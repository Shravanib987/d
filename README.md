managed_account_validation.py code


from util.logging_util import LOGGER
from service.managed_accounts_service import ManagedAccountsService


class ManagedAccountValidation:
    @staticmethod
    def validating_account_managed(answer_file,stp_data):
        account_Id = answer_file['brokerageAccount']['accountId']
        response_data = ManagedAccountsService.retrieve_managed_accounts(account_Id)
        is_managed_account = ManagedAccountsService.extract_managed_flag(response_data)
        if is_managed_account != 'MANAGED':
            LOGGER.info('Vanguard Account Registration Validation Failed, unable to determine if this a Managed Account.')
        else:
            stp_data['is_stp_eligible'] = False
            stp_data['message'] = stp_data['message'] + "Vanguard Account is Managed, out of scope for STP Automation. Sending to Manual Queue."
            LOGGER.info('Vanguard Account is Managed, out of scope for STP Automation. Sending to Manual Queue.')



managed_account_service.py code



import requests
from service.oauth_service import OAuthService
from service.vg_session_service import VGSessionService
from constants.endpoints import EndPoints
from util.logging_util import LOGGER
from constants.constant_values import Constants


class ManagedAccountsService:
    @staticmethod
    def retrieve_managed_accounts(account_id):
        managed_accounts_response_data = None
        url = EndPoints.retrieve_url_by_key(EndPoints.AAT_MANAGED_ACCOUNTS_KEY)
        headers = ManagedAccountsService.build_headers()
        payload = [
            {
                "accountId": account_id,
                "accountRecordKeeper": "VANGUARD_RETAIL"
            }
        ]
        try:
            response = requests.post(url, json=payload, headers=headers, timeout=10)
            if response.status_code != 200:
                LOGGER.error("Failed to get managed account details with code %s and message %s",
                             str(response.status_code), response.text)
            else:
                managed_accounts_response_data = response.json()
        except Exception as exception:
            LOGGER.error("Exception occurred during managed accounts service request: %s", str(exception))

        return managed_accounts_response_data

    @staticmethod
    def extract_managed_flag(managed_accounts_response_data):
        if managed_accounts_response_data:
            if managed_accounts_response_data[0]:
                # temp log for testing
                LOGGER.info('Account is managed? ' + str(managed_accounts_response_data[0].get("accountManagementType") == "MANAGED"))
                return managed_accounts_response_data[0].get("accountManagementType")
            return None
        return None

    @staticmethod
    def build_headers():
        vg_session = VGSessionService.retrieve_vg_session_token()
        oauth_token = OAuthService.retrieve_oauth_token(vg_session=vg_session)
        return {
            "Content-Type": "application/json",
            "Accept": "application/json",
            "Cookie": f"VGSESSION={vg_session}",
            "Consumer-Application-Code": Constants.CONSUMER_APPLICATION_CODE_KEY,
            "Authorization": f"Bearer {oauth_token}",
            "Origin": "https://www.vanguard.com",
        }



Please edit managed_account_validation.py  according to the acceptance criteria
Acceptance Criteria

Given the account listed in the answer file is NOT a managed account

And the service call is successful

And managed account data is returned for the given client/account

When the lambda validates the account is not managed

Then it logs a message (temporary)

And continues on to the next step

 

Given the account listed in the answer file is a managed account

And the service call is successful

And account data is returned for the given account

When the lambda validates the account

Then it logs a message (temporary)

and sets  STP flag to false

and moves on to the next validation

 

Given the account listed in the answer file is a managed account

And the service call is NOT successful

And account data is none

When the lambda validates the account

Then it logs a message (temporary)

and sets  STP flag to false

and moves on to the next validation

(Get Message from Billy for 1. unable to determine due to service failure; 2. Managed account=true)

SAT Testing 
