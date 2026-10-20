# LOGBOOK — Git & GitHub CLI (gh) Basics

Repository: https://github.com/babyclaire0222-cyber/unit-converter
Environment: Windows, PowerShell · git 2.55.0.windows.5 · gh 2.102.0

Tokens are never shown. Any credential in output is replaced with `<REDACTED>`.

---

## Before You Start

**Command:** `git --version` and `gh --version`
**What it did:** Checked that git and the GitHub CLI are both installed before starting.
**Output:**
```
git version 2.55.0.windows.5
gh version 2.102.0 (2026-09-30)
```

---

## Part A — Set Up gh

### A1 — Log in

**Command:** `gh auth login`
**What it did:** Logged gh into my GitHub account through the interactive prompts (the only task where prompts are allowed). I chose GitHub.com, HTTPS, said yes to authenticating Git, and logged in with a web browser using a one-time code.
**Choice:** HTTPS, because `gh auth setup-git` (A4) lets git reuse gh's login over HTTPS, so I don't have to create and manage SSH keys.
**Found at:** `gh auth login --help`
**Output:**
```
✓ Authentication complete.
- gh config set -h github.com git_protocol https
✓ Configured git protocol
✓ Logged in as babyclaire0222-cyber
```

### A2 — Check that it worked

**Command:** `gh auth status`
**What it did:** Reported which account gh is logged in to, how the login is stored (Windows keyring), the git protocol, and which permissions the token has.
**Found at:** `gh auth status --help`
**Output:**
```
github.com
  ✓ Logged in to github.com account babyclaire0222-cyber (keyring)
  - Active account: true
  - Git operations protocol: https
  - Token: <REDACTED>
  - Token scopes: 'gist', 'read:org', 'repo', 'workflow'
```
- **Host:** github.com
- **Username:** babyclaire0222-cyber
- **Token scopes:** gist, read:org, repo, workflow

### A3 — Choose your text editor

**Command:** `gh config set editor "code --wait"`
**What it did:** Saved VS Code as the editor gh opens when it needs longer text. `--wait` makes gh pause until I close the file, so it doesn't save an empty message.
**Found at:** `gh config set --help`

**Command:** `gh config list`
**What it did:** Showed all my gh settings, confirming the new editor and the HTTPS protocol saved in A1.
**Found at:** `gh config list --help`
**Output (trimmed):**
```
git_protocol=https
editor=code --wait
prompt=enabled
```

**Command:** `gh config get editor`
**What it did:** Read back only the editor setting to confirm it was stored.
**Found at:** `gh config get --help`
**Output:**
```
code --wait
```

### A4 — Let git borrow your gh login

**Command:** `gh auth setup-git`
**What it did:** Registered gh as git's credential helper for GitHub, so `git push` and `git pull` use my gh login and never ask for a password. It printed nothing, which means it succeeded.
**Found at:** `gh auth setup-git --help`

**Check:** `git config --list --show-origin | Select-String credential`
**Output (trimmed):**
```
file:C:/Program Files/Git/etc/gitconfig credential.helper=manager
file:C:/Users/yuyua/.gitconfig  credential.https://github.com.helper=
file:C:/Users/yuyua/.gitconfig  credential.https://github.com.helper=!'C:\Program Files\GitHub CLI\gh.exe' auth git-credential
```
The blank `helper=` line clears the older helper (Git Credential Manager, set in the system config) for github.com, and the next line makes gh the one git uses there.

### A5 — Make a shortcut

**Command:** `gh alias set bugs "issue list --label bug" --clobber`
**What it did:** Created a shortcut so `gh bugs` runs `gh issue list --label bug` in whichever repo I'm in. My first attempt saved a placeholder instead of the real `--label` flag, so I used `--clobber` to overwrite the wrong alias.
**Found at:** `gh alias set --help`, `gh issue list --help`

**Command:** `gh alias list`
**What it did:** Listed my aliases to confirm `bugs` saved correctly (`co` is a built-in default).
**Found at:** `gh alias list --help`
**Output:**
```
bugs: issue list --label bug
co: pr checkout
```

### A6 — The one you must not run

**Command (NOT run):** `gh auth token`
**What it does:** Prints the full login token for the active account on the default host (`--hostname` and `--user` choose others). Unlike `gh auth status`, it does not mask the token.
**Why it is needed / why it is dangerous:** Scripts and tools sometimes need the token to call GitHub's API as me, but it shows the token in plain text, so anyone who sees my screen, a screenshot or a recording could copy it and act as me until I revoke it.
**Found at:** `gh auth token --help` (only the help page was opened)

### A7 — Reading only — change nothing

**Variable:** `GH_HOST`
**What I would set it to:** my company's GitHub Enterprise Server hostname, e.g. `github.mycompany.com` (hostname only, no `https://`). In PowerShell that would be `$env:GH_HOST = "github.mycompany.com"`. Not actually set.
**What it does:** Makes gh use that host by default for commands where no host is given and none can be worked out from the local repo's git remote. If I had never logged in to that host, `GH_ENTERPRISE_TOKEN` would also be needed.
**Found at:** `gh help environment`

---

## Part B — Make the Repository

### B1 — Pick a license and a .gitignore

**Command:** `gh repo license list`
**What it did:** Listed the license templates gh can add to a new repo, with their keys.
**Found at:** `gh repo license list --help`

**Command:** `gh repo gitignore list`
**What it did:** Listed the .gitignore templates gh can add, by language or tool.
**Found at:** `gh repo gitignore list --help`

**Picked:**
- License: **MIT** (`mit`) — short and permissive, lets anyone use the code.
- .gitignore: **Python** — unit-converter will be written in Python.

### B2 — Create the repository, in one command

**Command:**
```
gh repo create unit-converter --public --add-readme --license mit --gitignore Python --description "A small command-line tool that converts units" --clone
```
**What it did:** Created a public repo on GitHub with a README, the MIT license, the Python .gitignore and a one-line description, then cloned it into my Documents folder. No prompts appeared because every choice was passed as a flag.
**Found at:** `gh repo create --help`
**Output:**
```
✓ Created repository babyclaire0222-cyber/unit-converter on github.com
  https://github.com/babyclaire0222-cyber/unit-converter
Cloning into 'unit-converter'...
```

### B3 — Look at what you made

**Command:** `cd unit-converter` then `gh repo view`
**What it did:** Displayed the repo's name, description and README in the terminal. It does not show the branch or visibility.
**Found at:** `gh repo view --help`

**Command:** `gh repo view --json defaultBranchRef,visibility`
**What it did:** Asked for just the two fields I needed, as JSON.
**Found at:** `gh repo view --help` (JSON FIELDS)
**Output:**
```json
{
  "defaultBranchRef": {
    "name": "main"
  },
  "visibility": "PUBLIC"
}
```
- **Default branch:** main
- **Visibility:** public

---

## Part C — Write Down the Work as Issues

### C1 — Labels

**Command:** `gh label list`
**What it did:** Listed the 10 labels GitHub creates by default. `bug` and `documentation` already existed, so only `enhancement-cli` had to be made.
**Found at:** `gh label list --help`

**Command:**
```
gh label create enhancement-cli --color 1d76db --description "New feature or improvement to the command-line interface"
```
**What it did:** Created a blue `enhancement-cli` label with a description. The colour is a hex code without `#`.
**Found at:** `gh label create --help`
**Output:**
```
✓ Label "enhancement-cli" created in babyclaire0222-cyber/unit-converter
```

### C2 — Create three issues

**Commands:**
```
gh issue create --title "Add support for temperature conversion" --body "The converter cannot handle temperatures yet. Users should be able to convert between Celsius, Fahrenheit and Kelvin. Temperature needs its own formulas because it uses offsets, not a simple multiplier like length or weight." --label enhancement-cli

gh issue create --title "Converter crashes on negative input" --body "Passing a negative number, for example -5, makes the tool exit with an error instead of converting it. Negative values are valid for things like temperature, so they need to be accepted. The input parsing seems to treat the minus sign as an unknown option." --label bug

gh issue create --title "Add --help output to the tool" --body "Running the tool with --help currently prints nothing useful. New users have no way to see which units are supported or how to format a conversion. The help text should list the available units and show one or two example commands." --label documentation,enhancement-cli
```
**What it did:** Created three issues, each in one command with title, body and labels passed as flags. Two labels are given as a comma-separated list with no space.
**Found at:** `gh issue create --help`
**Output:**
```
https://github.com/babyclaire0222-cyber/unit-converter/issues/1
https://github.com/babyclaire0222-cyber/unit-converter/issues/2
https://github.com/babyclaire0222-cyber/unit-converter/issues/3
```

### C3 — List and filter

**Command:** `gh issue list`
**What it did:** Listed every open issue.
**Output:**
```
ID  TITLE                                   LABELS                          UPDATED
#3  Add --help output to the tool           documentation, enhancement-cli  less than a minute ago
#2  Converter crashes on negative input     bug                             less than a minute ago
#1  Add support for temperature conversion  enhancement-cli                 less than a minute ago
```

**Command:** `gh issue list --label bug`
**What it did:** Listed only open issues labelled `bug`.
**Output:**
```
ID  TITLE                                LABELS  UPDATED
#2  Converter crashes on negative input  bug     about 2 minutes ago
```

**Command:** `gh bugs`
**What it did:** Ran the A5 shortcut. It gave the same result as the command above, so the alias works.
**Output:**
```
ID  TITLE                                LABELS  UPDATED
#2  Converter crashes on negative input  bug     about 2 minutes ago
```
**What was wrong earlier:** In A5 I first saved the alias with a placeholder instead of `--label`. I fixed it with `gh alias set bugs "issue list --label bug" --clobber` before testing here.
**Found at:** `gh issue list --help`

### C4 — Read one issue

**Command:** `gh issue view 2 --comments`
**What it did:** Showed issue #2 in full in the terminal: title, status, author, label and body. `--comments` includes comments (there were 0 at this point).
**Found at:** `gh issue view --help`
**Output (trimmed):**
```
Converter crashes on negative input babyclaire0222-cyber/unit-converter#2
Open • babyclaire0222-cyber opened about 2 minutes ago • 0 comments
Labels: bug
```

### C5 — Comment on it

**Command:**
```
gh issue comment 2 --body "Plan: tell the argument parser to treat values starting with a minus sign as numbers, not options. I will add a test that converts -5 and -40 to check negative input works, then fix the parsing so the test passes."
```
**What it did:** Added a comment to issue #2 explaining how I plan to fix it.
**Found at:** `gh issue comment --help`
**Output:**
```
https://github.com/babyclaire0222-cyber/unit-converter/issues/2#issuecomment-6237528548
```

### C6 — Assign them to yourself

**Command:** `gh issue edit 1 2 3 --add-assignee "@me"`
**What it did:** Assigned all three issues to me in one command. `@me` means the logged-in account; it is quoted because `@` is a special character in PowerShell.
**Can it take more than one issue?** Yes. The help says "Edit one or more issues" and the usage is `gh issue edit {<numbers> | <urls>}`. **Commands used: 1.**
**Found at:** `gh issue edit --help`
**Output:**
```
https://github.com/babyclaire0222-cyber/unit-converter/issues/1
https://github.com/babyclaire0222-cyber/unit-converter/issues/2
https://github.com/babyclaire0222-cyber/unit-converter/issues/3
```

### C7 — Your issues

**Command:** `gh issue status`
**What it did:** Showed a summary of issues relevant to me in three groups: assigned to me, mentioning me, and opened by me. When run inside a repo folder it summarises that repo.
**Found at:** `gh issue status --help`
**Output (trimmed):**
```
Issues assigned to you
  #1  Add support for temperature conversion  enhancement-cli
  #3  Add --help output to the tool           documentation, enhancement-cli
  #2  Converter crashes on negative input     bug

Issues mentioning you
  There are no issues mentioning you

Issues opened by you
  #1  Add support for temperature conversion  enhancement-cli
  #3  Add --help output to the tool           documentation, enhancement-cli
  #2  Converter crashes on negative input     bug
```
