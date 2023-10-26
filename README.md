Constants

class PrimaryOwnerValidation:

    @staticmethod
    def validating_primary_owner(toa_record_data, stp_data):
        if PrimaryOwnerValidation._has_required_fields(toa_record_data):
            matching_account_roles_data = PrimaryOwnerValidation._find_matching_account(toa_record_data)

            if matching_account_roles_data:
                poid = toa_record_data[Constants.VG_PRIMARY_OWNER_NAME].strip()
                client_name_response_data = PrimaryOwnerValidation._get_client_name_response(toa_record_data)
                if poid == client_name_response_data.strip():
                    is_primary_owner = PrimaryOwnerValidation._is_primary_owner(matching_account_roles_data)
                    if is_primary_owner:
                        LOGGER.info('Primary owner is True from the RNS response where validation passed')
                    else:
                        PrimaryOwnerValidation._set_stp_ineligible(stp_data, "Primary owner name doesn't match where validation failed.")
                else:
                    PrimaryOwnerValidation._set_stp_ineligible(stp_data, "Primary owner on form does not match primary owner on Vanguard account")
            else:
                PrimaryOwnerValidation._set_stp_ineligible(stp_data, "Unable to validate primary owner on Vanguard account")
        else:
            PrimaryOwnerValidation._set_stp_ineligible(stp_data, "Unable to validate primary owner on Vanguard account")

    @staticmethod
    def _has_required_fields(toa_record_data):
        return toa_record_data.get('clientNameResponse') and toa_record_data.get('accountRoles') and toa_record_data.get('brokerageAccount')

    @staticmethod
    def _find_matching_account(toa_record_data):
        brokerage_account_id = toa_record_data['brokerageAccount']['accountId']
        for account_roles_data in toa_record_data['accountRoles']:
            if str(account_roles_data.get('accountId')) == str(brokerage_account_id):
                return account_roles_data
        return None

    @staticmethod
    def _get_client_name_response(toa_record_data):
        client_name_data = toa_record_data['clientNameResponse']
        return client_name_data.get('firstName', '') + ' ' + client_name_data.get('lastName', '')

    @staticmethod
    def _is_primary_owner(account_data):
        for matching_register_roles_data in account_data.get('registeredRoles', []):
            if matching_register_roles_data.get('primaryOwner'):
                return True
        return False

    @staticmethod
    def _set_stp_ineligible(stp_data, message):
        stp_data['is_stp_eligible'] = False
        stp_data['message'] = stp_data.get('message', '') + "\n" + message
        LOGGER.info('Primary owner validation, set message: ' + stp_data['message'])
