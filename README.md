from util.logging_util import LOGGER
from service.managed_accounts_service import ManagedAccountsService

class ManagedAccountValidation:
    @staticmethod
    def validating_account_managed(answer_file, stp_data):
        account_Id = answer_file['brokerageAccount']['accountId']
        response_data = ManagedAccountsService.retrieve_managed_accounts(account_Id)
        
        if response_data is None:
            # Service call is NOT successful
            stp_data['is_stp_eligible'] = False
            stp_data['message'] = stp_data['message'] + "\nUnable to determine account type due to service failure. Moving on to the next validation."
            LOGGER.info('Unable to determine account type due to service failure. Moving on to the next validation. , set message: ' + stp_data['message'])
        else:
            # Service call is successful
            if not response_data:
                # Account data is none
                LOGGER.info("Vanguard Account is not Managed. Continuing to the next validation.")
            else:
                # Managed account data is returned for the given client/account
                if response_data[0].get('accountManagementType') == 'MANAGED':
                    # Account is a managed account
                    stp_data['is_stp_eligible'] = False
                    stp_data['message'] = stp_data['message'] + "\nVanguard Account is Managed. STP Automation is out of scope. Sending to Manual Queue."
                    LOGGER.info('Vanguard Account is Managed. STP Automation is out of scope. Sending to Manual Queue. , set message: ' + stp_data['message'])
                else:
                    # Account is NOT a managed account
                    LOGGER.info("Vanguard Account is not Managed. Continuing to the next validation.")
