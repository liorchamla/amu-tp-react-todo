# Ajouter du routage dans notre application

## Sommaire

## But de l'exercice

## Installation des dépendances (react-router-dom)

`npm install react-router-dom`

## Un composant pour afficher le détail d'une tâche

```jsx
// src/pages/TodoDetailsPage.js

import React, { useState } from "react";

const TodoDetailsPage = () => {
    return <>
        <p>Ca fonctionne !</p>
    </>
}

export default TodoDetailsPage;

```

## Mise en place du routeur

```jsx
// src/app.js

// React va permettre de dessiner notre arbre d'éléments HTML
import React from "react";
// ReactDOM va permettre de créer le rendu correspondant dans le DOM HTML
import ReactDOM from "react-dom";

// BrowserRouter permet de fournir à tous les composants qu'il contient des outils relatifs au routage
// Routes permet de décrire la configuration des routes
// Route permet de décrire la configuration d'une route (url => composant à afficher)
import { BrowserRouter, Routes, Route } from "react-router-dom";

import TodoDetailsPage from "./pages/TodoDetailsPage";
import TodoListPage from "./pages/TodoListPage";

const App = () => {
    return <BrowserRouter>
        <Routes>
            <Route
                path="/:id/details"
                element={<TodoDetailsPage />}
            />
            <Route
                path="/"
                element={<TodoListPage />}
            />
        </Routes>
    </BrowserRouter>
}

// Imprime l'arbre renvoyé par App() dans l'élément <main> du DOM HTML
ReactDOM.render(<App />, document.querySelector('main'));
```

## Récupérer un paramètre dans l'URL
```jsx
// src/pages/TodoDetailsPage.js

import { useParams } from "react-router-dom";

const TodoDetailsPage = () => {
    // useParams est un hook offert par ReactRouter qui permet
    // de retrouver les paramètres dans une URL.
    const params = useParams();

    // Comme nous sommes sur /:id/details (exemple : /120/details)
    // On peut retrouver dans ces params un élément "id" qui contiendra l'identifiant fourni (exemple : 120)
    const id = +params.id;

    console.log(id);

    return <>
        <p>Ca fonctionne !</p>
    </>
}
```

## Récupérer la tâche dans l'API et l'afficher
```js
// src/api/http.js

export const loadTaskFromApi = (id) => {
    return fetch(`${SUPABASE_URL}?id=eq.${id}`, {
        method: "GET",
        headers: {
            "Content-Type": "application/json",
            apiKey: SUPABASE_API_KEY,
            Prefer: "return=representation",
        }
    })
        .then(response => response.json())
        // La réponse contenant un tableau des tâches correspondantes
        // Nous ne retournons que la première (et la seule)
        .then(tasks => tasks[0]);
}
```

```jsx
import React, { useEffect, useState } from "react";
import { useParams } from "react-router-dom";
import { loadTaskFromApi } from "../api/http";

const TodoDetailsPage = () => {
    // Créons un état qui représentera la tâche à afficher
    // Par défaut sa valeur est null, mais on mettra à jour celle-ci
    // dès que l'API aura renvoyé les données de la tâche
    const [task, setTask] = useState(null);

    // useParams est un hook offert par ReactRouter qui permet
    // de retrouver les paramètres dans une URL.
    const params = useParams();

    // Comme nous sommes sur /:id/details (exemple : /120/details)
    // On peut retrouver dans ces params un élément "id" qui contiendra l'identifiant fourni (exemple : 120)
    const id = +params.id;

    // Un effet doit être lancé par React à chaque fois que l'ID change
    useEffect(() => {
        // On appelle l'API et lorsqu'on reçoit la tâche
        // correspondante, on remplace l'ancien state "task"
        // par les données que l'API a retourné
        loadTaskFromApi(id)
            .then(apiTask => setTask(apiTask));
    }, [id])

    // En fonction du state "task" (null ou pas), on retourne
    // un arbre JSX différent
    return task ?
        <>
            <h2>{task.text}</h2>
            <strong>Statut : </strong>
            {task.done ? "Fait" : "Pas fait"}
            <br />
            <a href="/">Retour aux tâches</a>
        </>
        :
        <p>Chargement en cours</p>
}

export default TodoDetailsPage;
```

## Le problème du rechargement de page
```diff
// src/pages/TodoDetailsPage.js

- import { useParams } from "react-router-dom";
+ import { Link, useParams } from "react-router-dom";

const TodoDetailsPage = () => {
    // ...

    return task ?
        <>
            <h2>{task.text}</h2>
            <strong>Statut : </strong>
            {task.done ? "Fait" : "Pas fait"}
            <br />
-           <a href="/">Retour aux tâches</a>
+           <Link to="/">Retour aux tâches</Link>
        </>
        :
        <p>Chargement en cours</p>
}
```

```diff
// src/components/TodoList.js

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

+                   <Link to={item.id + "/details"}>Details</Link>
                </label>
            </li>)}
        </ul>
    </>
}
```


## Ce que vous avez appris