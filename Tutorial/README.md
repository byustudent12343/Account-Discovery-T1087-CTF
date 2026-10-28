# Account Discovery (MITRE ATT&CK T1087): Tutorial

## Table of Contents

1. [What Is Account Discovery?](#what-is-account-discovery)
2. [Setup](#setup)
3. [Tutorial](#tutorial)
   - [Step 1: Figure out who you are first](#step-1-figure-out-who-you-are-first)
   - [Step 2: Read the master list of accounts](#step-2-read-the-master-list-of-accounts)
   - [Step 3: Ignore useless information](#step-3-ignore-useless-information)
   - [Step 4: Don't miss the hidden service account](#step-4-dont-miss-the-hidden-service-account)
   - [Step 5: Find out who the admins are](#step-5-find-out-who-the-admins-are)
   - [Step 6: A few more quick looks](#step-6-a-few-more-quick-looks)
   - [Step 7: What you can't see](#step-7-what-you-cant-see)
4. [Resources](#resources)

---

## What Is Account Discovery?

Think of an attacker who has just gotten into a building. Before doing anything else, they check the employee directory on the wall: *Who works here? Who's the manager? Who has keys to the server room?* **Account Discovery** is the computer version of reading that directory.

After an attacker lands on a machine (say, by stealing one employee's password), one of the first things they do is ask the computer, *"What other accounts exist here?"* The answer tells them:

- **Who to target next.** An administrator account is far more valuable than a regular user's.
- **Which usernames to guess passwords for.** You can't guess a password if you don't know the username.
- **Where the forgotten accounts are.** Old or automated accounts often have weak passwords and nobody watching them.

MITRE ATT&CK is a free, public encyclopedia of attacker behavior used by security teams worldwide. It files this behavior as technique **[T1087 – Account Discovery](https://attack.mitre.org/techniques/T1087/)**, under the **Discovery** tactic (the "looking around" stage of an attack). It has four sub-techniques, but we will only be focusing on one: T1087.001 Local Account 

Two important things about this technique:

- **It needs no exploit.** Every command in this tutorial is a normal, built-in command that any user is allowed to run. The attacker isn't breaking anything — they're just *reading* information the system hands out freely. That is exactly why it's sneaky and hard to detect.
- **It happens *after* the break-in.** Account discovery assumes the attacker is already on the machine. It's a step that comes after getting in, not the way in.

---

## Setup

In this tutorial you'll explore a pretend Linux server belonging to a fake company, **Acme Corp**. Instead of building a whole server, we'll use **Docker**.

**What is Docker?** Docker lets you run a small, self-contained "computer inside your computer" (called a **container**). The container is built from a recipe file called a `Dockerfile`. Because everyone builds from the same recipe, everyone in class gets an identical lab, and the output on your screen will match the examples below. When you're done, you delete it and nothing on your real computer is changed. ([Docker's beginner overview](https://docs.docker.com/get-started/docker-overview/))

### What's in this folder

| File | What it does |
|---|---|
| `readme.md` | This tutorial |
| `Dockerfile` | The recipe for the Acme Corp server. It creates the user accounts you'll discover |
| `docker-compose.yml` | A short settings file so you can start the lab with one command |

Since you're in this class, I'll assume you've already got docker installed and running. So, here's what you do next:

### Step A: Download the tutorial files

**Option 1 (easiest):** On this repository's GitHub page, click the green **Code** button → **Download ZIP**. Unzip it, then open the folder `code/Account-Discovery-T1087-CTF/Tutorial`.

**Option 2 (if you have Git):**

```bash
git clone <REPO-URL>
```

### Step B: Move your terminal into the Tutorial folder

Your terminal is always "standing" inside one folder. Use `cd` (change directory) to move it into the Tutorial folder. Type `cd ` (with a space after it), then **drag the Tutorial folder from your file explorer into the terminal window** — the full path fills in automatically. Press Enter.

It will look something like this (your path will differ):

```bash
cd C:\Users\yourname\Downloads\repo-main\code\account-discovery-T1087\Tutorial
```
Alternatively, if you are running Windows, you can navigate to the /Tutorial folder and click on the searchbar at the top. Then, when it highlights the filepath, delete everything and type in "cmd". This will open up a terminal window already inside the folder.

### Step C: Build and start the lab

```bash
docker compose up -d --build
```

What this means:
- `docker compose up` starts the lab described in `docker-compose.yml`.
- `--build` builds it from the `Dockerfile` first.
- `-d` runs it in the background so you get your terminal back.

The first time takes a minute or two while Docker downloads Ubuntu Linux. When it finishes you'll see a line ending in **Started**.

> **Error "no configuration file provided"?** Your terminal isn't in the Tutorial folder. Redo Step B.
>
> **`docker compose` not recognized?** On older installs, try `docker-compose up -d --build` (with a dash).

### Step D: Log in to the lab

```bash
docker exec -it t1087-tutorial bash
```

This opens a command line **inside** the Acme Corp server (`exec` = run a command in the container, `-it` = make it interactive, `bash` = the Linux command line program). Your prompt changes to:

```
student@acme-web01:~$
```

That means you are logged in as the user **`student`** on a machine named **`acme-web01`**. 🎉 Setup is done. Leave this window open for the tutorial.

> **How to tell where you are:** If your prompt starts with `student@acme-web01`, you're inside the lab. If it shows your normal computer prompt (like `PS C:\Users\...>`), you're back on your real computer.

---

## Tutorial

### The scenario

You are playing the attacker. You've just stolen the password for a low-level Acme Corp employee account called `student`, and you've logged in to their web server, `acme-web01`. You have a foothold — but `student` is a nobody. Your goal now is to **discover what other accounts exist** so you can find a better one to take over.

> Type each command exactly as shown and press **Enter**. The lines starting with `student@acme-web01:~$` are the prompt — you don't type that part, just the command after it. Your usernames may appear in a different order, but everything will otherwise match.

### Step 1: Figure out who you are first

Before mapping everyone else, an attacker checks their own account. `whoami` prints your username. `id` shows your user ID and the groups you belong to.

```
student@acme-web01:~$ whoami
student

student@acme-web01:~$ id
uid=1003(student) gid=1005(student) groups=1005(student)
```

**What's going on:** You're `student`, and you belong to no special groups — no `sudo`, no `admin`. In plain terms, you're a nobody on this machine. That matters, because your whole mission is to find a *more powerful* account to take over.

### Step 2: Read the master list of accounts

On Linux, every account is listed in a plain text file called `/etc/passwd`, and any user is allowed to read it. Use the `cat` command to print the contents of a file to the screen. Example output:

```
student@acme-web01:~$ cat /etc/passwd
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
...
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
alice:x:1000:1002:Alice Nguyen,,,Developer:/home/alice:/bin/bash
bob:x:1001:1003:Bob Martinez,,,Accountant:/home/bob:/bin/bash
carol:x:1002:1004:Carol Smith,,,IT Administrator:/home/carol:/bin/bash
svc_backup:x:999:999:Nightly backup job:/var/lib/backup:/bin/bash
student:x:1003:1005:Student (you!):/home/student:/bin/bash
```

Each line is one account, with fields separated by colons (`:`). Reading left to right:

```
alice : x : 1000 : 1002 : Alice Nguyen,,,Developer : /home/alice : /bin/bash
  |     |    |      |              |                      |            |
username |  user   group        full name / comment   home folder   login
     password  ID    ID                                             shell
  (x = stored elsewhere)
```

**What's going on:** You just got the entire employee roster in one command — usernames, real names, job titles (people often type these into the comment field), and home folders. You can already see that `carol` is an "IT Administrator," which makes her an obvious high-value target.

### Step 3: Ignore useless information

Most of those lines are **system accounts** (`daemon`, `www-data`, `nobody`, and so on) — they run background programs, not people. Attackers usually want the *human* accounts. Two common tricks:

**Just the usernames**, using `cut` to grab the first field (`-d:` means "split on colons," `-f1` means "field 1"):

```
student@acme-web01:~$ cut -d: -f1 /etc/passwd
root
daemon
...
alice
bob
carol
svc_backup
student
```

**Filter to the likely human accounts.** By convention, regular user accounts have an ID of 1000 or higher. `awk` is a tool for picking out lines and fields by rule:

```
student@acme-web01:~$ awk -F: '$3 >= 1000 && $3 < 65534 {print $1}' /etc/passwd
alice
bob
carol
student
```

That reads as: split each line on `:` (`-F:`), and if the third field (the user ID) is between 1000 and 65533, print the first field (the username).

**What's going on:** You've narrowed dozens of lines down to a short list of real people — a ready-made target list.

### Step 4: Don't miss the hidden service account

Look carefully: the filter in Step 3 **missed `svc_backup`**. Its ID is 999, just under the 1000 cutoff. That's not a bug — it's intentional. **Service accounts** (used by programs, not people) often sit just below 1000, so a lazy filter skips right over them. Attackers prize these accounts because they frequently have weak passwords and nobody monitoring them.

A smarter approach is to list accounts that have a real **login shell** (a value ending in `sh`, like `/bin/bash`). Accounts meant only for programs are usually set to `nologin` instead:

```
student@acme-web01:~$ awk -F: '$7 ~ /sh$/ {print $1, $7}' /etc/passwd
root /bin/bash
alice /bin/bash
bob /bin/bash
carol /bin/bash
svc_backup /bin/bash
student /bin/bash
```

**What's going on:** `svc_backup` *can* log in interactively — which is unusual for a backup job and exactly the kind of overlooked account an attacker would pounce on.

### Step 5: Find out who the admins are

Knowing the accounts isn't enough; you want the ones with **administrator** access. On Linux, administrative power usually comes from membership in the `sudo` group. The `getent group sudo` command shows who's in it:

```
student@acme-web01:~$ getent group sudo
sudo:x:27:carol
```

You can also inspect a specific user. `id carol` lists all of carol's groups:

```
student@acme-web01:~$ id carol
uid=1002(carol) gid=1004(carol) groups=1004(carol),27(sudo)
```

**What's going on:** `carol` is an administrator. If you can steal carol's password or hijack her session, you effectively own the machine. Your enumeration has paid off: out of five accounts, you now know exactly which one to go after.

### Step 6: A few more quick looks

```
student@acme-web01:~$ ls -l /home          # who has a home folder
drwxr-x--- 2 alice   alice   ... alice
drwxr-x--- 2 bob     bob     ... bob
drwxr-x--- 2 carol   carol   ... carol
drwxr-x--- 2 student student ... student

student@acme-web01:~$ who                   # who is logged in right now
                                            # (empty = nobody else online)

student@acme-web01:~$ getent passwd carol   # look up one account directly
carol:x:1002:1004:Carol Smith,,,IT Administrator:/home/carol:/bin/bash
```

**What's going on:** `ls /home` confirms the human accounts (each person gets a folder there). `who` shows who's currently online — useful for an attacker who wants to avoid tipping off an active user, or who wants to piggyback on an admin's live session.

### Step 7: What you can't see

Account discovery only reveals what the system hands out to any user. It does **not** hand over passwords:

```
student@acme-web01:~$ cat /etc/shadow
cat: /etc/shadow: Permission denied
```

The actual scrambled passwords live in `/etc/shadow`, which only the `root` superuser can read.

**What's going on:** Discovery gives the attacker a *target list*, not the keys. Actually stealing carol's password is a separate, later step (a different ATT&CK tactic called **Credential Access**). This is the key takeaway: account discovery is reconnaissance, not the break-in itself.

---

## Resources

- [MITRE ATT&CK: T1087 Account Discovery](https://attack.mitre.org/techniques/T1087/) — the parent technique
- [MITRE ATT&CK: T1087.001 Local Account](https://attack.mitre.org/techniques/T1087/001/) — commands, real-world examples, mitigations, detection
- [MITRE ATT&CK: Discovery tactic (TA0007)](https://attack.mitre.org/tactics/TA0007/) — where this step fits in the bigger picture of an attack
- [Linux user account management (Red Hat)](https://www.redhat.com/sysadmin/linux-user-account-management) — the `/etc/passwd` format explained in plain terms
- [Microsoft `net user` documentation](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/net-user) — the Windows equivalent
- [Docker: Get started](https://docs.docker.com/get-started/) — if you want to understand the lab tooling better
