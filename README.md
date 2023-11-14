@patch('main_test_class.MyUtil')
    def test_method_to_test(self, mock_util_class):
        # Create an instance of the mock MyUtil class
        mock_util_instance = mock_util_class.return_value

        # Set the return values for the methods you want to mock
        mock_util_instance.method1.return_value = "mocked_method1_result"
        mock_util_instance.method2.return_value = "mocked_method2_result"

        # Create an instance of the main test class
        main_test_instance = MainTestClass()

        # Call the method you want to test
        result = main_test_instance.method_to_test()

        # Your assertions based on the mocked results
        self.assertEqual(result, "mocked_method1_resultmocked_method2_result")
