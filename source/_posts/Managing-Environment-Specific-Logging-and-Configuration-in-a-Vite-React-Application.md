---
title: >-
  Managing Environment-Specific Logging and Configuration in a Vite React
  Application
date: 2025-01-30 18:34:06
categories:
- ReactJS
- TypeScript
- Vite
- Yarn
tags:
- ReactJS
- TypeScript
- Vite
- Yarn
---

1. **How to show console logs according to the environment**.
2. **How to load environment variables in your application**.
3. **How to load the correct environment file (with Vite or Yarn)**.
4. **How to start the application with the production build**.

---

## 1. Show Console Logs According to the Environment

To ensure that `console.log`, `console.warn`, and `console.error` only work in specific environments (e.g., test mode), you can create a utility module that conditionally logs based on the environment.

### Example: `logUtil.ts`

```typescript
const isTestEnv = import.meta.env.VITE_ENV === 'test';

const logUtil = {
    log: <T>(...args: T[]) => {
        if (isTestEnv) {
            console.log(...args);
        }
    },
    warn: <T>(...args: T[]) => {
        if (isTestEnv) {
            console.warn(...args);
        }
    },
    error: <T>(...args: T[]) => {
        if (isTestEnv) {
            console.error(...args);
        }
    },
};

export default logUtil;
```

### Explanation:
- **Environment Check**: The `isTestEnv` variable checks if the `VITE_ENV` environment variable is set to `"test"`.
- **Conditional Logging**: The `logUtil` methods (`log`, `warn`, `error`) only execute `console` statements if the environment is `test`.
- **Type Safety**: The utility uses TypeScript generics (`<T>`) to ensure type safety for the logged arguments.

### Usage:
```typescript
import logUtil from './utils/logUtil';

logUtil.log("This will only log in test mode");
logUtil.warn("This warning will only appear in test mode");
logUtil.error("This error will only appear in test mode");
```

---

## 2. Load Environment Variables in Your Application

Vite automatically exposes environment variables prefixed with `VITE_` to your application via `import.meta.env`. To use environment-specific variables:

### Example: `.env.test`
```env
VITE_ENV=test
VITE_API_URL=https://api.test.example.com
```

### Accessing Environment Variables:
```typescript
const apiUrl = import.meta.env.VITE_API_URL;
console.log(apiUrl); // Outputs: https://api.test.example.com
```

### Rules:
- Prefix all environment variables with `VITE_` (e.g., `VITE_ENV`, `VITE_API_URL`).
- Access them in your application using `import.meta.env`.

---

## 3. Load the Correct Environment File (with Vite or Yarn)

Vite loads environment variables from `.env` files based on the `mode` specified during the build or development process.

### Environment Files:
- `.env`: Default environment file (loaded in all modes).
- `.env.test`: Loaded when the mode is `test`.
- `.env.production`: Loaded when the mode is `production`.

### Specifying the Mode:
- **During Development**:
  ```bash
  yarn dev --mode test
  ```
- **During Build**:
  ```bash
  yarn build --mode test
  ```

### Example Workflow:
1. Create a `.env.test` file:
   ```env
   VITE_ENV=test
   VITE_API_URL=https://api.test.example.com
   ```

2. Build the application in test mode:
   ```bash
   yarn build --mode test
   ```

3. Vite will load the `.env.test` file and expose its variables via `import.meta.env`.

---

## 4. Start the Application with the Build

After building your application, you need to serve the production files. Use a static server like `serve` or Vite's built-in `preview` command.

### Option 1: Using `serve`
1. Install `serve` globally or locally:
   ```bash
   yarn global add serve
   ```
   Or:
   ```bash
   yarn add serve
   ```

2. Serve the production build:
   ```bash
   serve -s dist
   ```
   - The `-s` flag ensures all routes fall back to `index.html` (useful for SPAs).
   - The `dist` directory is the default output directory for Vite builds.

3. Access the application at `http://localhost:3000`.

### Option 2: Using Vite's Preview Command
1. Add a `preview` script to your `package.json`:
   ```json
   "scripts": {
     "build": "vite build",
     "preview": "vite preview"
   }
   ```

2. Build the application:
   ```bash
   yarn build --mode test
   ```

3. Start the preview server:
   ```bash
   yarn preview
   ```
   - This serves the production build at `http://localhost:4173` (or another port if specified).

---

## Summary

1. **Environment-Specific Logging**:
   - Use a utility like `logUtil.ts` to conditionally log based on the environment.

2. **Loading Environment Variables**:
   - Prefix variables with `VITE_` and access them via `import.meta.env`.

3. **Loading the Correct Environment File**:
   - Use the `--mode` flag with `yarn dev` or `yarn build` to load the appropriate `.env` file.

4. **Starting the Application with the Build**:
   - Serve the production build using `serve` or Vite's `preview` command.
