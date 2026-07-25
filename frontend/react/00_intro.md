# What is React?

**React** is a JavaScript library developed by Meta (Facebook) for building modern, dynamic, and responsive user interfaces (UI).  
It allows you to create **reusable components**, manage an application's state, and improve performance thanks to the **Virtual DOM**.

## Why use React?

- **Reusable components**: structure your app with logical building blocks  
- **Fast** thanks to the Virtual DOM  
- **Automatic UI updates** when data changes  
- **Rich ecosystem** (React Router, Redux, Tailwind CSS, etc.)  
- **Highly in demand** in the industry (modern frontend)

## Install React with Vite

### Step 1 – Create a new project

```bash
npm create vite@latest project-name
cd project-name
```

> Replace `project-name` with the name of your app (e.g., cinequest)

### Step 2 – Install dependencies

```bash
npm install
```

### Step 3 – Start the development server

```bash
npm run dev
```

Your app will be available at:

```
http://localhost:5173
```

### Initial project structure

```
project-name/
├── public/
├── src/
│   ├── App.jsx
│   ├── main.jsx
│   └── index.css
├── index.html
├── vite.config.js
└── package.json
```

---

### Scalable structure
```text
src/
├── components/         # All reusable components (buttons, cards, etc.)
├── pages/              # Full pages (if you're using React Router or Next.js)
├── layouts/            # Global layouts (Navbar + Footer, etc.)
├── hooks/              # Custom hooks (e.g., useForm, useAuth)
├── context/            # Context API for global state management
├── assets/             # Images, icons, static files
├── styles/             # CSS files or custom Tailwind styles
├── utils/              # Utility functions (date formatting, etc.)
├── services/           # API calls or business logic (e.g., authService.js)
├── App.js              # Root component
└── index.js            # Application entry point
```

---

## Adding Tailwind CSS and shadcn/ui

### Why this setup

A fresh Vite + React project has no styling system beyond plain CSS. Tailwind CSS provides utility classes written directly in JSX (`className="flex items-center"`), while shadcn/ui provides accessible, pre-built React components (buttons, cards, dialogs...) copied directly into the project rather than installed as an opaque dependency.

### Step 1 — Install Tailwind CSS

Tailwind v4 removed the old `tailwind.config.js` file and the PostCSS setup used in v3. Configuration now happens directly in CSS.

```bash
npm install tailwindcss @tailwindcss/vite
```

Register the Tailwind plugin in `vite.config.ts`, alongside the React plugin:

```ts
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'
import tailwindcss from '@tailwindcss/vite'

export default defineConfig({
  plugins: [react(), tailwindcss()],
})
```

Replace the entire content of `src/index.css` with a single import:

```css
@import "tailwindcss";
```

Tailwind scans every component file for class names in use and generates only the corresponding CSS — unused utility classes never ship to the browser.

### Step 2 — Configure the `@/` path alias

shadcn/ui relies on imports like `@/components/ui/button` instead of relative paths. This alias must be declared in **two places**, because Vite splits TypeScript config across multiple files.

`tsconfig.json`:

```jsonc
{
  "files": [],
  "references": [
    { "path": "./tsconfig.app.json" },
    { "path": "./tsconfig.node.json" }
  ],
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@/*": ["./src/*"]
    }
  }
}
```

`tsconfig.app.json` (the config that actually compiles `src/` — the alias must sit *inside* `compilerOptions`):

```jsonc
{
  "compilerOptions": {
    // ...existing options
    "baseUrl": ".",
    "paths": {
      "@/*": ["./src/*"]
    }
  },
  "include": ["src"]
}
```

### Step 3 — Make Vite resolve the same alias

TypeScript and Vite are separate systems — each needs to know about `@/` independently.

```bash
npm install -D @types/node
```

```ts
import path from "path"
import tailwindcss from "@tailwindcss/vite"
import react from "@vitejs/plugin-react"
import { defineConfig } from "vite"

export default defineConfig({
  plugins: [react(), tailwindcss()],
  resolve: {
    alias: {
      "@": path.resolve(__dirname, "./src"),
    },
  },
})
```

### Step 4 — Initialize shadcn/ui

```bash
npx shadcn@latest init
```

The CLI prompts for a preset — theme, font, and icons. The recommended preset at time of writing is **Nova** (Lucide icons, Geist font): a neutral, modern default, easy to restyle later.

This generates:
- `components.json` — the CLI's own config file
- `src/lib/utils.ts` — exports `cn()`, used by every shadcn component to merge Tailwind classes safely
- Additional imports appended to `src/index.css`:

```css
@import "tailwindcss";
@import "tw-animate-css";
@import "shadcn/tailwind.css";
@import "@fontsource-variable/geist";
```

### Step 5 — Add components on demand

Each component's source code is copied directly into the project, not installed via `node_modules`:

```bash
npx shadcn@latest add button
```

Creates `src/components/ui/button.tsx` — a real, editable file owned by the project. Add several at once as needed:

```bash
npx shadcn@latest add button card input dialog
```

Usage:

```tsx
import { Button } from "@/components/ui/button"

function App() {
  return (
    <div className="flex min-h-svh flex-col items-center justify-center">
      <Button>Click me</Button>
    </div>
  )
}

export default App
```

### Cleaning up the default Vite template CSS

The Vite React template ships with a default `App.css` (styling the spinning-logo demo page). Delete it entirely, along with its `import './App.css'` in `App.tsx`. Going forward, `index.css` is the only global stylesheet needed — styling happens through Tailwind classes and shadcn components in JSX.

---

## Alternative considered: DaisyUI

**DaisyUI** is a Tailwind plugin, not a component library — it adds class names (`btn`, `card`, `modal`...) that expand into pre-styled CSS, but markup is still written by hand.

```bash
npm install daisyui
```

```css
@import "tailwindcss";
@plugin "daisyui";
```

```tsx
<button className="btn btn-primary">Click me</button>
```

No CLI, no files added — everything comes from the single `@plugin` line.

**Decision:** shadcn/ui was chosen instead, since it's part of the target enterprise stack for the Next.js roadmap, while DaisyUI isn't.