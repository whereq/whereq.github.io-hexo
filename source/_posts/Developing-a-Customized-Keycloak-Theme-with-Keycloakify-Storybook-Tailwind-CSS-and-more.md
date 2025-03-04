---
title: >-
  Developing a Customized Keycloak Theme with Keycloakify, Storybook, Tailwind CSS and more...
date: 2025-03-02 12:35:31
categories:
- Keycloak
- Keycloakify
tags:
- Keycloak
- Keycloakify
---

## Table of Contents
1. [Introduction](#1-introduction)
   - [What is Keycloakify?](#what-is-keycloakify)
   - [Why Customize Keycloak Themes?](#why-customize-keycloak-themes)
2. [Prerequisites](#2-prerequisites)
   - [Tools and Dependencies](#tools-and-dependencies)
   - [Setting Up the Development Environment](#setting-up-the-development-environment)
3. [Project Setup](#3-project-setup)
   - [Initialize a Vite + React + TypeScript Project](#initialize-a-vite--react--typescript-project)
   - [Install Required Dependencies](#install-required-dependencies)
4. [Set Up Storybook for Local Testing](#4-set-up-storybook-for-local-testing)
   - [Initialize Storybook](#initialize-storybook)
   - [Configure Storybook](#configure-storybook)
5. [Integrate Keycloakify with Vite](#5-integrate-keycloakify-with-vite)
   - [Create the `src/keycloak-theme` Directory](#create-the-srckeycloak-theme-directory)
   - [Migrate the Source Code from Keycloakify Starter](#migrate-the-source-code-from-keycloakify-starter)
   - [Rename `src/main.tsx` to `src/main.app.tsx`](#rename-srcmaintsx-to-srcmainapptsx)
   - [Migrate Keycloakify Starter's `main.tsx`](#migrate-keycloakify-starters-maintsx)
   - [Update `index.html`](#update-indexhtml)
6. [Customizing the Theme](#6-customizing-the-theme)
   - [Adding Tailwind CSS](#adding-tailwind-css)
   - [Setting Up the Open Sans Font](#setting-up-the-open-sans-font)
   - [Implementing a Dark Theme](#implementing-a-dark-theme)
   - [Customizing Keycloak Pages](#customizing-keycloak-pages)
7. [Configuration Files](#7-configuration-files)
   - [`vite.config.ts`](#viteconfigts)
   - [`tailwind.config.js`](#tailwindconfigjs)
   - [`postcss.config.js`](#postcssconfigjs)
   - [`.eslintrc.js`](#eslintrcjs)
   - [`tsconfig.json`](#tsconfigjson)
8. [Building and Deploying the Theme](#8-building-and-deploying-the-theme)
   - [Building the Theme](#building-the-theme)
   - [Generating the JAR File](#generating-the-jar-file)
   - [Deploying to Keycloak](#deploying-to-keycloak)
9. [Advanced Customizations](#9-advanced-customizations)
   - [Adding Custom Pages](#adding-custom-pages)
   - [Theming Keycloak Emails](#theming-keycloak-emails)
   - [Localization and Internationalization](#localization-and-internationalization)
10. [Troubleshooting](#10-troubleshooting)
    - [Common Issues and Fixes](#common-issues-and-fixes)
11. [Conclusion](#11-conclusion)

---

## 1. Introduction

### What is Keycloakify?
**Keycloakify** is a tool that allows you to create custom themes for **Keycloak** using modern web technologies like **React.js** and **TypeScript**. It simplifies the process of building and deploying Keycloak themes by providing a streamlined workflow and integration with tools like **Vite**.

### Why Customize Keycloak Themes?
Customizing Keycloak themes allows you to:
- Match the look and feel of your application.
- Improve user experience with modern design and responsiveness.
- Add custom functionality to Keycloak pages (e.g., login, registration).

---

## 2. Prerequisites

### Tools and Dependencies
- **Node.js** (v18 or higher)
- **Yarn**
- **Keycloak** (latest version)
- **React.js**, **TypeScript**, **Vite**, **Tailwind CSS**, **Open Sans Font**, **Storybook**

### Setting Up the Development Environment
1. Install Node.js: [Download Node.js](https://nodejs.org/)
2. Install Yarn (if not already installed):
   ```bash
   npm install -g yarn
   ```

---

## 3. Project Setup

### Initialize a Vite + React + TypeScript Project
1. Create a new project using Vite:
   ```bash
   yarn create vite keycloakify-poc --template react-ts
   cd keycloakify-poc
   ```

2. Install dependencies:
   ```bash
   yarn install
   ```

### Install Required Dependencies
Install additional dependencies for Tailwind CSS, Open Sans, Storybook, and dark theme support:
```bash
yarn add keycloakify react react-dom
yarn add -D typescript vite vite-plugin-react
yarn add -D storybook @storybook/react-vite @storybook/addon-essentials @storybook/react
yarn add -D tailwindcss postcss autoprefixer @tailwindcss/postcss @tailwindcss/vite
yarn add @fontsource/open-sans
```

---

## 4. Set Up Storybook for Local Testing

### Initialize Storybook
1. Initialize Storybook:
   ```bash
   npx storybook init
   ```

2. Start Storybook:
   ```bash
   yarn storybook
   ```

### Configure Storybook
Ensure Storybook is configured to serve static files from the `public` directory. Update `.storybook/main.ts`:

```typescript
import type { StorybookConfig } from "@storybook/react-vite";

const config: StorybookConfig = {
  stories: [
    "../src/**/*.mdx",
    "../src/**/*.stories.@(js|jsx|mjs|ts|tsx)",
  ],
  addons: [
    "@storybook/addon-essentials",
    "@storybook/addon-onboarding",
    "@chromatic-com/storybook",
    "@storybook/experimental-addon-test",
  ],
  framework: {
    name: "@storybook/react-vite",
    options: {},
  },
  staticDirs: ["../public"], // Add this line
};

export default config;
```

---

## 5. Integrate Keycloakify with Vite

### Create the `src/keycloak-theme` Directory
Keycloakify expects a `keycloak-theme` directory in your `src` folder. This directory will contain all the theme-related files.

```bash
mkdir src/keycloak-theme
```

### Migrate the Source Code from Keycloakify Starter
To scaffold your project, you can use the **Keycloakify Starter** repository as a template.

1. Clone the Keycloakify Starter repository into a temporary directory:
   ```bash
   git clone https://github.com/keycloakify/keycloakify-starter tmp
   ```

2. Move the `src` folder from the cloned repository into your project's `src/keycloak-theme` directory:
   ```bash
   mv tmp/src src/keycloak-theme
   ```

3. Clean up the temporary directory:
   ```bash
   rm -rf tmp
   rm src/keycloak-theme/vite-env.d.ts
   ```

### Rename `src/main.tsx` to `src/main.app.tsx`
The `main.tsx` file is the entry point for your application. Rename it to `main.app.tsx` to avoid conflicts with Keycloakify's `main.tsx`.

```bash
mv src/main.tsx src/main.app.tsx
```

Update the content of `src/main.app.tsx` to ensure it exports a valid React component:

```tsx
import { StrictMode } from "react";
import { createRoot } from "react-dom/client";
import "./index.css";
import App from "./App";

// Define a React component that wraps the App component
const MainApp = () => (
  <StrictMode>
    <App />
  </StrictMode>
);

// Export the MainApp component as the default export
export default MainApp;

// Render the MainApp component to the DOM
const root = createRoot(document.getElementById("root")!);
root.render(<MainApp />);
```

### Migrate Keycloakify Starter's `main.tsx`
Move the `main.tsx` file from the `keycloak-theme` directory to the root `src` directory:

```bash
mv src/keycloak-theme/main.tsx src/main.tsx
```

Update the content of `src/main.tsx` to lazily load the `MainApp` component and handle Keycloak context:

```tsx
import { createRoot } from "react-dom/client";
import { lazy, StrictMode, Suspense } from "react";
import { KcPage, type KcContext } from "./keycloak-theme/kc.gen";

// Lazy load the MainApp component
const AppEntrypoint = lazy(() => import("./main.app"));

createRoot(document.getElementById("root")!).render(
  <StrictMode>
    {!window.kcContext ? (
      <Suspense fallback={<div>Loading...</div>}>
        <AppEntrypoint />
      </Suspense>
    ) : (
      <KcPage kcContext={window.kcContext} />
    )}
  </StrictMode>
);

declare global {
  interface Window {
    kcContext?: KcContext;
  }
}
```

### Update `index.html`
Exclude elements from the HTML `<head>` that are not relevant in the context of Keycloak pages. Use the `keycloakify-ignore` meta tags to achieve this.

Update `index.html`:

```html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <link rel="icon" type="image/svg+xml" href="/vite.svg" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />

    <meta name="keycloakify-ignore-start">
    <title>WhereQ --- Keycloakify POC</title>
    <script>
      window.ENV = {
        API_ADDRESS: '${API_ADDRESS}',
        SENTRY_DSN: '${SENTRY_DSN}'
      };
    </script>
    <meta name="keycloakify-ignore-end">
  </head>
  <body>
    <div id="root"></div>
    <script type="module" src="/src/main.tsx"></script>
  </body>
</html>
```

---

## 6. Customizing the Theme

### Adding Tailwind CSS
1. Create a `tailwind.config.js` file:
   ```javascript
   /** @type {import('tailwindcss').Config} */
   export default {
     content: [
       "./index.html",
       "./src/**/*.{js,ts,jsx,tsx}",
     ],
     theme: {
       extend: {},
     },
     plugins: [],
   };
   ```

2. Create a `postcss.config.js` file:
   ```javascript
   export default {
     plugins: {
       "@tailwindcss/postcss": {},
       autoprefixer: {},
     },
   };
   ```

3. Create a CSS file (`src/index.css`) and add Tailwind directives:
   ```css
   @import "tailwindcss";
   @tailwind base;
   @tailwind components;
   @tailwind utilities;
   ```

4. Import the CSS file in `src/main.tsx`:
   ```tsx
   import { createRoot } from "react-dom/client";
   import { lazy, StrictMode, Suspense } from "react";
   import { KcPage, type KcContext } from "./keycloak-theme/kc.gen";

   import "./index.css";

   // Lazy load the MainApp component
   const AppEntrypoint = lazy(() => import("./main.app"));

   createRoot(document.getElementById("root")!).render(
     <StrictMode>
       {!window.kcContext ? (
         <Suspense fallback={<div>Loading...</div>}>
           <AppEntrypoint />
         </Suspense>
       ) : (
         <KcPage kcContext={window.kcContext} />
       )}
     </StrictMode>
   );

   declare global {
     interface Window {
       kcContext?: KcContext;
     }
   }
   ```

### Setting Up the Open Sans Font
1. Install the Open Sans font:
   ```bash
   yarn add @fontsource/open-sans
   ```

2. Import the font in `src/index.css`:
   ```css
   @import "@fontsource/open-sans";

   body {
     font-family: "Open Sans", sans-serif;
   }
   ```

### Implementing a Dark Theme
1. Enable dark mode in `tailwind.config.js`:
   ```javascript
   export default {
     darkMode: "class", // or 'media' for system preference
     content: [
       "./index.html",
       "./src/**/*.{js,ts,jsx,tsx}",
     ],
     theme: {
       extend: {},
     },
     plugins: [],
   };
   ```

2. Add a theme toggle button in your React component:
   ```tsx
   import { useState } from "react";

   const ThemeToggle = () => {
     const [isDark, setIsDark] = useState(false);

     const toggleTheme = () => {
       setIsDark(!isDark);
       document.documentElement.classList.toggle("dark", !isDark);
     };

     return (
       <button onClick={toggleTheme}>
         {isDark ? "Switch to Light Mode" : "Switch to Dark Mode"}
       </button>
     );
   };

   export default ThemeToggle;
   ```

3. Add dark mode styles in `src/index.css`:
   ```css
   @layer components {
     .dark {
       @apply bg-gray-900 text-white;
     }
   }
   ```

### Customizing Keycloak Pages
1. Eject a Keycloak page (e.g., login page):
   ```bash
   npx keycloakify eject-page login
   ```

2. Customize the ejected page (`src/pages/Login.tsx`):
   ```tsx
   const Login = () => {
     return (
       <div className="min-h-screen flex items-center justify-center bg-gray-100 dark:bg-gray-900">
         <div className="bg-white dark:bg-gray-800 p-8 rounded-lg shadow-lg w-96">
           <h1 className="text-2xl font-bold mb-4 text-gray-900 dark:text-white">Login</h1>
           <form>
             <input
               type="text"
               placeholder="Username"
               className="w-full p-2 mb-4 border rounded-lg dark:bg-gray-700 dark:text-white"
             />
             <input
               type="password"
               placeholder="Password"
               className="w-full p-2 mb-4 border rounded-lg dark:bg-gray-700 dark:text-white"
             />
             <button
               type="submit"
               className="w-full bg-blue-600 text-white p-2 rounded-lg hover:bg-blue-700"
             >
               Log In
             </button>
           </form>
         </div>
       </div>
     );
   };

   export default Login;
   ```

---

## 7. Configuration Files

### `vite.config.ts`
```typescript
import { defineConfig } from "vite";
import react from "@vitejs/plugin-react";
import tailwindcss from "@tailwindcss/vite";
import { keycloakify } from "keycloakify/vite-plugin";

export default defineConfig({
  plugins: [
    react(),
    tailwindcss(),
    keycloakify({
      accountThemeImplementation: "none",
    }),
  ],
});
```

### `tailwind.config.js`
```javascript
/** @type {import('tailwindcss').Config} */
export default {
  darkMode: "class",
  content: [
    "./index.html",
    "./src/**/*.{js,ts,jsx,tsx}",
  ],
  theme: {
    extend: {},
  },
  plugins: [],
};
```

### `postcss.config.js`
```javascript
export default {
  plugins: {
    "@tailwindcss/postcss": {},
    autoprefixer: {},
  },
};
```

### `.eslintrc.js`
```javascript
export default {
  env: {
    node: true,
    es2022: true,
  },
  parserOptions: {
    ecmaVersion: "latest",
    sourceType: "module",
  },
  rules: {
    // Your ESLint rules...
  },
};
```

### `tsconfig.json`
```json
{
  "compilerOptions": {
    "target": "ES2020",
    "useDefineForClassFields": true,
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

---

## 8. Building and Deploying the Theme

### Building the Theme
Run the following command to build the theme:
```bash
yarn build-keycloak-theme
```

### Generating the JAR File
1. Package the theme into a JAR file:
   ```bash
   yarn keycloakify build
   ```

2. The JAR file will be generated in the `build/keycloak-theme` directory.

### Deploying to Keycloak
1. Copy the generated JAR file to your Keycloak server's `themes` directory.
2. Restart Keycloak to apply the new theme.

---

## 9. Advanced Customizations

### Adding Custom Pages
You can add custom pages (e.g., a custom error page) by ejecting and modifying the corresponding page:
```bash
npx keycloakify eject-page error
```

This will generate the necessary files for the error page in the `src/keycloak-theme` directory. You can then customize the page as needed.

---

### Theming Keycloak Emails
Keycloakify also supports customizing email templates. To customize email templates:

1. Eject the email template you want to modify:
   ```bash
   npx keycloakify eject-email <email-template-name>
   ```

   Replace `<email-template-name>` with the name of the email template (e.g., `verify-email`, `password-reset`).

2. Customize the ejected email template in the `src/keycloak-theme` directory.

---

### Localization and Internationalization
Keycloak supports multiple languages out of the box. To add localization to your theme:

1. Add language files to the `src/keycloak-theme/messages` directory. For example:
   - `messages_en.properties` for English
   - `messages_fr.properties` for French

2. Use the `msg()` function in your components to display localized text:
   ```tsx
   const Login = () => {
     const { msg } = useI18n();

     return (
       <div>
         <h1>{msg("loginTitle")}</h1>
         <p>{msg("loginDescription")}</p>
       </div>
     );
   };
   ```

3. Add translations to your language files:
   ```properties
   # messages_en.properties
   loginTitle=Login
   loginDescription=Please enter your credentials.

   # messages_fr.properties
   loginTitle=Connexion
   loginDescription=Veuillez entrer vos identifiants.
   ```

---

## 10. Troubleshooting

### Common Issues and Fixes
- **Tailwind CSS Not Working**:
  - Ensure `postcss.config.js` and `tailwind.config.js` are correctly configured.
  - Verify that the `@import "tailwindcss";` directive is included in `src/index.css`.

- **Font Not Loading**:
  - Verify the font import path in `index.css`.
  - Ensure the `@fontsource/open-sans` package is installed.

- **Dark Mode Not Applying**:
  - Ensure `darkMode: "class"` is set in `tailwind.config.js`.
  - Verify that the `dark` class is applied to the root element (e.g., `<html>`).

- **Storybook Not Rendering Components**:
  - Ensure the `staticDirs` configuration in `.storybook/main.ts` points to the `public` directory.
  - Verify that the `index.css` file is imported in your components.

- **Keycloakify Build Fails**:
  - Ensure the `keycloak-theme` directory exists and contains valid theme files.
  - Check for errors in the terminal output and resolve them.

---

## 11. Conclusion

By following this guide, you've created a fully customized Keycloak theme using **Keycloakify**, **React.js**, **TypeScript**, **Vite**, **Tailwind CSS**, and the **Open Sans** font. You've also implemented a **dark theme**, tested the theme locally with **Storybook**, and deployed it to **Keycloak**. This approach ensures a modern and streamlined workflow for creating custom Keycloak themes.

### Key Takeaways
- **Keycloakify** simplifies the process of building and deploying Keycloak themes.
- **Tailwind CSS** and **Storybook** enable rapid development and testing of your theme.
- **Localization** and **custom pages** allow you to tailor the theme to your application's needs.
