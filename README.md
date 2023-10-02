    @patch.object(VGSessionService, 'retrieve_vg_session_token', return_value=vg_session_token)
    @patch.object(OAuthService, 'retrieve_oauth_token', return_value=oauth_token)
    @patch.object(requests, 'post', side_effect=Exception("Service call failed"))
    def test_service_call_failure(self, mock_post, mock_retrieve_oauth_token, mock_retrieve_vg_session_token):
        answer_file = {"brokerageAccount": {"accountId": '889400030162730'}}
        ManagedAccountValidation.validating_account_managed(answer_file, stp_data)
        self.assertTrue(stp_data['is_stp_eligible'])
        self.assertIn("Unable to determine account type due to service failure. Moving on to the next validation.", stp_data['message'])
        mock_post.assert_called_once()
