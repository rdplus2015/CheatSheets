# React – Hooks 

## What is a Hook?

A hook is a special function provided by React (or custom-built) that lets a component "hook into" internal React mechanisms — memory (state), side effects, context, etc. — things a plain JavaScript function can't do on its own. The name comes from the idea of *hooking* your component into one of React's internal systems.

### The problem hooks solve

A plain JS function forgets everything between calls:

```js
function compte() {
  let x = 0;
  x = x + 1;
  return x;
}
compte(); // 1
compte(); // 1 again — x always resets to 0
```

A React component function is re-called on every re-render, so without a special mechanism it would lose its state the same way. `useState` is exactly what lets React preserve a value *outside* the function body, in memory tied to that specific component instance, and hand it back on every call.

Before hooks (pre React 16.8), this kind of memory and lifecycle access required writing class components (`this.state`, `componentDidMount`...). Hooks made it possible to get the same capabilities inside a plain function.

### The general pattern behind every hook

| Hook | What it hooks into |
|---|---|
| `useState` | Memory (state) that survives across re-renders |
| `useEffect` | A specific lifecycle moment (after render, on mount, on unmount) |
| `useRef` | A mutable value that survives re-renders without triggering one |
| `useContext` | Globally shared data, without prop drilling |
| `useMemo` / `useCallback` | A cache to avoid recomputing a value/function unnecessarily |

All of them follow the same shape: a function starting with `use`, giving access to something React manages internally on the component's behalf.

### Rules of Hooks — and why they exist

1. **Only called inside a component** (or another custom hook) — React needs to know which component a given piece of memory belongs to.

2. **Never inside `if`, loops, or nested functions** — React identifies each hook **by call order**, not by name. If a hook is sometimes called and sometimes skipped depending on a condition, the order shifts between renders, and React ends up attaching the wrong memory to the wrong hook.

```jsx
function MyComponent() {
  const [count, setCount] = useState(0);   // hook #1
  const [name, setName] = useState("Ridi"); // hook #2

  // React internally tracks: "position 1 = count, position 2 = name"
  // If a hook were called conditionally, this order would shift
  // and everything would get mismatched.
}
```

### In one sentence

A hook is the entry point that lets a plain JavaScript function become a real, "stateful" React component — with memory, side effects, and access to internal mechanisms — without needing a class.

##  Pourquoi les hooks ?

Le principal intérêt de React est de **créer une interface réactive**, qui évolue selon les interactions de l’utilisateur.  
Grâce aux **hooks**, un composant peut :
- Avoir une **mémoire interne (état)** accessible uniquement à lui
- Modifier cet état et **re-rendre automatiquement** l’interface

> L'état représente les données propres à un composant : ce sont des valeurs qui appartiennent uniquement à ce composant et qui peuvent changer dans le temps.
> Les hooks ne peuvent être utilisés qu’à l’intérieur d’un composant fonctionnel. On ne peut pas les appeler à l’extérieur d’un composant, ni dans des conditions (if, for, etc.).
> - Le **principe de base** : à chaque fois que **l’état change**, React **relance l’exécution du composant** (appel de la fonction) pour générer un nouveau JSX.  
Ensuite, **React compare l’ancien JSX au nouveau**, et met à jour **uniquement ce qui a changé** dans le DOM (grâce au Virtual DOM).

---

## Le hook `useState`

```jsx
import { useState } from 'react';

export function Counter() {
  const [count, setCount] = useState(0);
  const increment = () => setCount(count + 1);

  return (
    <div>
      <p>Compteur : {count}</p>
      <button onClick={increment}>Incrémenter</button>
    </div>
  );
}
```

- `useState(0)` crée une **valeur d’état** initiale (`0`)
- `count` contient la valeur actuelle
- `setCount` est une fonction pour **mettre à jour** l’état

---

##  Deux manières d'utiliser `setCount`

```js
setCount(3);             // valeur directe
setCount(count => count  + 1);    // avec fonction prenant l'ancienne valeur
```

---

## Comment React réagit ?

1. **Premier rendu** :
    - React appelle `Counter()` → retourne du JSX
    - `useState` réserve un espace mémoire pour la valeur (`0`)
    - React DOM affiche le JSX

2. **Changement d’état via `setCount`** :
    - React **met à jour la mémoire**
    - Il **ré-appelle `Counter()`**
    - Compare l’ancien et le nouveau JSX
    - Met à jour uniquement ce qui a changé dans le DOM

---

## Caractéristiques d’un Hook

- Toujours nommé en commençant par `use` 
- Ne peut être appelé qu’**à l’intérieur d’un composant React**
- **Toujours dans le même ordre**, pas dans des `if`, `for`, `while`, etc.
- **Toujours appelé au même nombre de fois** (pas conditionnel)


les hooks comme `useState`, `useEffect`, etc., **sont automatiquement appelés à chaque rendu du composant**, **dans l’ordre dans lequel ils ont été écrits** dans la fonction

```jsx
function MyComponent() {
  const [count, setCount] = useState(0);
  const [name, setName] = useState("Ridi");
  return <div>{count} - {name}</div>;
}
```
-  React appelle `MyComponent()` (comme une fonction normale)  
- Il **exécute ton code ligne par ligne**, donc :

1.  Appelle `useState(0)` → crée une **boîte mémoire 1**

2.  Appelle `useState("Ridi")` → crée une **boîte mémoire 2**


À chaque fois que le composant est **re-rendu**, React refait **toute la fonction**, mais **associe les bonnes valeurs aux bons hooks grâce à l’ordre**.


##  Bonnes pratiques

- Peut stocker **des valeurs complexes** (objets, tableaux, etc.)
- ❌ **Ne pas muter directement** un objet ou tableau :

```jsx

function App() {


      const [person, setPerson] = useState({
        age: 10,
        name:'john',
      })

      const increment = ()=> {
          /*person ++
          setPerson(person) // Mutation ne marchera pas , il faut donc creer un nouvel object */
        setPerson({
            ...person,              // copie toutes les propriétés actuelles
            age: person.age + 1     // modifie uniquement `age`
        });

      }

  return <div>
      {person}
  </div>
}
```
Dans le cadre d'un tableau, on ne les mutera pas directement : 
- Pour ajouter : `setTodos([...todos, "Nouvel élément"])`
- Pour supprimer : `setTodos(todos.filter(t => t !== "cible"))`
- Un composant peut avoir **plusieurs hooks** (`useState`, `useEffect`, etc.)

---
