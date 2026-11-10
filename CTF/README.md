# CTF: Account Discovery (MITRE ATT&CK T1087.001)

**Category:** Linux Enumeration
**ATT&CK:** T1087.001 – Account Discovery: Local Account (Discovery tactic)
**Flag format:** `CYBER266{A_B_C}`

## The scenario

You've stolen the password for a low-level account, `student`, on Acme Corp's database server, `acme-db02`. Before you can move further, you need to know who's who on this machine.

Three accounts matter:

| Piece | Find the account that... |
|-------|--------------------------|
| **A** | can actually run commands as an administrator through `sudo` |
| **B** | is a service account (used by a program, not a person) that can still log in interactively |
| **C** | is secretly just as powerful as `root`, but isn't named `root` |

Every account's comment field contains a `tag=` word. Most are decoys. Put the tags of accounts **A**, **B** and **C** together, in that order, separated by underscores:

```
CYBER266{tagA_tagB_tagC}
```

## Rules

- Everything you need can be read by an ordinary user. You don't need to break anything, guess passwords, or become root.
- Don't trust what people *say* about themselves; trust what the system says.
- The tutorial's commands are a starting point, not the answer. This server wasn't set up as neatly as the tutorial's, so think about what each command actually checks and what it might miss.

## Start the lab

Open a terminal in this `CTF` folder, then run:

```
docker compose up -d --build
docker exec -it t1087-ctf bash
```

Your prompt should change to `student@acme-db02:~$`.

## Check your answer

The lab includes a checker. Keep the single quotes around the flag:

```
checkflag 'CYBER266{tagA_tagB_tagC}'
```

## Hints

<details>
<summary>Hint 1: "Usually" isn't "always"</summary>

Service accounts *usually* sit below user ID 1000, but only if whoever created them did it properly. If the ID can't be trusted, what else in an account's `/etc/passwd` line gives away that it belongs to a program and not a person?
</details>

<details>
<summary>Hint 2: A group's member list isn't the whole story</summary>

If the admin group looks empty, that doesn't mean nobody's in it. An account can belong to a group in two different ways, and `/etc/group` only records one of them. Look at the fourth field of `/etc/passwd`.
</details>

<details>
<summary>Hint 3: Root is a number, not a name</summary>

Linux decides who's root by the user ID, not the username. What would it mean if two accounts had the same ID?
</details>

## Clean up

```
docker compose down --rmi local
```
