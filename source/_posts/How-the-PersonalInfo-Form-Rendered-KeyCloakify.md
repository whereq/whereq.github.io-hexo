---
title: How the PersonalInfo Form Rendered --- KeyCloakify
date: 2025-03-17 09:52:55
categories:
- Keycloak
- Keycloakify
tags:
- Keycloak
- Keycloakify
---

### How the Form is Rendered
1. **Component Structure**:
   - The `UserProfileFields` component is the main entry point, receiving props like `t` (translation function), `form` (React Hook Form instance), `userProfileMetadata` (attribute metadata), and others.
   - It renders a `ScrollForm` component, which takes a `sections` prop to display grouped form fields.
---
2. **Grouping Attributes**:
   - **Logic**: Inside `UserProfileFields`, `groupsWithAttributes` is computed using `useMemo`:
     - If `userProfileMetadata.attributes` is empty, it returns an empty array.
     - Filters attributes based on `hideReadOnly` (excludes read-only fields if `true`).
     - Groups attributes by their `group` property, including an "ungrouped" category (`{ name: undefined }`) for attributes without a group.
     - Result: An array of `GroupWithAttributes` objects, each containing a `group` and its `attributes`.
   - **Outcome**: Only groups with attributes are rendered (filtered by `.filter((group) => group.attributes.length > 0)`).
---
3. **Rendering Sections**:
   - **ScrollForm**: The `ScrollForm` component receives `sections`, where each section is an object with:
     - `title`: Derived from `group.displayHeader` or `group.name`, falling back to `t("general")`.
     - `panel`: A `<div className="space-y-4">` containing:
       - Optional group description (`<p>` with `pb-6 text-gray-700`) if `group.displayDescription` exists.
       - A list of `FormField` components, one per attribute in the group.
   - **Rendering**: `ScrollForm` likely renders these sections as collapsible or navigable panels (exact behavior depends on `ScrollForm` implementation, not shown here).
---
4. **FormField Rendering**:
   - **Logic**: The `FormField` component determines how each attribute is rendered:
     - **Special Case**: If `attribute.name === "locale"`, it renders a `LocaleSelector`.
     - **General Case**: Otherwise, it selects a component from `FIELDS` based on `inputType`:
       - `inputType` is computed by `determineInputType` (see below).
       - If `attribute.multivalued` or the value is multi-valued (`isMultiValue`), it defaults to `MultiInputComponent`.
       - Otherwise, it uses the mapped component from `FIELDS` (e.g., `TextComponent` for `"text"`, `SelectComponent` for `"select"`).
   - **Component Props**: Each `FIELDS` component receives `t`, `form`, `inputType`, `attribute`, and `renderer`.
   - **Outcome**: Each attribute is rendered as a form field (e.g., text input, select dropdown) based on its metadata.
---
5. **Field Types**:
   - **Mapping**: The `FIELDS` object maps `InputType` to components:
     - `"text"`, `"html5-*"` → `TextComponent`.
     - `"textarea"` → `TextAreaComponent`.
     - `"select"`, `"multiselect"` → `SelectComponent`.
     - `"select-radiobuttons"`, `"multiselect-checkboxes"` → `OptionComponent`.
     - `"multi-input"` → `MultiInputComponent`.
   - **Dynamic Selection**: `determineInputType` checks:
     - Root attributes (`username`, `firstName`, etc.) → `"text"`.
     - `attribute.annotations?.inputType` if valid (e.g., `"select"`, `"textarea"`).
     - Default: `"text"`.
---
6. **Visual Layout**:
   - **TailwindCSS**: `<div className="space-y-4">` adds vertical spacing between fields; `<p className="pb-6 text-gray-700">` styles group descriptions.
   - **Result**: A scrollable form with sections, each containing a group title, optional description, and a list of input fields.

### 1. **Form Rendering**

The `UserProfileFields` component is responsible for rendering the form. Here's how it works:

#### **Step 1: Group Attributes**
- The `userProfileMetadata` object contains metadata about the user profile, including attributes and groups.
- The `groupsWithAttributes` array is created by grouping attributes based on their `group` property. If an attribute doesn't belong to any group, it's placed in a default group.

```typescript
const groupsWithAttributes = useMemo(() => {
  if (!userProfileMetadata.attributes) {
    return [];
  }

  const attributes = hideReadOnly
    ? userProfileMetadata.attributes.filter(({ readOnly }) => !readOnly)
    : userProfileMetadata.attributes;

  return [
    { name: undefined }, // Default group for attributes without a group
    ...(userProfileMetadata.groups ?? []),
  ].map<GroupWithAttributes>((group) => ({
    group,
    attributes: attributes.filter(
      (attribute) => attribute.group === group.name,
    ),
  }));
}, [hideReadOnly, userProfileMetadata.groups, userProfileMetadata.attributes]);
```

#### **Step 2: Render the Form**
- The `ScrollForm` component is used to render the form sections. Each section corresponds to a group of attributes.
- For each group, the `title` is derived from the group's `displayHeader` or a fallback (e.g., "general").
- The `panel` contains the form fields for the group's attributes.

```typescript
return (
  <ScrollForm
    label={t("jumpToSection")}
    sections={groupsWithAttributes
      .filter((group) => group.attributes.length > 0)
      .map(({ group, attributes }) => ({
        title: label(t, group.displayHeader, group.name) || t("general"),
        panel: (
          <div className="space-y-4">
            {group.displayDescription && (
              <p className="pb-6 text-gray-700">
                {label(t, group.displayDescription, "")}
              </p>
            )}
            {attributes.map((attribute) => (
              <FormField
                key={attribute.name}
                t={t}
                form={form}
                supportedLocales={supportedLocales}
                currentLocale={currentLocale}
                renderer={renderer}
                attribute={attribute}
              />
            ))}
          </div>
        ),
      }))}
  />
);
```

### How Field Values are Populated
1. **Form State**:
   - **Source**: The `form` prop is a `UseFormReturn<UserFormFields>` from React Hook Form, managing the form state.
   - **Type**: `UserFormFields` (from `utils.ts`) includes user properties (e.g., `username`, `email`) and an `attributes` object or array for custom fields.
---
2. **Value Retrieval**:
   - **Logic**: In `FormField`, `form.watch(fieldName(attribute.name))` retrieves the current value of each field:
     - `fieldName` (from `utils.ts`) transforms the attribute name:
       - Root attributes (`username`, `firstName`, etc.) → direct property (e.g., `"username"`).
       - Custom attributes → prefixed with `"attributes."` and dots replaced with `"🍺"` (e.g., `"attributes.my.field"` → `"attributes.my🍺field"`).
     - **Result**: `value` reflects the current form state for that attribute (e.g., `"john.doe"` for `username`, `["option1"]` for a multivalued field).
---
3. **Initial Population**:
   - **Not Shown**: The initial values aren’t set in this file. They’re likely passed to `form` via `useForm({ defaultValues })` in a parent component (e.g., a user profile editor).
   - **Assumption**: `defaultValues` would match `UserFormFields`, populated from a `UserRepresentation` object (e.g., `{ username: "john.doe", attributes: { "custom.field": "value" } }`).
---
4. **Dynamic Behavior**:
   - **Watching Values**: `form.watch` ensures the rendered component reflects real-time updates (e.g., user typing in a `TextComponent`).
   - **Multi-Valued Check**: `isMultiValue(value)` determines if the value is an array with multiple items, influencing the choice of `MultiInputComponent`.
---
5. **Component Role**:
   - **Field Components**: Each `FIELDS` component (e.g., `TextComponent`) uses `form` to register its input (via `register`, `control`, etc., not shown here) and display the `value`.
   - **Example**: For `TextComponent`, it might render `<input {...form.register(fieldName(attribute.name))} value={value} />`.
---
### 2. **Field Rendering and Value Population**

The `FormField` component is responsible for rendering individual form fields and populating their values. Here's how it works:

#### **Step 1: Determine the Input Type**
- The `inputType` is determined based on the attribute's metadata. For example, if the attribute is multivalued, the `multi-input` type is used.
- The `determineInputType` function checks the attribute's `annotations.inputType` or falls back to a default type (`text`).

```typescript
const inputType = useMemo(() => determineInputType(attribute), [attribute]);

const determineInputType = (attribute: UserProfileAttributeMetadata): InputType => {
  if (isRootAttribute(attribute.name)) {
    return "text";
  }

  const inputType = attribute.annotations?.inputType;

  if (isValidInputType(inputType)) {
    return inputType;
  }

  return DEFAULT_INPUT_TYPE;
};
```

#### **Step 2: Render the Field**
- The `Component` variable holds the appropriate component for the field based on the `inputType`. For example, if the `inputType` is `text`, the `TextComponent` is used.
- If the attribute is `locale`, the `LocaleSelector` component is used instead.

```typescript
const Component =
  attribute.multivalued ||
  (isMultiValue(value) && attribute.annotations?.inputType === undefined)
    ? FIELDS["multi-input"]
    : FIELDS[inputType];

if (attribute.name === "locale")
  return (
    <LocaleSelector
      form={form}
      supportedLocales={supportedLocales}
      currentLocale={currentLocale}
      t={t}
      attribute={attribute}
    />
  );

return (
  <Component
    t={t}
    form={form}
    inputType={inputType}
    attribute={attribute}
    renderer={renderer}
  />
);
```

#### **Step 3: Populate Field Values**
- The `form.watch` method from `react-hook-form` is used to watch the value of the field. This ensures that the field's value is always in sync with the form state.
- The `fieldName` function generates the field name based on the attribute's name. For example, if the attribute is `username`, the field name is `username`. If the attribute is `customAttribute`, the field name is `attributes.customAttribute`.

```typescript
const value = form.watch(
  fieldName(attribute.name) as FieldPath<UserFormFields>,
);
```

---

### 3. **Form State Management**

The `react-hook-form` library manages the form state. Here's how it works:

#### **Step 1: Form Initialization**
- The `form` object is passed down from the parent component. It contains methods like `watch`, `setValue`, and `register` for managing form state.

#### **Step 2: Field Registration**
- Each field component (e.g., `TextComponent`, `SelectComponent`) registers itself with the form using the `register` method. This allows `react-hook-form` to track the field's value and validation state.

#### **Step 3: Value Updates**
- When a user interacts with a field (e.g., types in a text input), the `onChange` event updates the form state via `react-hook-form`.

---

### 4. **Utility Functions**

The `utils.ts` file contains utility functions that support form rendering and value population:

#### **`fieldName`**
- Generates the field name based on the attribute's name. For root attributes (e.g., `username`), the field name is the attribute name. For custom attributes, the field name is prefixed with `attributes.`.

```typescript
export const fieldName = (name?: string) =>
  `${isRootAttribute(name) ? "" : "attributes."}${name?.replaceAll(
    ".",
    "🍺",
  )}` as FieldPath<UserFormFields>;
```

#### **`label`**
- Generates the label for a field based on the attribute's `displayName` or a fallback.

```typescript
export const label = (
  t: TFunction,
  text: string | undefined,
  fallback?: string,
  prefix?: string,
) => {
  const value = text || fallback;
  const bundleKey = isBundleKey(value) ? unWrap(value!) : value;
  const key = prefix ? `${prefix}.${bundleKey}` : bundleKey;
  return t(key || "");
};
```

---

### Summary

1. **Form Rendering**:
   - The `UserProfileFields` component groups attributes and renders them in sections using the `ScrollForm` component.
   - Each section contains fields rendered by the `FormField` component.


2. **Field Rendering**:
   - The `FormField` component determines the input type and renders the appropriate field component (e.g., `TextComponent`, `SelectComponent`).
   - The `LocaleSelector` component is used for the `locale` attribute.


3. **Value Population**:
   - The `form.watch` method from `react-hook-form` ensures that field values are always in sync with the form state.
   - The `fieldName` function generates the correct field name for each attribute.


4. **Form State Management**:
   - The `react-hook-form` library manages the form state, including field registration, value updates, and validation.


5. **Utility Functions**:
   - Functions like `fieldName` and `label` support form rendering and value population.

### Reference:
```typescript
// src/keycloak-theme/shared/keycloak-ui-shared/user-profile/UserProfileFields.tsx
import {
  UserProfileAttributeGroupMetadata,
  UserProfileAttributeMetadata,
  UserProfileMetadata,
} from "@keycloak/keycloak-admin-client/lib/defs/userProfileMetadata";
import { TFunction } from "i18next";
import { ReactNode, useMemo, type JSX } from "react";
import { FieldPath, UseFormReturn } from "react-hook-form";

import { ScrollForm } from "../scroll-form/ScrollForm";
import { LocaleSelector } from "./LocaleSelector";
import { MultiInputComponent } from "./MultiInputComponent";
import { OptionComponent } from "./OptionsComponent";
import { SelectComponent } from "./SelectComponent";
import { TextAreaComponent } from "./TextAreaComponent";
import { TextComponent } from "./TextComponent";
import { UserFormFields, fieldName, isRootAttribute, label } from "./utils";

export type UserProfileError = {
  responseData: { errors?: { errorMessage: string }[] };
};

export type Options = {
  options?: string[];
};

export type InputType =
  | "text"
  | "textarea"
  | "select"
  | "select-radiobuttons"
  | "multiselect"
  | "multiselect-checkboxes"
  | "html5-email"
  | "html5-tel"
  | "html5-url"
  | "html5-number"
  | "html5-range"
  | "html5-datetime-local"
  | "html5-date"
  | "html5-month"
  | "html5-time"
  | "multi-input";

export type UserProfileFieldProps = {
  t: TFunction;
  form: UseFormReturn<UserFormFields>;
  inputType: InputType;
  attribute: UserProfileAttributeMetadata;
  renderer?: (attribute: UserProfileAttributeMetadata) => ReactNode;
};

export type OptionLabel = Record<string, string> | undefined;

export const FIELDS: {
  [type in InputType]: (props: UserProfileFieldProps) => JSX.Element;
} = {
  text: TextComponent,
  textarea: TextAreaComponent,
  select: SelectComponent,
  "select-radiobuttons": OptionComponent,
  multiselect: SelectComponent,
  "multiselect-checkboxes": OptionComponent,
  "html5-email": TextComponent,
  "html5-tel": TextComponent,
  "html5-url": TextComponent,
  "html5-number": TextComponent,
  "html5-range": TextComponent,
  "html5-datetime-local": TextComponent,
  "html5-date": TextComponent,
  "html5-month": TextComponent,
  "html5-time": TextComponent,
  "multi-input": MultiInputComponent,
} as const;

export type UserProfileFieldsProps = {
  t: TFunction;
  form: UseFormReturn<UserFormFields>;
  userProfileMetadata: UserProfileMetadata;
  supportedLocales: string[];
  currentLocale: string;
  hideReadOnly?: boolean;
  renderer?: (
    attribute: UserProfileAttributeMetadata,
  ) => JSX.Element | undefined;
};

type GroupWithAttributes = {
  group: UserProfileAttributeGroupMetadata;
  attributes: UserProfileAttributeMetadata[];
};

export const UserProfileFields = ({
  t,
  form,
  userProfileMetadata,
  supportedLocales,
  currentLocale,
  hideReadOnly = false,
  renderer,
}: UserProfileFieldsProps) => {
  // Group attributes by group, for easier rendering.
  const groupsWithAttributes = useMemo(() => {
    // If there are no attributes, there is no need to group them.
    if (!userProfileMetadata.attributes) {
      return [];
    }

    // Hide read-only attributes if 'hideReadOnly' is enabled.
    const attributes = hideReadOnly
      ? userProfileMetadata.attributes.filter(({ readOnly }) => !readOnly)
      : userProfileMetadata.attributes;

    return [
      // Insert an empty group for attributes without a group.
      { name: undefined },
      ...(userProfileMetadata.groups ?? []),
    ].map<GroupWithAttributes>((group) => ({
      group,
      attributes: attributes.filter(
        (attribute) => attribute.group === group.name,
      ),
    }));
  }, [
    hideReadOnly,
    userProfileMetadata.groups,
    userProfileMetadata.attributes,
  ]);

  if (groupsWithAttributes.length === 0) {
    return null;
  }

  return (
    <ScrollForm
      label={t("jumpToSection")}
      sections={groupsWithAttributes
        .filter((group) => group.attributes.length > 0)
        .map(({ group, attributes }) => ({
          title: label(t, group.displayHeader, group.name) || t("general"),
          panel: (
            <div className="space-y-4"> {/* Replaced pf-v5-c-form with Tailwind CSS */}
              {group.displayDescription && (
                <p className="pb-6 text-gray-700"> {/* Replaced pf-v5-u-pb-lg with Tailwind CSS */}
                  {label(t, group.displayDescription, "")}
                </p>
              )}
              {attributes.map((attribute) => (
                <FormField
                  key={attribute.name}
                  t={t}
                  form={form}
                  supportedLocales={supportedLocales}
                  currentLocale={currentLocale}
                  renderer={renderer}
                  attribute={attribute}
                />
              ))}
            </div>
          ),
        }))}
    />
  );
};

type FormFieldProps = {
  t: TFunction;
  form: UseFormReturn<UserFormFields>;
  supportedLocales: string[];
  currentLocale: string;
  attribute: UserProfileAttributeMetadata;
  renderer?: (
    attribute: UserProfileAttributeMetadata,
  ) => JSX.Element | undefined;
};

const FormField = ({
  t,
  form,
  renderer,
  supportedLocales,
  currentLocale,
  attribute,
}: FormFieldProps) => {
  const value = form.watch(
    fieldName(attribute.name) as FieldPath<UserFormFields>,
  );
  const inputType = useMemo(() => determineInputType(attribute), [attribute]);

  const Component =
    attribute.multivalued ||
    (isMultiValue(value) && attribute.annotations?.inputType === undefined)
      ? FIELDS["multi-input"]
      : FIELDS[inputType];

  if (attribute.name === "locale")
    return (
      <LocaleSelector
        form={form}
        supportedLocales={supportedLocales}
        currentLocale={currentLocale}
        t={t}
        attribute={attribute}
      />
    );
  return (
    <Component
      t={t}
      form={form}
      inputType={inputType}
      attribute={attribute}
      renderer={renderer}
    />
  );
};

const DEFAULT_INPUT_TYPE = "text" satisfies InputType;

function determineInputType(
  attribute: UserProfileAttributeMetadata,
): InputType {
  // Always treat the root attributes as a text field.
  if (isRootAttribute(attribute.name)) {
    return "text";
  }

  const inputType = attribute.annotations?.inputType;

  // if we have an valid input type use that to render
  if (isValidInputType(inputType)) {
    return inputType;
  }

  // In all other cases use the default
  return DEFAULT_INPUT_TYPE;
}

const isValidInputType = (value: unknown): value is InputType =>
  typeof value === "string" && value in FIELDS;

const isMultiValue = (value: unknown): boolean =>
  Array.isArray(value) && value.length > 1;
###

###  src/keycloak-theme/shared/keycloak-ui-shared/user-profile/utils.ts
import { UserProfileAttributeMetadata } from "@keycloak/keycloak-admin-client/lib/defs/userProfileMetadata";
import UserRepresentation from "@keycloak/keycloak-admin-client/lib/defs/userRepresentation";
import { TFunction } from "i18next";
import { FieldPath } from "react-hook-form";

export type KeyValueType = { key: string; value: string };

export type UserFormFields = Omit<
  UserRepresentation,
  "attributes" | "userProfileMetadata"
> & {
  attributes?: KeyValueType[] | Record<string, string | string[]>;
};

type FieldError = {
  field: string;
  errorMessage: string;
  params?: string[];
};

type ErrorArray = { errors?: FieldError[] };

export type UserProfileError = {
  responseData: ErrorArray | FieldError;
};

const isBundleKey = (displayName?: string) => displayName?.includes("${");
const unWrap = (key: string) => key.substring(2, key.length - 1);

export const label = (
  t: TFunction,
  text: string | undefined,
  fallback?: string,
  prefix?: string,
) => {
  const value = text || fallback;
  const bundleKey = isBundleKey(value) ? unWrap(value!) : value;
  const key = prefix ? `${prefix}.${bundleKey}` : bundleKey;
  return t(key || "");
};

export const labelAttribute = (
  t: TFunction,
  attribute: UserProfileAttributeMetadata,
) => label(t, attribute.displayName, attribute.name);

const ROOT_ATTRIBUTES = ["username", "firstName", "lastName", "email"];

export const isRootAttribute = (attr?: string) =>
  attr && ROOT_ATTRIBUTES.includes(attr);

export const fieldName = (name?: string) =>
  `${isRootAttribute(name) ? "" : "attributes."}${name?.replaceAll(
    ".",
    "🍺",
  )}` as FieldPath<UserFormFields>;

export const beerify = <T extends string>(name: T) =>
  name.replaceAll(".", "🍺");

export const debeerify = <T extends string>(name: T) =>
  name.replaceAll("🍺", ".");

export function setUserProfileServerError<T>(
  error: UserProfileError,
  setError: (field: keyof T, params: object) => void,
  t: TFunction,
) {
  (
    ((error.responseData as ErrorArray).errors !== undefined
      ? (error.responseData as ErrorArray).errors
      : [error.responseData]) as FieldError[]
  ).forEach((e) => {
    const params = Object.assign(
      {},
      e.params?.map((p) => (isBundleKey(p.toString()) ? t(unWrap(p)) : p)),
    );
    setError(fieldName(e.field) as keyof T, {
      message: t(
        isBundleKey(e.errorMessage) ? unWrap(e.errorMessage) : e.errorMessage,
        {
          ...params,
          defaultValue: e.errorMessage || e.field,
        },
      ),
      type: "server",
    });
  });
}

export function isRequiredAttribute({
  required,
  validators,
}: UserProfileAttributeMetadata): boolean {
  // Check if required is true or if the validators include a validation that would make the attribute implicitly required.
  return required || hasRequiredValidators(validators);
}

/**
 * Checks whether the given validators include a validation that would make the attribute implicitly required.
 */
function hasRequiredValidators(
  validators?: UserProfileAttributeMetadata["validators"],
): boolean {
  // If we don't have any validators, the attribute is not required.
  if (!validators) {
    return false;
  }

  // If the 'length' validator is defined and has a minimal length greater than zero the attribute is implicitly required.
  // We have to do a lot of defensive coding here, because we don't have type information for the validators.
  if (
    "length" in validators &&
    "min" in validators.length &&
    typeof validators.length.min === "number"
  ) {
    return validators.length.min > 0;
  }

  return false;
}

export function isUserProfileError(error: unknown): error is UserProfileError {
  // Check if the error is an object with a 'responseData' property.
  if (
    typeof error !== "object" ||
    error === null ||
    !("responseData" in error)
  ) {
    return false;
  }

  const { responseData } = error;

  if (isFieldError(responseData)) {
    return true;
  }

  // Check if 'responseData' is an object with an 'errors' property that is an array.
  if (
    typeof responseData !== "object" ||
    responseData === null ||
    !("errors" in responseData) ||
    !Array.isArray(responseData.errors)
  ) {
    return false;
  }

  // Check if all errors are field errors.
  return responseData.errors.every(isFieldError);
}

function isFieldError(error: unknown): error is FieldError {
  // Check if the error is an object.
  if (typeof error !== "object" || error === null) {
    return false;
  }

  // Check if the error object has a 'field' property that is a string.
  if (!("field" in error) || typeof error.field !== "string") {
    return false;
  }

  // Check if the error object has an 'errorMessage' property that is a string.
  if (!("errorMessage" in error) || typeof error.errorMessage !== "string") {
    return false;
  }

  return true;
}
```