# OverTheWire: Bandit

**Goal:** Get comfortable on the Linux command line through a free wargame.
**Status:** Draft

---

## Background
Bandit is a beginner wargame from OverTheWire. Each level is a Linux account on a remote server, and the goal is to find the password for the next level using only the command line. It is legal, intended for practice, and teaches the skills used in real security work.

## How it works
- Connect over SSH to the host and port given on the official Bandit page (I read the current details there).
- Each level's page describes the goal and lists commands that may help.
- The password I find opens the next level.

## Rules for my notes
- **Never publish passwords or level solutions.** The wargame asks for this, and spoilers ruin the learning for others.
- Write what **skills** I practised, and what I got stuck on.

## Skills the early levels practise
| Skill | Why it matters in real work |
|-------|-----------------------------|
| Logging in with SSH | Daily admin work |
| Reading files with unusual names | Real files are messy |
| Hidden files (`ls -a`) | Malware and configs often hide there |
| Identifying file types (`file`) | Never trust extensions |
| Finding files by size, owner, or name (`find`) | Incident response and audits |
| Searching inside files (`grep`) | Log analysis |
| Sorting and de-duplicating (`sort`, `uniq`) | Finding the odd one out in data |
| Encoding and decoding (base64) | Data is often encoded, not encrypted |
| Extracting text from binaries (`strings`) | Basic file analysis |
| Compression and archives | Unpacking suspicious samples safely |
| Permissions and SUID | Privilege escalation basics, from the defender's view |

## My progress
| Level range | Date finished | Commands I learned | What was hard |
|-------------|---------------|--------------------|---------------|
| 0 - 10 | | | |
| 10 - 20 | | | |

## What I learned
(Fill in after playing.)

## Defensive takeaway
The same commands used to explore a Bandit account are the ones I use to audit my own servers (`find`, `grep`, permissions, hidden files). Learning to read a Linux system fast makes me a better defender.
