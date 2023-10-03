from unittest import TestCase
from unittest.mock import patch
from src.service.managed_accounts_service import requests
from src.service.managed_accounts_service import VGSessionService
from src.service.managed_accounts_service import OAuthService
from tests.service.mock_response import MockResponse
from validation.managed_account_validation import ManagedAccountValidation


vg_session_token = "VGSESSION"
oauth_token = "OAUTHTOKEN"

stp_data = {
    "message": "",
    "is_stp_eligible": True
}


class TestManagedAccountValidation(TestCase):
    def setUp(self):
        stp_data["message"] = ""
        stp_data["is_stp_eligible"] = True

    @patch.object(VGSessionService, 'retrieve_vg_session_token', return_value=vg_session_token)
    @patch.object(OAuthService, 'retrieve_oauth_token', return_value=oauth_token)
    @patch.object(requests, 'post', return_value=MockResponse([{"accountManagementType": "MANAGED"}], 200))
    def test_validating_account_managed_is_managed(self, mock_post, mock_retrieve_oauth_token, mock_retrieve_vg_session_token):
        toa_record_data = {"brokerageAccount": {"accountId": "889400030162730"}}
        ManagedAccountValidation.validating_account_managed(toa_record_data, stp_data)
        self.assertFalse(stp_data['is_stp_eligible'])
        self.assertIn("\nVanguard Account is Managed. STP Automation is out of scope. Sending to Manual Queue.", stp_data['message'])
        mock_post.assert_called_once()

    @patch.object(VGSessionService, 'retrieve_vg_session_token', return_value=vg_session_token)
    @patch.object(OAuthService, 'retrieve_oauth_token', return_value=oauth_token)
    @patch.object(requests, 'post', side_effect=Exception("Service call failed"))
    def test_service_call_failure(self, mock_post, mock_retrieve_oauth_token, mock_retrieve_vg_session_token):
        toa_record_data = {"brokerageAccount": {"accountId": '889400030162730'}}
        ManagedAccountValidation.validating_account_managed(toa_record_data, stp_data)
        self.assertFalse(stp_data['is_stp_eligible'])
        self.assertIn("\nVanguard Account Registration Validation Failed, unable to determine if this a Managed Account.", stp_data['message'])
        mock_post.assert_called_once()

    @patch.object(VGSessionService, 'retrieve_vg_session_token', return_value=vg_session_token)
    @patch.object(OAuthService, 'retrieve_oauth_token', return_value=oauth_token)
    @patch.object(requests, 'post', return_value=MockResponse([{"accountManagementType": ''}], 200))
    def test_validating_account_managed_is_not_managed(self, mock_post, mock_retrieve_oauth_token, mock_retrieve_vg_session_token):
        toa_record_data = {"brokerageAccount": {"accountId": '889400030162730'}}
        ManagedAccountValidation.validating_account_managed(toa_record_data, stp_data)
        self.assertTrue(stp_data["is_stp_eligible"])
        self.assertEqual(stp_data["message"], "")
        mock_post.assert_called_once()
