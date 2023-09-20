    @staticmethod
    def verify_ssn_match(filter_data):
        ssn_name_match = False
        vanguard_ssn = ''
        contrafirm_ssn = ''
        for key,value in filter_data.items():
            if key == Constants.CONTRAFIRM_SSN:
                contrafirm_ssn = value
            elif key == Constants.VANGUARD_SSN:
                vanguard_ssn = value
        if vanguard_ssn == contrafirm_ssn:
            ssn_name_match = True
            LOGGER.info('Both Vanguard and Contrafirm SSN are matching')
        else:
            LOGGER.info('Both Vanguard and Contrafirm SSN are NOT matching')
        return ssn_name_match
