# Information disclosure in version control history

## Lab Description
This lab contains a vulnerability of Sensitive Information Disclosure via version control history and we need to take leverage of this vulnerability and search for the password of the administrator account and delete the user carlos.

---

## My First Move
After reading the lab description, I first didn't understand what it meant by "version control", and I started searching for random pages like `/history`, `/version`, `/version-history`, etc., because I completely forgot that version control is handled by Git and GitHub.

---

## What Failed and Why
As I was trying to search for the version control pages directly instead of searching for them inside the folders that actually contain version control history, and after 15 to 20 mins of brute-forcing random page names, I searched on Google and got to know that version control is usually handled by the `/.git` folder and at that I remembered how dumb I was for brute-forcing random pages instead of the relevant ones. 

---

## What Worked
And then, I visited the `/.git` folder by appending it to the parent URL, and then I got to see many folders in it, like `<branches>`, `<hooks>`,`<refs>`, `<objects>`, `<logs>`, etc. In all of this I found the `<logs>` folder, in which I had a thought that i will find something useful. I then visited the `<logs>` and found 2 options there: `HEAD` & `<refs>`, in which I opened the `HEAD` and saw 2 commits there like this:

<img width="1366" height="736" alt="image" src="https://github.com/user-attachments/assets/77e064d2-6473-4736-bfdd-1cbb76af7a29" />


In this, the First Commit got my attention more than the second commit, which has the commit message "Add skeleton admin panel", because the second commit has "Remove admin password from config", which absolutely doesn't have the chance of an admin password.

Then I went back to the `/.git` page and opened the `<objects>` because the `<logs>` only contain logs of that commit but don't contain the  password, and I had seen the `<objects>` at the start so, I know there is most chances that the password should be in this. So, I opened the `<objects>`, and there I got many folders, but the 2 folders got my attention; both of them have the starting 2 bytes of the commit hashes which I saw in the logs. Like in my case, one was `cf`, and the other was `8f` (NOTE: Every time you visit the lab, the hash changes, so your hashes will most probably be different).

<img width="1366" height="736" alt="image" src="https://github.com/user-attachments/assets/6b8a47dc-233c-4270-a230-aff2ea3314d6" />

And then I saw myself actually progressing in the lab after sitting for 30 minutes clueless about what to do next.

---

## Entire Pipeline & Solution
1. Go to `the /.git` folder by appending `/.git` to the parent URL. You will see something like this:

<img width="1366" height="736" alt="image" src="https://github.com/user-attachments/assets/3b337fd4-ad63-41c8-ac5c-db38a9e75fcb" />


<img width="1366" height="736" alt="image" src="https://github.com/user-attachments/assets/c27903d3-a556-498a-9f72-60fc868a7944" />


3. Remember, or you can write the First 2 bytes of the FIRST commit hash. In my case, it's `cf` in `cf9b90c957a2af2dfe358fe99816e15ff964073c` ( Your hash can be different).

<img width="1366" height="736" alt="Screenshot 2026-11-27 112940" src="https://github.com/user-attachments/assets/b1d9b9c3-08dd-4dc4-a1af-764b11de8fd4" />


4. Then come back to the `/.git` folder and open `<objects>`, and there you will see many folders, but try to find the folder starting with the first 2 bytes of the hash that you have written above. Open that `cf` folder, and there you will see the remaining 38-byte hash-named file; click it and download it.

<img width="1366" height="736" alt="Screenshot 2026-11-27 112955" src="https://github.com/user-attachments/assets/132cd4fa-5142-4af6-bc1c-3ace2ac649cd" />

5. After downloading, use this Python script to decode the file:

```
import zlib

with open(r"path/to/that/downloaded/file", "rb") as f:
    data = zlib.decompress(f.read())
    print(data)

```
6. After running the scirpt you have the output similar to this : `b'commit 209\x00tree d09bccb37c10e7ab4d3b5405d4756a2f97fbf6c3\nauthor Carlos Montoya <carlos@carlos-montoya.net> 1592843022 +0000\ncommitter Carlos Montoya <carlos@carlos-montoya.net> 1795758959 +0000\n\nAdd skeleton admin panel\n'`.
<img width="1366" height="736" alt="Screenshot 2026-11-27 113247" src="https://github.com/user-attachments/assets/b3ee57d9-a627-4ae7-a552-165a24b23817" />

7. In this output, extract the hash `d09bccb37c10e7ab4d3b5405d4756a2f97fbf6c3` and paste that hash into the URL to download the tree object. After pasting, put the `/` after the first 2 bytes of the hash, and your URL should look something like: `<lab-url>/.git/objects/d0/9bccb37c10e7ab4d3b5405d4756a2f97fbf6c3` and press Enter. The zlib file will be downloaded.
8. After that, you will copy the recently downloaded file name, replace this name with the previous file name you typed in the script and run the script. Your output should be something like: `b'tree 81\x00100644 admin.conf\x00\xc7\xc0b\xaa\xbdn\x046\xd7\x86\x91\x9d\xa47\x056\x844\x8e\x9a100644 admin_panel.php\x00\x89D\xe3\xb9\x856\x91C\x1d\xc5\x8d_Ix\xd3\x94\x0c\xeaJ\xf2'`.
<img width="1366" height="736" alt="Screenshot 2026-11-27 113305" src="https://github.com/user-attachments/assets/e6e7bfaa-c1a5-4d86-b0b1-d984d91a39f7" />

9. Here you can see that `admin.conf` and `admin_panel.php` are there, and a 20-byte Binary hash after their name. Copy the 20-byte hash after `admin.conf\x00` and before the number `100644` (its mode name is not part of the hash) and your copied hash should be like: `\xc7\xc0b\xaa\xbdn\x046\xd7\x86\x91\x9d\xa47\x056\x844\x8e\x9a`.
10. Now, to convert it into a 40-byte SHA-1 hash, simply use any AI chatbot or simply ask Google to decode it like this:

<img width="1366" height="736" alt="Screenshot 2026-11-27 113410" src="https://github.com/user-attachments/assets/4cc585f7-6d67-4472-b1a5-aeec91506131" />

11. Now, after decoding, it should look something like this: `c062aabd6e0436d786919da437053684348e9a`
12. Now paste that hash into your Python script again and run the script again. You will get the output with the admin password `b'blob 36\x00ADMIN_PASSWORD=rdhwq09xy7frlloi5eh7\n'`; copy the password `rdhwq09xy7frlloi5eh7` ( It can be different in your case).
<img width="1366" height="736" alt="Screenshot 2026-11-27 113451" src="https://github.com/user-attachments/assets/c7b783fc-0edf-45ba-b481-d13ec1fed822" />

13. Log in to the administrator account using this password and delete the user carlos.

<img width="1366" height="736" alt="Screenshot 2026-11-27 113521" src="https://github.com/user-attachments/assets/e997a285-e760-4a25-aa9b-0211a352c6f3" />

<img width="1366" height="736" alt="Screenshot 2026-11-27 113534" src="https://github.com/user-attachments/assets/fe8df519-f3ef-43d6-8799-34dd5e908ffc" />

---

## Explanation: How does /.git work and what we just exploited
So, in this lab, we have exploited the vulnerability of Information Disclosure, where the information is disclosed by version control history or the `/.git` file, which is publicly available. So, in the scenario of this lab, where the developer commits the admin panel but forgot to remove the admin password, he later realises his mistake and does another commit where he removes the password. But the problem here is that Git never deletes its old commits and files and points every change with a fresh new commit and also saves new hashes in the new file.

Therefore, for an In-depth understanding, we need to understand what happened when the file was created to understand what we just did:

### Step 1: Carlos Stages the Files
Carlos first ran the `git add admin.conf admin_panel.php` command to create two files in his project folder. Git immediately creates two separate blob objects for those two files:
1. #### Prepends Header:
• Creates the first blob of `admin.conf`, which has the admin password in it. It is like: `b'blob 36\x00ADMIN_PASSWORD=rdhwq09xy7frlloi5eh7\n'`.
• Creates the second blob of `admin_panel.php`, which has admin panel PHP code in it. It is like: `blob <M>\x00 + <PHP code>`.

2. #### Run SHA-1:
• Creates the 20-byte Binary Hash for both blobs which kind of looks like : `\xc7\xc0b\xaa\xbdn\x046\xd7\x86\x91\x9d\xa47\x056\x844\x8e\x9` & `\x89D\xe3\xb9\x856\x91C\x1d\xc5\x8d_Ix\xd3\x94\x0c\xeaJ\xf2`.

3. #### Zlib & Save:
• Git compresses both hashes into a separate zlib file for each, named after the 40-byte hash, which you will get after decoding these 20-byte binary hashes. Like the decoded hash for the first 20-byte hash for `admin.conf` content is: `c7c062aabd6e0436d786919da437053684348e` and saves it as `/.git/objetcs/c7/c062aabd6e0436d786919da437053684348e` and similarly for 2nd 20-byte hash, the decoded hash is : `8944e3b9853691431dc58d5f4978d3940cea4af2` and saves it as `/.git/objetcs/89/44e3b9853691431dc58d5f4978d3940cea4af2`.

### Step 2: Git Builds the Tree Object
When Carlos runs `git commit -m "Add skeleton admin panel"`. Git first need to record the folder structure. It reads the staged files and and combines them together into a single raw byte stream (in this case, 81 bytes).

• For the first file: `10064 admin.conf\x00`(18 bytes) + `\xc7\xc0b\xaa\xbdn\x046\xd7\x86\x91\x9d\xa47\x056\x844\x8e\x9`(20 bytes).
• For the second file: `10064  admin_panel.php\x00`(23 bytes) + `\x89D\xe3\xb9\x856\x91C\x1d\xc5\x8d_Ix\xd3\x94\x0c\xeaJ\xf2`(20 bytes).
• And combines both files with Total raw data: 38 + 43 = 81 bytes.

Now Git packages this Tree Object:

#### Prepends Header:
Attaches 8 byte header into this raw data: `b'tree 81\x00100644 admin.conf\x00\xc7\xc0b\xaa\xbdn\x046\xd7\x86\x91\x9d\xa47\x056\x844\x8e\x9a100644 admin_panel.php\x00\x89D\xe3\xb9\x856\x91C\x1d\xc5\x8d_Ix\xd3\x94\x0c\xeaJ\xf2'`.
#### Runs SHA-1:
Git hashes those 89 bytes and converts them into a SHA-1 Hash. Which was, in this case: `d09bccb37c10e7ab4d3b5405d4756a2f97fbf6c3`.

#### Zlib & Save:
Git now compress those 89 bytes into a zlib file and saves it into the folder named by the first 2 bytes of the generated SHA-1 hash, then the file name is the remaining 38 characters of that SHA-1 hash. The path is similar to: `/.git/objects/d0/9bccb37c10e7ab4d3b5405d4756a2f97fbf6c3`.

### Step 3: Git Builds the Commit Object & Updates Logs

Immediately after saving the Tree, Git grabs that SHA-1 hash generated for the Tree object above and includes it in the metadata and the Commit message.

#### Prepends Header:
Git adds the commit header and rest information for the commit message and generate something similar to this: `b'commit 209\x00tree d09bccb37c10e7ab4d3b5405d4756a2f97fbf6c3\nauthor Carlos Montoya <carlos@carlos-montoya.net> 1592843022 +0000\ncommitter Carlos Montoya <carlos@carlos-montoya.net> 1795758959 +0000\n\nAdd skeleton admin panel\n'`.

#### Runs SHA-1:
It then runs the generated commit message through SHA-1 and generates its SHA-1 hash, like this: `cf9b90c957a2af2dfe358fe99816e15ff964073c`.

#### Zlib & Save:
It then decompresses the commit message and saves it as: `/.git/objects/cf/9b90c957a2af2dfe358fe99816e15ff964073c`.

#### Update Refs & Logs:
Git writes the above hash generated in the `/.git/refs/heads/master/` and also creates a log into `/.git/logs/HEAD` ( and also in `/.git/logs/refs/heads/master` with the commit message: `Add skeleton admin panel`.

---

## What Defenders Should Have
1. To protect against this kind of vulnerability, the first precaution a developer has to take is never to push `/.git` into Production.
2. If the `/.git` has to be pushed into production, then default deny any request to `/.git` or any other dot file like `/.env`, `/.htaccess`, etc.
3. If the secrets are accidentally committed, rotate your credentials immediately and rewrite the commit history using the `git filter-repo --path <filename> --invert-paths` command.
4. Always keep secrets in the `/.env` file listed in the `/.gitignore`.


---
