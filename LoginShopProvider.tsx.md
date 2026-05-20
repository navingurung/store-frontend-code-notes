# LoginShopProvider.tsx — Overview

## What is this file?

A **global state container** for the app.
It stores who is logged in and shares that information to every component.

> It does NOT call any API. It only receives, stores, and shares data.

---

## Types

### `Shop`
Represents the logged-in shop. Comes from `/auth/token` API response.

```typescript
type Shop = {
  id: string;           // shop ID
  shop_name: string;    // shop name
  use_shopify: boolean; // feature flag — is Shopify connected?
  use_trj: boolean;     // feature flag — is TRJ connected?
  product_list: string[]; // list of products
};
```

### `LoginShopContextType`
Defines what data and functions are shared to all components.

```typescript
type LoginShopContextType = {
  loginShop: Shop | null;                    // current logged-in shop
  setLoginShop: (shop: Shop | null) => void; // write shop data in
  isLoggedIn: boolean;                       // true / false
  setIsLoggedIn: (value: boolean) => void;   // manually set login state
  logout: () => void;                        // clear everything
};
```

---

## Functions & State — What Each One Does

### 1. `loginShop` state
```typescript
const [loginShop, setLoginShopState] = useState<Shop | null>(() => {
  const raw = sessionStorage.getItem("authShop");
  return raw ? JSON.parse(raw) : null;
});
```
- Holds the current logged-in shop object
- On first load → checks sessionStorage for saved shop data
- Found → restores it (user stays logged in after refresh)
- Not found → null (user is not logged in)

---

### 2. `isLoggedIn` state
```typescript
const [isLoggedIn, setIsLoggedInState] = useState<boolean>(() => {
  return !!sessionStorage.getItem("authShop");
});
```
- Simple true/false flag
- On first load → true if sessionStorage has authShop, false if not
- Automatically managed by useEffect (no need to set manually)

---

### 3. `useEffect` — sessionStorage sync
```typescript
useEffect(() => {
  if (loginShop) {
    sessionStorage.setItem("authShop", JSON.stringify(loginShop));
    setIsLoggedInState(true);
  } else {
    sessionStorage.removeItem("authShop");
    setIsLoggedInState(false);
  }
}, [loginShop]);
```
- Runs every time `loginShop` changes
- loginShop has data → save to sessionStorage, isLoggedIn = true
- loginShop is null → remove from sessionStorage, isLoggedIn = false
- Keeps state and sessionStorage always in sync automatically

---

### 4. `logout` function
```typescript
const logout = () => {
  sessionStorage.removeItem("authShop");
  setLoginShopState(null);
};
```
- Clears authShop from sessionStorage
- Sets loginShop = null
- useEffect then sets isLoggedIn = false automatically
- Any component can call this to log the user out

---

### 5. `LoginShopContext.Provider` — the wrapper
```typescript
<LoginShopContext.Provider value={{ loginShop, setLoginShop, isLoggedIn, setIsLoggedIn, logout }}>
  {children}
</LoginShopContext.Provider>
```
- Wraps all child components
- Shares loginShop, isLoggedIn, setLoginShop, logout to everyone inside
- Any component can access these via useContext(LoginShopContext)

---

## What happens after wrapping

Because `LoginShopProvider` wraps the entire app in `App.tsx`:

```
Any component anywhere in the app can do:

const { loginShop, isLoggedIn, logout } = useContext(LoginShopContext);
```

| Component | What it uses |
|---|---|
| `Login` | `setLoginShop()` → writes shop data after API success |
| `Whoami` | `setLoginShop()` or `logout()` → based on session validity |
| `PrivateRoute` | `isLoggedIn` → allow or redirect to /login |
| Any page | `loginShop` → read shop name, product list, feature flags |

---

## Full Data Flow

```
FIRST TIME LOGIN
User enters ID + Password
        ↓
Login calls POST /auth/token
        ↓
Server responds with shop data + sets token in httpOnly cookie
        ↓
Login calls setLoginShop(shopData)
        ↓
loginShop state updates
        ↓
useEffect saves to sessionStorage
        ↓
isLoggedIn = true
        ↓
PrivateRoute allows access → Home page

PAGE REFRESH
sessionStorage still has authShop
        ↓
loginShop restored on mount
        ↓
isLoggedIn = true
        ↓
User stays logged in

LOGOUT
logout() called
        ↓
sessionStorage cleared
        ↓
loginShop = null
        ↓
isLoggedIn = false
        ↓
PrivateRoute redirects to /login

TAB CLOSED
Browser clears sessionStorage automatically
        ↓
Next visit → loginShop = null
        ↓
User must login again
```

---

## Simple Mental Model

```
LOGIN/WHOAMI  →  "here is the shop data"  →  LoginShopProvider
                                                    ↓
                                             saves to sessionStorage
                                                    ↓
ALL COMPONENTS  ←  "here is the shop data"  ←  shares via useContext
```

> `LoginShopProvider` is the postbox.
> `Login/Whoami` delivers the letter.
> Every component reads from the postbox.
