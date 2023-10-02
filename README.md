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
    def test_managed_account(self, mock_post, mock_retrieve_oauth_token, mock_retrieve_vg_session_token):
        answer_file = {"brokerageAccount": {"accountId": "889400030162730"}}
        ManagedAccountValidation.validating_account_managed(answer_file, stp_data)
        self.assertFalse(stp_data['is_stp_eligible'])
        self.assertIn("Vanguard Account is Managed. STP Automation is out of scope. Sending to Manual Queue.", stp_data['message'])
        mock_post.assert_called_once()

    @patch.object(VGSessionService, 'retrieve_vg_session_token', return_value=vg_session_token)
    @patch.object(OAuthService, 'retrieve_oauth_token', return_value=oauth_token)
    @patch.object(requests, 'post', side_effect=Exception("Service call failed"))
    def test_service_call_failure(self, mock_post, mock_retrieve_oauth_token, mock_retrieve_vg_session_token):
        answer_file = {"brokerageAccount": {"accountId": '889400030162730'}}
        ManagedAccountValidation.validating_account_managed(answer_file, stp_data)
        self.assertFalse(stp_data['is_stp_eligible'])
        self.assertIn("Unable to determine account type due to service failure. Moving on to the next validation.", stp_data['message'])
        mock_post.assert_called_once()

    @patch.object(VGSessionService, 'retrieve_vg_session_token', return_value=vg_session_token)
    @patch.object(OAuthService, 'retrieve_oauth_token', return_value=oauth_token)
    @patch.object(requests, 'post', return_value=MockResponse([{"accountManagementType": ''}], 200))
    def test_not_managed_account(self, mock_post, mock_retrieve_oauth_token, mock_retrieve_vg_session_token):
        answer_file = {"brokerageAccount": {"accountId": '889400030162730'}}
        ManagedAccountValidation.validating_account_managed(answer_file, stp_data)
        self.assertTrue(stp_data["is_stp_eligible"])
        self.assertEqual(stp_data["message"], "")
        mock_post.assert_called_once()
