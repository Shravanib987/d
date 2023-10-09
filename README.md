class DataExtractor:
    # ... (other methods remain the same)

    @staticmethod
    def execute_client_search(toa_record_data):
        client_search_response_data = ClientSearchService.retrieve_client_poids(toa_record_data.get(Constants.VG_ACCOUNT_NUMBER))
        if client_search_response_data and client_search_response_data.get("clients"):
            client_poids = client_search_response_data.get("clients").get("poid")
            toa_record_data["clientSearchResponse"] = {}
            for poid in client_poids:
                ovv_response_data = DataExtractor.get_ovv_response(poid)
                if ovv_response_data:
                    toa_record_data["clientSearchResponse"][poid] = ovv_response_data
        else:
            toa_record_data["clientSearchResponse"] = None

    @staticmethod
    def get_ovv_response(poid):
        try:
            # Make the OVV service call for the given poid
            client_names_response_data = ClientNamesService.retrieve_client_names(poid)
            if client_names_response_data and client_names_response_data.get("data"):
                return client_names_response_data.get("data").get("getClientUserContext")
            else:
                return None
        except Exception as exception:
            LOGGER.error("Exception occurred during OVV service request for poid %s: error=%s", poid, str(exception))
            return None
