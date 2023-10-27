DataExtractor file

    @staticmethod
    def execute_beta_spad_toar_transaction_details(toa_record_data):
        spad_transaction_response_data = BetaConnectorService.retrieve_spad_transaction_details(toa_record_data.get(Constants.VG_ACCOUNT_NUMBER))
        toar_transaction_response_data = BetaConnectorService.retrieve_toar_transaction_details(toa_record_data.get(Constants.VG_ACCOUNT_NUMBER))
        if spad_transaction_response_data and spad_transaction_response_data.get("getAccountNotesResponse"):
            toa_record_data["spadTransactionDetails"] = spad_transaction_response_data.get("getAccountNotesResponse").get("accountNotes")
        else:
            toa_record_data["spadTransactionDetails"] = None
        if toar_transaction_response_data and toar_transaction_response_data.get("getAccountTransferRequestsResponse"):
            toa_record_data["toarTransactionDetails"] = toar_transaction_response_data.get("getAccountTransferRequestsResponse").get("accountTransfers")
        else:
            toa_record_data["toarTransactionDetails"] = None


   Provide Test case for the above code, similar test cases below for reference 


       @patch.object(BetaConnectorService, 'retrieve_spad_transaction_details')
    def test_execute_beta_spad_transaction_details_success(self, mock_retrieve_spad_transaction_details):
        mock_retrieve_spad_transaction_details.return_value = {"getAccountNotesResponse": {"accountNotes": ["Mockdata"]}}
        self.toa_record_data[Constants.VG_ACCOUNT_NUMBER] = "1234"

        DataExtractor.execute_beta_spad_transaction_details(self.toa_record_data)
        self.assertIn('spadTransactionDetails', self.toa_record_data)
        self.assertEqual(self.toa_record_data['spadTransactionDetails'], ["Mockdata"])

    @patch.object(BetaConnectorService, 'retrieve_spad_transaction_details')
    def test_execute_beta_spad_transaction_details_response_none(self, mock_retrieve_spad_transaction_details):
        mock_retrieve_spad_transaction_details.return_value = None
        self.toa_record_data[Constants.VG_ACCOUNT_NUMBER] = "1234"

        DataExtractor.execute_beta_spad_transaction_details(self.toa_record_data)
        self.assertIn('spadTransactionDetails', self.toa_record_data)
        self.assertEqual(self.toa_record_data['spadTransactionDetails'], None)

    @patch.object(BetaConnectorService, 'retrieve_spad_transaction_details')
    def test_execute_beta_spad_transaction_details_getAccountNotesResponse_none(self, mock_retrieve_spad_transaction_details):
        mock_retrieve_spad_transaction_details.return_value = {"getAccountNotesResponse": None}
        self.toa_record_data[Constants.VG_ACCOUNT_NUMBER] = "1234"

        DataExtractor.execute_beta_spad_transaction_details(self.toa_record_data)
        self.assertIn('spadTransactionDetails', self.toa_record_data)
        self.assertEqual(self.toa_record_data['spadTransactionDetails'], None)



The below code is beta_connector_Service.py


import requests
from service.oauth_service import OAuthService
from service.vg_session_service import VGSessionService
from constants.endpoints import EndPoints
from util.logging_util import LOGGER
from constants.constant_values import Constants


class BetaConnectorService:

    @staticmethod
    def retrieve_spad_transaction_details(account_number):
        beta_spad_transaction_details_response_data = None
        url = EndPoints.retrieve_url_by_key(EndPoints.BCT_BETA_CONNECTOR_KEY)
        headers = BetaConnectorService.build_headers()
        payload = {
            "path": "/v1/AccountNotes/GetAccountNotes",
            "pageToken": "",
            "betaPayload": {
                "getAccountNotesRequest": {
                    "accountNumber": int(account_number)
                }
            }
        }
        try:
            response = requests.post(url, json=payload, headers=headers, timeout=10)
            if response.status_code != 200:
                LOGGER.error("Failed to get BETA SPAD transaction details with code %s and message %s",
                             str(response.status_code), response.text)
            else:
                beta_spad_transaction_details_response_data = response.json()
                # temp log for testing
                LOGGER.info('BETA SPAD transaction details response data: ' + str(beta_spad_transaction_details_response_data))
        except Exception as exception:
            LOGGER.error("Exception occurred during BETA SPAD transaction details service request: %s", str(exception))

        return beta_spad_transaction_details_response_data

    @staticmethod
    def retrieve_toar_transaction_details(account_number):
        beta_toar_transaction_details_response_data = None
        url = EndPoints.retrieve_url_by_key(EndPoints.BCT_BETA_CONNECTOR_KEY)
        headers = BetaConnectorService.build_headers()
        payload = {
            "path": "/v1/AccountTransfers/GetAccountTransferRequests",
            "pageToken": "",
            "betaPayload": {
                "getAccountTransferRequestsRequest": {
                    "accountTransferInstruction": "Receive",
                    "includeTransferAssets": True,
                    "filterOption": {
                        "accountNumber": int(account_number)
                    }
                }
            }
        }
        try:
            response = requests.post(url, json=payload, headers=headers, timeout=10)
            if response.status_code != 200:
                LOGGER.error("Failed to get BETA TOAR transaction details with code %s and message %s",
                             str(response.status_code), response.text)
            else:
                beta_toar_transaction_details_response_data = response.json()
                # temp log for testing
                LOGGER.info('BETA TOAR transaction details response data: ' + str(beta_toar_transaction_details_response_data))
        except Exception as exception:
            LOGGER.error("Exception occurred during BETA TOAR transaction details service request: %s", str(exception))

        return beta_toar_transaction_details_response_data

    @staticmethod
    def build_headers():
        vg_session = VGSessionService.retrieve_vg_session_token()
        oauth_token = OAuthService.retrieve_oauth_token(vg_session=vg_session)
        return {
            "Content-Type": "application/json",
            "Accept": "application/json",
            "Consumer-Application-Code": Constants.CONSUMER_APPLICATION_CODE_KEY,
            "Authorization": f"Bearer {oauth_token}"
        }
