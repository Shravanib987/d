from util.logging_util import LOGGER
from service.managed_accounts_service import ManagedAccountsService


class ManagedAccountValidation:
    @staticmethod
    def validating_account_managed(answer_file, stp_data):
        account_Id = answer_file['brokerageAccount']['accountId']
        response_data = ManagedAccountsService.retrieve_managed_accounts(account_Id)

        if response_data is None:
                # temporary message for service failure
            LOGGER.info("Unable to determine account type due to service failure. Moving on to the next validation.")
            return

        if response_data[0].get('accountManagementType') == 'MANAGED':
            # temporary message for managed account
            LOGGER.info("Vanguard Account is Managed. STP Automation is out of scope. Sending to Manual Queue.")
            stp_data['is_stp_eligible'] = False
            stp_data['message'] += "Vanguard Account is Managed, out of scope for STP Automation. Sending to Manual Queue."
        else:
            # temporary message for non-managed account
            LOGGER.info("Vanguard Account Registration Validation Failed, unable to determine if this a Managed Account.")



Please change the below test case according to the above code


from unittest import TestCase
from unittest.mock import patch
from src.service.managed_accounts_service import requests
from src.service.managed_accounts_service import VGSessionService
from src.service.managed_accounts_service import OAuthService
from tests.service.mock_response import MockResponse
from validation.managed_account_validation import ManagedAccountValidation

vg_session_token = "VGSESSION"
oauth_token = "OAUTHTOKEN"
successful_response = [{"accountManagementType": "MANAGED"}]

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
    @patch.object(requests, 'post', return_value=MockResponse(successful_response, 200))
    def test_managed_account(self, mock_post, mock_retrieve_oauth_token, mock_retrieve_vg_session_token):
        answer_file = {"brokerageAccount": {"accountId": "889400030162730"}}
        ManagedAccountValidation.validating_account_managed(answer_file, stp_data)
        self.assertFalse(stp_data['is_stp_eligible'])
        self.assertIn("Vanguard Account is Managed, out of scope for STP Automation. Sending to Manual Queue.", stp_data['message'])
        mock_post.assert_called_once()

    @patch.object(VGSessionService, 'retrieve_vg_session_token', return_value=vg_session_token)
    @patch.object(OAuthService, 'retrieve_oauth_token', return_value=oauth_token)
    @patch.object(requests, 'post', return_value=MockResponse({}, 404))
    def test_not_managed_account(self, mock_post, mock_retrieve_oauth_token, mock_retrieve_vg_session_token):
        answer_file = {"brokerageAccount": {"accountId": '889400030162730'}}
        ManagedAccountValidation.validating_account_managed(answer_file, stp_data)
        self.assertTrue(stp_data['is_stp_eligible'])
        mock_post.assert_called_once()
