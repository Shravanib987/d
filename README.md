import json
#Reading json file
with open('xmltojson.json', 'r') as json_file:
    data = json.load(json_file)
json_data = data['intelledoxAnswerFile']['submissions']['results']
# account number validation
account_number_checking = lambda obj: obj.get("name") == "Account number:"
account_number_validation = lambda obj: obj.get("value") != ""
for idx, obj in enumerate(json_data):
    if 'answer' in obj:
        if account_number_checking(obj['answer']):
            if account_number_validation(obj['answer']):
                print(f"Account number is not empty for object {idx + 1}. going to  the step.")
                #here will do log
            else:
                print(f"Account number is  empty for object {idx + 1}.  Stopping the process.")
                #here will do log
                break
