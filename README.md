import json

with open('xmltojson.json', 'r') as json_file:
    data = json.load(json_file)

json_data = data['intelledoxAnswerFile']['submissions']['results']

# Use lambda functions for filtering and checking
filter_answer_data = lambda data: filter(lambda obj: "answer" in obj, data)
answer_data = list(filter_answer_data(json_data))

check_names_and_values = lambda data: any(
    obj1["answer"]["name"] == "Primary account owner name or registration as it appears at the other firm" and
    obj2["answer"]["name"] == "Primary account owner name" and
    obj1["answer"]["value"] == obj2["answer"]["value"]
    for obj1 in data
    for obj2 in data
)

names_and_values_match = check_names_and_values(answer_data)

if names_and_values_match:
    print("matched")
else:
    print("not matched")


