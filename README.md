Detail Description
As a processor I want the lambda to validate the account is not a managed account.

 

As in the example link above, make sure to specifically compare against the MANAGED keyword. Anything other than MANAGED can be considered unmanaged.

 

Acceptance Criteria

Given the account listed in the answer file is NOT a managed account

And the service call is successful

And managed account data is returned for the given client/account

When the lambda validates the account is not managed

Then it logs a message (temporary)

And continues on to the next step

 

Given the account listed in the answer file is a managed account

And the service call is successful

And account data is returned for the given account

When the lambda validates the account

Then it logs a message (temporary)

and sets  STP flag to false

and moves on to the next validation

 

Given the account listed in the answer file is a managed account

And the service call is NOT successful

And account data is none

When the lambda validates the account

Then it logs a message (temporary)

and sets  STP flag to false

and moves on to the next validation
