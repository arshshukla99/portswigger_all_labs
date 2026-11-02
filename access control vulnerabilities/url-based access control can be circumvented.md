# URL-based access control can be circumvented

## Lab Description
This lab is configured to block access to the /admin page externally in the front-end, but the backend is configured that way that it supports the `X-Original-URL` header. Our goal is to delete the user carlos by accessing the /admin page.

---

## My First Move
When I first came across the lab, I tried accessing admin-panel, and it says "Access Denied", so I intercepted the request and tried to analyse the request content, and I found Nothing. I also used `X-Original-URL: /` and put the /admin in the real search query; it doesn't work either. So, I came to the conclusion that there is some sort of access control that exists in the front end that resists access.

---

## What failed and why

I have been constantly changing the URL in the X-Original-URL header, putting random URLs in the header like `/product`, `/users?username=carlos`, etc., but nothing worked. After sometime I figured out it returns the admin page when I put / in the request line path and put `/admin` in the `X-Original-URL` header. But still, after that, I cannot perform the deletion operation to complete the lab. I am supplying / in the request line path and supplying `X-Original-URL:/admin/delete?username=carlos`, but it doesn't work.

I was supplying the full path in the X-Original-URL header, and this suggests that the backend server parses the X-Original-URL header strictly as a path, so `/admin/delete?username=carlos` will be parsed into `/admin/delete` because it requires a parameter that is not supplied with the request line path (only / was supplied), so server thinks that URL parameter is missing. And that's the error I was facing.

---

## What Worked
After constantly failing, I tried separating the arguments and the URL and kept trying different methods by exchanging values, supplying no path but only a header, but nothing seemed to work. After like 30 minutes of constantly failing, I tried supplying  `/?username=carlos` as the request line and `X-Original-URL: /admin/delete` as a header, and It Worked.

---

## The Pipeline
1. You alter the request to the `/` page by putting an extra `X-Original-URL:/admin header`.
2.  Front-end server receives the request and because Front-end don't process any header it just sees the `/` path which is publicly accessible and let the request pass through.
3. Now, the backend receives the request, and because it is configured to support the X-Original-URL header, it uses that header to override the original `/` path URL to the `/admin` URL.
4. You found the full URL in the response in Burp Suite `/admin/delete?username=carlos`.So, you get to know that to delete there is a `/delete` path inside `/admin`. Now, you put /?username=carlos as a request line and put the `X-Original-URL:/admin/delete` header in another request.
5. The frontend again sees the `/` path and forwards it to the backend, and the backend receives the supplied arguments and path, because the backend is configured to override the `/` path to `/admin/delete` using the header.
6. It constructs `/admin/delete?username=carlos` as the full path; because the server doesn't have any access control for its own, it trusts that anything that the front end passes is authorized, then it executes the path.
7. It deletes user Carlos.

---

## What Defender Should Have
1. Always check for authentication and authorization for sensitive pathways and operations; never let the front end only handle the access control.
2. Use a reverse proxy to strip out headers like X-Original-URL and X-Rewrite-URL.
3. If it isn't necessary, you can remove those header configurations from the server side completely.

---
