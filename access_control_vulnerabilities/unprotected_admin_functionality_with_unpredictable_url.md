# Unprotected Admin Functionality

## Lab Description
In this lab there is an unprotected admin panel located somewhere in the website with an unpredictable URL. But that unpredicatable location is exposed somewhere in the website. Our goal is to find the admin panel, access it and delete user carlos.

---

## My First Move

When I first see the lab, I have a thought to check the `/robots.txt` file and have look at the paths in there, since the challenge says "location is exposed somewhere in a website". But there was no `/robots.txt` in that website. So, my second thought is to check the source code of the website and look if the developer had made any mistake of Security through Obscurity (hiding something and thinking it is protected".

---

## What Worked
When I looked at the source code and search the HTML of that website in just few scrolls I found a block of javascript code which is written to hide the `Admin Panel` button in the `my-account` page when logged in as admin. It is default set to `false`, so it will never show. The Javascript there looks like this:
<img width="1366" height="736" alt="image" src="https://github.com/user-attachments/assets/03e0fbe3-f0cd-4fee-b0b8-984310ab04e8" />

In this case, The Developer of this website thought:
1. The developer exposed that code on the client side thinking now one will see the source. 
2. The developer hide this admin page from the user, they can never access it (Security through Obscurity).
3. The developer also didn't protect the Admin-panel here with any type of authentication and authorization rules.

---

## Entire Pipeline and Solution
1. Open the Lab and right click on the home page of the lab and click `View Page Source`.
2. Scroll down a bit and you will see block of code which is written to hide that admin page with unpredictable URL.
3. Copy the admin-path written in the code and access it by appending the copied URL in the homepage's URL.
4. Now delete user Carlos.

---

## What Defenders Should Do (Mitigations)
1. Verify permissions with every request. the websiet must verify the user's session and privileges before giving access.
2. Implement role-based access control in the website to avoid unauthorized access.
3. Configure your application to implement authorization and authentication check on all endpoints.

---
