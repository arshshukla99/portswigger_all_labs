# User role controlled by request parameter

## Lab Description
This lab has an `/admin` page that identifies users with a session cookie and a cookie parameter. Our goal for this lab is to access the `/admin` page and delete user  `carlos`.

---

## My First Move and what failed
When I first saw that lab, I have tried to access that page without logging in with the given credentials, I thought there should a cookie in the request headers which i can modify and access it, I was right about that and I was just trying to find that cookie by intercepting the request for `/admin` with Burp but since I was not logged in, I can't find that because I think the cookie parameter is not supplied to the server for guest session, and if the parameter is not supplied it is default set to `false` or something else that restricts the page for guest or standard user.

---

## What Worked
When I logged in with the credentials and then tried to access the `/admin` page, it still says `"Admin interface only available if logged in as an administrator"` but when I saw the request in Burp Suite in HTTP History under proxy, I saw a cookie parameter in the Cookie header that is `Admin=false`. Then I figured out that for a logged-in user, the server makes authorization decisions with parameters that are generated from the client-side. So, in the next try, I intercepted the request to the `/admin` page and changed the cookie parameter `Admin=true` then sent the request. Then I was able to access that page.

---

## Entire Pipeline and Solution
1. I logged in with `wiener:peter` credentials. I turned on interception in Burp Suite, then tried to access the`/admin` page.
2. In the interception, I saw the request with a cookie parameter and modified `Admin=false` to `Admin=true` and sent it.
3. The server saw the request and because the server rely on the client-controlled input for authorisation and doesn't do any checks from his side, it blindly trust the request and grants the permission to access the `/admin' page. Hence, the Vertical Privilege Escalation is accomplished.
4. From there, I clicked on the delete button beside Carlos to delete the user Carlos while the interception is still ON.
5. A new request was generated with the `Admin=false` cookie parameter.
6. I again modified the request, turning `Admin=false` to `Admin=true` again.
7. User Carlos was deleted.

---

## What Defender should have
1. When a request comes for sensitive pages like the`/admin` page, the server must use check the session ID to identify the role of the user from the database in the backend.
2. Use a cryptographically secure session token like JWT with a secret key known only to the server. This can help against client-supplied boolean flags like `Admin=true`.
3. Deny access by default to sensitive pages like `/admin`, requiring a role validation check to access that page. 

----
