# Routing et affichage dynamique
Dans toute application web, le protocole HTTP étant primordial, l'accès à une ressource (que ce soit une page, des données, un fichier etc) se fait via un URL.

Le système de routage est donc lui aussi essentiel : le *routing* c'est le fait de décrire l'ensemble des *routes* disponibles dans votre application. Alors qu'est-ce qu'une *route* ? C'est **l'association entre une URL et un comportement** !

Le routing est donc l'ensemble des associations URL / Comportements de votre application.

Dans une Single Page Application, on chercher à faire croire au visiteur qu'il navigue dans différentes pages (et donc différentes URL) alors qu'en réalité il reste toujours sur la même page (index.html) et c'est Javascript qui donne l'impression que la page change lorsque l'URL change.

## Sommaire

## But de l'exercice
Nous allons voir comment gérer le routage dynamique (sans rechargement de page) dans une application React grâce à une librairie tierce supplémentaire : React Router.

Elle nous donne un ensemble de composants et de fonctions qui vont permettre de configurer et gérer le routage sur notre application.

## Installation des dépendances (react-router-dom)

Commençons par installer la librairie la plus populaire pour la gestion du routage sur une application React, React Router :

```bash
npm install react-router-dom
```

Cette librairie va nous offrir un certain nombre de composants et de fonctions dans le but de mettre en place le routage !

Mais pour pouvoir démontrer son fonctionnement, il nous faut déjà plusieurs pages à afficher. Or pour l'instant nous n'avons que la TodoListPage, remédions à cela.

## Un composant pour afficher le détail d'une tâche

Créons un composant qui représentera une seconde page, un deuxième affichage : le détail d'une tâche donnée.

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

Pour l'instant ce composant est très simple, on l'enrichira plus tard, ici le but est simplement de voir si l'on arrive à l'afficher simplement grâce à une URL donnée.

## Mise en place du routeur
Mettons donc en place le router livré par la librairie React Router afin de faire en sorte qu'une URL spécifique donne accès à la liste des tâches et une autre URL donne accès à la page de détails :

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
Ce que l'on constate ici c'est la présence de 3 nouveaux composants :
1. BrowserRouter : c'est un composant qui va utiliser l'API de l'historique du navigateur pour connaître à tout moment l'URL tapée et aussi pour pouvoir modifier l'historique dynamiquement ;
2. Routes : c'est un composant qui permet de configurer l'ensemble des associations entre URLs et composants, il affichera le composant associé à l'URL courante de l'historique ;
3. Route : c'est un composant qui permet d'associer une URL à un affichage ;

Par ailleurs, on voit que nous avons associé notre composant *TodoDetailsPage* à une URL un peu spéciale : */:id/details*. Ici, React Router va considérer que *:id* est un paramètre variable de l'URL. Il va donc aussi considérer que */12/details* ou */220/details* ou encore */toto/details* correspondent bien à cette Route.

Si vous essayez votre application, vous devriez constater :
* `/` donne accès à la liste des tâches ;
* `/12/details` donne accès à la page de détails ;

On comprend donc bien que l'id présent dans l'URL est un paramètre variable, et il serait bon qu'on puisse récupérer ce paramètre afin de travailler avec.

## Récupérer un paramètre dans l'URL
Lorsque l'on appelle `/12/details`, React Router va identifier la Route dont le path est `/:id/details` et va donc afficher la *TodoDetailsPage*.

Comment alors, dans ce composant, récupérer l'identifiant `12` ? React Router nous donne un outil très utile grâce à la fonction *hook* `useParams()`. Il va nous donner un objet qui contiendra les différents paramètres de l'URL :

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

Si vous vous rendez sur `/12/details`, vous verrez qu'on récupère effectivement `12` dans votre console.

## Récupérer la tâche dans l'API et l'afficher

Puisqu'on sait désormais récupérer un paramètre d'URL (ici l'identifiant de la tâche), nous pouvons désormais récupérer les données de la tâche sur l'API afin de les afficher dans le composant.

Commençons par créer une fonction dans le fichier *src/api/http.js* afin d'appeler l'API :

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

Et mettons maintenant en place le code final du composant *TodoDetailsPage* :

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
Vous pouvez désormais tester cette URL au sein du navigateur (en choisissant un identifiant dont vous savez qu'il existe bien dans votre base de données !).

## Le problème du rechargement de page

Maintenant que vous voyez la page de détails d'une tâche, vous constatez qu'en cliquant sur le lien *Retour aux tâches*, le navigateur va recharger la page.

C'est un comportement que l'on veut absolument éviter dans le cadre d'une Single Page Application que l'on souhaite la plus réactive possible.

La librairie React Router nous offre un composant spécifique qui pourra remplacer les élément `<a>`, de tel sorte que l'historique du navigateur change (et donc l'URL affichée), sans pour autant recharger ce dernier : le composant `<Link>` :

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

En essayant la page dans le navigateur, vous constaterez que désormais, on navigue dans l'application sans rechargement de page.

Pour faciliter la navigation au niveau global, créons aussi un lien dans la liste des tâches pour aller voir le détails d'une tâche qui nous intéresse :

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
Et voilà ! On a une navigation réactive, sans rechargement de page, et l'impression réelle que l'on a plusieurs pages sous les yeux (alors qu'on reste perpétuellement sur *index.html* :-)) !

## Ce que vous avez appris
* Utiliser les composants de React Router pour configurer la navigation sur notre application ;
* Créer des URLs paramétrées et récupérer des paramètres grâce au hook `useParams()` ;
* Modifier l'URL sans rechargement de page grâce au composant `<Link to="url">`

[Revenir au sommaire](../README.md) ou [Passer à la suite : Tester nos composants React](tests.md)
