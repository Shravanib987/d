from util.logging_util import LOGGER
from constants.constant_values import Constants
class FieldValidation:
    @staticmethod
    def account_number_checking(filter_data):
        for data in filter_data['results']:
            if data['answer']['name'] == Constants.ACCOUNT_NUMBER:
                if data['answer']['value'] !="":
                    LOGGER.info('Account number is not empty continuing the process')
                else:
                    LOGGER.info('Account number is empty stopping the process')
                    break
    @staticmethod
    def acc_name_match(filter_data):
        for data1 in filter_data['results']:
            for data2 in filter_data['results']:
                if data1 ['answer']['name'] == Constants.CONTRAFIRM_PRI_ACCNAME and data2 ['answer']['name'] == Constants.VG_PRI_ACCNAME:
                    if data1 ['answer']['value'] == data2 ['answer']['value']:
                        LOGGER.info('Both Account names are matching')
                    else:
                        LOGGER.info('Both Account names are NOT matching')
                if data1 ['answer']['name'] == Constants.CONTRAFIRM_SEC_ACCNAME and data2 ['answer']['name'] == Constants.VG_SEC_ACCNAME:
                    if data1 ['answer']['value'] == data2 ['answer']['value']:
                        LOGGER.info('Both primary and secondary names are matching')
                    else:
                        LOGGER.info('Both  primary and secondary names are NOT matching')
    @staticmethod
    def SSN_match(filter_data):
        for data1 in filter_data['results']:
            for data2 in filter_data['results']:
                if data1 ['answer']['name'] == Constants.CONTRAFIRM_SSN and data2 ['answer']['name'] == Constants.VG_SSN:
                    if data1 ['answer']['value'] == data2 ['answer']['value']:
                        LOGGER.info('Both ssn are matching')
                    else:
                        LOGGER.info('Both ssn are NOT matching')


