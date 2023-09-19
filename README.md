import unittest
from unittest.mock import patch, Mock
from util.field_validation import FieldValidation

class TestFieldValidation(unittest.TestCase):
    def test_non_empty_account_number(self):
        filter_data = {
            'results': [{'answer': {'name': 'Account number:', 'value': '12345'}},
                        {'answer': {'name': 'Account number:', 'value': '56789'}}
                        ]
        }
        # Capture log messages
        with patch('util.field_validation.LOGGER', Mock()) as mock_logger:
            result = FieldValidation.account_number_checking(filter_data)
            # Assert log messages
            mock_logger.info.assert_called_once_with('Account number is not empty continuing the process')
        self.assertEqual(result, 'Account number is not empty continuing the process')

    def test_empty_account_number(self):
        filter_data = {
            'results': [{'answer': {'name': 'Account number:', 'value': ''}},
                        {'answer': {'name': 'Account number:', 'value': '56789'}}
                        ]
        }
        # Capture log messages
        with patch('util.field_validation.LOGGER', Mock()) as mock_logger:
            result = FieldValidation.account_number_checking(filter_data)
            # Assert log messages
            mock_logger.info.assert_called_once_with('Account number is empty stopping the process')
        self.assertEqual(result, 'Account number is empty stopping the process')

    # Add similar changes to other test cases
