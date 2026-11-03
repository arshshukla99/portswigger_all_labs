# URL-based access control can be circumvented

## Lab Description
In this lab, the front-end is configured to block external access to the /admin page, but the backend supports the `X-Original-URL` header. Our goal is to delete the user carlos by accessing the /admin page.

---

## My First Move
When I first came across the lab, I tried accessing admin-panel, and it said "Access Denied", so I intercepted the request and tried to analyse the request content, and I found Nothing. I also used `X-Original-URL: /` and put `/admin` in the request line path; it didn't work either. So, I came to the conclusion that there is some sort of access control that exists in the front end that resists access.

Then I sent `/` in the request line path and put the `X-Original-URL:/doesnotexist` to determine whether this header was originally supported or not. It turns out the website gives a `404 Not Found` error, which strongly suggests that the backend supports this header.

---

## What failed and why

I have been constantly changing the URL in the X-Original-URL header, putting random URLs in the header like `/product`, `/users?username=carlos`, etc., but nothing worked. After sometime I figured out it returns the admin page when I put / in the request line path and put `/admin` in the `X-Original-URL` header. But still, after that, I cannot perform the deletion operation to complete the lab. I am supplying / in the request line path and supplying `X-Original-URL:/admin/delete?username=carlos`, but it doesn't work.

I was supplying the full path in the X-Original-URL header, and this suggests that the backend server parses the X-Original-URL header strictly as a path, so `/admin/delete?username=carlos` will be parsed into `/admin/delete` because it requires a parameter that is not supplied with the request line path (only / was supplied), so the server thinks that URL parameter is missing. And that's the error I was facing.

---

## What Worked
After constantly failing, I tried separating the arguments and the URL and kept trying different methods by exchanging values, supplying no path but only a header, but nothing seemed to work. After like 30 minutes of constantly failing, I tried supplying  `/?username=carlos` as the request line and `X-Original-URL: /admin/delete` as a header, and It Worked.

### To Access Admin Page
```
GET / HTTP/2
Host: 0ab100fd039449e28045cb1d00120020.web-security-academy.net
Cookie: session=<session-cookie>
X-Original-URL: /admin

```

### To Delete User carlos
```
GET /?username=carlos HTTP/2
Host: 0ab100fd039449e28045cb1d00120020.web-security-academy.net
Cookie: session=<session-cookie>
X-Original-URL: /admin/delete

```

---

## The entire Pipeline and Solution
1. You alter the request to the `/` page by putting an extra `X-Original-URL:/admin` header.
2.  The front-end server receives the request with a `/` request-line path, which is publicly accessible, and lets the request pass through. This suggests that the front-end block looks only at the request-line path.
3. Now, the backend receives the request, and because it is configured to support the X-Original-URL header, it uses that header to override the original `/` path URL to the `/admin` URL.
4. You will find the full URL in the admin's HTML page: `/admin/delete?username=carlos`.So, you get to know that there is a `/delete` path inside `/admin`. Now, you put `/?username=carlos` as a request line and put the `X-Original-URL:/admin/delete` header in another request.
5. The frontend again sees the `/` path and forwards it to the backend, and the backend receives the supplied arguments and path, because the backend is configured to override the `/` path to `/admin/delete` using the header.
6. It constructs `/admin/delete?username=carlos` as the full path; because the server doesn't have any access control for its own, it trusts that anything that the front end passes is authorized, then it executes the path.
7. It deletes user Carlos.

---

## What Defender Should Have
1. Enforce a check for authentication and authorization for sensitive pathways and operations in the backend and front-end both.
2. Use a reverse proxy to strip out headers like X-Original-URL and X-Rewrite-URL or configure you backend to ignore headers like this.
3. If it isn't necessary, you can remove those header configurations from the server side completely.
4. Ensure that both the front-end reverse proxy and the back-end application interpret the request URL in exactly the same way.

---
 
