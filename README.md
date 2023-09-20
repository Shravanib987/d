from util.logging_util import LOGGER
from constants.constant_values import Constants


class FieldValidation:
    @staticmethod
    def verify_account_number_exists(filter_data):
        account_number = filter_data.get(Constants.VG_ACCOUNT_NUMBER)  #trying to get the account number
        if account_number and account_number.strip():
            LOGGER.info('Account number is present in the answer file')
            return True
        else:
            LOGGER.info('Account number is NOT present in the answer file')
            return False

    @staticmethod
    def verify_account_name_match(filter_data):
        vg_primary_owner_name = filter_data.get(Constants.VG_PRIMARY_OWNER_NAME, '').strip()
        contrafirm_primary_owner_name = filter_data.get(Constants.CONTRAFIRM_PRIMARY_OWNER_NAME, '').strip()
        vg_secondary_owner_name = filter_data.get(Constants.VG_SECONDARY_OWNER_NAME, '').strip()
        contrafirm_secondary_owner_name = filter_data.get(Constants.CONTRAFIRM_SECONDARY_OWNER_NAME, '').strip()
        account_name_match = False

        if vg_primary_owner_name == contrafirm_primary_owner_name:
            account_name_match = True
            LOGGER.info('Both Vanguard and Contrafirm primary Account owner names are matching')
        else:
            LOGGER.info('Both Vanguard and Contrafirm Primary Account owner names are NOT matching')

        if vg_secondary_owner_name and contrafirm_secondary_owner_name:
            if vg_secondary_owner_name == contrafirm_secondary_owner_name:
            account_name_match = True
            LOGGER.info('Both Primary and Secondary names of Vanguard and Contrafirm are matching')
        else:
            account_name_match = False
            return account_name_match

    @staticmethod
    def verify_ssn_match(filter_data):
        vanguard_ssn = filter_data.get(Constants.VANGUARD_SSN, '').strip()
        contrafirm_ssn = filter_data.get(Constants.CONTRAFIRM_SSN, '').strip()
        ssn_name_match = False
        if vanguard_ssn == contrafirm_ssn:
            ssn_name_match = True
            LOGGER.info('Both Vanguard and Contrafirm SSN are matching')
        else:
            LOGGER.info('Both Vanguard and Contrafirm SSN are NOT matching')
            return ssn_name_match
