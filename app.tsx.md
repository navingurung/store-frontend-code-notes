# App.tsx — Overview

## What is App.tsx?

`App.tsx` is the **root component** of the application. It sets up global providers, side-effect components, routing, and the shared footer.

---

## Component Structure

```
SnackbarProvider
└── LoginShopProvider
    ├── Whoami                  ← fetches current user on mount
    ├── ScrollToTop             ← resets scroll position on route change
    ├── Routes
    │   ├── /login              → Login page (public, no auth required)
    │   ├── PrivateRoute        ← auth guard
    │   │   ├── /               → Home page
    │   │   └── /completed      → Completed page
    │   └── *                   → 404 Not Found
    └── Footer                  ← always visible on all pages
```

---

## Providers

| Provider | Purpose |
|---|---|
| `SnackbarProvider` | Enables toast notifications anywhere in the app (max 3 at a time) |
| `LoginShopProvider` | Holds global auth state and shop-related state |

---

## Side-effect Components

These components **render nothing visually** — they only run logic.

| Component | What it does |
|---|---|
| `Whoami` | Calls an API on mount to fetch the logged-in user and hydrate auth state |
| `ScrollToTop` | Listens to route changes and scrolls the page back to the top |

---

## Routing

| Path | Component | Auth Required |
|---|---|---|
| `/login` | `Login` | No |
| `/` | `Home` | Yes |
| `/completed` | `Completed` | Yes |
| `*` (anything else) | 404 message | No |

### PrivateRoute

`PrivateRoute` is an **auth guard**. It wraps protected routes and checks if the user is authenticated.

- ✅ Authenticated → renders the requested page
- ❌ Not authenticated → redirects to `/login`

---

## Footer

The `Footer` is placed **outside of `<Routes>`**, which means it renders on **every page** — including `/login` and the 404 page.

---

## i18n

`import "./shared/i18n/config"` initializes **internationalization (i18n)** at the app level, making translations available across all components.
