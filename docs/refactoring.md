# Refactoring des appels HTTP et système de modules

Notre application est en bonne voie ! Elle fonctionne plutôt bien mais son code, par contre, est assez peu clair et relativement entremêlé.

Comme tout langage de programmation, Javascript (même ça lui a manqué pendant des décennies) s'est doté de la possibilité de répartir du code dans différents fichiers grâce à son système de **modules** (qu'on utilise déjà depuis le début de notre projet).

## Sommaire :
  * [But de l'exercice](#but-de-l-exercice--)
  * [Créer un fichier http.js qui contiendra les appels HTTP](#créer-un-fichier-httpjs-qui-contiendra-les-appels-http)
  * [Ce que vous avez appris](#ce-que-vous-avez-appris)

## But de l'exercice :
Notre objectif est de centraliser toutes les lignes de code qui ont un rapport avec l'API dans un seul fichier, ainsi :
1. Ces traitements pourront être rappelés facilement si on a de nouveaux besoins ;
2. Notre code sera agnostique de ces détails d'appels HTTP et ne fera qu'appeler les fonctions telles que `loadTasksFromApi()` sans se préoccuper de quel serveur est appelé ou de quels traitements sont réalisés lors de la réponse ;

**Ici, on se concentrera sur le fonctionnement des modules et notament de la façon dont on peut rendre une fonction utilisable dans un autre fichier !**

## Créer un fichier http.js qui contiendra les appels HTTP
On veut donc centraliser dans un nouveau fichier *src/api/http.js* tout ce qui a un rapport avec notre service Supabase. 

On peut donc extraire l'ensemble des appels `fetch()` qui sont dans notre fichier *src/pages/TodoListPage.js* et les confier à des fonctions dans le fichier *src/api/http.js*.

**Attention : remplacez bien dans les 2 constantes votre identifiant Supabase ainsi que la clé d'API que vous trouverez tout deux sur votre projet Supabase** :

```js
// src/api/http.js

const SUPABASE_URL = "https://IDENTIFIANT_SUPABASE.supabase.co/rest/v1/todos";
const SUPABASE_API_KEY = "CLE_API_SUPABASE";

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
On peut désormais faire appel à ces fonctions dans notre composant, ce qui aura le mérite de rendre le code plus clair et surtout plus testable !

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

        addTaskToApi(task).then((savedTask) => {
            // Remplaçons le tableau de tâches actuel par une copie
            // qui contiendra en plus la nouvelle tâche :
            setState([...state, savedTask]);
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

Essayez votre application dans le navigateur, tout devrait fonctionner correctement !

# Ce que vous avez appris
* Créer différents modules (fichiers JS) afin de répartir correctement vos fonctions et vos responsabilités ;
* Exporter des fonctions d'un module afin qu'elles soient utilisables dans un autre module ;
* Importer des fonctions dans un module à partir d'un autre module ;
* Centraliser certaines fonctions afin de pouvoir les réutiliser et les faire évoluer plus facilement ;

[Revenir au sommaire](../README.md) ou [Passer à la suite : Routing et affichage dynamique](routing.md)
