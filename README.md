# Managing Multiple GitHub Accounts with SSH

This guide walks you through setting up multiple GitHub accounts on the same machine using SSH keys. Once done, you can clone any repo and push/pull directly without any extra steps.

> **Windows Users:** Install [Git for Windows](https://git-scm.com/) first. It comes with **Git Bash**, which gives you the same terminal experience as macOS/Linux. All commands in this guide should be run inside **Git Bash**, not PowerShell or CMD.

---

## 1. Generate SSH Keys

Create a separate SSH key for each GitHub account.

> You can use either **ed25519** (recommended, more secure and modern) or **RSA** (widely supported). Examples below use ed25519, but swap `-t ed25519` with `-t rsa -b 4096` if you prefer RSA.

```bash
ssh-keygen -t ed25519 -C "account1@email.com" -f ~/.ssh/account1-github
ssh-keygen -t ed25519 -C "account2@email.com" -f ~/.ssh/account2-github
```

Then print each public key and add it to its corresponding GitHub account:

```bash
cat ~/.ssh/account1-github.pub
cat ~/.ssh/account2-github.pub
```

Copy the output and go to **GitHub → Settings → SSH and GPG Keys → New SSH Key** for each account.

---

## 2. Configure SSH Config File

Open or create the SSH config file (no extension):

- **macOS / Linux / Windows (Git Bash):** `~/.ssh/config`

Add a block for each account with a unique host alias:

```
Host github-work
  HostName github.com
  User git
  IdentityFile ~/.ssh/account1-github
  AddKeysToAgent yes

Host github-personal
  HostName github.com
  User git
  IdentityFile ~/.ssh/account2-github
  AddKeysToAgent yes
```

> **macOS only:** Add `UseKeychain yes` to each block so keys persist across reboots via Keychain.

---

## 3. Add Keys to the SSH Agent

**macOS:**
```bash
ssh-add --apple-use-keychain ~/.ssh/account1-github
ssh-add --apple-use-keychain ~/.ssh/account2-github
```

> `--apple-use-keychain` saves the keys to macOS Keychain so they persist across reboots. Without it, you will need to run `ssh-add` again every time you restart your machine.

**Linux / Windows (Git Bash):**
```bash
ssh-add ~/.ssh/account1-github
ssh-add ~/.ssh/account2-github
```

Verify both keys are loaded:
```bash
ssh-add -l
```

Test each connection:
```bash
ssh -T git@github-work
ssh -T git@github-personal
```

You should see `Hi <username>!` for each.

---

## 4. Clone Using the Host Alias

When cloning a repository, you **must** replace `github.com` with the matching host alias from your config. This is required — skipping it will cause push/pull to fail.

```bash
# Work account
git clone git@github-work:org-name/repo-name.git

# Personal account
git clone git@github-personal:username/repo-name.git
```

That's it — the remote URL is set automatically. You can now **push and pull directly** without any extra configuration.

---

## Extra — Fix a Rejected Push

If you cloned a repo using `github.com` instead of the alias, or you get a `Permission denied` / `Repository not found` error, fix the remote URL:

```bash
# Work account
git remote set-url origin git@github-work:org-name/repo-name.git

# Personal account
git remote set-url origin git@github-personal:username/repo-name.git
```

After that, push and pull will work normally again.
