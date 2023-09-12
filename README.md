import unittest
import json
from AccountNumberValidation import account_number_checking, account_number_validation

class TestAccountNumberValidation(unittest.TestCase):

    def setUp(self):
        # Load test JSON data
        with open('test_data.json', 'r') as test_json_file:
            self.test_data = json.load(test_json_file)

    def test_account_number_checking(self):
        obj = {"name": "Account number:"}
        self.assertTrue(account_number_checking(obj))

    def test_account_number_not_empty(self):
        obj = {"value": "12345"}
        self.assertTrue(account_number_validation(obj))

    def test_account_number_empty(self):
        obj = {"value": ""}
        self.assertFalse(account_number_validation(obj))

if __name__ == '__main__':
    unittest.main()
