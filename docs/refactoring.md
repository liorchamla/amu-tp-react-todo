# Refactoring des appels HTTP

## Sommaire

## But de l'exercice

## Création d'un module spécifique aux appels HTTP

```js

// src/api/http.js


const SUPABASE_URL = "https://ubrnopsjlvwleakngnmr.supabase.co/rest/v1/todos";
const SUPABASE_API_KEY = "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJyb2xlIjoiYW5vbiIsImlhdCI6MTYzODE5MjQ4MSwiZXhwIjoxOTUzNzY4NDgxfQ.3JdQW11rNZNpvcehhwFVaofXL2agE5LDn_3O4BvSAHw";

const SUPABASE_URL = "https://ubrnopsjlvwleakngnmr.supabase.co/rest/v1/todos";
const SUPABASE_API_KEY = "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJyb2xlIjoiYW5vbiIsImlhdCI6MTYzODE5MjQ4MSwiZXhwIjoxOTUzNzY4NDgxfQ.3JdQW11rNZNpvcehhwFVaofXL2agE5LDn_3O4BvSAHw";

/**
 * Permet de modifier le statut de la tâche dans l'API
 * @param {number} id 
 * @param {boolean} status 
 * @returns Promise<{id: number, done: boolean, text: string}>
 */
export const toggleTaskInApi = (id, status) => {
    return fetch(`${SUPABASE_URL}?id=eq.${id}`, {
        method: "PATCH",
        headers: {
            "Content-Type": "application/json",
            apiKey: SUPABASE_API_KEY,
            Prefer: "return=representation",
        },
        body: JSON.stringify({ done: status }),
    });
}

/**
 * Ajoute une tâche dans l'API
 * @param {{text: string, done: boolean}} task 
 * @returns Promise<Array<{id: number, text: string, done: boolean}>>
 */
export const addTaskToApi = (task) => {
    return fetch(SUPABASE_URL, {
        method: "POST",
        body: JSON.stringify(task),
        headers: {
            "Content-Type": "application/json",
            apiKey: SUPABASE_API_KEY,
            Prefer: "return=representation",
        },
    }).then((response) => response.json())
}

/**
 * Récupère les donnes des tâches à partir de l'API
 * @returns Promise<Array<{id: number, text: string, done: boolean}>>
 */
export const loadTasksFromApi = () => {
    return fetch(`${SUPABASE_URL}?order=created_at`, {
        headers: {
            apiKey: SUPABASE_API_KEY,
        },
    }).then((response) => response.json())
}

```

```jsx
// src/pages/TodoListPage.js

import React, { useEffect, useState } from "react";
import { addTaskToApi, loadTasksFromApi, toggleTaskInApi } from "../api/http";
import TaskForm from "../components/TaskForm";
import TodoList from "../components/TodoList";

const TodoListPage = () => {
    const [state, setState] = useState([]);

    const toggle = (id) => {
        // Récupérons l'index de la tâche concernée
        const idx = state.findIndex(task => task.id === id);

        // Créons une copie de la tâche concernée 
        const item = { ...state[idx] };

        // Appel HTTP en PATCH pour modifier la tâche
        toggleTaskInApi(id, !item.done).then(() => {
            // Lorsque le serveur a pris en compte la demande et nous a répond
            // Nous modifions notre copie de tâche :
            item.done = !item.done;

            // Créons une copie du tableau d'origine
            const stateCopy = [...state];
            // Enfin remplaçons la tâche originale par la copie :
            stateCopy[idx] = item;
            // Et faisons évoluer le state : l'ancien tableau sera
            // remplacé par le nouveau, et le rendu sera déclenché à nouveau
            setState(stateCopy);
        });
    }

    const addNewTask = (text) => {
        // Créons une nouvelle tâche avec le text tapé dans l'input
        const task = {
            text: text,
            done: false
        };

        addTaskToApi(task).then(() => {
            // Remplaçons le tableau de tâches actuel par une copie
            // qui contiendra en plus la nouvelle tâche :
            setState([...state, task]);
        })
    }

    // On utilise le hook useEffect, qui permet de créer un comportement
    // qui aura lieu lors de CHAQUE rendu du composant React
    // mais en passant un tableau de dépendances vide en deuxième paramètres, on explique à React que ce comportement 
    // ne devra avoir lieu qu'une seule fois, au chargement du composant
    useEffect(() => {
        // Appel HTTP vers Supabase
        loadTasksFromApi()
            .then((items) => {
                // On remplace la valeur actuel de state
                // par le tableau d'items venant de l'API
                setState(items);
            });
    }, []);

    return <>
        <TodoList tasks={state} onTaskToggle={toggle} />
        <TaskForm onTaskAdded={addNewTask} />
    </>
}

export default TodoListPage;
```