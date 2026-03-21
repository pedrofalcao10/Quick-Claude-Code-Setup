# Installing Claude Code on Windows

> [!IMPORTANT]
> **All terminal installation commands must be run in an Administrator PowerShell session.**
> Right-click **PowerShell** → **"Run as administrator"** before executing any command below.

## 1. Install Node.js

Download and install Node.js from the official site:

👉 https://nodejs.org/en/download

- Choose the **LTS (Long Term Support)** version for Windows
- Run the installer and make sure to check **"Add to PATH"** during setup
- After installation, verify it works by opening **PowerShell as administrator** and running:

```powershell
node --version
npm --version
```

Both commands should print version numbers.

---

## 2. Install Git

Download and install Git from the official site:

👉 https://git-scm.com/download/win

- Run the installer using the default options
- After installation, verify it works:

```powershell
git --version
```

### Configure your identity (must match your GitHub account)

> ⚠️ The email **must be exactly the same** as the one registered on your GitHub account.
> You can check it at: https://github.com/settings/emails

```powershell
git config --global user.name "Your Name"
git config --global user.email "your@email.com"
```

### Set the default branch name to `main`

```powershell
git config --global init.defaultBranch main
```

Verify all your global settings:

```powershell
git config --global --list
```

---

## 3. Install pnpm (optional but recommended)

pnpm is a faster, more efficient alternative to npm.

> Run in **PowerShell as administrator**:

```powershell
npm install -g pnpm
```

Verify the installation:

```powershell
pnpm --version
```

---

## 4. Install GitHub CLI (gh)

The GitHub CLI lets Claude Code create pull requests, manage issues, and interact with GitHub directly from the terminal.

**Option 1 — winget (recommended):**

```powershell
winget install --id GitHub.cli
```

**Option 2 — installer:** Download from 👉 https://cli.github.com and run with default options.

After installing, verify it works:

```powershell
gh --version
```

### Authenticate with GitHub

```powershell
gh auth login
```

This will walk you through browser-based GitHub authentication. Once done, you can create PRs with `gh pr create`.

Verify you are logged in:

```powershell
gh auth status
```

---

## 5. Install Claude Code

> Run in **PowerShell as administrator**:

```powershell
npm install -g @anthropic-ai/claude-code
```

> If you're using pnpm (recommended):
> ```powershell
> pnpm add -g @anthropic-ai/claude-code
> ```

---

## 6. Fix PowerShell Execution Policy (if needed)

On Windows, PowerShell may block `.ps1` scripts by default. If you see an `UnauthorizedAccess` error when running `claude`, fix it with:

> Run in **PowerShell as administrator**:

```powershell
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
```

This only needs to be done once.

---

## 7. Add Claude to PATH (if needed)

If `claude` is still not recognized after installation, you may need to add its install location to your PATH manually.

The installer places `claude.exe` at:

```
C:\Users\<YourUsername>\.local\bin\claude.exe
```

To add it to your PATH permanently, run:

```powershell
[Environment]::SetEnvironmentVariable(
  "PATH",
  "C:\Users\$env:USERNAME\.local\bin;" + [Environment]::GetEnvironmentVariable("PATH", "User"),
  "User"
)
```

Then **restart PowerShell** for the change to take effect.

---

## 8. Run Claude

```powershell
claude
```

On first run, Claude will prompt you to log in with your Anthropic account.

---

## Troubleshooting

| Problem | Fix |
|---|---|
| `claude` not found | Ensure the install bin folder is in your PATH (step 7) |
| `UnauthorizedAccess` on `.ps1` | Run execution policy fix (step 6) |
| `gh` not found | Re-install GitHub CLI and restart your terminal |
| `gh auth` fails | Try `gh auth login --web` or check your browser for the auth prompt |
| `node` not found | Re-install Node.js and ensure "Add to PATH" was checked |
| `git` not found | Re-install Git and restart your terminal |
| Wrong Git email | Run `git config --global user.email "correct@email.com"` |
| Permission errors with `npm install -g` | Run PowerShell as Administrator |

---

## Optional: Compound Engineering Plugin

Want a structured workflow for planning, reviewing, and retaining knowledge across projects?
See [COMPOUND-ENGINEERING.md](./COMPOUND-ENGINEERING.md) for a quick guide.
