# Git Repository Setup

This command sets up Git configuration and GitHub repository for a CroMetrics project that already has the Node.js infrastructure in place.

## Prerequisites

### GitHub CLI Installation

```bash
# Check if GitHub CLI is installed
if command -v gh &> /dev/null; then
  echo "✓ GitHub CLI is installed"
  # Check if authenticated
  if gh auth status &> /dev/null; then
    echo "✓ GitHub CLI is authenticated"
  else
    echo "⚠️ Setting up GitHub authentication..."
    gh auth login
  fi
else
  echo "⚠️ Installing GitHub CLI..."
  # Mac
  if [[ "$OSTYPE" == "darwin"* ]]; then
    brew install gh
  # Windows (if using WSL or Git Bash)
  elif [[ "$OSTYPE" == "msys" ]] || [[ "$OSTYPE" == "cygwin" ]]; then
    winget install --id GitHub.cli
  # Linux
  else
    curl -fsSL https://cli.github.com/packages/githubcli-archive-keyring.gpg | sudo dd of=/usr/share/keyrings/githubcli-archive-keyring.gpg
    echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/githubcli-archive-keyring.gpg] https://cli.github.com/packages stable main" | sudo tee /etc/apt/sources.list.d/github-cli.list > /dev/null
    sudo apt update
    sudo apt install gh
  fi
  
  # Authenticate after installation
  gh auth login
fi
```

## Complete Git Setup Script

```bash
#!/bin/bash

# Colors for output
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
NC='\033[0m' # No Color

echo -e "${GREEN}🚀 CroMetrics Git Repository Setup${NC}"
echo "=================================="

# Function to check and install GitHub CLI
check_and_install_gh() {
  if command -v gh &> /dev/null; then
    echo -e "${GREEN}✓ GitHub CLI is installed${NC}"
    # Check if authenticated
    if gh auth status &> /dev/null; then
      echo -e "${GREEN}✓ GitHub CLI is authenticated${NC}"
      return 0
    else
      echo -e "${YELLOW}⚠️ Setting up GitHub authentication...${NC}"
      gh auth login
      return $?
    fi
  else
    echo -e "${YELLOW}⚠️ GitHub CLI not found. Installing...${NC}"
    
    # Install based on OS
    if [[ "$OSTYPE" == "darwin"* ]]; then
      if command -v brew &> /dev/null; then
        brew install gh
      else
        echo -e "${RED}Homebrew not found. Please install GitHub CLI manually.${NC}"
        return 1
      fi
    elif [[ "$OSTYPE" == "linux-gnu"* ]]; then
      if [ "$EUID" -ne 0 ]; then
        curl -fsSL https://cli.github.com/packages/githubcli-archive-keyring.gpg | sudo dd of=/usr/share/keyrings/githubcli-archive-keyring.gpg
        echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/githubcli-archive-keyring.gpg] https://cli.github.com/packages stable main" | sudo tee /etc/apt/sources.list.d/github-cli.list > /dev/null
        sudo apt update
        sudo apt install gh -y
      else
        curl -fsSL https://cli.github.com/packages/githubcli-archive-keyring.gpg | dd of=/usr/share/keyrings/githubcli-archive-keyring.gpg
        echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/githubcli-archive-keyring.gpg] https://cli.github.com/packages stable main" | tee /etc/apt/sources.list.d/github-cli.list > /dev/null
        apt update
        apt install gh -y
      fi
    elif [[ "$OSTYPE" == "msys" ]] || [[ "$OSTYPE" == "cygwin" ]]; then
      if command -v winget &> /dev/null; then
        winget install --id GitHub.cli
      elif command -v choco &> /dev/null; then
        choco install gh
      else
        echo -e "${RED}Please install GitHub CLI manually from: https://cli.github.com/${NC}"
        return 1
      fi
    fi
    
    # Authenticate after installation
    if command -v gh &> /dev/null; then
      echo -e "${YELLOW}Setting up GitHub authentication...${NC}"
      gh auth login
      return $?
    else
      echo -e "${RED}Failed to install GitHub CLI${NC}"
      return 1
    fi
  fi
}

# Function to check Git
check_git() {
  if command -v git &> /dev/null; then
    echo -e "${GREEN}✓ Git v$(git --version | cut -d' ' -f3) is installed${NC}"
    return 0
  else
    echo -e "${YELLOW}⚠️ Git is not installed. Installing...${NC}"
    
    if [[ "$OSTYPE" == "darwin"* ]]; then
      brew install git
    elif [[ "$OSTYPE" == "linux-gnu"* ]]; then
      if [ "$EUID" -ne 0 ]; then
        sudo apt-get update && sudo apt-get install git -y
      else
        apt-get update && apt-get install git -y
      fi
    elif [[ "$OSTYPE" == "msys" ]] || [[ "$OSTYPE" == "cygwin" ]]; then
      if command -v winget &> /dev/null; then
        winget install --id Git.Git
      else
        echo -e "${RED}Please install Git manually from: https://git-scm.com/${NC}"
        return 1
      fi
    fi
    
    if command -v git &> /dev/null; then
      echo -e "${GREEN}✅ Git installed successfully${NC}"
      return 0
    else
      echo -e "${RED}Failed to install Git${NC}"
      return 1
    fi
  fi
}

# =======================
# MAIN SETUP STARTS HERE
# =======================

# Use current directory name as project name
PROJECT_NAME=$(basename "$PWD")
PROJECT_DESC="CroMetrics prototype: $PROJECT_NAME"
REPO_NAME="$PROJECT_NAME"

echo ""
echo "Step 1: Checking prerequisites..."
echo "---------------------------------"

# Check Git
if ! check_git; then
  echo -e "${RED}Cannot proceed without Git. Exiting.${NC}"
  exit 1
fi

# Check GitHub CLI
if ! check_and_install_gh; then
  echo -e "${YELLOW}⚠️ GitHub CLI setup failed. You'll need to manually push to GitHub later.${NC}"
  GH_AVAILABLE=false
else
  GH_AVAILABLE=true
fi

# Check if git repo already initialized
if [ -d .git ]; then
  echo -e "${YELLOW}⚠️ Git repository already initialized${NC}"
  
  # Check current branch
  CURRENT_BRANCH=$(git branch --show-current)
  echo "Current branch: $CURRENT_BRANCH"
  
  # Check for uncommitted changes
  if ! git diff --quiet || ! git diff --staged --quiet; then
    echo -e "${YELLOW}⚠️ You have uncommitted changes. Committing them now...${NC}"
    git add .
    git commit -m "chore: save current work before GitHub setup"
  fi
  
  # Check if remote exists
  if git remote get-url origin &> /dev/null; then
    echo -e "${YELLOW}⚠️ Remote 'origin' already exists${NC}"
    REMOTE_URL=$(git remote get-url origin)
    echo "Remote URL: $REMOTE_URL"
    
    read -p "Do you want to replace the existing remote? (y/n) " -n 1 -r
    echo
    if [[ $REPLY =~ ^[Yy]$ ]]; then
      git remote remove origin
      REMOTE_EXISTS=false
    else
      REMOTE_EXISTS=true
    fi
  else
    REMOTE_EXISTS=false
  fi
else
  # Initialize git
  echo ""
  echo -e "${GREEN}Step 2: Initializing Git repository${NC}"
  echo "---------------------------------"
  git init
  git branch -M main
  
  # Initial commit
  git add .
  git commit -m "feat: initial project setup with Node/React/TypeScript"
  
  REMOTE_EXISTS=false
  CURRENT_BRANCH="main"
fi

# Create GitHub repository if gh is available and no remote exists
if [ "$GH_AVAILABLE" = true ] && [ "$REMOTE_EXISTS" = false ]; then
  echo ""
  echo -e "${GREEN}Step 3: Creating GitHub repository${NC}"
  echo "---------------------------------"
  
  echo "Project name: $REPO_NAME"
  echo "Description: $PROJECT_DESC"
  echo ""
  
  read -p "Do you want to create a GitHub repository for this project? (y/n) " -n 1 -r
  echo
  
  if [[ $REPLY =~ ^[Yy]$ ]]; then
    gh repo create "CROmetrics/$REPO_NAME" \
      --public \
      --description "$PROJECT_DESC" \
      --source=. \
      --remote=origin \
      --push
    
    if [ $? -eq 0 ]; then
      echo -e "${GREEN}✅ Repository created: https://github.com/CROmetrics/$REPO_NAME${NC}"
      
      # Switch to develop branch if not already there
      if [ "$CURRENT_BRANCH" != "develop" ]; then
        git checkout -b develop
        git push -u origin develop
        echo -e "${GREEN}✅ Switched to develop branch${NC}"
      fi
      
      GITHUB_CREATED=true
    else
      echo -e "${YELLOW}⚠️ Failed to create GitHub repository automatically${NC}"
      GITHUB_CREATED=false
    fi
  else
    echo -e "${YELLOW}Skipping GitHub repository creation${NC}"
    GITHUB_CREATED=false
  fi
elif [ "$REMOTE_EXISTS" = true ]; then
  echo ""
  echo -e "${YELLOW}Step 3: GitHub repository already configured${NC}"
  GITHUB_CREATED=true
else
  echo ""
  echo -e "${YELLOW}Step 3: Skipping GitHub repository creation (CLI not available)${NC}"
  GITHUB_CREATED=false
fi

# Final status
echo ""
echo "======================================"
echo -e "${GREEN}✅ Git setup complete!${NC}"
echo "======================================"
echo ""

if [ "$GITHUB_CREATED" = true ]; then
  echo "🌐 GitHub repository: https://github.com/CROmetrics/$REPO_NAME"
  echo "🔧 Current branch: $(git branch --show-current)"
  echo ""
  echo "Git workflow reminders:"
  echo "  • Always work on 'develop' branch"
  echo "  • Only merge to 'main' when ready for production"
  echo "  • Use conventional commit messages (feat:, fix:, etc.)"
else
  echo -e "${YELLOW}⚠️ GitHub repository not created. To create manually:${NC}"
  echo "   1. Create repository at: https://github.com/CROmetrics/new"
  echo "   2. Repository name: $REPO_NAME"
  echo "   3. Then run:"
  echo "      git remote add origin https://github.com/CROmetrics/$REPO_NAME.git"
  echo "      git push -u origin main"
  echo "      git checkout -b develop"
  echo "      git push -u origin develop"
fi

echo ""
echo "Happy coding! 🚀"
```

## Quick Setup Command

For projects that already have the Node.js infrastructure:

```bash
# Save and run the script
curl -o setup-git.sh https://raw.githubusercontent.com/CROmetrics/setup-scripts/main/setup-git.sh && chmod +x setup-git.sh && ./setup-git.sh
```

## Manual GitHub Setup

If the automated setup fails, follow these steps:

1. **Create repository on GitHub**:
   - Go to: https://github.com/CROmetrics/new
   - Repository name: Use your project folder name
   - Make it public
   - Don't initialize with README, .gitignore, or license

2. **Connect local repository to GitHub**:
   ```bash
   # Add remote
   git remote add origin https://github.com/CROmetrics/YOUR_REPO_NAME.git
   
   # Push main branch
   git push -u origin main
   
   # Create and switch to develop branch
   git checkout -b develop
   git push -u origin develop
   ```

## GitHub CLI Manual Installation

### macOS
```bash
brew install gh
```

### Windows
```bash
winget install --id GitHub.cli
# or
choco install gh
```

### Linux (Ubuntu/Debian)
```bash
curl -fsSL https://cli.github.com/packages/githubcli-archive-keyring.gpg | sudo dd of=/usr/share/keyrings/githubcli-archive-keyring.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/githubcli-archive-keyring.gpg] https://cli.github.com/packages stable main" | sudo tee /etc/apt/sources.list.d/github-cli.list > /dev/null
sudo apt update
sudo apt install gh
```

## Authentication

After installing GitHub CLI, authenticate with:

```bash
gh auth login
```

Choose:
- GitHub.com
- HTTPS protocol
- Authenticate with browser or token
- Follow the prompts

## Troubleshooting

### GitHub CLI Authentication Issues

If authentication fails:

```bash
# Try with specific protocol
gh auth login --protocol https

# Or use personal access token
gh auth login --with-token < token.txt
```

### Permission Denied

If you get permission denied when pushing:

```bash
# Check your authentication status
gh auth status

# Re-authenticate if needed
gh auth refresh
```

### Repository Already Exists

If the repository name is taken:
1. Choose a different name
2. Or delete the existing repository (if you own it)
3. Update the script with the new name