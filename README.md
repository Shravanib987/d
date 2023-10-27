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
