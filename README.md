in data_extractor.py   Instead of in this initial batch in execute multiple services for execute client name service , we'll likely want to do this call in a loop after this group of service calls completes because we want to call OVV for EACH of the poids we got back from the CDM call. To that end, we'll also likely want to store them in our toa_record_data a little differently. Something like toa_record_data["clientNameResponse"] being set to an object and in the object each poid can be used as the key and the OVV response object can be the value? That's one idea, but feel free to set it up in a way you think will make the most sense for us to reference this client data later in our validations.




data extractor.py

import time
from concurrent.futures import ThreadPoolExecutor, as_completed
from threading import Lock

from constants.constant_values import Constants
from service.account_roles_service import AccountRolesService
from service.brokerage_accounts_service import BrokerageAccountsService
from service.client_name_service import ClientNamesService
from service.client_search_service import ClientSearchService
from service.custodian_service import CustodianService
from service.managed_accounts_service import ManagedAccountsService
from service.restrictions_service import RestrictionService


class DataExtractor:

    @staticmethod
    def execute_account_restrictions(toa_record_data):
        restriction_response_data = RestrictionService.retrieve_account_restrictions(
            toa_record_data[Constants.CLIENT_ID_FIELD])
        toa_record_data["restrictions"] = None if restriction_response_data is None else restriction_response_data.get(
            "accountLevelRestrictions")

    @staticmethod
    def execute_account_roles(toa_record_data):
        account_roles_response_data = AccountRolesService.retrieve_account_roles(
            toa_record_data[Constants.CLIENT_ID_FIELD])
        toa_record_data["accountRoles"] = None if account_roles_response_data is None else account_roles_response_data.get(
            "accounts")

    @staticmethod
    def execute_brokerage_accounts(toa_record_data):
        brokerage_accounts_response_data = BrokerageAccountsService.retrieve_brokerage_accounts(toa_record_data[Constants.CLIENT_ID_FIELD])
        toa_record_data["brokerageAccount"] = None if brokerage_accounts_response_data is None \
            else BrokerageAccountsService.extract_matching_account(brokerage_accounts_response_data["accounts"], toa_record_data.get(Constants.VG_ACCOUNT_NUMBER))

    @staticmethod
    def execute_client_search(toa_record_data):
        client_search_response_data = ClientSearchService.retrieve_client_poids(toa_record_data.get(Constants.VG_ACCOUNT_NUMBER))
        if client_search_response_data and client_search_response_data.get("clients"):
            toa_record_data["clientSearchResponse"] = client_search_response_data.get("clients").get("poid")
        else:
            toa_record_data["clientSearchResponse"] = None

    @staticmethod
    def execute_client_name(toa_record_data):
        client_names_response_data = ClientNamesService.retrieve_client_names(toa_record_data[Constants.CLIENT_ID_FIELD])
        if client_names_response_data and client_names_response_data.get("data"):
            toa_record_data["clientNameResponse"] = client_names_response_data.get("data").get("getClientUserContext")
        else:
            toa_record_data["clientNameResponse"] = None

    @staticmethod
    def execute_custodian_details(toa_record_data):
        custodian_response_data = CustodianService.retrieve_custodian_details(
            toa_record_data[Constants.CLIENT_ID_FIELD], toa_record_data[Constants.CUSTODIAN_NAME])
        toa_record_data[
            "custodianserviceresponse"] = None if not custodian_response_data else custodian_response_data.get(
            "custodians")

    @staticmethod
    def execute_managed_accounts(toa_record_data):
        account_id = None if not toa_record_data.get('brokerageAccount') else toa_record_data.get('brokerageAccount').get('accountId')
        managed_accounts_response_data = ManagedAccountsService.retrieve_managed_accounts(account_id)
        toa_record_data["isAccountManaged"] = None if not managed_accounts_response_data else ManagedAccountsService.extract_managed_flag(managed_accounts_response_data)

    @staticmethod
    def execute_multiple_services(toa_record_data):
        list_methods = [DataExtractor.execute_account_restrictions, DataExtractor.execute_account_roles,
                        DataExtractor.execute_brokerage_accounts, DataExtractor.execute_custodian_details,
                        DataExtractor.execute_client_search, DataExtractor.execute_client_name]
        with ThreadPoolExecutor(max_workers=len(list_methods)) as executor:
            lock = Lock()
            start_time = time.perf_counter()
            results = [executor.submit(method, toa_record_data) for method in
                       list_methods]
            for future in as_completed(results):
                future.result()
            end_time = time.perf_counter()

    @staticmethod
    def execute(toa_record_data):
        DataExtractor.execute_multiple_services(toa_record_data)
        DataExtractor.execute_managed_accounts(toa_record_data)




client_name_service.py

import requests

from constants.constant_values import Constants
from constants.endpoints import EndPoints
from service.eup_token_service import EupTokenService
from service.oauth_service import OAuthService
from service.vg_session_service import VGSessionService
from util.logging_util import LOGGER


class ClientNamesService:
    REQUEST_BODY = {'query': '{getClientUserContext{clientPoId firstName lastName middleName}}'}

    @staticmethod
    def retrieve_client_names(poid):
        client_names_response_data = None
        url = EndPoints.retrieve_url_by_key(EndPoints.OVV_CLIENT_NAMES_KEY)
        headers = ClientNamesService.build_headers(poid)

        try:
            response = requests.post(url, json=ClientNamesService.REQUEST_BODY, headers=headers, timeout=10)
            if response.status_code != 200:
                LOGGER.error("Failed to get client names: status_code=%s, message=%s",
                             str(response.status_code), response.text)
            else:
                client_names_response_data = response.json()
        except Exception as exception:
            LOGGER.error("Exception occurred during client names service request: error=%s", str(exception))

        return client_names_response_data

    @staticmethod
    def build_headers(poid):
        vg_session = VGSessionService.retrieve_vg_session_token()
        oauth_token = OAuthService.retrieve_oauth_token(vg_session=vg_session)
        eup_token = EupTokenService.retrieve_eup_token(vg_session, oauth_token, poid)
        return {
            "Accept": "application/json",
            "Authorization": f"Bearer {oauth_token}",
            "Cookie": "XSRF-TOKEN=1",
            "Consumer-Application-Code": Constants.CONSUMER_APPLICATION_CODE_KEY,
            "Content-Type": "application/json",
            "ext-user-profile-compressed": eup_token,
            "Origin": "https://www.vanguard.com",
            "Skip-Scopes-Check": "true",
            "X-XSRF-TOKEN": "1"
        }
