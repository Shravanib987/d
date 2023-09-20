@staticmethod
def verify_ssn_match(filter_data):
    vanguard_ssn = filter_data.get(Constants.VANGUARD_SSN, '').strip()
    contrafirm_ssn = filter_data.get(Constants.CONTRAFIRM_SSN, '').strip()

    ssn_name_match = False

    if vanguard_ssn == contrafirm_ssn:
        ssn_name_match = True
        LOGGER.info('Both Vanguard and Contrafirm SSN are matching')
    else:
        LOGGER.info('Both Vanguard and Contrafirm SSN are NOT matching')

    return ssn_name_match
