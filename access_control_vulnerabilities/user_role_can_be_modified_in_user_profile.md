# User role can be modified in user profile

## Lab Description

This lab contains a `/admin` page that has strict role-based access control implemented in it, making it only accessible to users logged
in with a `roleid` of 2. Our goal for this lab is the same, i.e., access the `/admin` page and delete user `carlos`.

---

## My First Move
My first move for this lab is to try to access that `/admin` page after logging in with the given `wiener:peter` credentials and try to
find a roleid parameter after intercepting the request in Burp Suite, but unfortunately I found nothing; that request was clean without
any parameter passed from the client-side.
<img width="1365" height="735" alt="image" src="https://github.com/user-attachments/assets/9ab1815c-f6c0-48eb-992c-d8a716436699" />


---

## What failed and Why
After that, I kept exploring the entire website content to find something useful, I visited product page, intercepted login request, tried changing userid at the profile's URL but again I found absolutely nothing, all requets was clean without any parameter passed at any request in a website. My mistake was that I kept exploring the website but didn't check the "Change Email" feature, which was right in front of me on the profile page.

---

## What Worked
I was confused how can I find roleid parameter until I tried to change email of my profile and intercepted the `/my-account/change-email` request, then I sent it to Repeater to see how the website responded to that request. The request was:

```

POST /my-account/change-email HTTP/2
Host: 0a8700c003f221558069b21f00ba0041.web-security-academy.net
Cookie: session=<redacted>

{
"email":"a@gmail.com"    // My new email request
}

```


And I saw there were 4 parameters in the JSON response:

```
HTTP/2 302 Found
Location: /my-account
Content-Type: application/json; charset=utf-8
X-Frame-Options: SAMEORIGIN
Content-Length: 115

{
  "username": "wiener",
  "email": "a@gmail.com",
  "apikey": "<api-key>",
  "roleid": 1
}
```
<img width="1365" height="738" alt="image" src="https://github.com/user-attachments/assets/44b1a832-d0f9-41c1-b911-31df4f0a9a53" />

The server is returning the full information about the profile, including the `roleid` parameter. When I saw that response, I immediately thought of altering the request by introducing `"roleid": 2` along with the `email` parameter. The altered request looks like this:

```

POST /my-account/change-email HTTP/2
Host: 0a8700c003f221558069b21f00ba0041.web-security-academy.net
Cookie: session=<redacted>

{
"email":"ab@gmail.com",
"roleid": 2
}
```
<img width="1365" height="739" alt="image" src="https://github.com/user-attachments/assets/f25fb5c2-e225-46d3-a233-768b0392d29d" />


And in the response for this, the server changed my `roleid` to 2: 

```
HTTP/2 302 Found
Location: /my-account
Content-Type: application/json; charset=utf-8
X-Frame-Options: SAMEORIGIN
Content-Length: 115

{
  "username": "wiener",
  "email": "ab@gmail.com",
  "apikey": "<api-key>",
  "roleid": 2
}

```

This suggests that the server fails to restrict or validate whether the user submitting the request is actually authorised to change the `roleid` parameter.

---

## Entire Pipeline and Solution
1. I send a `my-account/change-email` request by introducing an extra parameter `"roleid": 2` in the request.
2. The server receives the request, and because the server doesn't implement an authorisation check on the `roleid` parameter, it blindly trusts the request and binds the provided parameters to the database called mass assignment (also called autobinding).
3. Then the response I get to see that the `roleid` parameter is changed to 2.
4. Then, if I now go to the `/admin` page, I can access it because, by the logic, I am now an authorised member and can delete the user Carlos.
5. I deleted the user Carlos and completed the lab.
<img width="1365" height="736" alt="image" src="https://github.com/user-attachments/assets/4d1a277a-9fc9-4ad5-a988-106e98c6f34f" />


---

## What Defender Should have (Mitigations)
1. Ensure new accounts or standard users are strictly set to the lowest privileges. There should be an allowlist of fields from the client side.
2. The role modification should be handled separately by the dedicated endpoint and can only be modified by authorised administrators.
3. Always check permissions for every sensitive field modification in the backend, and sensitive endpoints should be completely immutable for low-privileged users.

---
