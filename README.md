    def execute_client_name(toa_record_data):
        client_names_response_data = {}
        for poid in toa_record_data["clientSearchResponse"]:
            response = ClientNamesService.retrieve_client_names(poid)
            client_names_response_data[str(poid)] = response
        toa_record_data["clientNameResponse"] = client_names_response_data if client_names_response_data else None

Test case for this similar like below


    @patch.object(ClientSearchService, 'retrieve_client_poids')
    def test_execute_client_search_response_none(self, mock_retrieve_clients):
        mock_retrieve_clients.return_value = None

        DataExtractor.execute_client_search(self.toa_record_data)
        self.assertIn('clientSearchResponse', self.toa_record_data)
        self.assertEqual(self.toa_record_data['clientSearchResponse'], None)

    @patch.object(ClientSearchService, 'retrieve_client_poids')
    def test_execute_client_search_clients_none(self, mock_retrieve_clients):
        mock_retrieve_clients.return_value = {"clients": None}

        DataExtractor.execute_client_search(self.toa_record_data)
        self.assertIn('clientSearchResponse', self.toa_record_data)
        self.assertEqual(self.toa_record_data['clientSearchResponse'], None)

    @patch.object(ClientNamesService, 'retrieve_client_names')
    def test_execute_client_name_success(self, mock_retrieve_client_names):
        self.toa_record_data["clientSearchResponse"] = [2000044468]
        for poid in self.toa_record_data["clientSearchResponse"]:
            mock_retrieve_client_names.return_value = {str(poid): ["Mockdata"]}
            DataExtractor.execute_client_name(self.toa_record_data)
            self.assertIn('clientNameResponse', self.toa_record_data)
            self.assertEqual(self.toa_record_data['clientNameResponse'], ["Mockdata"])
