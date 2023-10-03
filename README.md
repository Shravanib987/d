from util.logging_util import LOGGER

class ManagedAccountValidation:
    @staticmethod
    def validating_account_managed(toa_record_data, stp_data):
        is_account_managed = toa_record_data.get("isAccountManaged")
        if is_account_managed is None:
            stp_data['is_stp_eligible'] = False
            stp_data['message'] = stp_data['message'] + "\nVanguard Account Registration Validation Failed, unable to determine if this a Managed Account."
            LOGGER.info('Vanguard Account Registration Validation Failed, unable to determine if this a Managed Account. , set message: ' + stp_data['message'])
        elif is_account_managed == 'MANAGED':
            stp_data['is_stp_eligible'] = False
            stp_data['message'] = stp_data['message'] + "\nVanguard Account is Managed. STP Automation is out of scope. Sending to Manual Queue."
            LOGGER.info('Vanguard Account is Managed. STP Automation is out of scope. Sending to Manual Queue. , set message: ' + stp_data['message'])
        else:
            LOGGER.info("Vanguard account is not a Managed account.")
