# Afficher la liste des tâches

## Sommaire

## But de l'exercice

## Modéliser une liste de tâches dans le JS

Créons tout d'abord la liste des tâches (en dur pour l'instant) au sein de notre fichier src/app.js

```js
// src/app.js

// On créé ici un tableau TODO_ITEMS qui contient deux objets 
const TODO_ITEMS = [
  { id: 1, text: "Faire les courses", done: false },
  { id: 2, text: "Aller chercher les enfants", done: true },
];
```

```jsx
// React va permettre de dessiner notre arbre d'éléments HTML
import React from "react";
// ReactDOM va permettre de créer le rendu correspondant dans le DOM HTML
import ReactDOM from "react-dom";

const TodoList = () => {
    // On retourne une list <ul> qui contient un tableau d'éléments React
    // Chaque objet de TODO_ITEMS sera transformé en un <li> contenant les détails de la tâche
    // Les éléments React générés par une boucle doivent avoir une "key" unique
    return <ul>
        {TODO_ITEMS.map(item => <li key={item.id}>
            <label>
                <input type="checkbox" id="todo-${item.id}" checked={item.done} />
                {item.text}
            </label>
        </li>)}
    </ul>
}

// Imprime l'arbre renvoyé par TodoList() dans l'élément <main> du DOM HTML
ReactDOM.render(<TodoList />, document.querySelector('main'));
```

## Impossibilité d'agir sur les checkbox


## Dynamisme des composants : la notion d'état (state)

```diff
+ import React, { useState } from "react";


const TodoList = () => {
+    // Créons un état (qui pourra évoluer dynamiquement) à partir du tableau TODO_ITEMS
+    const [state, setState] = useState(TODO_ITEMS);

+    const toggle = (id) => {
+        // Récupérons l'index de la tâche concernée
+        const idx = state.findIndex(task => task.id === id);
+        // Créons une copie de la tâche concernée tout en modifiant son état
+        const item = { ...state[idx], done: !state[idx].done };
+        // Créons une copie du tableau d'origine
+        const stateCopy = [...state];
+        // Enfin remplaçons la tâche originale par la copie :
+        stateCopy[idx] = item;
+        // Et faisons évoluer le state : l'ancien tableau sera
+        // remplacé par le nouveau, et le rendu sera déclenché à nouveau
+        setState(stateCopy);
+    }

    // On retourne une list <ul> qui contient un tableau d'éléments React
    // Chaque objet de TODO_ITEMS sera transformé en un <li> contenant les détails de la tâche
    // Les éléments React générés par une boucle doivent avoir une "key" unique
    return <ul>
        {state.map(item => <li key={item.id}>
            <label>
                <input 
                    type="checkbox" 
                    id="todo-${item.id}" 
                    checked={item.done} 
+                   onChange={() => toggle(item.id)} 
                />
                {item.text}
            </label>
        </li>)}
    </ul>
}
```

## Refactoring avant de continer

```jsx
// src/pages/TodoList.js
import React, { useState } from "react";

// On créé ici un tableau TODO_ITEMS qui contient deux objets 
const TODO_ITEMS = [
    { id: 1, text: "Faire les courses", done: false },
    { id: 2, text: "Aller chercher les enfants", done: true },
];

const TodoList = () => {
    // Créons un état (qui pourra évoluer dynamiquement) à partir du tableau TODO_ITEMS
    const [state, setState] = useState(TODO_ITEMS);

    const toggle = (id) => {
        // Récupérons l'index de la tâche concernée
        const idx = state.findIndex(task => task.id === id);
        // Créons une copie de la tâche concernée tout en modifiant son état
        const item = { ...state[idx], done: !state[idx].done };
        // Créons une copie du tableau d'origine
        const stateCopy = [...state];
        // Enfin remplaçons la tâche originale par la copie :
        stateCopy[idx] = item;
        // Et faisons évoluer le state : l'ancien tableau sera
        // remplacé par le nouveau, et le rendu sera déclenché à nouveau
        setState(stateCopy);
    }

    // On retourne une list <ul> qui contient un tableau d'éléments React
    // Chaque objet de TODO_ITEMS sera transformé en un <li> contenant les détails de la tâche
    // Les éléments React générés par une boucle doivent avoir une "key" unique
    return <ul>
        {state.map(item => <li key={item.id}>
            <label>
                <input type="checkbox" id="todo-${item.id}" checked={item.done} onChange={() => toggle(item.id)} />
                {item.text}
            </label>
        </li>)}
    </ul>
}

export default TodoList;
```

```jsx
// src/app.js
// React va permettre de dessiner notre arbre d'éléments HTML
import React from "react";
// ReactDOM va permettre de créer le rendu correspondant dans le DOM HTML
import ReactDOM from "react-dom";
// On importe la fonction TodoList
import TodoList from "./pages/TodoList";


// Imprime l'arbre renvoyé par TodoList() dans l'élément <main> du DOM HTML
ReactDOM.render(<TodoList />, document.querySelector('main'));
```

# Ce que vous avez appris