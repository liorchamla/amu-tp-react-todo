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
// src/pages/TodoList.js

import React, { useState } from "react";

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

        <TaskForm />
    </>
}
```

## Le problème du partage d'état

## La solution : les props
```diff
// src/pages/TodoList 

const TodoList = () => {
    // ...

+   const onNewTask = (text) => {
+         // Créons une nouvelle tâche avec le text tapé dans l'input
+         const task = {
+             id: Date.now(),
+             text: text,
+             done: false
+         };
+         // Remplaçons le tableau de tâches actuel par une copie
+         // qui contiendra en plus la nouvelle tâche :
+         setState([...state, task]);
+     }

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
+       <TaskForm onNewTask={onNewTask} />
    </>
}
```

```diff
// src/components/TaskForm.js

- const TaskForm = () => {
+ const TaskForm = (props) => {
    // ...

    const handleSubmit = (event) => {
        // Annulons le comportement par défaut de l'événement
        // qui serait de recharger la page
        event.preventDefault();

-        // Créons une nouvelle tâche avec le text tapé dans l'input
-        const task = {
-            id: Date.now(),
-            text: text,
-            done: false
-        };
-        // Remplaçons le tableau de tâches actuel par une copie
-        // qui contiendra en plus la nouvelle tâche :
-        setState([...state, task]);

+       // Appelons la fonction onNewTask passée en props
+       props.onNewTask(text);
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