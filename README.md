import json

with open('xmltojson.json', 'r') as json_file:
    data = json.load(json_file)

json_data = data['intelledoxAnswerFile']['submissions']['results']

# Convert the filtered result into a list
answer_data = list(filter(lambda obj: "answer" in obj, json_data))

def check_names_and_values(data):
    for obj1 in data:
        for obj2 in data:
            if (
                "answer" in obj1
                and "answer" in obj2
                and obj1["answer"]["name"] == "Primary account owner name or registration as it appears at the other firm"
                and obj2["answer"]["name"] == "Primary account owner name"
                and obj1["answer"]["value"] == obj2["answer"]["value"]
            ):
                return True
    return False

names_and_values_match = check_names_and_values(answer_data)

if names_and_values_match:
    print("matched")
else:
    print("not matched")
