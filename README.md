# Git Configuration Documentation

**Last Updated:** 2025-10-31
**Purpose:** Production-grade git configuration optimized for software engineering workflows

---

## Table of Contents
1. [User Identity & Signing](#user-identity--signing)
2. [Commit Configuration](#commit-configuration)
3. [GPG/SSH Signing](#gpgssh-signing)
4. [Aliases](#aliases)
5. [Colors & UI](#colors--ui)
6. [Branch & Tag Management](#branch--tag-management)
7. [Diff Configuration](#diff-configuration)
8. [Merge Configuration](#merge-configuration)
9. [Push & Fetch](#push--fetch)
10. [Performance Optimization](#performance-optimization)
11. [Workflow Enhancements](#workflow-enhancements)
12. [Security](#security)
13. [Advanced Features](#advanced-features)

---

## User Identity & Signing

```ini
[user]
    name = Your Name
    email = your.email@example.com
    signingkey = ~/.ssh/id_ed25519.pub
```

**Purpose:**
- **name/email**: Identity for all commits you make
- **signingkey**: Path to your SSH public key for signing commits

**Why:**
- Ensures proper attribution in git history
- SSH key signing is simpler than GPG (no key management complexity)
- Verified commits provide authenticity and non-repudiation

---

## Commit Configuration

```ini
[commit]
    gpgsign = true
    verbose = true
    template = ~/.config/git/commit-template.txt
    cleanup = scissors
```

**Purpose:**
- **gpgsign**: Automatically sign all commits
- **verbose**: Show diff while writing commit message
- **template**: Pre-populate commit messages with template
- **cleanup**: Use scissors mode for cleaning commit messages

**Why:**
- **Signed commits** prove authorship and prevent impersonation
- **Verbose mode** helps write better commit messages by showing what changed
- **Template** enforces consistent commit message format (conventional commits)
- **Scissors cleanup** preserves content above scissors line, removes below

**Template Format:**
```
<type>(<scope>): <subject>
- Types: feat, fix, docs, style, refactor, perf, test, chore, build, ci
- Enforces conventional commits standard
- Improves changelog generation and semantic versioning
```

---

## GPG/SSH Signing

```ini
[gpg]
    program = gpg
    format = ssh

[gpg "ssh"]
    allowedSignersFile = ~/.config/git/allowed_signers
    defaultKeyCommand = ssh-add -L
```

**Purpose:**
- **format = ssh**: Use SSH keys instead of GPG keys for signing
- **allowedSignersFile**: Verification of SSH signatures
- **defaultKeyCommand**: Automatically use SSH agent keys

**Why:**
- SSH signing is **simpler** than GPG (no separate keyring)
- Most developers already have SSH keys
- GitHub/GitLab support SSH signature verification
- SSH agent integration makes signing seamless

**allowed_signers format:**
```
your.email@example.com namespaces="git" ssh-ed25519 AAAAC3... your.email@example.com
```
This file allows git to verify your own signatures locally.

---

## Aliases

### Status & Logging
```ini
sb = status -sb                 # Short branch status
lol = log --oneline --graph --all  # Visual commit graph
```

**Why:**
- **sb**: Cleaner, more readable status output
- **lol**: Visualize branch topology quickly

### Undo & Amend
```ini
undo = reset HEAD~1 --mixed     # Undo last commit, keep changes
amend = commit --amend --no-edit  # Quick amend without changing message
fixup = commit --fixup          # Create fixup commit for interactive rebase
```

**Why:**
- **undo**: Safe way to uncommit (preserves working directory)
- **amend**: Quick fixes to last commit
- **fixup**: Part of interactive rebase workflow for clean history

### Branch Management
```ini
gone = ! git fetch -p && git for-each-ref --format '%(refname:short) %(upstream:track)' | awk '$2 == "[gone]" {print $1}' | xargs -r git branch -D
branches = branch -vv
recent = branch --sort=-committerdate --format='%(committerdate:relative)%09%(refname:short)'
```

**Why:**
- **gone**: Delete local branches whose remote has been deleted (cleanup after merged PRs)
- **branches**: See all branches with tracking info
- **recent**: Find branches you worked on recently (sorted by commit date)

### Better Diffs
```ini
staged = diff --cached          # Show staged changes
last = log -1 HEAD --stat      # Show last commit with stats
diffw = diff --word-diff       # Word-level diff (better for prose)
```

**Why:**
- **staged**: Preview what will be committed
- **last**: Quick review of what you just committed
- **diffw**: Better for reviewing documentation/text changes

### Interactive Operations
```ini
unstage = restore --staged      # Remove from staging area
discard = restore              # Discard working directory changes
```

**Why:**
- Modern git commands (replace `reset HEAD` and `checkout --`)
- More intuitive naming
- Safer operations (less destructive defaults)

### Workflow Shortcuts
```ini
pushu = push -u origin HEAD    # Push current branch and set upstream
sync = !git fetch --all --prune && git pull --rebase  # Sync with remote
wip = commit -am "WIP" --no-verify  # Quick work-in-progress commit
```

**Why:**
- **pushu**: Create remote branch automatically on first push
- **sync**: Update local repo with all remotes in one command
- **wip**: Quick save point (bypasses hooks for temporary commits)

### Search & Debugging
```ini
grep = grep -Ii                # Case-insensitive grep
find = log --all --full-history --  # Find file in history
contributors = shortlog -sn    # Show commit counts by author
```

**Why:**
- **grep**: Search codebase efficiently (case-insensitive)
- **find**: Locate deleted files or changes across all branches
- **contributors**: See team contribution metrics

---

## Colors & UI

```ini
[color]
    ui = auto

[color "diff"]
    meta = yellow bold
    commit = green bold
    frag = magenta bold
    old = red bold
    new = green bold
    whitespace = red reverse

[color "branch"]
    current = yellow reverse
    local = yellow
    remote = green

[color "status"]
    added = green
    changed = yellow
    untracked = red
```

**Purpose:**
- Enable automatic color in terminal output
- Customize colors for better readability

**Why:**
- **Visual clarity**: Quickly distinguish added/removed/modified
- **Consistency**: Same color scheme across all git commands
- **Accessibility**: Bold colors easier to see in various terminal themes
- **Whitespace highlighting**: Catch trailing spaces (red reverse)

---

## Branch & Tag Management

```ini
[branch]
    sort = -committerdate

[tag]
    sort = version:refname
    gpgsign = true
```

**Purpose:**
- **branch.sort**: Show most recently committed branches first
- **tag.sort**: Sort tags by semantic version
- **tag.gpgsign**: Sign all tags automatically

**Why:**
- Recent branches are usually more relevant
- Version sorting shows v1.9.0 before v1.10.0 (not alphabetically)
- Signed tags provide release authenticity

---

## Diff Configuration

```ini
[diff]
    algorithm = histogram
    colorMoved = plain
    mnemonicPrefix = true
    renames = true
    tool = vimdiff
    indentHeuristic = true
    colorMovedWS = allow-indentation-change
    submodule = log

[difftool]
    prompt = false
    trustExitCode = true

[diff "lockfile"]
    textconv = cat
    binary = true
```

**Purpose:**
- **algorithm = histogram**: More accurate diff algorithm
- **colorMoved**: Highlight moved code blocks
- **mnemonicPrefix**: Use meaningful prefixes (i/, w/, c/ instead of a/, b/)
- **renames**: Detect renamed files
- **indentHeuristic**: Better diff chunk boundaries
- **colorMovedWS**: Ignore whitespace when detecting moves
- **submodule = log**: Show submodule changes as log entries

**Why:**
- **Histogram algorithm**: Better than Myers for most code (faster, more intuitive)
- **Move detection**: Refactoring shows as moves, not delete+add
- **Mnemonic prefixes**: i/ = index, w/ = working tree, c/ = commit (clearer context)
- **Indent heuristic**: Diffs break at logical boundaries (functions, classes)
- **Lockfile handling**: Treat as binary (they're generated, not meaningful to diff)

**Tool Configuration:**
- **vimdiff**: Powerful side-by-side diff viewer
- **prompt = false**: Don't ask before launching difftool
- **trustExitCode**: Honor tool's exit code for success/failure

---

## Merge Configuration

```ini
[merge]
    conflictstyle = zdiff3
    tool = vimdiff
    ff = only

[mergetool]
    prompt = false
    keepBackup = false
    trustExitCode = true
```

**Purpose:**
- **conflictstyle = zdiff3**: Show 3-way merge conflicts with common ancestor
- **ff = only**: Only allow fast-forward merges (prevent merge commits)
- **keepBackup = false**: Don't keep .orig files after merge

**Why:**
- **zdiff3**: Shows original code + both changes = easier conflict resolution
  ```
  <<<<<<< HEAD
  your changes
  ||||||| base
  original code (this is the extra context!)
  =======
  their changes
  >>>>>>> branch
  ```
- **ff = only**: Enforces linear history (fails if fast-forward not possible)
- **No backups**: .orig files clutter working directory
- **No prompt**: Faster merge resolution workflow

---

## Push & Fetch

```ini
[push]
    autoSetupRemote = true
    followTags = true
    default = current

[fetch]
    prune = true
    pruneTags = true
    fsckObjects = true
    parallel = 0
    writeCommitGraph = true
```

**Purpose:**
- **autoSetupRemote**: Create remote branch on first push
- **followTags**: Push annotated tags with commits
- **default = current**: Push current branch to same name
- **prune**: Delete local references to deleted remote branches
- **pruneTags**: Delete local tags deleted on remote
- **fsckObjects**: Verify objects during fetch (detect corruption)
- **parallel = 0**: Auto-detect optimal parallel fetch count
- **writeCommitGraph**: Update commit graph on fetch

**Why:**
- **autoSetupRemote**: No need for `-u origin branch` on first push
- **followTags**: Releases (tags) are pushed with commits automatically
- **Pruning**: Keep local repo clean (no stale references)
- **Integrity checks**: Catch repository corruption early
- **Parallel fetching**: Faster fetches on multi-remote repos
- **Commit graph**: Faster log/blame operations after fetch

---

## Performance Optimization

```ini
[core]
    editor = nvim
    excludesfile = ~/.gitignore_global
    fsmonitor = true
    untrackedCache = true
    preloadIndex = true
    commitGraph = true
    multiPackIndex = true

[gc]
    auto = 256
    writeCommitGraph = true

[maintenance]
    auto = true
    strategy = incremental

[pack]
    useSparse = true
    writeReverseIndex = true

[index]
    version = 4

[protocol]
    version = 2
```

**Purpose:**
- **fsmonitor**: Use filesystem events to track changes (faster status)
- **untrackedCache**: Cache untracked files (faster status)
- **preloadIndex**: Load index in parallel (faster operations)
- **commitGraph**: Accelerate commit traversal
- **multiPackIndex**: Faster object lookups in repos with many packfiles
- **gc.auto = 256**: Run garbage collection after 256 loose objects
- **maintenance**: Automatic background optimization
- **pack optimizations**: Faster clones and fetches
- **index v4**: Compressed index format
- **protocol v2**: Faster wire protocol

**Why:**
- **Large repos**: These settings make big repositories faster
- **Background optimization**: Maintenance runs automatically
- **Modern git**: Uses latest performance features
- **Minimal overhead**: All settings are backwards compatible

**Performance Impact:**
- `git status` up to **10x faster** on large repos
- `git log` and `git blame` significantly faster
- Reduced disk space usage
- Better clone/fetch performance

---

## Workflow Enhancements

```ini
[help]
    autocorrect = prompt

[rerere]
    enabled = true
    autoupdate = true

[rebase]
    autoSquash = true
    autoStash = true
    updateRefs = true

[pull]
    rebase = true

[status]
    showUntrackedFiles = all
    submoduleSummary = true

[log]
    abbrevCommit = true
    follow = true
    decorate = auto
    date = relative

[submodule]
    recurse = true

[blame]
    ignoreRevsFile = .git-blame-ignore-revs
    markIgnoredLines = true
    markUnblamableLines = true
```

**Purpose & Why:**

### help.autocorrect = prompt
- Suggests corrections for typos: `git comit` → prompt to run `git commit`
- Safer than `autocorrect = 1` (auto-runs after 1 second)

### rerere (Reuse Recorded Resolution)
- **enabled**: Records how you resolved merge conflicts
- **autoupdate**: Automatically applies previously learned resolutions
- **Why**: If you rebase and get the same conflict twice, it auto-resolves the second time

### rebase
- **autoSquash**: Automatically arrange `fixup!` commits during interactive rebase
- **autoStash**: Stash changes before rebase, apply after
- **updateRefs**: Update all branches pointing to rebased commits
- **Why**: Cleaner rebase workflow, less manual work

### pull.rebase = true
- Always rebase instead of merge when pulling
- **Why**: Linear history, no merge commits from `git pull`

### status
- **showUntrackedFiles = all**: Show all files in untracked directories
- **submoduleSummary**: Show submodule changes in status
- **Why**: Better visibility of repository state

### log
- **abbrevCommit**: Show short commit hashes (7 chars)
- **follow**: Track file renames in log
- **decorate**: Show branch/tag names in log
- **date = relative**: "2 hours ago" instead of timestamps
- **Why**: More readable, informative output

### submodule.recurse = true
- Automatically recurse into submodules for all commands
- **Why**: Submodules stay in sync with parent repo

### blame
- **ignoreRevsFile**: Ignore bulk formatting commits in blame
- **markIgnoredLines**: Mark lines from ignored commits
- **Why**: `git blame` shows who actually wrote the code, not who ran Prettier

---

## Security

```ini
[transfer]
    fsckObjects = true

[receive]
    fsckObjects = true

[url "https://github.com/"]
    insteadOf = git://github.com/

[url "https://"]
    insteadOf = git://

[url "ssh://git@github.com/"]
    pushInsteadOf = https://github.com/
```

**Purpose:**
- **fsckObjects**: Verify object integrity during transfer
- **URL rewrites**: Force HTTPS/SSH, block insecure git:// protocol

**Why:**
- **Corruption detection**: Catch corrupted objects during push/pull
- **Security**: git:// protocol is unencrypted and unauthenticated
- **Best practice**: Always use HTTPS for fetching, SSH for pushing
- **GitHub optimization**: Uses SSH for push (faster, more reliable)

**Attack prevention:**
- Man-in-the-middle attacks (git:// protocol)
- Repository corruption
- Malicious object injection

---

## Advanced Features

```ini
[init]
    defaultBranch = main

[feature]
    manyFiles = true

[filter "lfs"]
    process = git-lfs filter-process
    required = true
    clean = git-lfs clean -- %f
    smudge = git-lfs smudge -- %f

[column]
    ui = auto
```

**Purpose:**
- **defaultBranch = main**: Use "main" instead of "master" for new repos
- **manyFiles**: Optimize for repositories with many files
- **lfs**: Git Large File Storage support
- **column.ui**: Use columns in output when space permits

**Why:**
- **main**: Industry standard, inclusive naming
- **manyFiles**: Better performance for monorepos
- **LFS**: Required for binary assets, datasets, media files
- **Columns**: More compact output in wide terminals

---

## Global Ignore Patterns

**File:** `~/.config/git/ignore`

### Categories:
1. **OS files**: .DS_Store, Thumbs.db, desktop.ini
2. **IDEs**: .vscode/, .idea/, *.swp
3. **Dependencies**: node_modules/, vendor/, .venv/
4. **Secrets**: .env, *.key, *.pem, credentials.json
5. **Build artifacts**: dist/, build/, target/, *.egg-info/
6. **DevOps**: .terraform/, *.tfstate, docker-compose.override.yml
7. **Languages**: __pycache__/, *.pyc, *.class, *.o

**Why:**
- **Never commit secrets**: Prevent credential leaks
- **Clean repos**: Don't commit generated files
- **Universal**: Applies to all your repositories
- **Team safety**: Even if .gitignore is incomplete

---

## Git Attributes

**File:** `~/.config/git/attributes`

```gitattributes
* text=auto                    # Auto-detect text files
*.js text eol=lf              # Force LF line endings
*.png binary                   # Mark as binary
package-lock.json binary       # Don't diff lockfiles
```

**Purpose:**
- **Line ending normalization**: Consistent LF across platforms
- **Binary file detection**: Don't diff images/binaries
- **Merge strategies**: Special handling for lockfiles

**Why:**
- **Cross-platform**: Windows/Mac/Linux consistency
- **Clean diffs**: No "entire file changed" due to line endings
- **Lockfile handling**: Merge lockfiles correctly (union strategy)
- **Performance**: Don't waste time diffing binary files

---

## File Structure

```
~/.config/git/
├── config                    # Main configuration file
├── commit-template.txt      # Commit message template
├── allowed_signers          # SSH signature verification
├── attributes               # Global gitattributes
├── ignore                   # Global gitignore
└── README.md                # This documentation
```

---

## Why ~/.config/git Instead of ~/.gitconfig?

This configuration uses the **XDG Base Directory Specification** for organizing git configuration files.

### Benefits of ~/.config/git/

1. **Better Organization**: All git files in one directory instead of scattered in home directory
2. **Modern Standard**: XDG Base Directory is the modern Linux/Unix standard (adopted by git 2.32+)
3. **Cleaner Home Directory**: Reduces clutter in ~/
4. **Easier Backup**: Single directory to backup all git configuration
5. **Version Control**: Easier to track in dotfiles repositories

### Directory Structure Comparison

**Old approach (traditional):**
```
~/
├── .gitconfig              # Main config
├── .gitignore_global       # Global ignore
├── .gitmessage            # Commit template
└── .git-templates/        # Templates
```

**New approach (XDG):**
```
~/.config/git/
├── config                 # Main config (was ~/.gitconfig)
├── ignore                 # Global ignore (was ~/.gitignore_global)
├── commit-template.txt    # Commit template
├── allowed_signers        # SSH signing verification
└── attributes             # Global attributes
```

### Migration from ~/.gitconfig

If you have an existing `~/.gitconfig` file, here's how to migrate:

```bash
# 1. Backup existing configuration
cp ~/.gitconfig ~/.gitconfig.backup

# 2. Create XDG directory
mkdir -p ~/.config/git

# 3. Move existing config (if you want to keep your current settings)
mv ~/.gitconfig ~/.config/git/config

# 4. Or start fresh with this configuration
cp config ~/.config/git/config

# 5. Verify git finds the config
git config --list --show-origin
```

**Git will automatically check both locations in this order:**
1. `~/.config/git/config` (XDG, preferred)
2. `~/.gitconfig` (traditional, fallback)

**Important:** If both files exist, git will read from **both**, with `~/.gitconfig` taking precedence. To avoid conflicts, choose one location.

---

## Installation

### Step 1: Generate SSH Key (if you don't have one)

SSH keys are used for both authentication (pushing/pulling) and signing commits.

#### Generate ED25519 Key (Recommended)

```bash
# Generate a new ED25519 SSH key
ssh-keygen -t ed25519 -C "your.email@example.com"

# When prompted:
# - File location: Press Enter for default (~/.ssh/id_ed25519)
# - Passphrase: Enter a strong passphrase (recommended for security)
```

**Why ED25519?**
- ✅ More secure than RSA
- ✅ Smaller key size (faster)
- ✅ Better performance
- ✅ Recommended by GitHub, GitLab, and security experts

#### Alternative: RSA Key (if ED25519 not supported)

```bash
# Generate 4096-bit RSA key (if ED25519 not available)
ssh-keygen -t rsa -b 4096 -C "your.email@example.com"
```

#### Add SSH Key to SSH Agent

```bash
# Start ssh-agent (if not running)
eval "$(ssh-agent -s)"

# Add your SSH key to the agent
ssh-add ~/.ssh/id_ed25519

# Verify key is loaded
ssh-add -L

# Make it persist across reboots (macOS)
# Add to ~/.ssh/config:
cat >> ~/.ssh/config <<'EOF'
Host *
  AddKeysToAgent yes
  UseKeychain yes
  IdentityFile ~/.ssh/id_ed25519
EOF
```

---

### Step 2: Configure SSH Keys for GitHub

SSH keys serve two purposes on GitHub:
1. **Authentication** - Push/pull repositories
2. **Signing** - Sign commits and tags

You'll add the **same public key** for both purposes.

#### 2.1 Add SSH Key for Authentication

```bash
# Copy your public key to clipboard
# macOS:
pbcopy < ~/.ssh/id_ed25519.pub

# Linux (requires xclip):
xclip -selection clipboard < ~/.ssh/id_ed25519.pub

# Or just display it:
cat ~/.ssh/id_ed25519.pub
```

**On GitHub:**
1. Go to **Settings** → **SSH and GPG keys** → **New SSH key**
2. **Title**: `Work Laptop - Authentication` (descriptive name)
3. **Key type**: `Authentication Key`
4. **Key**: Paste your public key
5. Click **Add SSH key**

**Test SSH connection:**
```bash
ssh -T git@github.com
# Should see: "Hi username! You've successfully authenticated..."
```

#### 2.2 Add SSH Key for Signing (Same Key)

**On GitHub:**
1. Go to **Settings** → **SSH and GPG keys** → **New SSH key**
2. **Title**: `Work Laptop - Signing` (descriptive name)
3. **Key type**: `Signing Key` ⚠️ Important: Select "Signing Key" not "Authentication Key"
4. **Key**: Paste the **same public key** again
5. Click **Add SSH key**

**Why add the same key twice?**
- GitHub separates authentication keys from signing keys
- You can use the same key for both purposes
- Or use different keys if you prefer separation

#### 2.3 Enable Vigilant Mode (Optional but Recommended)

**On GitHub:**
1. Go to **Settings** → **SSH and GPG keys**
2. Scroll down to **Vigilant mode**
3. Check **Flag unsigned commits as unverified**

**Benefits:**
- All unsigned commits will be marked as "Unverified"
- Makes it obvious when commits aren't signed
- Improves security awareness

---

### Step 3: Configure SSH Keys for GitLab

Similar to GitHub, GitLab uses SSH keys for authentication and signing.

#### 3.1 Add SSH Key for Authentication

```bash
# Copy your public key (same commands as above)
cat ~/.ssh/id_ed25519.pub
```

**On GitLab:**
1. Go to **Preferences** → **SSH Keys**
2. **Key**: Paste your public key
3. **Title**: `Work Laptop - Authentication` (descriptive name)
4. **Usage type**: `Authentication` or `Authentication & Signing`
5. **Expiration date**: Optional (recommended for security)
6. Click **Add key**

**Test SSH connection:**
```bash
ssh -T git@gitlab.com
# Should see: "Welcome to GitLab, @username!"
```

#### 3.2 Configure Commit Signing on GitLab

**On GitLab:**
1. Go to **Preferences** → **GPG Keys** (yes, even for SSH signing!)
2. GitLab supports SSH signing via the same SSH key
3. Alternatively, if you want separate signing keys, add another SSH key

**Note:** GitLab's SSH signing support may vary by version. For best compatibility:
- **GitLab 14.0+**: Native SSH signing support
- **Earlier versions**: May require GPG signing instead

---

### Step 4: Install Configuration Files

```bash
# 1. Create config directory
mkdir -p ~/.config/git

# 2. Copy all configuration files
cp config ~/.config/git/config
cp commit-template.txt ~/.config/git/commit-template.txt
cp allowed_signers ~/.config/git/allowed_signers
cp attributes ~/.config/git/attributes
cp ignore ~/.config/git/ignore

# 3. Make sure old ~/.gitconfig doesn't conflict
# Option A: Remove it (if you're using this config)
mv ~/.gitconfig ~/.gitconfig.old

# Option B: Keep both (not recommended - can cause conflicts)
# Git will read both, with ~/.gitconfig taking precedence
```

---

### Step 5: Update Personal Information

#### 5.1 Edit Main Config

Edit `~/.config/git/config`:

```bash
# Open in your editor
nvim ~/.config/git/config  # or nano, vim, code, etc.
```

**Replace these values:**
```ini
[user]
    name = Your Name              → Your actual name
    email = your.email@example.com → Your actual email
    signingkey = ~/.ssh/id_ed25519.pub → Path to your SSH PUBLIC key
```

**Important:**
- Use your **public key** (.pub file), not private key
- Must match the email you used with GitHub/GitLab
- Path can be absolute or relative to home (~/)

#### 5.2 Create allowed_signers File

The `allowed_signers` file tells git which SSH keys are authorized to sign commits.

```bash
# Edit the allowed_signers file
nvim ~/.config/git/allowed_signers
```

**Format:**
```
your.email@example.com namespaces="git" ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAA... your.email@example.com
```

**Generate your entry:**
```bash
# Get your public key content
cat ~/.ssh/id_ed25519.pub

# Format it correctly (replace with your actual email and key)
echo "your.email@example.com namespaces=\"git\" $(cat ~/.ssh/id_ed25519.pub)" > ~/.config/git/allowed_signers
```

**Explanation of format:**
- `your.email@example.com` - Email (must match git config)
- `namespaces="git"` - Limits key to git signing
- `ssh-ed25519 AAAA...` - Your actual public key content
- `your.email@example.com` - Comment (usually same email)

#### 5.3 Verify SSH Key Paths

```bash
# Verify public key exists and is readable
cat ~/.ssh/id_ed25519.pub

# Verify private key exists (don't display it!)
ls -la ~/.ssh/id_ed25519

# Check file permissions (private key should be 600)
chmod 600 ~/.ssh/id_ed25519
chmod 644 ~/.ssh/id_ed25519.pub
```

---

### Step 6: Configure SSH Signing in Git

Your configuration already has SSH signing enabled, but let's verify:

```bash
# Check signing configuration
git config --global gpg.format
# Should output: ssh

git config --global user.signingkey
# Should output: ~/.ssh/id_ed25519.pub (or your key path)

git config --global commit.gpgsign
# Should output: true

git config --global gpg.ssh.allowedSignersFile
# Should output: ~/.config/git/allowed_signers
```

**If any are missing, set them:**
```bash
git config --global gpg.format ssh
git config --global user.signingkey ~/.ssh/id_ed25519.pub
git config --global commit.gpgsign true
git config --global gpg.ssh.allowedSignersFile ~/.config/git/allowed_signers
```

---

### Step 7: Test Your Configuration

#### Test 1: Verify Config is Loaded

```bash
# See where git is reading config from
git config --list --show-origin | grep "user.name"

# Should show: file:/home/you/.config/git/config

# Check all important settings
git config user.name
git config user.email
git config user.signingkey
git config commit.gpgsign
```

#### Test 2: Test Commit Signing

```bash
# Create a test repository
mkdir /tmp/git-signing-test
cd /tmp/git-signing-test
git init

# Make a test commit
git commit --allow-empty -m "test: verify SSH signing works"

# Verify the commit is signed
git log --show-signature -1

# Should see:
# Good "git" signature for your.email@example.com with ED25519 key SHA256:...
```

**If signing fails, troubleshoot:**
```bash
# Check ssh-agent has your key
ssh-add -L

# If empty, add your key
ssh-add ~/.ssh/id_ed25519

# Try signing again
GIT_TRACE=1 git commit --allow-empty -m "test" -S
```

#### Test 3: Test GitHub Integration

```bash
# Clone a repository via SSH
git clone git@github.com:yourusername/your-repo.git
cd your-repo

# Make a signed commit
echo "test" >> README.md
git add README.md
git commit -m "test: verify signed commit on GitHub"

# Push to GitHub
git push

# Check on GitHub:
# - Go to the commit on GitHub's web interface
# - Should see "Verified" badge next to your commit
```

#### Test 4: Test Aliases

```bash
git sb              # Short status
git recent          # Recent branches
git lol             # Visual log
git branches        # Branch list with tracking
```

---

### Step 8: Configure Additional Settings (Optional)

#### Set Default Editor

```bash
# Already set to nvim in config, but you can change it:
git config --global core.editor "code --wait"  # VS Code
git config --global core.editor "vim"          # Vim
git config --global core.editor "nano"         # Nano
```

#### Configure Diff/Merge Tools

```bash
# VS Code as difftool
git config --global diff.tool vscode
git config --global difftool.vscode.cmd 'code --wait --diff $LOCAL $REMOTE'

# VS Code as mergetool
git config --global merge.tool vscode
git config --global mergetool.vscode.cmd 'code --wait --merge $REMOTE $LOCAL $BASE $MERGED'
```

#### Enable Git Maintenance (Performance)

```bash
# Enable automatic background maintenance
git maintenance start

# This will optimize your repositories automatically:
# - Garbage collection
# - Commit-graph updates
# - Incremental repack
```

---

## Recommendations

### For Teams
1. Share `allowed_signers` file (add team keys)
2. Agree on `.git-blame-ignore-revs` for formatting commits
3. Use same `commit-template.txt` for consistency
4. Document custom aliases in team wiki

### For Large Repos
1. Enable `core.fsmonitor` (requires watchman or built-in)
2. Use `git maintenance start` for background optimization
3. Consider partial clone: `git clone --filter=blob:none`
4. Use sparse checkout for monorepos

### Security Checklist
- ✅ All commits signed
- ✅ git:// protocol blocked
- ✅ Secrets in global ignore
- ✅ fsckObjects enabled
- ✅ SSH key in ssh-agent

---

## Troubleshooting

### Signing Issues
```bash
# Verify SSH key is loaded
ssh-add -L

# Test signing
git commit --allow-empty -m "test" -S

# Check allowed_signers format
cat ~/.config/git/allowed_signers
```

### Performance Issues
```bash
# Run maintenance
git maintenance run --task=gc

# Check repository health
git fsck

# Rebuild commit graph
git commit-graph write --reachable
```

### Merge Tool Not Working
```bash
# Verify vimdiff is available
which vimdiff

# Or use vscode
git config --global merge.tool vscode
git config --global mergetool.vscode.cmd 'code --wait --merge $REMOTE $LOCAL $BASE $MERGED'
```

---

## Resources

- [Git Documentation](https://git-scm.com/doc)
- [Conventional Commits](https://www.conventionalcommits.org/)
- [Git Attributes](https://git-scm.com/docs/gitattributes)
- [SSH Signing](https://docs.github.com/en/authentication/managing-commit-signature-verification/about-commit-signature-verification)
- [Git Performance](https://git-scm.com/docs/git-config#_performance)

---

## License

This configuration is free to use and modify. Attribution appreciated but not required.
