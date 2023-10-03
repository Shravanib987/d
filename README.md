I am getting below error for the test case

    @patch('validation.managed_account_validation.ManagedAccountsService.retrieve_managed_accounts')
    def test_validating_account_managed_is_managed(self, mock_retrieve_managed_accounts):
        mock_retrieve_managed_accounts.return_value = [{"accountManagementType": "MANAGED"}]
        toa_record_data = {"isAccountManaged": "MANAGED"}
        ManagedAccountValidation.validating_account_managed(toa_record_data, self.stp_data)
        self.assertFalse(self.stp_data['is_stp_eligible'])
        self.assertIn("\nVanguard Account is Managed. STP Automation is out of scope. Sending to Manual Queue.", self.stp_data['message'])


        "C:\Program Files\Python3913\python.exe" C:/Users/u5hn/AppData/Roaming/JetBrains/IntelliJIdea2023.1/plugins/python/helpers/pycharm/_jb_pytest_runner.py --target managed_account_validation_test.py::TestManagedAccountValidation.test_validating_account_managed_is_managed 
Testing started at 11:55 AM ...
Launching pytest with arguments managed_account_validation_test.py::TestManagedAccountValidation::test_validating_account_managed_is_managed --no-header --no-summary -q in C:\Repos\nat-processor.lambda\tests\validation

============================= test session starts =============================
collecting ... collected 1 item

managed_account_validation_test.py::TestManagedAccountValidation::test_validating_account_managed_is_managed 

============================== 1 failed in 0.96s ==============================
FAILED [100%]{"timestamp": "2023-10-03 11:55:10,883.883", "module": "[managed_account_validation.validating_account_managed:10  ]", "level": "INFO    ", "message": "Vanguard Account Registration Validation Failed, unable to determine if this a Managed Account.
Vanguard Account Registration Validation Failed, unable to determine if this a Managed Account."}

managed_account_validation_test.py:33 (TestManagedAccountValidation.test_validating_account_managed_is_managed)
self = <tests.validation.managed_account_validation_test.TestManagedAccountValidation testMethod=test_validating_account_managed_is_managed>
mock_post = <MagicMock name='post' id='1851904559568'>
mock_retrieve_oauth_token = <MagicMock name='retrieve_oauth_token' id='1851904702832'>
mock_retrieve_vg_session_token = <MagicMock name='retrieve_vg_session_token' id='1851904718928'>

    @patch.object(VGSessionService, 'retrieve_vg_session_token', return_value=vg_session_token)
    @patch.object(OAuthService, 'retrieve_oauth_token', return_value=oauth_token)
    @patch.object(requests, 'post', return_value=MockResponse([{"accountManagementType": "MANAGED"}], 200))
    def test_validating_account_managed_is_managed(self, mock_post, mock_retrieve_oauth_token, mock_retrieve_vg_session_token):
        toa_record_data = {"brokerageAccount": {"accountId": '889400030162730'}}
        ManagedAccountValidation.validating_account_managed(toa_record_data, stp_data)
        self.assertFalse(stp_data['is_stp_eligible'])
>       self.assertIn("\nVanguard Account is Managed. STP Automation is out of scope. Sending to Manual Queue.", stp_data['message'])
E       AssertionError: '\nVanguard Account is Managed. STP Automation is out of scope. Sending to Manual Queue.' not found in '\nVanguard Account Registration Validation Failed, unable to determine if this a Managed Account.'

managed_account_validation_test.py:41: AssertionError

Process finished with exit code 1
