import unittest
import json

# Import the function you want to test
from your_module_name import check_names_and_values

class TestCheckNamesAndValues(unittest.TestCase):

    def test_matching_names_and_values(self):
        data = [
            {"answer": {"name": "Primary account owner name or registration as it appears at the other firm", "value": "John"}},
            {"answer": {"name": "Primary account owner name", "value": "John"}}
        ]
        result = check_names_and_values(data)
        self.assertTrue(result, "Expected matching names and values")

    def test_non_matching_names_and_values(self):
        data = [
            {"answer": {"name": "Primary account owner name or registration as it appears at the other firm", "value": "John"}},
            {"answer": {"name": "Primary account owner name", "value": "Jane"}}
        ]
        result = check_names_and_values(data)
        self.assertFalse(result, "Expected non-matching names and values")

if __name__ == '__main__':
    unittest.main()
