---
title: Customizing Keycloak Themes with Keycloakify and Tailwind CSS
date: 2025-03-09 21:09:42
categories:
- Keycloak
- Keycloakify
tags:
- Keycloak
- Keycloakify
---


## What is Keycloakify?

Keycloakify is a library that simplifies the process of creating custom Keycloak themes using modern frontend tools like React and Tailwind CSS. It provides:

- A set of utility functions (e.g., `clsx`, `kcClsx`) to combine Keycloak's default classes with custom styles.
- Predefined classes (e.g., `kcLoginClass`, `kcHeaderClass`) to ensure compatibility with Keycloak's theming system.
- Tools to integrate custom themes with Keycloak's login, registration, and account management pages.

Keycloakify is particularly useful for developers who want to use React and Tailwind CSS to create custom Keycloak themes without dealing with Keycloak's legacy theming system.

---

## Keycloakify's `kc*` Classes

### Structure of `kc*` Classes

Keycloakify provides a set of predefined classes prefixed with `kc*` (e.g., `kcLoginClass`, `kcHeaderClass`). These classes are designed to:

- Ensure compatibility with Keycloak's default theming system.
- Provide a base set of styles for common components (e.g., headers, forms, buttons).
- Allow developers to extend or override styles using custom CSS or Tailwind CSS.

### Common `kc*` Classes

Here are some of the most common `kc*` classes provided by Keycloakify:

| Class Name          | Description                                                                 |
|---------------------|-----------------------------------------------------------------------------|
| `kcLoginClass`      | Applied to the root container of the login page.                            |
| `kcHeaderClass`     | Applied to the header section of the login page.                            |
| `kcFormCardClass`   | Applied to the form container (e.g., login form, registration form).         |
| `kcFormHeaderClass` | Applied to the header of forms (e.g., "Login to Your Account").              |
| `kcFormGroupClass`  | Applied to form groups (e.g., username/password fields).                    |
| `kcAlertClass`      | Applied to alert messages (e.g., error messages, success messages).         |
| `kcSignUpClass`     | Applied to the footer section of the login page.                            |

---

## Pros and Cons of Using Keycloakify Classes

### Pros

1. **Compatibility**:
   - Keycloakify's `kc*` classes ensure compatibility with Keycloak's default theming system, reducing the risk of layout issues.

2. **Quick Start**:
   - Using Keycloakify's default classes allows developers to quickly create custom themes without starting from scratch.

3. **Consistency**:
   - The predefined classes provide a consistent structure for common components (e.g., headers, forms, buttons).

4. **Integration with Keycloak**:
   - Keycloakify handles the integration between custom themes and Keycloak's theming system, simplifying the deployment process.

### Cons

1. **Styling Conflicts**:
   - Keycloakify's default classes often come with predefined styles (e.g., padding, margins) that can conflict with custom Tailwind CSS styles.

2. **Complexity**:
   - Combining Keycloakify's default classes with custom styles can introduce unnecessary complexity, especially when using utility-first frameworks like Tailwind CSS.

3. **Limited Flexibility**:
   - Relying on Keycloakify's default classes can limit your ability to fully customize the theme, as you may need to override or work around the predefined styles.

4. **Performance Overhead**:
   - Including unnecessary default styles can increase the size of your CSS bundle, impacting performance.

---

## Cleaning Up Keycloakify Classes

### Why Clean Up Keycloakify Classes?

Keycloakify's default classes (e.g., `kcLoginClass`, `kcHeaderClass`) are designed to ensure compatibility with Keycloak's theming system. However, they come with several drawbacks:

1. **Avoid Styling Conflicts**:
   - Removing Keycloakify's default classes eliminates the risk of conflicts with custom Tailwind CSS styles.

2. **Simplify the Codebase**:
   - Relying solely on Tailwind CSS makes the codebase cleaner and easier to maintain.

3. **Full Control Over Styling**:
   - Using only Tailwind CSS gives you complete control over the theme's appearance and behavior.

4. **Reduce Complexity**:
   - Removing unnecessary classes and functions simplifies the development process.

5. **Limited Flexibility**:
   - Relying on Keycloakify's default classes limits your ability to fully customize the theme, as you may need to override or work around the predefined styles.

6. **Performance Overhead**:
   - Including unnecessary default styles increases the size of your CSS bundle, impacting performance.

---

### Steps to Clean Up Keycloakify Classes

1. **Remove `clsx` and `kcClsx`**:
   - Keycloakify provides `clsx` and `kcClsx` functions to combine its default classes with custom styles. Since we're removing the default classes, these functions are no longer needed.

#### Before:
```tsx
<div id="template-full" className={clsx(kcClsx("kcLoginClass"), "h-screen flex flex-col overflow-hidden")}>
```

#### After:
```tsx
<div id="template-full" className="h-screen flex flex-col overflow-hidden bg-gray-700">
```

2. **Replace `kc*` Classes with Tailwind CSS**:
   - Replace all instances of `clsx(kcClsx(...))` with plain Tailwind CSS classes. For example:

#### Before:
```tsx
<div id="kc-header" className={clsx(kcClsx("kcHeaderClass"), "bg-blue-700 border-b border-orange-700 h-[3.125rem] flex items-center px-4 fixed top-0 left-0 right-0 z-50 w-full")}>
```

#### After:
```tsx
<div id="kc-header" className="bg-blue-700 border-b border-orange-700 h-[3.125rem] flex items-center px-4 fixed top-0 left-0 right-0 z-50 w-full">
```

3. **Remove Unnecessary Imports**:
   - Remove imports like `clsx`, `getKcClsx`, and `useSetClassName` since they are no longer needed.

#### Before:
```tsx
import { clsx } from "keycloakify/tools/clsx";
import { getKcClsx } from "keycloakify/login/lib/kcClsx";
import { useSetClassName } from "keycloakify/tools/useSetClassName";
```

#### After:
```tsx
// These imports are no longer needed
```

4. **Ensure Consistent Styling**:
   - Use Tailwind CSS utility classes to define all styles, ensuring consistency and avoiding conflicts. For example:

#### Before:
```tsx
<div id="kc-header-wrapper" className={clsx(kcClsx("kcHeaderWrapperClass"), "flex items-center h-full")}>
```

#### After:
```tsx
<div id="kc-header-wrapper" className="flex items-center h-full">
```

---

### Example: Fully Customized Keycloak Theme with Tailwind CSS

Here’s an example of a fully customized Keycloak theme using only Tailwind CSS:

```tsx
import { useEffect } from "react";
import { kcSanitize } from "keycloakify/lib/kcSanitize";
import type { TemplateProps } from "keycloakify/login/TemplateProps";
import { useInitialize } from "keycloakify/login/Template.useInitialize";
import type { I18n } from "./i18n";
import type { KcContext } from "./KcContext";

import "./template.css"; // Import the CSS file

import logoPngUrl from "../../assets/img/logo-s.png";

export default function Template(props: TemplateProps<KcContext, I18n>) {
    const {
        displayMessage = true,
        displayRequiredFields = false,
        headerNode,
        socialProvidersNode = null,
        documentTitle,
        kcContext,
        i18n,
        children
    } = props;

    const { msg, msgStr, currentLanguage, enabledLanguages } = i18n;

    const { realm, auth, url, message, isAppInitiatedAction } = kcContext;

    useEffect(() => {
        document.title = documentTitle ?? msgStr("loginTitle", kcContext.realm.displayName);
    }, [documentTitle, kcContext.realm.displayName, msgStr]);

    const { isReadyToRender } = useInitialize({ kcContext, doUseDefaultCss: false });

    if (!isReadyToRender) {
        return null;
    }

    return (
        <div id="template-full" className="h-screen flex flex-col overflow-hidden bg-gray-700">
            {/* Header */}
            <div id="kc-header" className="bg-blue-700 border-b border-orange-700 h-[3.125rem] flex items-center px-4 fixed top-0 left-0 right-0 z-50 w-full">
                <div id="kc-header-wrapper" className="flex items-center h-full">
                    <img src={logoPngUrl} width={40} height={40} className="mr-2" />
                    <span className="text-white">{msg("loginTitleHtml", realm.displayNameHtml)}</span>
                </div>
            </div>

            {/* Main Content */}
            <div id="template-main-content" className="flex-grow mt-[3.125rem] mb-[3.125rem] bg-gray-700">
                <header className="border-b border-orange-700">
                    {enabledLanguages.length > 1 && (
                        <div id="kc-locale">
                            <div id="kc-locale-wrapper" className="flex gap-2">
                                {enabledLanguages
                                    .filter(({ languageTag }) => languageTag === "en" || languageTag === "zh-CN")
                                    .map(({ languageTag, label, href }) => (
                                        <button
                                            key={languageTag}
                                            className="px-3 py-1 rounded-md bg-gray-200 text-gray-700 hover:bg-gray-300"
                                            onClick={() => (window.location.href = href)}
                                            aria-label={label}
                                        >
                                            {languageTag === "en" ? "🇺🇸" : "🇨🇳"}
                                        </button>
                                    ))}
                            </div>
                        </div>
                    )}
                    {(() => {
                        const node = !(auth !== undefined && auth.showUsername && !auth.showResetCredentials) ? (
                            <h1 id="kc-page-title">{headerNode}</h1>
                        ) : (
                            <div id="kc-username">
                                <label id="kc-attempted-username">{auth.attemptedUsername}</label>
                                <a id="reset-login" href={url.loginRestartFlowUrl} aria-label={msgStr("restartLoginTooltip")}>
                                    <div className="kc-login-tooltip">
                                        <i className="kcResetFlowIcon"></i>
                                        <span className="kc-tooltip-text">{msg("restartLoginTooltip")}</span>
                                    </div>
                                </a>
                            </div>
                        );

                        if (displayRequiredFields) {
                            return (
                                <div>
                                    <div className="subtitle">
                                        <span className="required">*</span>
                                        {msg("requiredFields")}
                                    </div>
                                    <div>{node}</div>
                                </div>
                            );
                        }

                        return node;
                    })()}
                </header>

                {/* Content */}
                <div id="kc-content" className="border-b border-orange-700">
                    <div id="kc-content-wrapper">
                        {displayMessage && message !== undefined && (message.type !== "warning" || !isAppInitiatedAction) && (
                            <div className={`alert-${message.type}`}>
                                <div className="pf-c-alert__icon">
                                    {message.type === "success" && <span className="kcFeedbackSuccessIcon"></span>}
                                    {message.type === "warning" && <span className="kcFeedbackWarningIcon"></span>}
                                    {message.type === "error" && <span className="kcFeedbackErrorIcon"></span>}
                                    {message.type === "info" && <span className="kcFeedbackInfoIcon"></span>}
                                </div>
                                <span
                                    className="kcAlertTitleClass"
                                    dangerouslySetInnerHTML={{
                                        __html: kcSanitize(message.summary)
                                    }}
                                />
                            </div>
                        )}
                        {children}
                        {auth !== undefined && auth.showTryAnotherWayLink && (
                            <form id="kc-select-try-another-way-form" action={url.loginAction} method="post">
                                <div>
                                    <input type="hidden" name="tryAnotherWay" value="on" />
                                    <a
                                        href="#"
                                        id="try-another-way"
                                        onClick={() => {
                                            document.forms["kc-select-try-another-way-form" as never].submit();
                                            return false;
                                        }}
                                    >
                                        {msg("doTryAnotherWay")}
                                    </a>
                                </div>
                            </form>
                        )}
                        {socialProvidersNode}
                    </div>
                </div>
            </div>

            {/* Footer */}
            <div id="kc-footer" className="bg-blue-700 border-t border-orange-700 h-[3.125rem] flex items-center justify-center fixed bottom-0 left-0 right-0 z-50">
                <div>
                    <span className="text-white">© 2025 WhereQ Inc. All rights reserved.</span>
                </div>
            </div>
        </div>
    );
}
```


## Again, benefits of Using Tailwind CSS Exclusively

1. **No Styling Conflicts**:
   - By removing Keycloakify's default classes, you eliminate the risk of conflicts with custom Tailwind CSS styles.

2. **Full Control Over Styling**:
   - Using only Tailwind CSS gives you complete control over the theme's appearance and behavior.

3. **Simplified Codebase**:
   - The code is cleaner and easier to maintain, with no reliance on Keycloakify's default theming system.

4. **Improved Performance**:
   - Removing unnecessary default styles reduces the size of your CSS bundle, improving performance.

---

