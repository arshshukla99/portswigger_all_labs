# Information disclosure in version control history

---

## Lab Description
This lab contains a vulnerbaility of Sensitive Information Disclosure via version control history and we need to take leverage of this vulnerability and search for the password of administrator account and delete user carlos.

---

## My First Move
After reading the lab description I first don't understand what it meant by "version control" and I started searching for random pages like `/history`, `/version`, `/version-history`, etc beacuse I completely forgot that time that version control is handled by Git and GitHub.

---

## What Failed and Why
As I was trying to search for ther version control pages directly instead of searching for them inside the folders that actually contain version control history, and after 15 to 20 mins of brute-forcing random pages names I searched on google and get to know the version control is handled usually by `/.git` folder and at that I remebered how dumb I am brute-forcing random pages instead of the relevent once. 

---

## What Worked
And then, I visitied the `/.git` folder by appending it in the parent URL. and Then I got to see many folder in that like `<branches>`,`<hooks>`,`<refs>`,`<objects>`,`<logs>`, etc. In all of this I found the `<logs>` folder, in which I had a thought that i will find something useful. I then visited the `<logs>` and found 2 options there `HEAD` & `<refs>`, I which I opened the `HEAD` and see 2 commits are there like this:

<img width="1366" height="736" alt="image" src="https://github.com/user-attachments/assets/77e064d2-6473-4736-bfdd-1cbb76af7a29" />


In this the First Commit got my attention more that second commit, which has the commit message "Add skeleton admin panel" beacuse the second commit has "Remove admin password from config" which absolutely don't have the chances of admin password.

Then I got bach to the `/.git` page and opened the `<objects>` because the `<logs>` only contain logs of that commit but don't contain password and I had seen the `<objects>` at the start so, I know there is most chances that the password should be in this. So, I opened the `<objects>` and there I have got many folders but the 2 folders got my attention there, both of the have the starting 2-bytes of the commit hashes which I have seen in logs. Like in my case the one was `cf` and other was `8f` (NOTE: Every time you visit the lab the hash changes to your hashes will be Most Probably different).

<img width="1366" height="736" alt="image" src="https://github.com/user-attachments/assets/6b8a47dc-233c-4270-a230-aff2ea3314d6" />

And then I saw myself actually progressing in the lab after sitting for 30-minutes clueless of what to do next.

---

## Entire Pipeline & Solution
1. Go to `/.git` folder by appending `/.git` in the parent URL. You will se something like this:

<img width="1366" height="736" alt="image" src="https://github.com/user-attachments/assets/3b337fd4-ad63-41c8-ac5c-db38a9e75fcb" />

2. Go to `<logs>` file which is in the last the list then open `HEAD`. There you will see 2 commits by carlos. It should look like this:

<img width="1366" height="736" alt="image" src="https://github.com/user-attachments/assets/c27903d3-a556-498a-9f72-60fc868a7944" />


3. Remember or you can Write the First 2 bytes of the FIRST commit hash. In my case its `cf` in `cf9b90c957a2af2dfe358fe99816e15ff964073c` ( Your hash can be different).

<img width="1366" height="736" alt="Screenshot 2026-11-27 112940" src="https://github.com/user-attachments/assets/b1d9b9c3-08dd-4dc4-a1af-764b11de8fd4" />


4. Then come back to the `/.git` folder and open `<objects>` and and there you will see many folders but try to find the folder starting from the first 2-bytes of the hash that you have written above. Open that `cf` folder and there you will see the remaining 38-byte hash named file, click it and download it.

<img width="1366" height="736" alt="Screenshot 2026-11-27 112955" src="https://github.com/user-attachments/assets/132cd4fa-5142-4af6-bc1c-3ace2ac649cd" />

5. After Downloading use this python script to decode the file :

```
import zlib

with open(r"path/to/that/file", "rb") as f:
    data = zlib.decompress(f.read())
    print(data)

```


---

## What Defenders Should have



---
