import unittest
from unittest.mock import patch
from src.util.data_extractor import DataExtractor
from src.util.data_extractor import BetaConnectorService
from src.constants.constant_values import Constants

class TestDataExtractor(unittest.TestCase):
    def setUp(self):
        self.toa_record_data = {Constants.VG_ACCOUNT_NUMBER: "1234"}

    @patch.object(BetaConnectorService, 'retrieve_spad_transaction_details')
    @patch.object(BetaConnectorService, 'retrieve_toar_transaction_details')
    def test_execute_beta_spad_toar_transaction_details(self, mock_retrieve_toar, mock_retrieve_spad):
        # Mock the responses for both functions
        mock_retrieve_spad.return_value = {"getAccountNotesResponse": {"accountNotes": ["Mockdata"]}}
        mock_retrieve_toar.return_value = {"getAccountTransferRequestsResponse": {"accountTransfers": ["Mockdata"]}}

        DataExtractor.execute_beta_spad_toar_transaction_details(self.toa_record_data)

        # Check if the spadTransactionDetails and toarTransactionDetails have been updated correctly
        self.assertEqual(self.toa_record_data['spadTransactionDetails'], ["Mockdata"])
        self.assertEqual(self.toa_record_data['toarTransactionDetails'], ["Mockdata"])

if __name__ == '__main__':
    unittest.main()
