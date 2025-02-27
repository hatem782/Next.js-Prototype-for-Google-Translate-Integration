# Next.js Prototype for Google Translate Integration

This is a prototype for integrating **Google Translate** into a Next.js app. The integration uses **Google Translate** to provide a language switcher that allows users to change the language of the website.

### Features:

- Google Translate integration for translation.
- Language switcher component.
- Cookie-based language persistence.
- Non-blocking script loading for better performance (using `next/script`).
- Styling to hide the Google Translate UI components from the page.

---

## Getting Started

### 1. Prerequisites

- **Node.js** (v16 or higher).
- A Next.js application.

### 2. Install Dependencies

You need to install the `nookies` package to handle cookies.

```bash
npm install nookies
```

### 3. Configure Public Files

Create the following files in the `public` folder to set up the Google Translate configuration and initialization:

#### `public/assets/scripts/lang-config.js`

```javascript
window.__GOOGLE_TRANSLATION_CONFIG__ = {
  languages: [
    { title: "English", name: "en" },
    { title: "Deutsch", name: "de" },
    { title: "Español", name: "es" },
    { title: "Français", name: "fr" },
  ],
  defaultLanguage: "en",
};
```

#### `public/assets/scripts/translation.js`

```javascript
function TranslateInit() {
  if (!window.__GOOGLE_TRANSLATION_CONFIG__) {
    return;
  }
  new google.translate.TranslateElement({
    pageLanguage: window.__GOOGLE_TRANSLATION_CONFIG__.defaultLanguage,
  });
}
```

### 4. Update Layout File

Modify your `app/layout.tsx` (or equivalent) to include the necessary scripts for Google Translate. Import `Script` from `next/script` and add the following in the `<head>` section:

```tsx
import Script from "next/script";

export default function RootLayout({ children }) {
  return (
    <html lang="en">
      <head>
        <Script
          src="/assets/scripts/lang-config.js"
          strategy="beforeInteractive"
        />
        <Script
          src="/assets/scripts/translation.js"
          strategy="beforeInteractive"
        />
        <Script
          src="//translate.google.com/translate_a/element.js?cb=TranslateInit"
          strategy="afterInteractive"
        />
      </head>
      <body>{children}</body>
    </html>
  );
}
```

### 5. Add Language Switcher Component

Create a new component file `components/LangSwitch.tsx` to handle language switching:

```tsx
"use client";
import { useEffect, useState } from "react";
import { parseCookies, setCookie } from "nookies";

const COOKIE_NAME = "googtrans";

interface LanguageDescriptor {
  name: string;
  title: string;
}

declare global {
  namespace globalThis {
    var __GOOGLE_TRANSLATION_CONFIG__: {
      languages: LanguageDescriptor[];
      defaultLanguage: string;
    };
  }
}

const LanguageSwitcher = () => {
  const [currentLanguage, setCurrentLanguage] = useState<string>();
  const [languageConfig, setLanguageConfig] = useState<any>();

  useEffect(() => {
    const cookies = parseCookies();
    const existingLanguageCookieValue = cookies[COOKIE_NAME];

    let languageValue;
    if (existingLanguageCookieValue) {
      const sp = existingLanguageCookieValue.split("/");
      if (sp.length > 2) {
        languageValue = sp[2];
      }
    }
    if (global.__GOOGLE_TRANSLATION_CONFIG__ && !languageValue) {
      languageValue = global.__GOOGLE_TRANSLATION_CONFIG__.defaultLanguage;
    }
    if (languageValue) {
      setCurrentLanguage(languageValue);
    }
    if (global.__GOOGLE_TRANSLATION_CONFIG__) {
      setLanguageConfig(global.__GOOGLE_TRANSLATION_CONFIG__);
    }
  }, []);

  if (!currentLanguage || !languageConfig) {
    return null;
  }

  const switchLanguage = (lang: string) => () => {
    setCookie(null, COOKIE_NAME, "/auto/" + lang);
    window.location.reload();
  };

  return (
    <div className="text-center notranslate">
      {languageConfig.languages.map((ld: LanguageDescriptor, i: number) => (
        <>
          {currentLanguage === ld.name ||
          (currentLanguage === "auto" &&
            languageConfig.defaultLanguage === ld.name) ? (
            <span key={`l_s_${ld.name}`} className="mx-3 text-orange-300">
              {ld.title}
            </span>
          ) : (
            <a
              key={`l_s_${ld.name}`}
              onClick={switchLanguage(ld.name)}
              className="mx-3 text-blue-300 cursor-pointer hover:underline"
            >
              {ld.title}
            </a>
          )}
        </>
      ))}
    </div>
  );
};

export { LanguageSwitcher, COOKIE_NAME };
```

### 6. Apply Styling

Update `app/globals.css` to hide Google Translate elements and style the page:

```css
body {
  color: rgb(var(--foreground-rgb));
  background: linear-gradient(
      to bottom,
      transparent,
      rgb(var(--background-end-rgb))
    ) rgb(var(--background-start-rgb));
  padding: 5px;
  top: 0px !important;
}

.skiptranslate {
  display: none !important;
}
```

> **Note**: The `notranslate` class in the `LanguageSwitcher` component ensures that language titles (e.g., "English", "Deutsch") are not translated by Google Translate.

---

## Usage

1. Start the Next.js app:
   ```bash
   npm run dev
   ```
2. Open the app in your browser (default: `http://localhost:3000`).
3. Use the language switcher to toggle between supported languages (English, German, Spanish, French).

## How It Works

- **Scripts**: `lang-config.js` defines the available languages and default language, while `translation.js` initializes Google Translate.
- **Cookies**: The `nookies` package manages the `googtrans` cookie to persist the user's language choice.
- **Component**: `LanguageSwitcher` reads the cookie, displays the current language, and allows switching via clickable links.
- **Styling**: CSS hides Google Translate's default UI elements while keeping the custom switcher visible.

## Notes

- An internet connection is required to load the Google Translate script from `//translate.google.com`.
- The `notranslate` class is critical to prevent translation of the language switcher text.

## License

This project is unlicensed and intended for prototyping purposes only.
