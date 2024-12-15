# AppSec_Vulnerabilities

The repository provides definition, explanation, attack patterns, examples of applications security vulnerabities.
This is a live document and I will keep updating it when I get time and whenever i increase my knowledge and get to know new attack
patterns and new type of vulnerabilities. The guide covers all of https://owasp.org/www-project-top-ten/ and far beyond that. 
I have used portswigger academy, owasp top 10, medium articles, personal research, personal exploit development, my experience and testing as
a guide to create this document. 

## Mass Assignment 
Mass assignment vulnerabilities occurs when an attacker/user can change internal objects fields value that they are not authorized to perform, because
week security settings. 

### Detecting the vulnerability in run time application

For example, consider a PATCH API endpoint /api/users/ request, which enables users to update their username and email, and includes the following JSON:

```json
{
    "username": "wiener",
    "email": "wiener@example.com",
}
```

A concurrent GET /api/users/123 request returns the following JSON:

```json
{
    "id": 123,
    "name": "John Doe",
    "email": "john@example.com",
    "isAdmin": "false"
}
```

To test whether you can modify the enumerated isAdmin parameter value, add it to the PATCH request:

```json
{
    "username": "wiener",
    "email": "wiener@example.com",
    "isAdmin": false,
}
```
In addition, send a PATCH request with an invalid isAdmin parameter value:

```json
{
    "username": "wiener",
    "email": "wiener@example.com",
    "isAdmin": "foo",
}
```

If the application behaves differently, this may suggest that the invalid value impacts the query logic, but the valid value doesn't

Then finally you can provide the true value and then check the response what you get, if you get a return with the value of isadmin as true, that
means that you have successfully exploited the vulnerability

```json
{
    "username": "wiener",
    "email": "wiener@example.com",
    "isAdmin": true,
}
```


### Detecting the vulnerability during source code review
Step1: Look for Patterns of Automatic Mapping
Reviewers should identify locations in the code where user input (e.g., req.body, params, or raw JSON payload) is directly assigned to objects or models:
  - Direct assignment of user input to objects, such as:

```
@user = User.new(params[:user])
```

```
const user = new User(req.body);
```

  - Framework methods that map input automatically (e.g., bind, assign, update_attributes):

```
user.update_attributes(params[:user])
```

```
$user->fill($request->all());
```

These patterns are common sources of mass assignment vulnerabilities.

Step2: When you see direct mappings, ask:
- Which fields are being mapped? If the code uses something like req.body or params[:object], it maps all the input properties by default. This is a red flag.
- Does the function explicitly filter sensitive fields? Check if there’s a whitelist of allowed fields (e.g., permit, pick, or similar mechanisms). If not, it’s a potential vulnerability.

Another important consideration is to understand properties of the object and understanding whether all properties are meant to be controlled by the user is a critical step.
If all properties must not be controlled by the user then this is potential a vulnerability, if user should control all properties of the object then this is an expected functionality.

**Potential Vulnerability:** 

If the developer confirms that not all properties are user-controlled, you need to check:
Which fields are sensitive or internal-only?

Example fields: 

- isAdmin
- user_role
- account_balance
- status
- id

Whether there is a mechanism to filter out these sensitive fields.

