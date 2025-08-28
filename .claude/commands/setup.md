# Project Setup Instructions

You are helping build an internal tool for CroMetrics. This setup will automatically create a complete project with GitHub repository.

## Prerequisites Check

First, ensure GitHub CLI is installed and authenticated:

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

## Automated Project Creation

### Project Naming
```bash
# Get the current folder name as the project name
PROJECT_NAME=$(basename "$PWD")
PROJECT_DESC="CroMetrics prototype: $PROJECT_NAME"

# The repository will use the current folder name
REPO_NAME="$PROJECT_NAME"

echo "Setting up project in current directory: $REPO_NAME"
```

## Project Structure

Create a monorepo in the current directory with the following structure:
```
./
├── backend/
│   ├── src/
│   │   ├── routes/
│   │   ├── controllers/
│   │   ├── services/
│   │   ├── types/
│   │   └── index.ts
│   ├── package.json
│   └── tsconfig.json
├── frontend/
│   ├── src/
│   │   ├── components/
│   │   ├── pages/
│   │   ├── services/
│   │   ├── types/
│   │   └── App.tsx
│   ├── package.json
│   ├── tsconfig.json
│   └── vite.config.ts
├── .husky/
│   ├── pre-commit
│   └── pre-push
├── .gitignore
├── .eslintrc.json
├── .prettierrc
├── commitlint.config.js
├── package.json (root)
└── README.md
```

### Create Directory Structure
```bash
# Create all directories
mkdir -p backend/src/{routes,controllers,services,types}
mkdir -p frontend/src/{components,pages,services,types}
mkdir -p .husky
```

## Technology Stack

### Backend
- **Runtime**: Node.js with TypeScript
- **Framework**: Express.js
- **Database ORM**: Prisma (if database is needed)
- **Validation**: Zod
- **CORS**: Enabled for local development
- **Port**: 3001

### Frontend  
- **Framework**: React 18 with TypeScript
- **Build Tool**: Vite
- **Styling**: Tailwind CSS
- **HTTP Client**: Axios
- **State Management**: React Context + useReducer (if needed)
- **Port**: 3000

## File Creation

### Backend Package.json (backend/package.json)
```json
{
  "name": "backend",
  "version": "1.0.0",
  "scripts": {
    "dev": "nodemon --exec ts-node src/index.ts",
    "build": "tsc",
    "start": "node dist/index.js"
  },
  "dependencies": {
    "express": "latest",
    "cors": "latest",
    "dotenv": "latest",
    "zod": "latest"
  },
  "devDependencies": {
    "@types/express": "latest",
    "@types/cors": "latest",
    "@types/node": "latest",
    "typescript": "latest",
    "nodemon": "latest",
    "ts-node": "latest"
  }
}
```

### Frontend Package.json (frontend/package.json)
```json
{
  "name": "frontend",
  "version": "1.0.0",
  "scripts": {
    "dev": "vite",
    "build": "tsc && vite build",
    "preview": "vite preview"
  },
  "dependencies": {
    "react": "latest",
    "react-dom": "latest",
    "axios": "latest"
  },
  "devDependencies": {
    "@types/react": "latest",
    "@types/react-dom": "latest",
    "@vitejs/plugin-react": "latest",
    "autoprefixer": "latest",
    "postcss": "latest",
    "tailwindcss": "latest",
    "typescript": "latest",
    "vite": "latest"
  }
}
```

### Root Package.json
```json
{
  "name": "project-root",
  "scripts": {
    "install:all": "npm install && cd backend && npm install && cd ../frontend && npm install",
    "dev": "concurrently \"npm run dev:backend\" \"npm run dev:frontend\"",
    "dev:backend": "cd backend && npm run dev",
    "dev:frontend": "cd frontend && npm run dev",
    "lint": "eslint . --ext .ts,.tsx,.js,.jsx",
    "lint:fix": "eslint . --ext .ts,.tsx,.js,.jsx --fix",
    "format": "prettier --write \"**/*.{ts,tsx,js,jsx,json,md}\"",
    "type-check": "tsc --noEmit && cd backend && tsc --noEmit && cd ../frontend && tsc --noEmit",
    "prepare": "husky install"
  },
  "devDependencies": {
    "concurrently": "latest",
    "husky": "latest",
    "lint-staged": "latest",
    "@commitlint/cli": "latest",
    "@commitlint/config-conventional": "latest",
    "eslint": "latest",
    "eslint-config-prettier": "latest",
    "eslint-plugin-react": "latest",
    "eslint-plugin-react-hooks": "latest",
    "@typescript-eslint/eslint-plugin": "latest",
    "@typescript-eslint/parser": "latest",
    "prettier": "latest"
  },
  "lint-staged": {
    "*.{ts,tsx,js,jsx}": [
      "eslint --fix",
      "prettier --write"
    ],
    "*.{json,md}": [
      "prettier --write"
    ]
  }
}
```

### Backend TypeScript Config (backend/tsconfig.json)
```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "commonjs",
    "lib": ["ES2022"],
    "outDir": "./dist",
    "rootDir": "./src",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "resolveJsonModule": true,
    "moduleResolution": "node"
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist"]
}
```

### Frontend TypeScript Config (frontend/tsconfig.json)
```json
{
  "compilerOptions": {
    "target": "ES2020",
    "lib": ["ES2020", "DOM", "DOM.Iterable"],
    "module": "ESNext",
    "skipLibCheck": true,
    "moduleResolution": "bundler",
    "allowImportingTsExtensions": true,
    "resolveJsonModule": true,
    "isolatedModules": true,
    "noEmit": true,
    "jsx": "react-jsx",
    "strict": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noFallthroughCasesInSwitch": true
  },
  "include": ["src"],
  "references": [{ "path": "./tsconfig.node.json" }]
}
```

### Frontend tsconfig.node.json
```json
{
  "compilerOptions": {
    "composite": true,
    "skipLibCheck": true,
    "module": "ESNext",
    "moduleResolution": "bundler",
    "allowSyntheticDefaultImports": true
  },
  "include": ["vite.config.ts"]
}
```

### ESLint Config (.eslintrc.json)
```json
{
  "extends": [
    "eslint:recommended",
    "plugin:@typescript-eslint/recommended",
    "plugin:react/recommended",
    "plugin:react-hooks/recommended",
    "prettier"
  ],
  "parser": "@typescript-eslint/parser",
  "plugins": ["@typescript-eslint", "react", "react-hooks"],
  "rules": {
    "react/react-in-jsx-scope": "off",
    "react/prop-types": "off",
    "@typescript-eslint/explicit-module-boundary-types": "off",
    "@typescript-eslint/no-unused-vars": ["error", { "argsIgnorePattern": "^_" }]
  },
  "settings": {
    "react": {
      "version": "detect"
    }
  }
}
```

### Prettier Config (.prettierrc)
```json
{
  "semi": true,
  "trailingComma": "es5",
  "singleQuote": true,
  "printWidth": 100,
  "tabWidth": 2,
  "useTabs": false,
  "bracketSpacing": true,
  "arrowParens": "always",
  "endOfLine": "lf"
}
```

### Commitlint Config (commitlint.config.js)
```javascript
module.exports = {
  extends: ['@commitlint/config-conventional'],
  rules: {
    'type-enum': [
      2,
      'always',
      ['feat', 'fix', 'docs', 'style', 'refactor', 'test', 'chore', 'perf']
    ]
  }
};
```

### Tailwind Config (frontend/tailwind.config.js)
```javascript
module.exports = {
  content: [
    "./index.html",
    "./src/**/*.{js,ts,jsx,tsx}",
  ],
  theme: {
    extend: {},
  },
  plugins: [],
}
```

### Tailwind CSS (frontend/src/index.css)
```css
@tailwind base;
@tailwind components;
@tailwind utilities;
```

### PostCSS Config (frontend/postcss.config.js)
```javascript
module.exports = {
  plugins: {
    tailwindcss: {},
    autoprefixer: {},
  },
}
```

### Vite Config (frontend/vite.config.ts)
```typescript
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'

export default defineConfig({
  plugins: [react()],
  server: {
    port: 3000,
    proxy: {
      '/api': {
        target: 'http://localhost:3001',
        changeOrigin: true,
      }
    }
  }
})
```

### Frontend index.html
```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>CroMetrics Prototype</title>
  </head>
  <body>
    <div id="root"></div>
    <script type="module" src="/src/main.tsx"></script>
  </body>
</html>
```

### Frontend main.tsx (frontend/src/main.tsx)
```typescript
import React from 'react'
import ReactDOM from 'react-dom/client'
import App from './App'
import './index.css'

ReactDOM.createRoot(document.getElementById('root')!).render(
  <React.StrictMode>
    <App />
  </React.StrictMode>,
)
```

### Git Ignore (.gitignore)
```
# Dependencies
node_modules/
.pnp
.pnp.js

# Production
dist/
build/

# Environment
.env
.env.local
.env.production.local
.env.development.local
.env.test.local

# Logs
npm-debug.log*
yarn-debug.log*
yarn-error.log*
lerna-debug.log*
*.log

# OS Files
.DS_Store
Thumbs.db

# IDE
.vscode/
.idea/
*.swp
*.swo
*.swn
*.bak

# Testing
coverage/
.nyc_output/

# TypeScript
*.tsbuildinfo

# Optional npm cache
.npm

# Optional eslint cache
.eslintcache

# Misc
*.pid
*.seed
*.pid.lock
```

### Husky Pre-commit Hook (.husky/pre-commit)
```bash
#!/usr/bin/env sh
. "$(dirname -- "$0")/_/husky.sh"

npx lint-staged
```

### Husky Pre-push Hook (.husky/pre-push)
```bash
#!/usr/bin/env sh
. "$(dirname -- "$0")/_/husky.sh"

npm run type-check
```

## Environment Variables

### Backend (.env)
```
PORT=3001
NODE_ENV=development
DATABASE_URL=your_database_url_here
```

### Frontend (.env)
```
VITE_API_URL=http://localhost:3001
```

## Starter Code Templates

### Backend Express Server (backend/src/index.ts)
```typescript
import express from 'express';
import cors from 'cors';
import dotenv from 'dotenv';

dotenv.config();

const app = express();
const PORT = process.env.PORT || 3001;

app.use(cors());
app.use(express.json());

// Health check endpoint
app.get('/api/health', (req, res) => {
  res.json({ status: 'ok', timestamp: new Date().toISOString() });
});

// Add routes here

app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});
```

### Frontend App Component (frontend/src/App.tsx)
```tsx
import React, { useEffect, useState } from 'react';
import axios from 'axios';

function App() {
  const [health, setHealth] = useState<string>('');

  useEffect(() => {
    axios.get('/api/health')
      .then(response => setHealth(response.data.status))
      .catch(error => console.error('Error:', error));
  }, []);

  return (
    <div className="min-h-screen bg-gray-100">
      <div className="container mx-auto px-4 py-8">
        <h1 className="text-3xl font-bold text-gray-900">
          CroMetrics Internal Tool
        </h1>
        <p className="mt-4 text-gray-600">
          API Status: {health || 'Checking...'}
        </p>
      </div>
    </div>
  );
}

export default App;
```

## UI/UX Requirements

1. **Design System**
   - Use Tailwind CSS utility classes
   - Primary color: Blue (bg-blue-500, text-blue-600, etc.)
   - Maintain consistent spacing (use Tailwind's spacing scale)
   - All buttons should have hover states
   - Include loading states for all async operations

2. **Layout**
   - Responsive design (mobile-first)
   - Maximum content width: 1280px (max-w-7xl)
   - Consistent padding: px-4 on mobile, px-8 on desktop
   - Navigation bar at top if multiple pages

3. **Components**
   - Cards for displaying data groups
   - Tables must be responsive (overflow-x-auto on mobile)
   - Forms should have proper validation and error states
   - Use modals sparingly, prefer inline editing

## API Design Patterns

1. **RESTful Endpoints**
   ```
   GET    /api/resources     - List all
   GET    /api/resources/:id - Get one
   POST   /api/resources     - Create
   PUT    /api/resources/:id - Update
   DELETE /api/resources/:id - Delete
   ```

2. **Response Format**
   ```typescript
   // Success
   {
     "success": true,
     "data": { ... }
   }
   
   // Error
   {
     "success": false,
     "error": "Error message"
   }
   ```

3. **Error Handling**
   - Always use try-catch blocks
   - Return appropriate HTTP status codes
   - Log errors to console in development

## Database Patterns (if needed)

If the project requires a database:
1. Use Prisma as the ORM
2. Include migrations in the project
3. Add seed data for development
4. Use environment variables for connection strings

## Automated Git and GitHub Setup

### Initialize Git and Create GitHub Repository
```bash
# Initialize git with main branch
git init
git branch -M main

# Create initial .gitignore if not exists
if [ ! -f .gitignore ]; then
  echo "Creating .gitignore..."
  # .gitignore content created above
fi

# Add all files and create initial commit
git add .
git commit -m "feat: initial project setup with Node/React/TypeScript"

# Create GitHub repository using GitHub CLI
echo "Creating GitHub repository..."

# Check if gh is authenticated
if ! gh auth status &> /dev/null; then
  echo "⚠️ GitHub CLI not authenticated. Running authentication..."
  gh auth login
fi

# Create the repository on GitHub
gh repo create "CROmetrics/$REPO_NAME" \
  --public \
  --description "$PROJECT_DESC" \
  --source=. \
  --remote=origin \
  --push

# Check if repository creation was successful
if [ $? -eq 0 ]; then
  echo "✅ GitHub repository created successfully: https://github.com/CROmetrics/$REPO_NAME"
  
  # Create and switch to develop branch
  git checkout -b develop
  git push -u origin develop
  
  echo "✅ Develop branch created and pushed"
  echo "📍 Current branch: develop (all development work happens here)"
else
  echo "⚠️ Failed to create GitHub repository automatically"
  echo "Please create it manually at: https://github.com/CROmetrics/new"
  echo "Repository name: $REPO_NAME"
  echo "Then run:"
  echo "  git remote add origin https://github.com/CROmetrics/$REPO_NAME.git"
  echo "  git push -u origin main"
  echo "  git checkout -b develop"
  echo "  git push -u origin develop"
fi
```

## Branch Strategy

**IMPORTANT: Never push directly to main during development**

All development work happens on the `develop` branch. Only push to `main` when explicitly instructed with phrases like:
- "Push to production"
- "Deploy to main"
- "This is ready for main"
- "Merge to main branch"

### Branch Naming Conventions
- `feature/` - New features (e.g., feature/user-dashboard)
- `fix/` - Bug fixes (e.g., fix/login-validation)
- `chore/` - Maintenance tasks (e.g., chore/update-dependencies)
- `refactor/` - Code refactoring (e.g., refactor/api-service)

### Commit Message Format
Follow conventional commits:
- `feat:` - New features
- `fix:` - Bug fixes
- `docs:` - Documentation changes
- `style:` - Formatting changes
- `refactor:` - Code refactoring
- `test:` - Adding tests
- `chore:` - Maintenance tasks
- `perf:` - Performance improvements

## Complete Automated Setup Script

Run this complete setup sequence:

```bash
#!/bin/bash

# Use current directory name as project name
PROJECT_NAME=$(basename "$PWD")
PROJECT_DESC="CroMetrics prototype: $PROJECT_NAME"
REPO_NAME="$PROJECT_NAME"

echo "🚀 Setting up project: $REPO_NAME in current directory"

# Create all directories in current folder
echo "📁 Creating directory structure..."
mkdir -p backend/src/{routes,controllers,services,types}
mkdir -p frontend/src/{components,pages,services,types}
mkdir -p frontend/public
mkdir -p .husky

# Create all configuration files
echo "📝 Creating configuration files..."
# [All file creation commands go here - using cat or echo to create each file]

# Initialize git
echo "🔧 Initializing Git..."
git init
git branch -M main

# Initial commit
git add .
git commit -m "feat: initial project setup with Node/React/TypeScript"

# Create GitHub repository
echo "🌐 Creating GitHub repository..."
if command -v gh &> /dev/null; then
  if gh auth status &> /dev/null; then
    gh repo create "CROmetrics/$REPO_NAME" \
      --public \
      --description "$PROJECT_DESC" \
      --source=. \
      --remote=origin \
      --push
    
    if [ $? -eq 0 ]; then
      echo "✅ Repository created: https://github.com/CROmetrics/$REPO_NAME"
      
      # Switch to develop branch
      git checkout -b develop
      git push -u origin develop
      
      echo "✅ Switched to develop branch"
    fi
  else
    echo "⚠️ Please authenticate GitHub CLI first: gh auth login"
    exit 1
  fi
else
  echo "⚠️ GitHub CLI not found. Please install it first:"
  echo "   Mac: brew install gh"
  echo "   Windows: winget install --id GitHub.cli"
  echo "   Linux: See https://cli.github.com/manual/installation"
  exit 1
fi

# Install dependencies
echo "📦 Installing dependencies..."
npm run install:all

# Setup git hooks
echo "🪝 Setting up git hooks..."
npm run prepare

# Make hooks executable
chmod +x .husky/pre-commit
chmod +x .husky/pre-push

# Final status
echo ""
echo "✅ Project setup complete!"
echo "📁 Project location: $(pwd)"
echo "🌐 GitHub repository: https://github.com/CROmetrics/$REPO_NAME"
echo "🔧 Current branch: develop"
echo ""
echo "To start development:"
echo "  npm run dev"
echo ""
echo "Frontend: http://localhost:3000"
echo "Backend:  http://localhost:3001"
```

## Quick Start Summary

After running this setup, the project will be:
1. ✅ Fully structured with all files and folders
2. ✅ Git initialized with proper branching
3. ✅ GitHub repository created automatically
4. ✅ All dependencies installed
5. ✅ Git hooks configured for code quality
6. ✅ Ready to start development on `develop` branch

Simply run `npm run dev` to start both frontend and backend servers.