"C:\Program Files\Python3913\python.exe" C:/Users/u5hn/AppData/Roaming/JetBrains/IntelliJIdea2023.1/plugins/python/helpers/pycharm/_jb_pytest_runner.py --path C:\Repos\nat-processor.lambda\tests\util\field_validation_test.py 
Testing started at 4:39 PM ...
Launching pytest with arguments C:\Repos\nat-processor.lambda\tests\util\field_validation_test.py --no-header --no-summary -q in C:\Repos\nat-processor.lambda\tests\util

============================= test session starts =============================
collecting ... collected 6 items

field_validation_test.py::TestFieldValidation::test_diff_SSN_match 
field_validation_test.py::TestFieldValidation::test_diff_acc_name_match 
field_validation_test.py::TestFieldValidation::test_empty_account_number 
field_validation_test.py::TestFieldValidation::test_non_empty_account_number 
field_validation_test.py::TestFieldValidation::test_same_SSN_match 
field_validation_test.py::TestFieldValidation::test_same_acc_name_match 

========================= 5 failed, 1 passed in 0.42s =========================
FAILED [ 16%]
field_validation_test.py:74 (TestFieldValidation.test_diff_SSN_match)
'Both ssn are NOT matching' != None

Expected :None
Actual   :'Both ssn are NOT matching'
<Click to see difference>

self = <tests.util.field_validation_test.TestFieldValidation testMethod=test_diff_SSN_match>

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
>       self.assertEqual(result, 'Both ssn are NOT matching')

field_validation_test.py:86: AssertionError
FAILED [ 33%]
field_validation_test.py:46 (TestFieldValidation.test_diff_acc_name_match)
self = <tests.util.field_validation_test.TestFieldValidation testMethod=test_diff_acc_name_match>

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
>           mock_logger.info.assert_any_call('Both primary and secondary names are NOT matching')

field_validation_test.py:60: 
_ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _

self = <Mock name='mock.info' id='1854570880016'>
args = ('Both primary and secondary names are NOT matching',), kwargs = {}
expected = call('Both primary and secondary names are NOT matching')
cause = None
actual = [call('Both Account names are NOT matching'), call('Both primary and secondary names are matching')]
expected_string = "info('Both primary and secondary names are NOT matching')"

    def assert_any_call(self, /, *args, **kwargs):
        """assert the mock has been called with the specified arguments.
    
        The assert passes if the mock has *ever* been called, unlike
        `assert_called_with` and `assert_called_once_with` that only pass if
        the call is the most recent one."""
        expected = self._call_matcher(_Call((args, kwargs), two=True))
        cause = expected if isinstance(expected, Exception) else None
        actual = [self._call_matcher(c) for c in self.call_args_list]
        if cause or expected not in _AnyComparer(actual):
            expected_string = self._format_mock_call_signature(args, kwargs)
>           raise AssertionError(
                '%s call not found' % expected_string
            ) from cause
E           AssertionError: info('Both primary and secondary names are NOT matching') call not found

C:\Program Files\Python3913\lib\unittest\mock.py:978: AssertionError
FAILED [ 50%]
field_validation_test.py:18 (TestFieldValidation.test_empty_account_number)
'Account number is empty stopping the process' != None

Expected :None
Actual   :'Account number is empty stopping the process'
<Click to see difference>

self = <tests.util.field_validation_test.TestFieldValidation testMethod=test_empty_account_number>

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
>       self.assertEqual(result, 'Account number is empty stopping the process')

field_validation_test.py:30: AssertionError
FAILED [ 66%]
field_validation_test.py:5 (TestFieldValidation.test_non_empty_account_number)
'Account number is not empty continuing the process' != None

Expected :None
Actual   :'Account number is not empty continuing the process'
<Click to see difference>

self = <tests.util.field_validation_test.TestFieldValidation testMethod=test_non_empty_account_number>

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
>       self.assertEqual(result, 'Account number is not empty continuing the process')

field_validation_test.py:17: AssertionError
FAILED [ 83%]
field_validation_test.py:61 (TestFieldValidation.test_same_SSN_match)
'Both ssn are matching' != None

Expected :None
Actual   :'Both ssn are matching'
<Click to see difference>

self = <tests.util.field_validation_test.TestFieldValidation testMethod=test_same_SSN_match>

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
>       self.assertEqual(result, 'Both ssn are matching')

field_validation_test.py:73: AssertionError
PASSED [100%]
Process finished with exit code 1
