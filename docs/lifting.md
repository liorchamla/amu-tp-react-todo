# Remontée d'état : communication entre composants

```jsx
// src/components/TaskForm.js
import React, { useState } from "react";

const TaskForm = () => {
    // Créons un état (qui pourra évoluer dynamiquement) qui représentera
    // la chaîne tapée par l'utilisateur dans l'<input>
    const [text, setText] = useState('');

    // La fonction sera appelée à chaque changement sur l'<input>
    // Elle recevra les détails de l'événement
    const updateText = (event) => {
        // On fait évoluer le state "text" en remplaçant la valeur
        // acutelle par la valeur mise à jour de l'<input>
        setText(event.target.value);
    }

    // Cette fonction sera appelée lors de la soumission du <form> et recevra les détails de l'événement
    const handleSubmit = (event) => {
        // Annulons le comportement par défaut de l'événement
        // qui serait de recharger la page
        event.preventDefault();

        // Créons une nouvelle tâche avec le text tapé dans l'input
        const task = {
            id: Date.now(),
            text: text,
            done: false
        };

        // Remplaçons le tableau de tâches actuel par une copie
        // qui contiendra en plus la nouvelle tâche :
        setState([...state, task]);
    }
    return <form onSubmit={handleSubmit}>
        <input
            type="text"
            name="todo-text"
            placeholder="Ajouter une tâche"
            onChange={updateText}
            value={text}
        />
        <button>Ajouter</button>
    </form>
}
```

```jsx
// src/components/TodoList.js

import React, { useState } from "react";
import TaskForm from "./TaskForm.js";

const TodoList = () => {
    // ...

    return <>
        <ul>
            {state.map(item => <li key={item.id}>
                <label>
                    <input type="checkbox" id="todo-${item.id}" checked={item.done} onChange={() => toggle(item.id)} />
                    {item.text}
                </label>
            </li>)}
        </ul>

        {// On appelle le composant TaskForm ici afin d'afficher le formulaire}
        <TaskForm />
    </>
}
```

## Le problème du partage d'état

## La solution : *lifting state up*

```jsx
// src/pages/TodoListPage.js

import React from "react";
import TaskForm from "../components/TaskForm";
import TodoList from "../components/TodoList";

const TodoListPage = () => {
    return <>
        <TodoList />
        <TaskForm />
    </>
}

export default TodoListPage;
```

```diff
// src/app.js
// React va permettre de dessiner notre arbre d'éléments HTML
import React from "react";
// ReactDOM va permettre de créer le rendu correspondant dans le DOM HTML
import ReactDOM from "react-dom";
// On importe la fonction TodoList
- import TodoList from "./components/TodoList";
+ import TodoListPage from "./pages/TodoListPage";


// Imprime l'arbre renvoyé par TodoListPage() dans l'élément <main> du DOM HTML
- ReactDOM.render(<TodoList />, document.querySelector('main'));
+ ReactDOM.render(<TodoListPage />, document.querySelector('main'));

```


```diff
// src/components/TodoList.js 

const TodoList = () => {
    // ...

    return <>
        <ul>
            {state.map(item => <li key={item.id}>
                <label>
                    <input type="checkbox" id="todo-${item.id}" checked={item.done} onChange={() => toggle(item.id)} />
                    {item.text}
                </label>
            </li>)}
        </ul>
-       <TaskForm />
    </>
}
```

L'affichage revient au même mais on n'a pas de fonctionnalités

```jsx
// src/pages/TodoListPage.js

import React, { useState } from "react";
import TaskForm from "../components/TaskForm";
import TodoList from "../components/TodoList";

// On créé ici un tableau TODO_ITEMS qui contient deux objets 
const TODO_ITEMS = [
    { id: 1, text: "Faire les courses", done: false },
    { id: 2, text: "Aller chercher les enfants", done: true },
];

const TodoListPage = () => {
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

    const addNewTask = (text) => {
        // Créons une nouvelle tâche avec le text tapé dans l'input
        const task = {
            id: Date.now(),
            text: text,
            done: false
        };

        // Remplaçons le tableau de tâches actuel par une copie
        // qui contiendra en plus la nouvelle tâche :
        setState([...state, task]);
    }
    
    // Pour que nos composants profitent du state et des fonctions
    // associées, on leur transmet les informations nécessaires
    // via le biais des props
    return <>
        <TodoList tasks={state} onTaskToggle={toggle} />
        <TaskForm onTaskAdded={addNewTask} />
    </>
}

export default TodoListPage;
```

```jsx
// src/components/TodoList.js

// Notez bien ici à quel point le composant TodoList est devenu
// simple et clair. Il ne gère plus d'état par lui même et ne fait
// que recevoir des propriétés qu'il va afficher à l'écran
// les tâches sont dans props.tasks et la fonction permettant
// de modifier le statut d'une tâche est dans props.onTaskToggle
const TodoList = (props) => {
    return <>
        <ul>
            {props.tasks.map(item => <li key={item.id}>
                <label>
                    <input
                        type="checkbox"
                        id="todo-${item.id}"
                        checked={item.done}
                        onChange={() => props.onTaskToggle(item.id)}
                    />
                    {item.text}
                </label>
            </li>)}
        </ul>
    </>
}

```

```jsx
// src/components/TaskForm.js

// Notez bien que le composant recevra désormais des props
// qui représentent les informations passées par le composant
// parent (appelant)
// Contrairement au composant de liste, le composant TaskForm
// conserve une gestion d'état car il est de sa responsabilité
// de "monitorer" l'<input> du formulaire
const TaskForm = (props) => {
    // ...

    const handleSubmit = (event) => {
        // Annulons le comportement par défaut de l'événement
        // qui serait de recharger la page
        event.preventDefault();

       // Appelons la fonction onTaskAdded passée en props
       props.onTaskAdded(text);
    }

    return <form onSubmit={handleSubmit}>
        <input
            type="text"
            name="todo-text"
            placeholder="Ajouter une tâche"
            onChange={updateText}
            value={text}
        />
        <button>Ajouter</button>
    </form>
}
```

# Ce que vous avez appris