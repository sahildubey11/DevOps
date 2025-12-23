# Version Control with Git

## What is Git?

Git is a distributed version control system that tracks changes in source code during software development. It enables multiple developers to work together on non-linear development.

## Why Git?

- **Distributed Development**: Every developer has a full copy of the repository
- **Branching & Merging**: Create isolated branches for features
- **Fast Performance**: Most operations are local
- **Data Integrity**: Everything is checksummed
- **Staging Area**: Control what goes into a commit

## Installation

### Linux
```bash
sudo apt-get update
sudo apt-get install git
```

### macOS
```bash
brew install git
```

### Windows
Download from [git-scm.com](https://git-scm.com/download/win)

## Basic Configuration

```bash
# Set your identity
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"

# Set default editor
git config --global core.editor "vim"

# Check your settings
git config --list
```

## Essential Git Commands

### Repository Initialization
```bash
# Initialize a new repository
git init

# Clone an existing repository
git clone <repository-url>
```

### Basic Workflow
```bash
# Check status
git status

# Add files to staging area
git add <file>
git add .  # Add all files

# Commit changes
git commit -m "Commit message"

# View commit history
git log
git log --oneline
git log --graph --oneline --all
```

### Branching
```bash
# List branches
git branch

# Create a new branch
git branch <branch-name>

# Switch to a branch
git checkout <branch-name>

# Create and switch to a new branch
git checkout -b <branch-name>

# Delete a branch
git branch -d <branch-name>

# Merge a branch
git merge <branch-name>
```

### Remote Operations
```bash
# Add remote repository
git remote add origin <repository-url>

# View remotes
git remote -v

# Push changes
git push origin <branch-name>

# Pull changes
git pull origin <branch-name>

# Fetch changes
git fetch origin
```

### Undoing Changes
```bash
# Discard changes in working directory
git checkout -- <file>

# Unstage a file
git reset HEAD <file>

# Amend last commit
git commit --amend

# Reset to previous commit (dangerous!)
git reset --hard HEAD~1

# Revert a commit (safe)
git revert <commit-hash>
```

## Git Workflow Strategies

### 1. Centralized Workflow
- Single main branch
- All commits go to main
- Simple but not recommended for teams

### 2. Feature Branch Workflow
- Create feature branches from main
- Merge back to main when complete
- Better isolation of features

### 3. Gitflow Workflow
- Main branches: main, develop
- Supporting branches: feature, release, hotfix
- Structured release management

### 4. Forking Workflow
- Each developer has their own server-side repository
- Common in open source projects
- Pull requests for contributions

## Advanced Git Concepts

### Stashing
```bash
# Save changes temporarily
git stash

# List stashes
git stash list

# Apply stash
git stash apply

# Apply and remove stash
git stash pop
```

### Rebasing
```bash
# Rebase current branch onto another
git rebase <branch-name>

# Interactive rebase
git rebase -i HEAD~3
```

### Cherry-picking
```bash
# Apply a specific commit to current branch
git cherry-pick <commit-hash>
```

### Tags
```bash
# Create a tag
git tag v1.0.0

# Create annotated tag
git tag -a v1.0.0 -m "Version 1.0.0"

# Push tags
git push origin --tags
```

## .gitignore

Create a `.gitignore` file to exclude files from version control:

```
# Dependencies
node_modules/
vendor/

# Build outputs
dist/
build/
*.exe

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
```

## Best Practices

1. **Commit Often**: Make small, logical commits
2. **Write Meaningful Commit Messages**: Describe what and why
3. **Use Branches**: Isolate features and fixes
4. **Review Before Committing**: Use `git diff` and `git status`
5. **Don't Commit Sensitive Data**: Use .gitignore
6. **Pull Before Push**: Stay synchronized with remote
7. **Use Pull Requests**: Enable code review

## Commit Message Conventions

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
feat(auth): add user login functionality

Implement JWT-based authentication for user login.
Added login form and API endpoint.

Closes #123
```

## Common Issues and Solutions

### Merge Conflicts
```bash
# When you see merge conflict
git status

# Edit files to resolve conflicts
# Look for <<<<<<< HEAD markers

# After resolving
git add <resolved-files>
git commit
```

### Undo Last Push (with caution)
```bash
# Reset to previous commit
git reset --hard HEAD~1

# Force push (dangerous!)
git push -f origin <branch-name>
```

### Remove File from Git but Keep Locally
```bash
git rm --cached <file>
```

## Git Aliases

Make Git easier with aliases:

```bash
git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.ci commit
git config --global alias.st status
git config --global alias.unstage 'reset HEAD --'
git config --global alias.last 'log -1 HEAD'
git config --global alias.visual 'log --graph --oneline --all'
```

## Interview Questions

1. What is the difference between `git pull` and `git fetch`?
2. Explain the difference between `git merge` and `git rebase`
3. What is a merge conflict and how do you resolve it?
4. What is the difference between `git reset` and `git revert`?
5. How do you undo the last commit?
6. What is git stash and when would you use it?
7. Explain the Gitflow workflow
8. What is a detached HEAD state?
9. How do you remove a file from Git without deleting it locally?
10. What is the difference between a lightweight tag and an annotated tag?

## Practice Exercises

1. Create a repository and make 5 commits
2. Create a feature branch, make changes, and merge it
3. Practice resolving merge conflicts
4. Use git stash to save and restore work
5. Practice interactive rebase to squash commits
6. Set up git aliases for common commands
7. Create a .gitignore file for your project type

## Resources

- [Official Git Documentation](https://git-scm.com/doc)
- [Pro Git Book](https://git-scm.com/book/en/v2)
- [Git Cheat Sheet](https://education.github.com/git-cheat-sheet-education.pdf)
- [Interactive Git Tutorial](https://learngitbranching.js.org/)
- [GitHub Flow Guide](https://guides.github.com/introduction/flow/)
