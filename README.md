from util.logging_util import LOGGER
from service.managed_accounts_service import ManagedAccountsService

class ManagedAccountValidation:
    @staticmethod
    def validating_account_managed(answer_file, stp_data):
        account_Id = answer_file['brokerageAccount']['accountId']
        response_data = ManagedAccountsService.retrieve_managed_accounts(account_Id)
        
        if response_data is None:
            # Log a temporary message for service failure
            LOGGER.info("Unable to determine account type due to service failure. Moving on to the next validation.")
            return

        if response_data[0].get('accountManagementType') == 'MANAGED':
            # Log a temporary message for managed account
            LOGGER.info("Vanguard Account is Managed. STP Automation is out of scope. Sending to Manual Queue.")
            stp_data['is_stp_eligible'] = False
            stp_data['message'] += "Vanguard Account is Managed, out of scope for STP Automation. Sending to Manual Queue."
        else:
            # Log a temporary message for non-managed account
            LOGGER.info("Vanguard Account is not Managed. Continuing to the next validation.")
