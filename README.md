import re
import Constants

def validate_contrafirm_account_number(toa_recrddata,stp_data):
    account_numbers = re.split(r'[;,]', toa_recrddata[Constants.CONTRAFIRM_ACCOUNT_NUMBER])

    stp_flag = True

    if len(account_numbers) > 1:
        stp_flag = False

    return stp_flag
