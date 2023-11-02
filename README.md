class TestValidatingTransferType(unittest.TestCase):
    def test_broker_dealer_full_transfer_true(self):
        toa_record_data = {Constants.BROKER_DEALER_FULL_TRANSFFERTYE: 'true'}
        validating_transfer_type(toa_record_data)
        self.assertTrue(toa_record_data['liquidation_stamp'])

    def test_mutual_fund_transfer_all_assets_true(self):
        toa_record_data = {Constants.MUTAL_FUND_TRANSFER_ALL_ASSETS: 'true'}
        validating_transfer_type(toa_record_data)
        self.assertTrue(toa_record_data['liquidation_stamp'])

    def test_mutual_fund_liquidate_all_assets_true(self):
        toa_record_data = {Constants.MUTAL_FUND_LIQUIDATE_ALL_ASSETS: 'true'}
        validating_transfer_type(toa_record_data)
        self.assertTrue(toa_record_data['liquidation_stamp'])

    def test_all_false(self):
        toa_record_data = {
            Constants.BROKER_DEALER_FULL_TRANSFFERTYE: 'false',
            Constants.MUTAL_FUND_TRANSFER_ALL_ASSETS: 'false',
            Constants.MUTAL_FUND_LIQUIDATE_ALL_ASSETS: 'false',
        }
        validating_transfer_type(toa_record_data)
        self.assertFalse(toa_record_data['liquidation_stamp'])
