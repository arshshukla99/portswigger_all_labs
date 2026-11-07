# Unprotected Admin Functionality

## Lab Description
This lab has an unprotected admin panel, whose location is exposed somewhere on the website. Our goal is to discover that unprotected admin page's path, access that page, and delete user carlos.

---

## My First Move
When I started this lab, I immediately thought of directly accessing that admin page by typing `/admin` in the website's URL, but it gave me a `Not Found` Error. So, it suggested to me that the name of the admin path is NOT `/admin`; it is something else, so I tried many URL paths like `/administrator`, `/adminitration`, etc. But nothing seems to work. 

---

## What Failed and Why
So, the mistake I was making is that I am not trying to find the actual path of the page; I am just brute-forcing paths that are similar to the admin page and hoping that something will work.

---

## What Worked
But after a few minutes, I started trying to find the actual path hidden in the website, inspecting the source code of the website, HTML, JavaScript, and CSS files, but I found nothing. So, I randomly thought that Do this website have `robots.txt`? And when I typed `/robots.txt` in the URL, it turned out that it has a `robots.txt` with the text.

```
User-agent: *
Disallow: /administrator-panel
```
This is for Search Engine Crawlers.

Basically, search engine crawlers are automated bots whose task is to crawl each and every website on the internet and each and every webpage on the website; then they give the data to the search engine to index it and show it in people's search results.

But sometimes, the developer wants to hide a particular page from a website from the general public, so he mentions paths of those pages in the `robots.txt` file. Therefore, he gives this file the full instructions for crawlers on how to crawl this particular website. The crawlers also, before crawling the website, first visit the `/robots.txt` to get the instructions on how to crawl this website, what to avoid, and what to crawl.

And that's how I discovered the actual path of the admin page: `/administrator-panel`.

---

## Entire Pipeline and Solution
1. Check whether the website has `/robots.txt` page in it.
2. In `/robots.txt` there is some text like:

```
User-agent: *
Disallow: /administrator-panel
```

3. So, You Discover the actual path of the admin page `/administrator-panel`.
4. You visit that page and delete the user Carlos.

---

## What Defenders Should Do (Mitigations)

1. NEVER put highly confidential pages like `/administrator-panel` in `/robots.txt`. Since they are publicly accessible.
2. Put the highly confidential pages behind strong server-side authentication.
3. Put strict server-side authorization that verifies the user's session ID and privileges on the server side.
4. If the sensitive pages exist in a particular directory or subfolder, name that particular directory or subfolder in the `/robots.txt`.
5. Configure your application framework to `Deny Default` so all sensitive pages deny access unless an explicit permission rule allows it.

---
