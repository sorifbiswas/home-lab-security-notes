# Git Safety: Do Not Leak Secrets

## Before every commit
```bash
python secret_scanner.py                 # scan the working folder
python secret_scanner.py --staged        # scan only what I am about to commit
git status                               # look at what is staged
git diff --cached                        # read the actual changes
```

## What should never be in a repo
- Passwords, API keys, tokens, private keys (`id_ed25519`, `*.pem`)
- `.env` files and cloud credentials
- Unblurred screenshots with real IPs or usernames
- Packet captures with real traffic
- Flags and answers from CTF platforms
- VM images and ISOs (huge and pointless in Git)

My `.gitignore` already covers most of these.

## If I committed a secret by mistake
1. **Revoke or rotate the secret first.** Assume it is already exposed. This is the step that matters.
2. Remove it from the current files and commit that fix.
3. Deleting it is **not enough**: it still exists in the history. Check with:
   ```bash
   python secret_scanner.py --git-history
   ```
4. To clean history, use a tool such as `git filter-repo` or BFG Repo-Cleaner (read their documentation carefully), then force-push. Everyone who cloned the repo must also be told.

## Stop a file that is already tracked
```bash
git rm --cached secrets.txt     # untrack but keep the file on disk
echo "secrets.txt" >> .gitignore
```

## Good habits
- Store secrets in environment variables or a password manager, never in code.
- Use a separate, minimal-permission key for each service.
- Turn on two-factor authentication for GitHub.
- Use SSH keys or tokens with the smallest scope that works.
- Review pull requests and commits before they go public.
