import unittest
from unittest.mock import patch, Mock
from your_module import YourClass

class YourClassTest(unittest.TestCase):

    @patch('your_module.YourClass.toa_record_data.get')
    def test_has_required_fields_matching_account_data(self, mock_get):
        # Prepare mock data and instance
        mock_data = {'clientNameResponse': 'response', 'accountRoles': [{'accountId': '123'}], 'brokerageAccount': {'accountId': '123'}}
        instance = YourClass()

        # Set the return value for the mocked get method
        mock_get.side_effect = mock_data.get

        # Call the method under test
        result = instance.has_required_fields_matching_account_data(mock_data)

        # Assertions
        self.assertEqual(result, mock_data['accountRoles'][0])

    @patch('your_module.YourClass.toa_record_data.get')
    @patch('your_module.YourClass.LOGGER.info')
    def test_validating_owner_data(self, mock_logger_info, mock_get):
        # Prepare mock data and instance
        mock_data = {'firstName': 'John', 'middleName': 'Doe', 'lastName': 'Smith'}
        toa_record_data = {'owner_name_type': 'some_owner_name_type'}

        instance = YourClass()

        # Set the return values for the mocked get method
        mock_get.side_effect = lambda key: mock_data.get(key, '')

        # Call the method under test
        result = instance.validating_owner_data(toa_record_data, mock_data, 'owner_name_type')

        # Assertions
        mock_logger_info.assert_called_once()  # Add appropriate arguments
        self.assertTrue(result)  # Adapt based on the actual behavior of your method

if __name__ == '__main__':
    unittest.main()
