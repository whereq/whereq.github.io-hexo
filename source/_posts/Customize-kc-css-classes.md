---
title: Customize kc-* css classes
date: 2025-03-09 13:39:10
categories:
- Keycloak
- Keycloakify
tags:
- Keycloak
- Keycloakify
---

### Summary of the Debugging Process, and Solutions about Customzing Template.tsx
![Customize kc-header-wrapper clss](images/Customize-kc-css-classes/kc-header-wrapper-customization.png)


In this case, the issue stemmed from the `kc-header-wrapper` class, which is part of Keycloak's original default theme (`login.css`). This class had predefined styles (e.g., `padding`, `line-height`, `font-size`, etc.) that were overriding the Tailwind CSS styles applied via inline classes in the React component. The `kc-` prefixed classes are part of Keycloak's default theme, and customizing them with Tailwind CSS can be challenging because Tailwind's utility classes are often overridden by these predefined styles.

#### Debugging Process:
1. **Initial Observation**:
   - The `kc-header-wrapper` was exceeding the height of its parent (`kc-header`), causing layout issues.
   - The `h-full` Tailwind class was not working as expected.

2. **Root Cause Identification**:
   - The `kc-header-wrapper` class from Keycloak's `login.css` was overriding the Tailwind styles with its own `padding`, `line-height`, and other properties.

3. **Attempted Fixes**:
   - Initially, Tailwind CSS classes like `p-0`, `m-0`, and `h-full` were used inline, but they were ineffective due to the higher specificity of the `login.css` styles.
   - The `!important` rule was considered to force override the styles.

4. **Final Solution**:
   - A separate CSS file (`template.css`) was created to reset the problematic properties of `#kc-header-wrapper` using `!important`. This ensured that the styles from `login.css` were overridden.
   - The `template.css` file was imported into the `Template.tsx` component to apply the reset styles.

#### Final Solution Code:
```css
/* template.css */
#kc-header-wrapper {
    padding: 0 !important;
    margin: 0 !important;
    line-height: normal !important;
    font-size: inherit !important;
    text-transform: none !important;
    letter-spacing: normal !important;
    white-space: normal !important;
}
```

```tsx
{/* Header */}
<div id="kc-header" className={clsx(kcClsx("kcHeaderClass"), "bg-blue-700 border-b border-orange-700 h-[3.125rem] flex items-center px-4 fixed top-0 left-0 right-0 z-50 w-full")}>
    <div id="kc-header-wrapper" className={clsx(kcClsx("kcHeaderWrapperClass"), "flex items-center h-full")}>
        <img src={logoPngUrl} width={40} height={40} className="mr-2" />
        {msg("loginTitleHtml", realm.displayNameHtml)}
    </div>
</div>
```

#### Alternatives:
1. **Inline Styles**:
   - As a temporary debugging measure, inline styles can be used to override the `login.css` styles:
     ```tsx
     <div id="kc-header-wrapper" style={{ padding: 0, margin: 0, lineHeight: "normal", fontSize: "inherit", textTransform: "none", letterSpacing: "normal", whiteSpace: "normal" }}>
         <img src={logoPngUrl} width={40} height={40} className="mr-2" />
         {msg("loginTitleHtml", realm.displayNameHtml)}
     </div>
     ```

2. **Tailwind `!important` Utility**:
   - Tailwind CSS supports the `!important` utility (e.g., `p-0!`), but this may not always work due to specificity issues with Keycloak's default styles.

3. **Custom CSS File**:
   - The most robust solution is to use a separate CSS file to reset or override the `kc-` prefixed classes, as shown in the final solution.
