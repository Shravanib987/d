from unittest import TestCase
from unittest.mock import patch, Mock
from validation.managed_account_validation import ManagedAccountValidation

class TestManagedAccountValidation(TestCase):
    def setUp(self):
        self.stp_data = {
            "message": "",
            "is_stp_eligible": True
        }

    @patch('validation.managed_account_validation.ManagedAccountsService.retrieve_managed_accounts')
    def test_validating_account_managed_is_managed(self, mock_retrieve_managed_accounts):
        # Simulate that the account is managed
        mock_retrieve_managed_accounts.return_value = [{"accountManagementType": "MANAGED"}]
        
        toa_record_data = {"isAccountManaged": True}
        ManagedAccountValidation.validating_account_managed(toa_record_data, self.stp_data)
        
        self.assertFalse(self.stp_data['is_stp_eligible'])
        self.assertIn("\nVanguard Account is Managed. STP Automation is out of scope. Sending to Manual Queue.", self.stp_data['message'])

    @patch('validation.managed_account_validation.ManagedAccountsService.retrieve_managed_accounts')
    def test_validating_account_managed_is_not_managed(self, mock_retrieve_managed_accounts):
        # Simulate that the account is not managed
        mock_retrieve_managed_accounts.return_value = [{"accountManagementType": "NON_MANAGED"}]
        
        toa_record_data = {"isAccountManaged": False}
        ManagedAccountValidation.validating_account_managed(toa_record_data, self.stp_data)
        
        self.assertTrue(self.stp_data['is_stp_eligible'])
        self.assertEqual(self.stp_data['message'], "")

    @patch('validation.managed_account_validation.ManagedAccountsService.retrieve_managed_accounts')
    def test_service_call_failure(self, mock_retrieve_managed_accounts):
        # Simulate a service call failure
        mock_retrieve_managed_accounts.side_effect = Exception("Service call failed")
        
        toa_record_data = {"isAccountManaged": True}
        ManagedAccountValidation.validating_account_managed(toa_record_data, self.stp_data)
        
        self.assertFalse(self.stp_data['is_stp_eligible'])
        self.assertIn("\nVanguard Account Registration Validation Failed, unable to determine if this a Managed Account.", self.stp_data['message'])

    def test_is_account_managed_is_none(self):
        # Simulate a case where 'isAccountManaged' is None
        toa_record_data = {"isAccountManaged": None}
        ManagedAccountValidation.validating_account_managed(toa_record_data, self.stp_data)
        
        self.assertFalse(self.stp_data['is_stp_eligible'])
        self.assertIn("\nVanguard Account Registration Validation Failed, unable to determine if this a Managed Account.", self.stp_data['message'])
