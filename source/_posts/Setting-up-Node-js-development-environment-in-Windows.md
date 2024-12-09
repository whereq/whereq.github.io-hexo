---
title: Setting up Node.js development environment in Windows
date: 2024-12-04 19:19:43
categories:
- Node
- Type Script
tags:
- Node
- Type Script
---

## **1. Install Node.js**
1. Download Node.js from the [official website](https://nodejs.org/). Use the **LTS version** for stability.
2. Install Node.js by running the installer and ensure you check the box to install **npm** (Node Package Manager).
3. Verify the installation:
   ```bash
   node -v
   npm -v
   ```

---

## **2. Install Yarn (Optional)**
While npm is included with Node.js, Yarn is a popular alternative for managing dependencies.
1. Install Yarn globally:
   ```bash
   npm install --global yarn
   ```
2. Verify the installation:
   ```bash
   yarn -v
   ```

---

## **3. Install Git**
1. Download and install Git from [git-scm.com](https://git-scm.com/).
2. During installation, choose "Git Bash Here" and "Use Git from the Command Prompt."
3. Verify Git installation:
   ```bash
   git --version
   ```

---

## **4. Install Visual Studio Code**
1. Download and install [VS Code](https://code.visualstudio.com/).
2. Install the following extensions in VS Code:
   - **ESLint**: Linting and code quality.
   - **Prettier**: Code formatting.
   - **TypeScript**: Provides TypeScript language features.
   - **React Developer Tools**: Debugging React apps.
   - **REST Client**: For testing APIs.

---

## **5. Setup Node.js + TypeScript Project**
### **Step 1: Initialize the Project**
1. Create a new folder for your project:
   ```bash
   mkdir my-node-react-app
   cd my-node-react-app
   ```
2. Initialize the project with Yarn or npm:
   ```bash
   yarn init -y
   # OR
   npm init -y
   ```

---

### **Step 2: Install TypeScript**
1. Install TypeScript and Node.js types as dev dependencies:
   ```bash
   yarn add -D typescript @types/node
   ```
2. Create a `tsconfig.json`:
   ```bash
   npx tsc --init
   ```

**Example `tsconfig.json`:**
```json
{
  "compilerOptions": {
    "target": "ESNext",
    "module": "ESNext",
    "lib": ["ESNext", "DOM"],
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "moduleResolution": "Node",
    "resolveJsonModule": true,
    "outDir": "./dist",
    "rootDir": "./src",
    "baseUrl": "./",
    "paths": {
      "@/*": ["src/*"]
    }
  },
  "include": ["src"],
  "exclude": ["node_modules", "dist"]
}
```

---

### **Step 3: Setup Front-End (React + Vite)**
1. Create the React front-end:
   ```bash
   yarn create vite react-frontend --template react-ts
   ```
2. Navigate to the front-end folder:
   ```bash
   cd react-frontend
   ```
3. Install dependencies for React development:
   ```bash
   yarn add react react-dom
   yarn add -D @types/react @types/react-dom eslint prettier
   ```
4. Install additional libraries:
   ```bash
   yarn add axios zustand zod clsx @tanstack/react-query
   ```

---

### **Step 4: Setup Back-End (Node.js)**
1. Navigate back to the root directory.
2. Create a folder for the back-end:
   ```bash
   mkdir backend
   cd backend
   ```
3. Install Express and related types:
   ```bash
   yarn add express
   yarn add -D @types/express nodemon
   ```
4. Create a `src` folder and start building your API:
   ```bash
   mkdir src
   touch src/index.ts
   ```

**Example `src/index.ts`:**
```typescript
import express from 'express';

const app = express();
app.use(express.json());

app.get('/api', (req, res) => {
  res.send('Hello from the back-end!');
});

const PORT = 3001;
app.listen(PORT, () => {
  console.log(`Server is running on http://localhost:${PORT}`);
});
```

5. Add a `nodemon` script to automatically restart the server during development:
   ```json
   "scripts": {
     "dev": "nodemon src/index.ts"
   }
   ```

---

### **Step 5: Configure Path Aliases**
1. Add the following to both `react-frontend` and `backend` projects' `tsconfig.json`:
   ```json
   "paths": {
     "@/*": ["src/*"]
   }
   ```

2. In Vite's `vite.config.ts` for the front-end, install and configure `vite-tsconfig-paths`:
   ```bash
   yarn add -D vite-tsconfig-paths
   ```

   **vite.config.ts:**
   ```typescript
   import { defineConfig } from 'vite';
   import react from '@vitejs/plugin-react';
   import tsconfigPaths from 'vite-tsconfig-paths';

   export default defineConfig({
     plugins: [react(), tsconfigPaths()],
   });
   ```

---

### **Step 6: Setup Docker**
1. Create a `Dockerfile` in the root directory for both front-end and back-end.

**Front-End `Dockerfile`:**
```dockerfile
FROM node:18-alpine

WORKDIR /app
COPY package.json yarn.lock ./
RUN yarn install
COPY . .
CMD ["yarn", "dev"]
```

**Back-End `Dockerfile`:**
```dockerfile
FROM node:18-alpine

WORKDIR /app
COPY package.json yarn.lock ./
RUN yarn install
COPY . .
CMD ["yarn", "dev"]
```

2. Create a `docker-compose.yml`:
```yaml
version: '3.8'
services:
  frontend:
    build:
      context: ./react-frontend
    ports:
      - "5173:5173"
  backend:
    build:
      context: ./backend
    ports:
      - "3001:3001"
```

---

### **Step 7: Debugging in VS Code**
1. Install the **Remote - Containers** extension in VS Code.
2. Create a `.devcontainer` folder in the root directory.
3. Add a `devcontainer.json`:
   ```json
   {
     "name": "Node.js Dev",
     "dockerComposeFile": "docker-compose.yml",
     "service": "frontend",
     "workspaceFolder": "/app",
     "extensions": [
       "dbaeumer.vscode-eslint",
       "esbenp.prettier-vscode"
     ],
     "settings": {
       "terminal.integrated.shell.linux": "/bin/bash"
     }
   }
   ```

4. Reopen the project in the container using the Command Palette (`Ctrl + Shift + P`).

---

### **8. Start Development**
1. Start the Docker containers:
   ```bash
   docker-compose up --build
   ```
2. Access the front-end at `http://localhost:5173` and the back-end at `http://localhost:3001`.

---

### **Project Structure**
```
my-node-react-app/
├── backend/
│   ├── src/
│   │   └── index.ts
│   ├── package.json
│   └── tsconfig.json
├── react-frontend/
│   ├── src/
│   │   ├── App.tsx
│   │   ├── main.tsx
│   │   └── ...
│   ├── package.json
│   ├── tsconfig.json
│   └── vite.config.ts
├── docker-compose.yml
└── .devcontainer/
    └── devcontainer.json
```