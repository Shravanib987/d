This is the reference test case, Could you please write OwnerValidationUtil test cases..



import sys
sys.path.append("../../src")
import unittest
from unittest.mock import patch
from validation.primary_owner_validation import OwnerValidationUtil
from validation.secondary_owner_validation import SecondaryOwnerValidation
from src.constants.constant_values import Constants

stp_data = {
    "message": "",
    "is_stp_eligible": True
}

toa_record_data = {Constants.VG_SECONDARY_OWNER_NAME: 'John Doe', Constants.AN_ACCOUNT_OWNER: 'false', 'clientNameResponse': {'123': {'firstName': 'John', 'middleName': '', 'lastName': 'Doe'}}, 'accountRoles': [{'accountId': '123', 'registeredRoles': [{'clientPoId': '123', 'permissionType': 'TRNS'}]}], 'brokerageAccount': {'accountId': '123'}}


class TestSecondaryOwnerValidation(unittest.TestCase):
    def setUp(self):
        stp_data["message"] = ""
        stp_data["is_stp_eligible"] = True

    @patch.object(OwnerValidationUtil, 'has_required_fields_matching_account_data')
    @patch.object(OwnerValidationUtil, 'validating_owner_data')
    def test_validating_secondary_owner_matching_secondary_owner_and_client_name_response_data_no_middle_names_in_both(self, mock_validating_owner_data, mock_has_required_fields_matching_account_data):
        SecondaryOwnerValidation.validating_secondary_owner(toa_record_data, stp_data)
        mock_validating_owner_data.return_value = True
        mock_has_required_fields_matching_account_data.return_value = True
        self.assertTrue(stp_data['is_stp_eligible'])
        self.assertEqual(stp_data['message'], '')

    @patch.object(OwnerValidationUtil, 'has_required_fields_matching_account_data')
    @patch.object(OwnerValidationUtil, 'validating_owner_data')
    def test_validating_secondary_owner_AN_ACCOUNT_OWNER_true(self, mock_validating_owner_data, mock_has_required_fields_matching_account_data):
        toa_record_data[Constants.AN_ACCOUNT_OWNER] = 'true'
        SecondaryOwnerValidation.validating_secondary_owner(toa_record_data, stp_data)
        mock_validating_owner_data.return_value = True
        mock_has_required_fields_matching_account_data.return_value = True
        self.assertTrue(stp_data['is_stp_eligible'])
        self.assertEqual(stp_data['message'], '')

    @patch.object(OwnerValidationUtil, 'has_required_fields_matching_account_data')
    @patch.object(OwnerValidationUtil, 'validating_owner_data')
    def test_validating_secondary_owner_secondary_owner_name_empty(self, mock_validating_owner_data, mock_has_required_fields_matching_account_data):
        toa_record_data[Constants.VG_SECONDARY_OWNER_NAME] = ''
        toa_record_data[Constants.AN_ACCOUNT_OWNER] = 'false'
        SecondaryOwnerValidation.validating_secondary_owner(toa_record_data, stp_data)
        mock_validating_owner_data.return_value = True
        mock_has_required_fields_matching_account_data.return_value = True
        self.assertFalse(stp_data['is_stp_eligible'])
        self.assertIn("Secondary owner does not exists in the answer file", stp_data['message'])

    @patch.object(OwnerValidationUtil, 'has_required_fields_matching_account_data')
    @patch.object(OwnerValidationUtil, 'validating_owner_data')
    def test_validating_secondary_owner_secondary_owner_name_not_matching_with_client_name_response(self, mock_validating_owner_data, mock_has_required_fields_matching_account_data):
        toa_record_data[Constants.VG_SECONDARY_OWNER_NAME] = 'jones hoe'
        toa_record_data[Constants.AN_ACCOUNT_OWNER] = 'false'
        mock_validating_owner_data.return_value = False
        mock_has_required_fields_matching_account_data.return_value = True
        SecondaryOwnerValidation.validating_secondary_owner(toa_record_data, stp_data)
        self.assertFalse(stp_data['is_stp_eligible'])
        self.assertIn("Secondary owner on form does not match with client name response", stp_data['message'])

    @patch.object(OwnerValidationUtil, 'validating_owner_data')
    def test_validating_secondary_owner_secondary_owner_name_matching_with_client_name_response_with_not_trns(self, mock_validating_owner_data):
        toa_record_data = {Constants.VG_SECONDARY_OWNER_NAME: 'John Doe', Constants.AN_ACCOUNT_OWNER: 'false', 'clientNameResponse': {'123': {'firstName': 'John', 'middleName': '', 'lastName': 'Doe'}}, 'accountRoles': [{'accountId': '123', 'registeredRoles': [{'clientPoId': '123', 'permissionType': 'VIEW'}]}], 'brokerageAccount': {'accountId': '123'}}
        SecondaryOwnerValidation.validating_secondary_owner(toa_record_data, stp_data)
        mock_validating_owner_data.return_value = True
        self.assertFalse(stp_data['is_stp_eligible'])
        self.assertIn("Secondary owner name validation failed, permission type is not TRNS.", stp_data['message'])

    @patch.object(OwnerValidationUtil, 'validating_owner_data')
    def test_validating_secondary_owner_valid_case_with_permission_type_trns(self, mock_validating_owner_data):
        toa_record_data[Constants.VG_SECONDARY_OWNER_NAME] = 'John Doe'
        toa_record_data['accountRoles'][0]['registeredRoles'][0]['permissionType'] = 'TRNS'
        SecondaryOwnerValidation.validating_secondary_owner(toa_record_data, stp_data)
        mock_validating_owner_data.return_value = True
        self.assertTrue(stp_data['is_stp_eligible'])
        self.assertEqual(stp_data['message'], '')

    @patch.object(OwnerValidationUtil, 'validating_owner_data')
    def test_validating_secondary_owner_not_matching_account_roles_clientPoId_with_client_name_response_clientPoId(self, mock_validating_owner_data):
        toa_record_data = {Constants.VG_SECONDARY_OWNER_NAME: 'John Doe', Constants.AN_ACCOUNT_OWNER: 'false', 'clientNameResponse': {'123': {'firstName': 'John', 'middleName': '', 'lastName': 'Doe'}}, 'accountRoles': [{'accountId': '123', 'registeredRoles': [{'clientPoId': '321', 'permissionType': 'TRNS'}]}], 'brokerageAccount': {'accountId': '123'}}
        SecondaryOwnerValidation.validating_secondary_owner(toa_record_data, stp_data)
        mock_validating_owner_data.return_value = True
        self.assertTrue(stp_data['is_stp_eligible'])
        self.assertEqual(stp_data['message'], '')

    @patch.object(OwnerValidationUtil, 'validating_owner_data')
    def test_validating_secondary_owner_not_matching_account_roles_accountId_with_brokerageaccountId(self, mock_validating_owner_data):
        toa_record_data = {Constants.VG_SECONDARY_OWNER_NAME: 'John Doe', Constants.AN_ACCOUNT_OWNER: 'false', 'clientNameResponse': {'123': {'firstName': 'John', 'middleName': '', 'lastName': 'Doe'}}, 'accountRoles': [{'accountId': '123', 'registeredRoles': [{'clientPoId': '123', 'permissionType': 'TRNS'}]}], 'brokerageAccount': {'accountId': '321'}}
        SecondaryOwnerValidation.validating_secondary_owner(toa_record_data, stp_data)
        mock_validating_owner_data.return_value = False
        self.assertFalse(stp_data['is_stp_eligible'])
        self.assertIn("Unable to validate secondary owner on vanguard account", stp_data['message'])

    @patch.object(OwnerValidationUtil, 'has_required_fields_matching_account_data')
    @patch.object(OwnerValidationUtil, 'validating_owner_data')
    def test_validating_secondary_owner_client_name_response_none(self, mock_validating_owner_data, mock_has_required_fields_matching_account_data):
        toa_record_data = {Constants.VG_SECONDARY_OWNER_NAME: 'John Doe', Constants.AN_ACCOUNT_OWNER: 'false', 'clientNameResponse': {'123': {'firstName': 'John', 'middleName': '', 'lastName': 'Doe'}}, 'accountRoles': [{'accountId': '123', 'registeredRoles': [{'clientPoId': '123', 'permissionType': 'TRNS'}]}], 'brokerageAccount': {'accountId': '123'}}
        toa_record_data['clientNameResponse']['123'] = None
        SecondaryOwnerValidation.validating_secondary_owner(toa_record_data, stp_data)
        mock_has_required_fields_matching_account_data.return_value = True
        mock_validating_owner_data.return_value = False
        self.assertTrue(stp_data['is_stp_eligible'])
        self.assertEqual(stp_data['message'], '')

    @patch.object(OwnerValidationUtil, 'has_required_fields_matching_account_data')
    @patch.object(OwnerValidationUtil, 'validating_owner_data')
    def test_validating_secondary_owner_client_poid_not_matching_with_rns_register_roles_poid(self, mock_validating_owner_data, mock_has_required_fields_matching_account_data):
        toa_record_data = {Constants.VG_SECONDARY_OWNER_NAME: 'John Doe', Constants.AN_ACCOUNT_OWNER: 'false', 'clientNameResponse': {'123': {'firstName': 'John', 'middleName': '', 'lastName': 'Doe'}}, 'accountRoles': [{'accountId': '123', 'registeredRoles': [{'clientPoId': '321', 'permissionType': 'TRNS'}]}], 'brokerageAccount': {'accountId': '123'}}
        SecondaryOwnerValidation.validating_secondary_owner(toa_record_data, stp_data)
        mock_has_required_fields_matching_account_data.return_value = True
        mock_validating_owner_data.return_value = False
        self.assertTrue(stp_data['is_stp_eligible'])
        self.assertEqual(stp_data['message'], '')

    @patch.object(OwnerValidationUtil, 'validating_owner_data')
    def test_validating_secondary_owner_none_responses(self, mock_validating_owner_data):
        toa_record_data = {
            'brokerageAccount': {
                'accountId': '123'
            }
        }
        toa_record_data[Constants.AN_ACCOUNT_OWNER] = 'false'
        SecondaryOwnerValidation.validating_secondary_owner(toa_record_data, stp_data)
        mock_validating_owner_data.return_value = False
        self.assertFalse(stp_data['is_stp_eligible'])
        self.assertIn("Unable to validate secondary owner on vanguard account", stp_data['message'])



from util.logging_util import LOGGER
from constants.constant_values import Constants


class OwnerValidationUtil:

    @staticmethod
    def has_required_fields_matching_account_data(toa_record_data):
        response_data_is_not_empty = toa_record_data.get('clientNameResponse') and toa_record_data.get('accountRoles') and toa_record_data.get('brokerageAccount')
        if response_data_is_not_empty:
            brokerage_account_id = toa_record_data.get('brokerageAccount').get('accountId')
            for account_roles_data in toa_record_data.get('accountRoles'):
                if str(account_roles_data.get('accountId')) == brokerage_account_id:
                    return account_roles_data
            return None
        return None

    @staticmethod
    def validating_owner_data(toa_record_data, data, owner_name_type):
        client_name_response_data = data.get('firstName', '')+data.get('middleName', '')+data.get('lastName', '')
        owner_name = toa_record_data[owner_name_type].replace(' ', '').lower()
        full_name = (data.get('firstName', '') + data.get('lastName','')).lower()
        names_split = toa_record_data[owner_name_type].split()
        firstname_lastname_check = len(names_split) == 3 and ((names_split[0] + names_split[2]).lower() == full_name)
        matching_data = (owner_name == client_name_response_data.lower()) or (firstname_lastname_check or owner_name == full_name)
        return matching_data

    @staticmethod
    def set_stp_ineligible(stp_data, message, owner_type):
        stp_data['is_stp_eligible'] = False
        stp_data['message'] = "\n" + message
        LOGGER.info(owner_type+ ', set message: ' + stp_data['message'])         #temp log for testing
