You shouldn't need to call the ManagedAccountsService in the managed_account_validation.py file, it's already being called in the data_extractor.py file. You'll just want to get the isAccountManaged string from the toa_record_data (check in the data_extractor.py to see how the response from the service call is being set on the toa_record_data).
I have added all the required files here. Please edit managed_account_validation.py file according to the above statement



managed_account_validation.py file

from util.logging_util import LOGGER
from service.managed_accounts_service import ManagedAccountsService


class ManagedAccountValidation:
    @staticmethod
    def validating_account_managed(toa_record_data, stp_data):
        account_id = toa_record_data['brokerageAccount']['accountId']
        response_data = ManagedAccountsService.retrieve_managed_accounts(account_id)
        if response_data is None:
            stp_data['is_stp_eligible'] = False
            stp_data['message'] = stp_data['message'] + "\nUnable to determine account type due to service failure. Moving on to the next validation."
            LOGGER.info('Unable to determine account type due to service failure. Moving on to the next validation. , set message: ' + stp_data['message'])
            return
        if response_data[0].get('accountManagementType') == 'MANAGED':
            stp_data['is_stp_eligible'] = False
            stp_data['message'] = stp_data['message'] + "\nVanguard Account is Managed. STP Automation is out of scope. Sending to Manual Queue."
            LOGGER.info('Vanguard Account is Managed. STP Automation is out of scope. Sending to Manual Queue. , set message: ' + stp_data['message'])
        else:
            LOGGER.info("Vanguard Account Registration Validation Failed, unable to determine if this a Managed Account.")






data_extractor.py file


import time
from concurrent.futures import ThreadPoolExecutor, as_completed
from threading import Lock
from constants.constant_values import Constants
from service.account_roles_service import AccountRolesService
from service.brokerage_accounts_service import BrokerageAccountsService
from service.custodian_service import CustodianService
from service.restrictions_service import RestrictionService
from service.managed_accounts_service import ManagedAccountsService


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
                        DataExtractor.execute_brokerage_accounts, DataExtractor.execute_custodian_details]
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
