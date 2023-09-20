@staticmethod
    def verify_account_name_match(filter_data):
        name_match = False
        vg_primary_owner_name = contrafirm_primary_owner_name = vg_secondary_owner_name = contrafirm_secondary_owner_name = ''
        for key,value in filter_data.items():
            if key == Constants.VG_PRIMARY_OWNER_NAME:
                vg_primary_owner_name = value
            elif key == Constants.CONTRAFIRM_PRIMARY_OWNER_NAME:
                contrafirm_primary_owner_name = value
            elif key == Constants.VG_SECONDARY_OWNER_NAME:
                vg_secondary_owner_name = value
            elif key == Constants.CONTRAFIRM_SECONDARY_OWNER_NAME:
                contrafirm_secondary_owner_name = value
        if vg_primary_owner_name.strip() == contrafirm_primary_owner_name.strip():
            name_match = True
            LOGGER.info('Both Vanguard and Contrafirm primary Account owner names are matching')
        else:
            LOGGER.info('Both Vanguard and Contrafirm Primary Account owner names are NOT matching')
        if vg_secondary_owner_name != '' and contrafirm_secondary_owner_name != '':
            if vg_primary_owner_name == vg_secondary_owner_name and contrafirm_primary_owner_name == contrafirm_secondary_owner_name:
                name_match = True
                LOGGER.info('Both Primary and Secondary names of Vanguard and Contrafirm are matching')
            else:
                name_match = False
        return name_match
