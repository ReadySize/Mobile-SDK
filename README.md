# ReadySize React Native SDK

Embed our 3‑step sizing modal into any React Native **or Expo** app so your shoppers always pick the perfect fit.

---

## Table of Contents

1. [Features](#features)
2. [Requirements](#requirements)
3. [Installation](#installation)

   - [Expo / Expo Router](#expo--expo-router)
   - [Bare React Native](#bare-react-native)

4. [Quick Start](#quick-start)
5. [API Reference](#api-reference)
6. [Internationalisation (i18n)](#internationalisation-i18n)
7. [Navigation Recipes](#navigation-recipes)
8. [Troubleshooting](#troubleshooting)

---

## Features

- **Drop‑in sizing modal** — covers the screen and returns the recommended size.
- **Locale‑aware** — ships with _es‑ES, pt‑PT, en‑US, fr‑FR, de‑DE, nl‑NL_ and falls back to English.
- **TypeScript‑first** — full typings for every public API.
- **Pure‑JS** — _no native modules_, so no Cocoapods or Gradle edits 🎉.

---

## Requirements

| Package                                     | Minimum Version | Notes             |
| ------------------------------------------- | --------------- | ----------------- |
| `react`                                     | **18.2.0**      | peer dependency   |
| `react-native`                              | **0.76.0**      | 0.71+ also works  |
| `@react-native-async-storage/async-storage` | **1.23.1**      | caches user input |
| `react-intl`                                | **7.0.1**       | runtime i18n      |
| `react-native-svg`                          | **15.8.0**      | vector assets     |
| `@react-native-picker/picker`               | **2.9.0**       | size picker       |

> **Expo users** already get compatible versions of `react`, `react-native`, and `react-native-svg`. The remaining peers are installed automatically when you add the SDK.

---

## Installation

> **Private registry access**   This package is published under the _readysize_ organization with **restricted access**. You’ll need the personal or CI token we provide before you can install.
>
> ```bash
> # one‑time setup (terminal)
> npm config set //registry.npmjs.org/:_authToken="YOUR_READYSIZE_TOKEN"
> # or in CI add an environment variable ⬩  NPM_TOKEN=YOUR_READYSIZE_TOKEN
> ```

### Expo / Expo Router

```bash
npx expo install @readysize/readysizesdk
# Expo will also install:
#   @react-native-async-storage/async-storage
#   @react-native-picker/picker
#   react-intl
#   react-native-svg
```

Ensure **package.json** has the Expo Router entry point:

```json
{
  "main": "expo-router/entry"
}
```

### Bare React Native

```bash
npm install @readysize/readysizesdk react@^18.2.0 react-native@^0.76.0 @react-native-async-storage/async-storage@^1.23.1 react-intl@^7.0.1 react-native-svg@^15.11.1 @react-native-picker/picker@^2.11.0
```

Because the SDK is **100 % JavaScript**, there is nothing to link and **no** `pod install` or `build.gradle` edits required.

---

## Quick Start

```tsx
// HomeScreen.tsx (Expo example)
import React, { useState } from "react";
import { View, TextInput, Button, StyleSheet, Text } from "react-native";
import { Picker } from "@react-native-picker/picker";
import { useRouter } from "expo-router";

const languageOptions = [
  { code: "es-ES", emoji: "🇪🇸", label: "Español" },
  { code: "pt-PT", emoji: "🇧🇷", label: "Português" },
  { code: "en-US", emoji: "🇺🇸", label: "English" },
  { code: "fr-FR", emoji: "🇫🇷", label: "Français" },
  { code: "de-DE", emoji: "🇩🇪", label: "Deutsch" },
  { code: "nl-NL", emoji: "🇳🇱", label: "Nederlands" },
];

export default function HomeScreen() {
  const router = useRouter();

  const [sku, setSku] = useState("");
  const [token, setToken] = useState("");
  const [modalLanguage, setModalLanguage] = useState("es-ES");

  const handleStart = () => {
    if (!sku || !token) {
      alert("Please provide both Token and SKU");
      return;
    }

    router.push({
      pathname: "/modal",
      params: { sku, token, culture: modalLanguage },
    });
  };

  return (
    <View style={styles.container}>
      <Text style={styles.label}>Public token</Text>
      <TextInput
        style={styles.input}
        value={token}
        onChangeText={setToken}
        placeholder="e.g. readysize.shop"
      />

      <Text style={styles.label}>Product SKU</Text>
      <TextInput
        style={styles.input}
        value={sku}
        onChangeText={setSku}
        placeholder="e.g. S10253501"
      />

      <Text style={styles.label}>Modal language</Text>
      <Picker
        selectedValue={modalLanguage}
        onValueChange={setModalLanguage}
        style={styles.picker}
      >
        {languageOptions.map((l) => (
          <Picker.Item
            key={l.code}
            label={`${l.emoji} ${l.label}`}
            value={l.code}
          />
        ))}
      </Picker>

      <Button title="Open Size Modal" onPress={handleStart} />
    </View>
  );
}

const styles = StyleSheet.create({
  container: { padding: 20 },
  label: { marginTop: 16, marginBottom: 4, fontWeight: "600" },
  input: {
    borderWidth: 1,
    borderColor: "#ccc",
    borderRadius: 6,
    padding: 8,
  },
  picker: { height: 48, width: "100%" },
});
```

```tsx
// app/modal.tsx – screen that hosts the SDK
import { ReadysizeAppNative } from "@readysize/readysizesdk";
import { useLocalSearchParams, useRouter } from "expo-router";

export default function Modal() {
  const { sku, token, culture } = useLocalSearchParams();
  const router = useRouter();

  return (
    <ReadysizeAppNative
      sku={String(sku)}
      token={String(token)}
      cultureDefault={String(culture)}
      sizeRanges={[]}
      garmentFitType=""
      onCloseApp={router.back}
      onAcceptSize={(val) => {
        console.log("Recommended size", val);
        router.back();
      }}
    />
  );
}
```

---

## API Reference

### `<ReadysizeAppNative />` Props

| Prop             | Type                                 | Required | Description                             |     |                   |
| ---------------- | ------------------------------------ | -------- | --------------------------------------- | --- | ----------------- |
| `sku`            | `string`                             | ✅       | Product identifier                      |     |                   |
| `token`          | `string`                             | ✅       | Public token provided by ReadySize      |     |                   |
| `cultureDefault` | `string`                             | –        | Locale tag (e.g. `"en-US"`)             |     |                   |
| `sizeRanges`     | `string[]`                           | –        | Restrict selectable sizes               |     |                   |
| `garmentFitType` | `number    `                         | -        | id:1 Regular id:2 Ajustado id:3 Holgado | –   | Optional UX tweak |
| `onCloseApp`     | `() => void`                         | –        | Called when modal closes                |     |                   |
| `onAcceptSize`   | `(payload: RecommendedSize) => void` | ✅       | Fires when user accepts size            |     |                   |

---

## Internationalisation (i18n)

The SDK ships its own translations via **react-intl**. Pass `cultureDefault` to override auto‑detection.

Need your host UI translated too? Wrap the app with `LanguageProvider`:

```tsx
import { LanguageProvider } from "@readysize/readysizesdk/context/LanguageContext";

<LanguageProvider initialLocale="fr-FR">
  <App />
</LanguageProvider>;
```

---

## Navigation Recipes

### Expo Router

- `app/index.tsx` pushes to `app/modal.tsx`.
- Ensure `"main": "expo-router/entry"` is in **package.json**.

### React Navigation

```tsx
import { NavigationContainer } from "@react-navigation/native";
import { createStackNavigator } from "@react-navigation/stack";

const Stack = createStackNavigator();

<NavigationContainer>
  <Stack.Navigator screenOptions={{ headerShown: false }}>
    <Stack.Screen name="Home" component={HomeScreen} />
    <Stack.Screen
      name="SizeModal"
      component={ModalScreen}
      options={{ presentation: "modal" }}
    />
  </Stack.Navigator>
</NavigationContainer>;
```

---

## Troubleshooting

| Symptom                                             | Fix                                                                  |
| --------------------------------------------------- | -------------------------------------------------------------------- |
| **White screen** when modal opens                   | Check peer‑dependency versions; clear Metro cache (`expo start -c`). |
| Android build fails with “duplicate class \_Picker” | Remove `@react-native-picker/picker-legacy`.                         |
| Text appears in the wrong language                  | Verify `cultureDefault`; default is device locale or `en-US`.        |

---
