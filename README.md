for spad_data in data['spadTransactionDetails']:
    print(spad_data)
    control_Number = re.search(r'CTL-NO (\d+)', spad_data['note']).group(1) if re.search(r'CTL-NO (\d+)', data['spadTransactionDetails'][0]['note']) else None
    type = (re.search(r'TYPE (\w+)', spad_data['note']).group(1)).replace('PART','')
    participent_Number= re.search( r'FIKPART (\d+)', spad_data['note']).group(1)
    external_account_number = re.search(r'# (\d+)', spad_data['note']).group(1)

    print(control_Number)
    print(type)
    print(participent_Number)
    print(external_account_number)
