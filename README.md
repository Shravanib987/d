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
            mock_logger.info.assert_called_with('Account number is not empty continuing the process')
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
            mock_logger.info.assert_called_with('Account number is empty stopping the process')
        self.assertEqual(result, 'Account number is empty stopping the process')

    def test_same_acc_name_match(self):
        filter_data = {
            'results': [{'answer': {'name': 'Primary account owner name or registration as it appears at the other firm', 'value': '56789'}},
                        {'answer': {'name': 'Primary account owner name', 'value': '56789'}},
                        {'answer': {'name': 'Name of secondary owner', 'value': '56789'}},
                        {'answer': {'name': 'Secondary account owner name', 'value': '56789'}}
                        ]
        }
        # Capture log messages
        with patch('util.field_validation.LOGGER', Mock()) as mock_logger:
            FieldValidation.acc_name_match(filter_data)
            # Assert log messages
            mock_logger.info.assert_any_call('Both Account names are matching')
            mock_logger.info.assert_any_call('Both primary and secondary names are matching')

    def test_diff_acc_name_match(self):
        filter_data = {
            'results': [{'answer': {'name': 'Primary account owner name or registration as it appears at the other firm', 'value': '56789'}},
                        {'answer': {'name': 'Primary account owner name', 'value': '12345'}},
                        {'answer': {'name': 'Name of secondary owner', 'value': '56789'}},
                        {'answer': {'name': 'Secondary account owner name', 'value': '56789'}}
                        ]
        }
        # Capture log messages
        with patch('util.field_validation.LOGGER', Mock()) as mock_logger:
            FieldValidation.acc_name_match(filter_data)
            # Assert log messages
            mock_logger.info.assert_any_call('Both Account names are NOT matching')
            mock_logger.info.assert_any_call('Both primary and secondary names are NOT matching')

    def test_same_SSN_match(self):
        filter_data = {
            'results': [{'answer': {'name': 'Primary Social Security number', 'value': '12345'}},
                        {'answer': {'name': 'Primary SSN or EIN&nbsp;under which the account is registered', 'value': '12345'}}
                        ]
        }
        # Capture log messages
        with patch('util.field_validation.LOGGER', Mock()) as mock_logger:
            result = FieldValidation.SSN_match(filter_data)
            # Assert log messages
            mock_logger.info.assert_called_with('Both ssn are matching')
        self.assertEqual(result, 'Both ssn are matching')

    def test_diff_SSN_match(self):
        filter_data = {
            'results': [{'answer': {'name': 'Primary Social Security number', 'value': '56789'}},
                        {'answer': {'name': 'Primary SSN or EIN&nbsp;under which the account is registered', 'value': '12345'}}
                        ]
        }
        # Capture log messages
        with patch('util.field_validation.LOGGER', Mock()) as mock_logger:
            result = FieldValidation.SSN_match(filter_data)
            # Assert log messages
            mock_logger.info.assert_called_with('Both ssn are NOT matching')
        self.assertEqual(result, 'Both ssn are NOT matching')

if __name__ == '__main__':
    unittest.main()
