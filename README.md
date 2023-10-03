from util.logging_util import LOGGER

class ManagedAccountValidation:
    @staticmethod
    def validating_account_managed(toa_record_data, stp_data):
        is_account_managed = toa_record_data.get("isAccountManaged")
        if is_account_managed is None:
            stp_data['is_stp_eligible'] = False
            stp_data['message'] = stp_data['message'] + "\nVanguard Account Registration Validation Failed, unable to determine if this a Managed Account."
            LOGGER.info('Vanguard Account Registration Validation Failed, unable to determine if this a Managed Account.' + stp_data['message'])
        elif is_account_managed == 'MANAGED':
            stp_data['is_stp_eligible'] = False
            stp_data['message'] = stp_data['message'] + "\nVanguard Account is Managed. STP Automation is out of scope. Sending to Manual Queue."
            LOGGER.info('Vanguard Account is Managed. STP Automation is out of scope. Sending to Manual Queue.' + stp_data['message'])
        else:
            LOGGER.info("Vanguard account is not a Managed account.")





Please change this test case, According to the above code



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
        answer_file = {"brokerageAccount": {"accountId": "889400030162730"}}
        ManagedAccountValidation.validating_account_managed(answer_file, stp_data)
        self.assertFalse(stp_data['is_stp_eligible'])
        self.assertIn("\nVanguard Account is Managed. STP Automation is out of scope. Sending to Manual Queue.", stp_data['message'])
        mock_post.assert_called_once()

    @patch.object(VGSessionService, 'retrieve_vg_session_token', return_value=vg_session_token)
    @patch.object(OAuthService, 'retrieve_oauth_token', return_value=oauth_token)
    @patch.object(requests, 'post', side_effect=Exception("Service call failed"))
    def test_service_call_failure(self, mock_post, mock_retrieve_oauth_token, mock_retrieve_vg_session_token):
        answer_file = {"brokerageAccount": {"accountId": '889400030162730'}}
        ManagedAccountValidation.validating_account_managed(answer_file, stp_data)
        self.assertFalse(stp_data['is_stp_eligible'])
        self.assertIn("\nUnable to determine account type due to service failure. Moving on to the next validation.", stp_data['message'])
        mock_post.assert_called_once()

    @patch.object(VGSessionService, 'retrieve_vg_session_token', return_value=vg_session_token)
    @patch.object(OAuthService, 'retrieve_oauth_token', return_value=oauth_token)
    @patch.object(requests, 'post', return_value=MockResponse([{"accountManagementType": ''}], 200))
    def test_validating_account_managed_is_not_managed(self, mock_post, mock_retrieve_oauth_token, mock_retrieve_vg_session_token):
        answer_file = {"brokerageAccount": {"accountId": '889400030162730'}}
        ManagedAccountValidation.validating_account_managed(answer_file, stp_data)
        self.assertTrue(stp_data["is_stp_eligible"])
        self.assertEqual(stp_data["message"], "")
        mock_post.assert_called_once()
