"C:\Program Files\Python3913\python.exe" C:/Users/u5hn/AppData/Roaming/JetBrains/IntelliJIdea2023.1/plugins/python/helpers/pycharm/_jb_pytest_runner.py --path C:\Repos\nat-processor.lambda\tests\util\field_validation_test.py 
Testing started at 4:36 PM ...
Launching pytest with arguments C:\Repos\nat-processor.lambda\tests\util\field_validation_test.py --no-header --no-summary -q in C:\Repos\nat-processor.lambda\tests\util

============================= test session starts =============================
collecting ... collected 6 items

field_validation_test.py::TestFieldValidation::test_diff_SSN_match 
field_validation_test.py::TestFieldValidation::test_diff_acc_name_match 
field_validation_test.py::TestFieldValidation::test_empty_account_number 
field_validation_test.py::TestFieldValidation::test_non_empty_account_number 
field_validation_test.py::TestFieldValidation::test_same_SSN_match 
field_validation_test.py::TestFieldValidation::test_same_acc_name_match 

============================== 6 failed in 0.50s ==============================
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
            mock_logger.info.assert_called_once_with('Both ssn are NOT matching')
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
>           mock_logger.info.assert_called_with('Both Account names are NOT matching')

field_validation_test.py:59: 
_ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _

self = <Mock name='mock.info' id='2377409421120'>
args = ('Both Account names are NOT matching',), kwargs = {}
expected = call('Both Account names are NOT matching')
actual = call('Both primary and secondary names are matching')
_error_message = <function NonCallableMock.assert_called_with.<locals>._error_message at 0x0000022988A7E8B0>
cause = None

    def assert_called_with(self, /, *args, **kwargs):
        """assert that the last call was made with the specified arguments.
    
        Raises an AssertionError if the args and keyword args passed in are
        different to the last call to the mock."""
        if self.call_args is None:
            expected = self._format_mock_call_signature(args, kwargs)
            actual = 'not called.'
            error_message = ('expected call not found.\nExpected: %s\nActual: %s'
                    % (expected, actual))
            raise AssertionError(error_message)
    
        def _error_message():
            msg = self._format_mock_failure_message(args, kwargs)
            return msg
        expected = self._call_matcher(_Call((args, kwargs), two=True))
        actual = self._call_matcher(self.call_args)
        if actual != expected:
            cause = expected if isinstance(expected, Exception) else None
>           raise AssertionError(_error_message()) from cause
E           AssertionError: expected call not found.
E           Expected: info('Both Account names are NOT matching')
E           Actual: info('Both primary and secondary names are matching')

C:\Program Files\Python3913\lib\unittest\mock.py:907: AssertionError
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
            mock_logger.info.assert_called_once_with('Account number is empty stopping the process')
>       self.assertEqual(result, 'Account number is empty stopping the process')

field_validation_test.py:30: AssertionError
FAILED [ 66%]
field_validation_test.py:5 (TestFieldValidation.test_non_empty_account_number)
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
>           mock_logger.info.assert_called_once_with('Account number is not empty continuing the process')

field_validation_test.py:16: 
_ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _

self = <Mock name='mock.info' id='2377379893008'>
args = ('Account number is not empty continuing the process',), kwargs = {}
msg = "Expected 'info' to be called once. Called 2 times.\nCalls: [call('Account number is not empty continuing the process'),\n call('Account number is not empty continuing the process')]."

    def assert_called_once_with(self, /, *args, **kwargs):
        """assert that the mock was called exactly once and that that call was
        with the specified arguments."""
        if not self.call_count == 1:
            msg = ("Expected '%s' to be called once. Called %s times.%s"
                   % (self._mock_name or 'mock',
                      self.call_count,
                      self._calls_repr()))
>           raise AssertionError(msg)
E           AssertionError: Expected 'info' to be called once. Called 2 times.
E           Calls: [call('Account number is not empty continuing the process'),
E            call('Account number is not empty continuing the process')].

C:\Program Files\Python3913\lib\unittest\mock.py:918: AssertionError
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
            mock_logger.info.assert_called_once_with('Both ssn are matching')
>       self.assertEqual(result, 'Both ssn are matching')

field_validation_test.py:73: AssertionError
FAILED [100%]
field_validation_test.py:31 (TestFieldValidation.test_same_acc_name_match)
self = <tests.util.field_validation_test.TestFieldValidation testMethod=test_same_acc_name_match>

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
>           mock_logger.info.assert_called_with('Both Account names are matching')

field_validation_test.py:44: 
_ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _

self = <Mock name='mock.info' id='2377409749536'>
args = ('Both Account names are matching',), kwargs = {}
expected = call('Both Account names are matching')
actual = call('Both primary and secondary names are matching')
_error_message = <function NonCallableMock.assert_called_with.<locals>._error_message at 0x0000022987BC38B0>
cause = None

    def assert_called_with(self, /, *args, **kwargs):
        """assert that the last call was made with the specified arguments.
    
        Raises an AssertionError if the args and keyword args passed in are
        different to the last call to the mock."""
        if self.call_args is None:
            expected = self._format_mock_call_signature(args, kwargs)
            actual = 'not called.'
            error_message = ('expected call not found.\nExpected: %s\nActual: %s'
                    % (expected, actual))
            raise AssertionError(error_message)
    
        def _error_message():
            msg = self._format_mock_failure_message(args, kwargs)
            return msg
        expected = self._call_matcher(_Call((args, kwargs), two=True))
        actual = self._call_matcher(self.call_args)
        if actual != expected:
            cause = expected if isinstance(expected, Exception) else None
>           raise AssertionError(_error_message()) from cause
E           AssertionError: expected call not found.
E           Expected: info('Both Account names are matching')
E           Actual: info('Both primary and secondary names are matching')

C:\Program Files\Python3913\lib\unittest\mock.py:907: AssertionError

Process finished with exit code 1
