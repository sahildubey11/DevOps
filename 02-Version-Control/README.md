# Version Control with Git

## What is Version Control?

Version Control Systems (VCS) are tools that help manage changes to source code over time. They track modifications, allow collaboration, and enable reverting to previous versions.

## Why Version Control?

- **Track Changes**: See who changed what and when
- **Collaboration**: Multiple developers can work simultaneously
- **Backup**: Code is stored in multiple locations
- **Branching**: Experiment without affecting main code
- **History**: Complete audit trail of all changes
- **Revert**: Roll back to previous working versions

## Types of Version Control Systems

### 1. Local VCS
- Changes stored locally on one machine
- Example: RCS (Revision Control System)

### 2. Centralized VCS (CVCS)
- Single central server stores all versions
- Clients check out files from central place
- Examples: SVN, Perforce

### 3. Distributed VCS (DVCS)
- Every developer has complete repository
- Can work offline
- Examples: Git, Mercurial

## Git Fundamentals

Git is a distributed version control system created by Linus Torvalds in 2005.

### Key Concepts

#### Repository (Repo)
A repository is a directory that contains your project files and the entire revision history.

#### Commit
A snapshot of your repository at a specific point in time. Each commit has:
- Unique SHA-1 hash
- Author information
- Timestamp
- Commit message
- Parent commit(s)

#### Branch
A parallel version of the repository. The default branch is usually `main` or `master`.

#### Remote
A version of your repository hosted on a server (e.g., GitHub, GitLab).

#### Clone
Creating a local copy of a remote repository.

#### Fork
Creating a personal copy of someone else's repository.

## Git Workflow

### Three States of Git

1. **Modified**: File has been changed but not committed
2. **Staged**: Modified file marked to go into next commit
3. **Committed**: Data safely stored in local database

### Three Main Sections

1. **Working Directory**: Files you're currently working on
2. **Staging Area (Index)**: Files ready to be committed
3. **Git Directory (Repository)**: Where Git stores metadata and object database

## Essential Git Commands

### Initial Setup

```bash
# Configure username and email
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"

# View configuration
git config --list

# Set default editor
git config --global core.editor "vim"
```

### Creating Repositories

```bash
# Initialize a new repository
git init

# Clone an existing repository
git clone <repository-url>
git clone https://github.com/user/repo.git
```

### Basic Commands

```bash
# Check status of files
git status

# Add files to staging area
git add <file>
git add .                    # Add all files
git add *.js                 # Add all JavaScript files

# Commit changes
git commit -m "Commit message"
git commit -am "Message"     # Add and commit in one step

# View commit history
git log
git log --oneline
git log --graph --oneline --all
```

### Branching and Merging

```bash
# List branches
git branch
git branch -a                # List all branches (including remote)

# Create a new branch
git branch <branch-name>

# Switch to a branch
git checkout <branch-name>
git switch <branch-name>     # Modern alternative

# Create and switch to new branch
git checkout -b <branch-name>
git switch -c <branch-name>

# Merge branch into current branch
git merge <branch-name>

# Delete a branch
git branch -d <branch-name>
git branch -D <branch-name>  # Force delete
```

### Remote Repositories

```bash
# View remote repositories
git remote -v

# Add a remote
git remote add origin <url>

# Fetch changes from remote
git fetch origin

# Pull changes (fetch + merge)
git pull origin main

# Push changes to remote
git push origin main
git push -u origin main      # Set upstream and push

# Remove a remote
git remote remove origin
```

### Undoing Changes

```bash
# Discard changes in working directory
git checkout -- <file>
git restore <file>           # Modern alternative

# Unstage a file
git reset HEAD <file>
git restore --staged <file>  # Modern alternative

# Undo last commit (keep changes)
git reset --soft HEAD^

# Undo last commit (discard changes)
git reset --hard HEAD^

# Revert a commit (creates new commit)
git revert <commit-hash>
```

### Viewing Changes

```bash
# View unstaged changes
git diff

# View staged changes
git diff --staged
git diff --cached

# View changes between commits
git diff <commit1> <commit2>

# View changes in a file
git diff <file>
```

### Stashing

```bash
# Save changes temporarily
git stash
git stash save "Description"

# List stashes
git stash list

# Apply most recent stash
git stash apply

# Apply and remove stash
git stash pop

# Drop a stash
git stash drop
git stash drop stash@{0}
```

## Git Branching Strategies

### 1. Git Flow

```
main (production)
  └── develop
      ├── feature/feature-name
      ├── release/version
      └── hotfix/bug-fix
```

- **main**: Production-ready code
- **develop**: Integration branch for features
- **feature**: New features
- **release**: Preparing for production release
- **hotfix**: Emergency fixes for production

### 2. GitHub Flow

```
main
  ├── feature-1
  ├── feature-2
  └── bugfix
```

- Single main branch
- Create feature branches from main
- Open pull request for review
- Merge to main after approval
- Deploy from main

### 3. Trunk-Based Development

- Everyone commits to main (trunk)
- Short-lived feature branches
- Frequent integration
- Feature flags for incomplete features

## Best Practices

### Commit Messages

Follow the conventional commits format:

```
<type>(<scope>): <subject>

<body>

<footer>
```

Types:
- `feat`: New feature
- `fix`: Bug fix
- `docs`: Documentation changes
- `style`: Code style changes
- `refactor`: Code refactoring
- `test`: Adding tests
- `chore`: Maintenance tasks

Example:
```
feat(auth): add JWT authentication

Implement JWT-based authentication for API endpoints.
Includes token generation, validation, and refresh logic.

Closes #123
```

### General Best Practices

1. **Commit Often**: Make small, logical commits
2. **Write Good Messages**: Clear, concise, descriptive
3. **Use Branches**: Keep main branch stable
4. **Pull Before Push**: Stay updated with remote changes
5. **Review Before Commit**: Check what you're committing
6. **Use .gitignore**: Don't commit build artifacts, secrets
7. **Never Commit Secrets**: Use environment variables
8. **Atomic Commits**: One logical change per commit

## .gitignore File

Example `.gitignore`:

```gitignore
# Dependencies
node_modules/
vendor/

# Build outputs
dist/
build/
*.pyc
*.class

# IDE files
.vscode/
.idea/
*.swp

# OS files
.DS_Store
Thumbs.db

# Environment files
.env
.env.local

# Logs
*.log
logs/

# Temporary files
tmp/
temp/
```

## Git Tags

Tags mark specific points in history (usually releases).

```bash
# Create lightweight tag
git tag v1.0.0

# Create annotated tag
git tag -a v1.0.0 -m "Version 1.0.0"

# List tags
git tag
git tag -l "v1.*"

# Push tags to remote
git push origin v1.0.0
git push origin --tags

# Delete tag
git tag -d v1.0.0
git push origin --delete v1.0.0
```

## Advanced Git Concepts

### Rebasing

Reapply commits on top of another base:

```bash
# Rebase current branch onto main
git rebase main

# Interactive rebase (last 3 commits)
git rebase -i HEAD~3

# Continue after resolving conflicts
git rebase --continue

# Abort rebase
git rebase --abort
```

### Cherry-picking

Apply specific commits to current branch:

```bash
git cherry-pick <commit-hash>
```

### Submodules

Include other Git repositories within your repository:

```bash
# Add submodule
git submodule add <repository-url> <path>

# Clone repository with submodules
git clone --recursive <repository-url>

# Update submodules
git submodule update --init --recursive
```

## GitHub/GitLab Features

### Pull Requests (PR) / Merge Requests (MR)

- Propose changes to repository
- Code review mechanism
- Discussion and feedback
- CI/CD integration
- Approval workflows

### Issues

- Track bugs and features
- Assign to team members
- Labels and milestones
- Link to commits and PRs

### Actions/CI CD

- Automated workflows
- Build, test, deploy
- Triggered by events (push, PR, etc.)

## Git Aliases

Speed up common commands:

```bash
git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.ci commit
git config --global alias.st status
git config --global alias.lg "log --oneline --graph --all"
```

## Troubleshooting

### Common Issues

```bash
# Merge conflicts
# 1. Edit conflicted files
# 2. Remove conflict markers
# 3. git add <file>
# 4. git commit

# Detached HEAD state
git checkout main  # Return to branch

# Accidentally committed to wrong branch
git log  # Note commit hash
git checkout correct-branch
git cherry-pick <commit-hash>
git checkout wrong-branch
git reset --hard HEAD^

# Remove file from Git but keep locally
git rm --cached <file>
```

## Resources

- Official Git Documentation: https://git-scm.com/doc
- Pro Git Book (Free): https://git-scm.com/book
- GitHub Guides: https://guides.github.com
- Git Cheat Sheet: https://education.github.com/git-cheat-sheet-education.pdf

## Next Steps

Continue to [CI/CD](../03-CI-CD/) to learn about Continuous Integration and Continuous Deployment.
