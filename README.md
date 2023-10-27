This is data_extractor.py file

import time
from concurrent.futures import ThreadPoolExecutor, as_completed
from threading import Lock

from constants.constant_values import Constants
from service.account_roles_service import AccountRolesService
from service.beta_connector_service import BetaConnectorService
from service.brokerage_accounts_service import BrokerageAccountsService
from service.client_name_service import ClientNamesService
from service.client_search_service import ClientSearchService
from service.custodian_service import CustodianService
from service.managed_accounts_service import ManagedAccountsService
from service.restrictions_service import RestrictionService


class DataExtractor:

    @staticmethod
    def execute_account_restrictions(toa_record_data):
        restriction_response_data = RestrictionService.retrieve_account_restrictions(
            toa_record_data[Constants.CLIENT_ID_FIELD])
        toa_record_data["restrictions"] = None if restriction_response_data is None else restriction_response_data.get(
            "accountLevelRestrictions")

    @staticmethod
    def execute_account_roles(toa_record_data):
        account_roles_response_data = AccountRolesService.retrieve_account_roles(
            toa_record_data[Constants.CLIENT_ID_FIELD])
        toa_record_data["accountRoles"] = None if account_roles_response_data is None else account_roles_response_data.get(
            "accounts")

    @staticmethod
    def execute_beta_spad_toar_transaction_details(toa_record_data):
        spad_transaction_response_data = BetaConnectorService.retrieve_spad_transaction_details(toa_record_data.get(Constants.VG_ACCOUNT_NUMBER))
        toar_transaction_response_data = BetaConnectorService.retrieve_toar_transaction_details(toa_record_data.get(Constants.VG_ACCOUNT_NUMBER))
        if spad_transaction_response_data and spad_transaction_response_data.get("getAccountNotesResponse"):
            toa_record_data["spadTransactionDetails"] = spad_transaction_response_data.get("getAccountNotesResponse").get("accountNotes")
        else:
            toa_record_data["spadTransactionDetails"] = None
        if toar_transaction_response_data and toar_transaction_response_data.get("getAccountTransferRequestsResponse"):
            toa_record_data["toarTransactionDetails"] = toar_transaction_response_data.get("getAccountTransferRequestsResponse").get("accountTransfers")
        else:
            toa_record_data["toarTransactionDetails"] = None

    @staticmethod
    def execute_brokerage_accounts(toa_record_data):
        brokerage_accounts_response_data = BrokerageAccountsService.retrieve_brokerage_accounts(toa_record_data[Constants.CLIENT_ID_FIELD])
        toa_record_data["brokerageAccount"] = None if brokerage_accounts_response_data is None \
            else BrokerageAccountsService.extract_matching_account(brokerage_accounts_response_data["accounts"], toa_record_data.get(Constants.VG_ACCOUNT_NUMBER))

    @staticmethod
    def execute_client_search(toa_record_data):
        client_search_response_data = ClientSearchService.retrieve_client_poids(toa_record_data.get(Constants.VG_ACCOUNT_NUMBER))
        if client_search_response_data and client_search_response_data.get("clients"):
            toa_record_data["clientSearchResponse"] = client_search_response_data.get("clients").get("poid")
        else:
            toa_record_data["clientSearchResponse"] = None

    @staticmethod
    def execute_client_name(toa_record_data):
        if not toa_record_data.get("clientSearchResponse"):
            toa_record_data["clientNameResponse"] = None
        else:
            client_names_response_data = {}
            for poid in toa_record_data["clientSearchResponse"]:
                client_names_response = ClientNamesService.retrieve_client_names(poid)
                if client_names_response and "data" in client_names_response and "getClientUserContext" in client_names_response.get("data"):
                    client_names_response = client_names_response.get("data").get("getClientUserContext")
                else:
                    client_names_response = None
                client_names_response_data[str(poid)] = client_names_response
            toa_record_data["clientNameResponse"] = client_names_response_data

    @staticmethod
    def execute_custodian_details(toa_record_data):
        custodian_response_data = CustodianService.retrieve_custodian_details(
            toa_record_data[Constants.CLIENT_ID_FIELD], toa_record_data[Constants.CUSTODIAN_NAME])
        toa_record_data[
            "custodianserviceresponse"] = None if not custodian_response_data else custodian_response_data.get(
            "custodians")

    @staticmethod
    def execute_managed_accounts(toa_record_data):
        account_id = None if not toa_record_data.get('brokerageAccount') else toa_record_data.get('brokerageAccount').get('accountId')
        managed_accounts_response_data = ManagedAccountsService.retrieve_managed_accounts(account_id)
        toa_record_data["isAccountManaged"] = None if not managed_accounts_response_data else ManagedAccountsService.extract_managed_flag(managed_accounts_response_data)

    @staticmethod
    def execute_multiple_services(toa_record_data):
        list_methods = [DataExtractor.execute_account_restrictions, DataExtractor.execute_account_roles,
                        DataExtractor.execute_beta_spad_toar_transaction_details,
                        DataExtractor.execute_brokerage_accounts, DataExtractor.execute_custodian_details, DataExtractor.execute_client_search]
        with ThreadPoolExecutor(max_workers=len(list_methods)) as executor:
            lock = Lock()
            start_time = time.perf_counter()
            results = [executor.submit(method, toa_record_data) for method in
                       list_methods]
            for future in as_completed(results):
                future.result()
            end_time = time.perf_counter()

    @staticmethod
    def execute(toa_record_data):
        DataExtractor.execute_multiple_services(toa_record_data)
        DataExtractor.execute_client_name(toa_record_data)
        DataExtractor.execute_managed_accounts(toa_record_data)



This is data_extractor_test.py file


import sys


sys.path.append("../../src")
import unittest
from unittest.mock import patch
from src.util.data_extractor import AccountRolesService
from src.util.data_extractor import BetaConnectorService
from src.util.data_extractor import BrokerageAccountsService
from src.util.data_extractor import ClientNamesService
from src.util.data_extractor import ClientSearchService
from src.util.data_extractor import CustodianService
from src.util.data_extractor import DataExtractor
from src.util.data_extractor import ManagedAccountsService
from src.util.data_extractor import RestrictionService
from src.constants.constant_values import Constants


class TestDataExtractor(unittest.TestCase):

    def setUp(self):
        self.toa_record_data = {Constants.CLIENT_ID_FIELD: "12345"}

    @patch.object(RestrictionService, 'retrieve_account_restrictions')
    @patch.object(AccountRolesService, 'retrieve_account_roles')
    def test_execute_account_restrictions_success(self, mock_retrieve_account_roles, mock_retrieve_account_restrictions):
        mock_retrieve_account_restrictions.return_value = {"accountLevelRestrictions": "Mocked data"}
        mock_retrieve_account_roles.return_value = {"accounts": "Mocked data"}

        DataExtractor.execute_account_restrictions(self.toa_record_data)
        mock_retrieve_account_restrictions.assert_called_with("12345")
        DataExtractor.execute_account_roles(self.toa_record_data)
        mock_retrieve_account_roles.assert_called_with("12345")
        self.assertIsNotNone(self.toa_record_data['restrictions'])

    @patch.object(RestrictionService, 'retrieve_account_restrictions')
    @patch.object(AccountRolesService, 'retrieve_account_roles')
    def test_execute_account_restrictions_failure(self, mock_retrieve_account_roles, mock_retrieve_account_restrictions):
        mock_retrieve_account_restrictions.return_value = None
        mock_retrieve_account_roles.return_value = None

        DataExtractor.execute_account_restrictions(self.toa_record_data)
        mock_retrieve_account_restrictions.assert_called_with("12345")
        DataExtractor.execute_account_roles(self.toa_record_data)
        mock_retrieve_account_roles.assert_called_with("12345")
        self.assertIsNone(self.toa_record_data['restrictions'])

    @patch.object(ClientNamesService, 'retrieve_client_names')
    @patch.object(ClientSearchService, 'retrieve_client_poids')
    @patch.object(CustodianService, 'retrieve_custodian_details')
    @patch.object(RestrictionService, 'retrieve_account_restrictions')
    @patch.object(AccountRolesService, 'retrieve_account_roles')
    @patch.object(BrokerageAccountsService, 'retrieve_brokerage_accounts')
    @patch.object(BetaConnectorService, 'retrieve_spad_transaction_details')
    @patch.object(ManagedAccountsService, 'retrieve_managed_accounts')
    def test_execute_success(self, mock_retrieve_managed_accounts, mock_retrieve_spad_transaction_details,
                             mock_retrieve_brokerage_accounts, mock_retrieve_account_restrictions,
                             mock_retrieve_account_roles, mock_retrieve_custodian_details,
                             mock_client_search_service, mock_client_names_service):
        self.toa_record_data[Constants.CUSTODIAN_NAME] = "finname"
        self.toa_record_data[Constants.VG_ACCOUNT_NUMBER] = "1234"
        mock_client_search_service.return_value = {"clients": {"poid":[2000044468]}}

        DataExtractor.execute(self.toa_record_data)
        mock_retrieve_brokerage_accounts.assert_called_once()
        mock_retrieve_account_restrictions.assert_called_once()
        mock_retrieve_account_roles.assert_called_once()
        mock_retrieve_custodian_details.assert_called_once()
        mock_client_search_service.assert_called_once()
        mock_client_names_service.assert_called_once()
        mock_retrieve_spad_transaction_details.assert_called_once()
        mock_retrieve_managed_accounts.assert_called_once()

    @patch.object(AccountRolesService, 'retrieve_account_roles')
    def test_execute_account_roles_success(self, mock_retrieve_account_roles):
        mock_retrieve_account_roles.return_value = {"accounts": ["Mockdata"]}

        DataExtractor.execute_account_roles(self.toa_record_data)
        self.assertIn('accountRoles', self.toa_record_data)
        self.assertEqual(self.toa_record_data['accountRoles'], ["Mockdata"])

    @patch.object(AccountRolesService, 'retrieve_account_roles')
    def test_execute_account_roles_accounts_list_null(self, mock_retrieve_account_roles):
        mock_retrieve_account_roles.return_value = {"accounts": []}

        DataExtractor.execute_account_roles(self.toa_record_data)
        self.assertIn('accountRoles', self.toa_record_data)
        self.assertEqual(self.toa_record_data['accountRoles'], [])

    @patch.object(AccountRolesService, 'retrieve_account_roles')
    def test_execute_account_roles_response_none(self, mock_retrieve_account_roles):
        mock_retrieve_account_roles.return_value = None

        DataExtractor.execute_account_roles(self.toa_record_data)
        self.assertIn('accountRoles', self.toa_record_data)
        self.assertEqual(self.toa_record_data['accountRoles'], None)

    @patch.object(BetaConnectorService, 'retrieve_spad_transaction_details')
    def test_execute_beta_spad_transaction_details_success(self, mock_retrieve_spad_transaction_details):
        mock_retrieve_spad_transaction_details.return_value = {"getAccountNotesResponse": {"accountNotes": ["Mockdata"]}}
        self.toa_record_data[Constants.VG_ACCOUNT_NUMBER] = "1234"

        DataExtractor.execute_beta_spad_transaction_details(self.toa_record_data)
        self.assertIn('spadTransactionDetails', self.toa_record_data)
        self.assertEqual(self.toa_record_data['spadTransactionDetails'], ["Mockdata"])

    @patch.object(BetaConnectorService, 'retrieve_spad_transaction_details')
    def test_execute_beta_spad_transaction_details_response_none(self, mock_retrieve_spad_transaction_details):
        mock_retrieve_spad_transaction_details.return_value = None
        self.toa_record_data[Constants.VG_ACCOUNT_NUMBER] = "1234"

        DataExtractor.execute_beta_spad_transaction_details(self.toa_record_data)
        self.assertIn('spadTransactionDetails', self.toa_record_data)
        self.assertEqual(self.toa_record_data['spadTransactionDetails'], None)

    @patch.object(BetaConnectorService, 'retrieve_spad_transaction_details')
    def test_execute_beta_spad_transaction_details_getAccountNotesResponse_none(self, mock_retrieve_spad_transaction_details):
        mock_retrieve_spad_transaction_details.return_value = {"getAccountNotesResponse": None}
        self.toa_record_data[Constants.VG_ACCOUNT_NUMBER] = "1234"

        DataExtractor.execute_beta_spad_transaction_details(self.toa_record_data)
        self.assertIn('spadTransactionDetails', self.toa_record_data)
        self.assertEqual(self.toa_record_data['spadTransactionDetails'], None)

    @patch.object(BrokerageAccountsService, 'retrieve_brokerage_accounts')
    @patch.object(BrokerageAccountsService, 'extract_matching_account')
    def test_execute_brokerage_accounts_success(self, mock_extract_matching_account, mock_retrieve_brokerage_accounts):
        mock_retrieve_brokerage_accounts.return_value = {"accounts": ["Mockdata"]}
        mock_extract_matching_account.return_value = "Mockdata"

        DataExtractor.execute_brokerage_accounts(self.toa_record_data)
        self.assertIn('brokerageAccount', self.toa_record_data)
        self.assertEqual(self.toa_record_data['brokerageAccount'], "Mockdata")
        mock_extract_matching_account.assert_called_once()

    @patch.object(BrokerageAccountsService, 'retrieve_brokerage_accounts')
    @patch.object(BrokerageAccountsService, 'extract_matching_account')
    def test_execute_brokerage_accounts_list_null(self, mock_extract_matching_account, mock_retrieve_brokerage_accounts):
        mock_retrieve_brokerage_accounts.return_value = {"accounts": []}
        mock_extract_matching_account.return_value = None

        DataExtractor.execute_brokerage_accounts(self.toa_record_data)
        self.assertIn('brokerageAccount', self.toa_record_data)
        self.assertIsNone(self.toa_record_data['brokerageAccount'])
        mock_extract_matching_account.assert_called_once()

    @patch.object(BrokerageAccountsService, 'retrieve_brokerage_accounts')
    @patch.object(BrokerageAccountsService, 'extract_matching_account')
    def test_execute_brokerage_accounts_response_none(self, mock_extract_matching_account, mock_retrieve_brokerage_accounts):
        mock_retrieve_brokerage_accounts.return_value = None

        DataExtractor.execute_brokerage_accounts(self.toa_record_data)
        self.assertIn('brokerageAccount', self.toa_record_data)
        self.assertEqual(self.toa_record_data['brokerageAccount'], None)
        mock_extract_matching_account.assert_not_called()

    @patch.object(ClientSearchService, 'retrieve_client_poids')
    def test_execute_client_search_success(self, mock_retrieve_clients):
        mock_retrieve_clients.return_value = {"clients": {"poid": ["Mockdata"]}}

        DataExtractor.execute_client_search(self.toa_record_data)
        self.assertIn('clientSearchResponse', self.toa_record_data)
        self.assertEqual(self.toa_record_data['clientSearchResponse'], ["Mockdata"])

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
    def test_execute_client_name_success(self, mock_client_names_service):
        self.toa_record_data["clientSearchResponse"] = [2000044468]
        poid = str(self.toa_record_data['clientSearchResponse'][0])
        mock_data = {poid : {"data": {"getClientUserContext":["Mockdata"]}}}
        mock_client_names_service.return_value = mock_data.get(poid)
        DataExtractor.execute_client_name(self.toa_record_data)
        self.assertIn('clientNameResponse', self.toa_record_data)
        self.assertEqual(self.toa_record_data['clientNameResponse'].get(poid), mock_data.get(poid).get("data").get("getClientUserContext"))

    @patch.object(ClientNamesService, 'retrieve_client_names')
    def test_execute_client_name_response_none(self, mock_client_names_service):
        self.toa_record_data["clientSearchResponse"] = None
        mock_data = None
        mock_client_names_service.return_value = mock_data
        DataExtractor.execute_client_name(self.toa_record_data)
        self.assertIn('clientNameResponse', self.toa_record_data)
        self.assertEqual(self.toa_record_data['clientNameResponse'], mock_data)

    @patch.object(ClientNamesService, 'retrieve_client_names')
    def test_execute_client_name_client_names_none(self, mock_client_names_service):
        self.toa_record_data["clientSearchResponse"] = [2000044468]
        mock_data = {str(self.toa_record_data["clientSearchResponse"][0]):None}
        mock_client_names_service.return_value = mock_data
        DataExtractor.execute_client_name(self.toa_record_data)
        self.assertIn('clientNameResponse', self.toa_record_data)
        self.assertEqual((self.toa_record_data['clientNameResponse']), mock_data)

    @patch.object(CustodianService, 'retrieve_custodian_details')
    def test_execute_custodian_details_success(self, mock_retrieve_custodian_details):
        mock_retrieve_custodian_details.return_value = {"custodians": ["Mockdata"]}
        self.toa_record_data[Constants.CUSTODIAN_NAME] = "finname"

        DataExtractor.execute_custodian_details(self.toa_record_data)
        self.assertIn('custodianserviceresponse', self.toa_record_data)
        self.assertEqual(self.toa_record_data['custodianserviceresponse'], ["Mockdata"])

    @patch.object(CustodianService, 'retrieve_custodian_details')
    def test_execute_custodian_details_list_null(self, mock_retrieve_custodian_details):
        mock_retrieve_custodian_details.return_value = {"custodians": []}
        self.toa_record_data[Constants.CUSTODIAN_NAME] = "finname"

        DataExtractor.execute_custodian_details(self.toa_record_data)
        self.assertIn('custodianserviceresponse', self.toa_record_data)
        self.assertEqual(self.toa_record_data['custodianserviceresponse'], [])

    @patch.object(CustodianService, 'retrieve_custodian_details')
    def test_execute_custodian_details_list_response_none(self, mock_retrieve_custodian_details):
        mock_retrieve_custodian_details.return_value = None
        self.toa_record_data[Constants.CUSTODIAN_NAME] = "finname"

        DataExtractor.execute_custodian_details(self.toa_record_data)
        self.assertIn('custodianserviceresponse', self.toa_record_data)
        self.assertEqual(self.toa_record_data['custodianserviceresponse'], None)

    @patch.object(ManagedAccountsService, 'retrieve_managed_accounts')
    @patch.object(ManagedAccountsService, 'extract_managed_flag')
    def test_execute_managed_accounts_success(self, mock_extract_managed_flag, mock_retrieve_managed_accounts):
        self.toa_record_data['brokerageAccount'] = {"accountId": "1234"}
        mock_retrieve_managed_accounts.return_value = {"accountManagementType": "MANAGED"}
        mock_extract_managed_flag.return_value = "MANAGED"

        DataExtractor.execute_managed_accounts(self.toa_record_data)
        self.assertIn('isAccountManaged', self.toa_record_data)
        self.assertEqual(self.toa_record_data['isAccountManaged'], "MANAGED")
        mock_extract_managed_flag.assert_called_once()

    @patch.object(ManagedAccountsService, 'retrieve_managed_accounts')
    @patch.object(ManagedAccountsService, 'extract_managed_flag')
    def test_execute_managed_accounts_no_account_id_none_response(self, mock_extract_managed_flag, mock_retrieve_managed_accounts):
        self.toa_record_data['brokerageAccount'] = None
        mock_retrieve_managed_accounts.return_value = None

        DataExtractor.execute_managed_accounts(self.toa_record_data)
        self.assertIn('isAccountManaged', self.toa_record_data)
        self.assertEqual(self.toa_record_data['isAccountManaged'], None)
        mock_extract_managed_flag.assert_not_called()

    @patch.object(ManagedAccountsService, 'retrieve_managed_accounts')
    @patch.object(ManagedAccountsService, 'extract_managed_flag')
    def test_execute_managed_accounts_empty_response(self, mock_extract_managed_flag, mock_retrieve_managed_accounts):
        self.toa_record_data['brokerageAccount'] = {"accountId": "1234"}
        mock_retrieve_managed_accounts.return_value = []

        DataExtractor.execute_managed_accounts(self.toa_record_data)
        self.assertIn('isAccountManaged', self.toa_record_data)
        self.assertEqual(self.toa_record_data['isAccountManaged'], None)
        mock_extract_managed_flag.assert_not_called()





Please edit the spad toar test case and provide me.. It should be similar way of style with all other test cases
