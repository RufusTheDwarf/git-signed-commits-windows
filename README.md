<div align="center">

# 🔐 Git SSH Signing on Windows

**Sign your Git commits with SSH and get the "Verified" badge on GitHub.**

[![Platform](https://img.shields.io/badge/Platform-Windows-0078D6?style=flat&logo=windows&logoColor=white)](https://www.microsoft.com/windows)
[![Git](https://img.shields.io/badge/Git-%E2%89%A5%202.34-F05032?style=flat&logo=git&logoColor=white)](https://git-scm.com/)
[![License](https://img.shields.io/badge/License-MIT-2ea44f?style=flat)](LICENSE)
[![Status](https://img.shields.io/badge/Status-Tested-success?style=flat)](https://github.com/RufusTheDwarf/git-signing-test)
[![PRs Welcome](https://img.shields.io/badge/PRs-Welcome-brightgreen?style=flat)](CONTRIBUTING.md)

<br>

A step-by-step guide for beginners. No prior Git, SSH, or GPG knowledge required.

<br>

[📖 Quick Start](#-quick-start) · [📚 Full Guide](#-full-guide) · [🔒 Security](#-security) · [🤝 Contributing](#-contributing)

</div>

---

## 📋 Overview

This guide walks you through configuring **Git on Windows** to automatically sign your commits with an **SSH key**, so that GitHub displays the **"Verified"** badge next to each of your commits.

> [!NOTE]
> This configuration is **local to your PC**. On another computer, you will need to generate a new signing key and repeat the setup.

### ✨ What You Get

| Feature | Description |
|---|---|
| 🔑 **Dedicated signing key** | A separate ed25519 key used only for signing — your authentication keys stay untouched. |
| 🤖 **Automatic signing** | Every commit is signed automatically — no extra flag needed. |
| ✅ **Verified badge** | GitHub recognizes your signature and shows the green checkmark. |
| 🛡️ **Secure** | The `.ssh` folder is hidden and its permissions are locked down. |
| 🖥️ **Works everywhere** | Git Bash, PowerShell, VS Code, and GitHub Desktop all respect the config. |

---

## 🚀 Quick Start

> [!TIP]
> If you already have Git and a verified GitHub email, you can jump straight to the commands below.

```bash
# 1. Create the .ssh folder in your user profile
mkdir -p /c/Users/$USERNAME/.ssh

# 2. Generate a dedicated signing key
ssh-keygen -t ed25519 -C "your.email@example.com" -f /c/Users/$USERNAME/.ssh/id_ed25519_signing

# 3. Tell Git to use SSH signing
git config --global gpg.format ssh
git config --global user.signingkey C:/Users/$USERNAME/.ssh/id_ed25519_signing.pub
git config --global commit.gpgsign true

# 4. Create the allowed_signers file
echo "your.email@example.com $(cat /c/Users/$USERNAME/.ssh/id_ed25519_signing.pub)" > /c/Users/$USERNAME/.ssh/allowed_signers
git config --global gpg.ssh.allowedSignersFile C:/Users/$USERNAME/.ssh/allowed_signers

cat /c/Users/$USERNAME/.ssh/id_ed25519_signing.pub
```

Then [add the public key to GitHub](#step-6--add-the-public-key-to-github) and [test the signature](#step-7--test-the-signature-locally).

---

## 📚 Full Guide

<details>
<summary><b>📖 Click to expand the full step-by-step guide</b></summary>

<br>

### Prerequisites

- Windows 10 or 11
- Git for Windows installed (version 2.34 or later)
- A GitHub account with a **verified** email address
- Git Bash (installed with Git for Windows)

---

### Step 1 — Check your environment

Open **Git Bash** and run:

```bash
git --version
```

You should see a version ≥ 2.34.0 (e.g. `git version 2.50.0.windows.1`).

Check your Git identity:

```bash
git config --global user.name
git config --global user.email
```

- If these commands output nothing, configure them:
  ```bash
  git config --global user.name "Your Name"
  git config --global user.email "your.email@example.com"
  ```
- The email must be **verified on GitHub** (Settings → Emails).

> [!WARNING]
> If your email is not verified on GitHub, the "Verified" badge will **not** appear.

---

### Step 2 — Create the `.ssh` folder in your user profile

On Windows, your home folder is `C:\Users\<YOUR_USERNAME>`.  
In Git Bash, `~` might point elsewhere (e.g. `H:\`). **Always use the full path** to avoid confusion.

```bash
# Replace $USERNAME with your Windows username (e.g. pv52wcl)
mkdir -p /c/Users/$USERNAME/.ssh
```

Verify the folder was created in the right place:

```bash
ls -la /c/Users/$USERNAME/.ssh
```

> [!TIP]
> You can find your Windows username by running `echo $USERNAME` in Git Bash or by checking `C:\Users\`.

---

### Step 3 — Generate the signing key

We create an **ed25519** key dedicated to signing, with an explicit name.

```bash
ssh-keygen -t ed25519 -C "your.email@example.com" -f /c/Users/$USERNAME/.ssh/id_ed25519_signing
```

- `-t ed25519`: modern, fast key type.
- `-C`: comment (your GitHub email).
- `-f`: key file path.

**Passphrase**:
- You can leave it empty (press Enter) so you don't have to type it on every commit.
- Or set one (more secure) and use `ssh-agent` to type it only once per session.

> [!IMPORTANT]
> **Never show the contents of the private file** `id_ed25519_signing` (without `.pub`). The `.pub` file is safe to share.

---

### Step 4 — Configure Git for SSH signing

```bash
git config --global gpg.format ssh
git config --global user.signingkey C:/Users/$USERNAME/.ssh/id_ed25519_signing.pub
git config --global commit.gpgsign true
```

- `gpg.format ssh`: use SSH instead of GPG.
- `user.signingkey`: path to the **public** key.
- `commit.gpgsign true`: automatically sign all commits.

---

### Step 5 — Create the `allowed_signers` file

This file lets Git verify signatures locally.

```bash
echo "your.email@example.com $(cat /c/Users/$USERNAME/.ssh/id_ed25519_signing.pub)" > /c/Users/$USERNAME/.ssh/allowed_signers
git config --global gpg.ssh.allowedSignersFile C:/Users/$USERNAME/.ssh/allowed_signers
```

---

### Step 6 — Add the public key to GitHub

Display your public key:

```bash
cat /c/Users/$USERNAME/.ssh/id_ed25519_signing.pub
```

Copy the entire output (starts with `ssh-ed25519` and ends with your email).

On GitHub:
1. Go to **Settings** → **SSH and GPG keys**.
2. Click **New SSH key**.
3. **Title**: `Windows signing key` (or any name).
4. **Key type**: **Signing Key** (⚠️ not "Authentication Key").
5. Paste the public key.
6. Click **Add SSH key**.

> [!IMPORTANT]
> Make sure you select **Signing Key** as the key type. If you select "Authentication Key", the "Verified" badge will not appear.

---

### Step 7 — Test the signature locally

Create a test repository:

```bash
mkdir -p /c/Users/$USERNAME/git-signing-test
cd /c/Users/$USERNAME/git-signing-test
git init
echo "test" > test.txt
git add test.txt
git commit -m "Test signed commit"
```

Verify the signature:

```bash
git log --show-signature -1
```

You should see:

```
Good "git" signature for your.email@example.com with ED25519 key SHA256:...
```

If you see an error `gpg.ssh.allowedSignersFile needs to be configured`, re-run Step 5.

---

### Step 8 — Push to GitHub and verify the "Verified" badge

1. Create an **empty** repository on GitHub (no README, no .gitignore).
2. In Git Bash, inside the `git-signing-test` folder:

```bash
git remote add origin https://github.com/<YOUR_USER>/<YOUR_REPO>.git
git push -u origin master
```

Authenticate (Git Credential Manager or personal access token).

3. Go to the GitHub repository page. The commit `Test signed commit` should display a **Verified** badge (green check).

> [!TIP]
> Click on the commit to see "This commit was signed with the committer's verified signature."

---

### Step 9 — Secure the `.ssh` folder (hidden + permissions)

On Windows, the `.ssh` folder is not hidden by default. You can make it invisible and restrict access to your user account only.

Open **PowerShell** (no administrator rights needed) and run:

```powershell
# Make the folder hidden
attrib +h "C:\Users\$env:USERNAME\.ssh"

# Remove inheritance and grant full control to the current user
icacls "C:\Users\$env:USERNAME\.ssh" /inheritance:r
icacls "C:\Users\$env:USERNAME\.ssh" /grant:r "$env:USERNAME:(OI)(CI)F"

# Also restrict access to the private key file
icacls "C:\Users\$env:USERNAME\.ssh\id_ed25519_signing" /inheritance:r
icacls "C:\Users\$env:USERNAME\.ssh\id_ed25519_signing" /grant:r "$env:USERNAME:F"
```

> [!NOTE]
> The "hidden" attribute is purely visual: files remain accessible if "Show hidden files" is enabled. Real protection comes from the NTFS permissions above.

Verify the folder is hidden:

```powershell
Get-Item "C:\Users\$env:USERNAME\.ssh" | Select-Object Name, Attributes
```

You should see `Hidden` in the attributes.

</details>

---

## 🔒 Security

> [!WARNING]
> **Never share your private key** (`id_ed25519_signing` without `.pub`).
> Never paste passwords, GitHub tokens, or passphrases into a repository, chat, or email.
> The `.ssh` folder must stay **outside any Git repository**.

| Do | Don't |
|---|---|
| ✅ Keep your private key safe | ❌ Share your private key |
| ✅ Use a passphrase (optional) | ❌ Commit your `.ssh` folder |
| ✅ Add one signing key per machine | ❌ Reuse a leaked key |
| ✅ Backup your key securely | ❌ Paste secrets in issues or PRs |

If you suspect a key has leaked, remove it from GitHub and generate a new one.

---

## ✅ Verification Commands

| Goal | Command |
|---|---|
| Check Git version | `git --version` |
| View Git identity | `git config --global user.name` and `user.email` |
| View signing config | `git config --global --list \| grep -iE "sign\|gpg"` |
| Verify a signed commit | `git log --show-signature -1` |
| Show public key | `cat /c/Users/$USERNAME/.ssh/id_ed25519_signing.pub` |
| Check .ssh folder location | `ls -la /c/Users/$USERNAME/.ssh` |

---

## 🤝 Contributing

Contributions are welcome! Please read [CONTRIBUTING.md](CONTRIBUTING.md) before opening a pull request.

1. Fork the repository.
2. Create a branch: `git switch -c fix/my-fix`.
3. Commit **signed** (this repo is about signing, so let's practice 😉):
   ```bash
   git commit -S -m "fix: correct ..."
   ```
4. Push and open a pull request.

---

## 📄 License

Distributed under the MIT License. See [LICENSE](LICENSE) for more information.

---

## 🙏 References

- [Git documentation — SSH signing](https://git-scm.com/docs/git-config#Documentation/git-config.txt-gpgformat)
- [GitHub Docs — About commit signature verification](https://docs.github.com/en/authentication/managing-commit-signature-verification/about-commit-signature-verification)
- [GitHub Docs — Telling Git about your signing key](https://docs.github.com/en/authentication/managing-commit-signature-verification/telling-git-about-your-signing-key)
- [Shields.io — Badge generator](https://shields.io)

---

<div align="center">

### ⭐ Found this useful?

**If this guide helped you get your "Verified" badge, consider giving it a star!**

[![Stars](https://img.shields.io/github/stars/RufusTheDwarf/git-ssh-signing-windows?style=flat&color=yellow&label=Stars)](https://github.com/RufusTheDwarf/git-ssh-signing-windows/stargazers)
[![Forks](https://img.shields.io/github/forks/RufusTheDwarf/git-ssh-signing-windows?style=flat&color=blue&label=Forks)](https://github.com/RufusTheDwarf/git-ssh-signing-windows/forks)
[![Issues](https://img.shields.io/github/issues/RufusTheDwarf/git-ssh-signing-windows?style=flat&color=red&label=Issues)](https://github.com/RufusTheDwarf/git-ssh-signing-windows/issues)

<br>

**[⬆ Back to top](#-git-ssh-signing-on-windows)**

<br>

<sub>Made with ❤️ for the Git community</sub>

<br>

<sub>MIT License · © 2026 RufusTheDwarf</sub>

</div>
