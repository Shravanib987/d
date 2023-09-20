@staticmethod
def verify_account_name_match(filter_data):
    vg_primary_owner_name = filter_data.get(Constants.VG_PRIMARY_OWNER_NAME, '').strip()
    contrafirm_primary_owner_name = filter_data.get(Constants.CONTRAFIRM_PRIMARY_OWNER_NAME, '').strip()
    vg_secondary_owner_name = filter_data.get(Constants.VG_SECONDARY_OWNER_NAME, '').strip()
    contrafirm_secondary_owner_name = filter_data.get(Constants.CONTRAFIRM_SECONDARY_OWNER_NAME, '').strip()

    name_match = False

    if vg_primary_owner_name == contrafirm_primary_owner_name:
        name_match = True
        LOGGER.info('Both Vanguard and Contrafirm primary Account owner names are matching')
    else:
        LOGGER.info('Both Vanguard and Contrafirm Primary Account owner names are NOT matching')

    if vg_secondary_owner_name and contrafirm_secondary_owner_name:
        if vg_secondary_owner_name == contrafirm_secondary_owner_name:
            name_match = True
            LOGGER.info('Both Primary and Secondary names of Vanguard and Contrafirm are matching')
        else:
            name_match = False

    return name_match
