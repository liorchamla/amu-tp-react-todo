# Tester ses composants React
Notre application est désormais terminée : elle affiche une liste des tâches sauvegardées à distance et permet aussi de les modifier et d'en ajouter de nouvelles. Elle permet enfin de voir le détail d'une tâche donnée.

On veut désormais s'assurer que de futures modifications de notre code n'aille pas à l'encontre de ce fonctionnement et pour ce faire nous allons écrire des .. Tests !

Comme tout langage de programmation, Javascript possède des utilitaires et frameworks de tests qui vont permettre d'écrire des tests unitaires afin de valider le bon fonctionnement du code.

Plus encore, le code React étant tout de même assez spécifique dans son fonctionnement, des librairies et utilitaires sont disponibles pour encore plus faciliter l'écriture des tests.

## Sommaire

## But de l'exercice
Nous allons découvrir ici les tests unitaires en Javascript avec le framework de test **Jest**.

**Ici, les notions importantes sur lesquelles se concentrer sont l'écriture de tests, la simulation des intéractions avec le DOM et les Composants React, ainsi que l'utilisation de fonctions simulées (mocks)**

## Installation de Jest
Encore une fois, NPM va être notre allié le plus précieux ! Il va nous permettre d'installer un nouvel outil ainsi que des plugins nécessaires pour écrire nos tests unitaires !

```bash
npm install --save-dev jest babel-jest @types/jest
```
Ce qu'on installe ici c'est :
1. Le framework de tests Jest ;
2. Un plugin babel-jest qui permettra à Jest de demander à Babel de compiler le code JS avant de lancer les tests ;
3. Enfin, une librairie de types spécifique à Jest qui permettra simplement à notre éditeur de code de connaître les fonctions et méthodes disponibles dans Jest afin d'obtenir une belle autocomplétion :-) ;


Et bien sur, il nous faut ajouter une tâche qu'on pourra appeler depuis NPM :

```diff
// package.json

"scripts": {
+   "test": "jest --watch",
    "dev": "webpack --mode development --watch",
    "serve": "live-server --entry-file=./index.html"
},
```

## Premier test avec Jest

Ecrivons un premier test juste pour s'assurer que le framework fonctionne bien et que notre configuration est valide :

```js
// tests/app.test.js

// test("it should work fine", () => {})
it("should work fine", () => {
    expect(true).toBe(true);
})
```

Lançons notre tâche de test pour voir la sortie de Jest : `npm test`

Si tout va bien, on peut continuer et écrire un véritable test !

## Installation de React Testing Library
Le code React étant spécifique, on a des besoins particuliers en termes de tests et React Testing Library va nous fournir les outils adéquats pour tester le comportement de nos composants

```bash
npm install --save-dev @testing-library/react
```

## Tester le composant TodoList

Découvrons désormais comment tester facilement un composant tel que le *TodoList*. Pour cela nous devons tester plusieurs comportements :

1. Le fait que la liste affichera bien les tâches qu'on lui confie en props ;
2. Le fait qu'une tâche sur laquelle on click appellera bien la fonction que l'on confie en props ;

```jsx
// tests/TodoList.test.js

import { 
    // Permet de déclencher le rendu d'un composant
    render, 
    // Permet de déclencher un événement dans l'interface
    fireEvent, 
    // Permet d'aller chercher des éléments sur un "écran" virtuel
    screen 
} from '@testing-library/react'
import React from 'react';
import { BrowserRouter } from 'react-router-dom';
import TodoList from '../src/components/TodoList';

// Suite de tests pour le composant TodoList
describe("TodoList Component", () => {
    // Dans ce premier test, on veut s'assurer que le composant
    // affichera bien les tâches qu'on lui donne via les props
    it("should render tasks given in props", async () => {
        // On créé un tableau arbitraire de tâches
        const tasks = [
            { id: 1, text: 'MOCK_TASK_1', done: true },
            { id: 2, text: 'MOCK_TASK_2', done: false },
        ];

        // On déclenche le rendu du composant, en s'assurant
        // de l'envelopper dans un BrowserRouter car il contient
        // des fonctionnalités (<Link>) liées à React Router
        render(<BrowserRouter>
            <TodoList tasks={tasks} />
        </BrowserRouter>);

        // On recherche sur l' "écran" tous les éléments qui contiennent
        // le texte "MOCK_TASK"
        const items = await screen.getAllByText("MOCK_TASK", { exact: false });

        // On s'attend à avoir trouvé autant d'éléments que
        // de tâches dans le tableau passé
        expect(items).toHaveLength(tasks.length);
    })

    // Dans ce deuxième test, on veut s'assurer qu'un click sur
    // tâche déclenchera bien la fonction "onTaskToggle"
    it("should call onTaskToggle on a click", async () => {
        // On créé un tableau arbitraire de tâches
        const tasks = [
            { id: 1, text: 'MOCK_TASK_1', done: true },
            { id: 2, text: 'MOCK_TASK_2', done: false },
        ];

        // On créé une "fausse" fonction, qui nous permettra
        // de l'espionner et de savoir si elle a été appelée ou pas
        const mockFunction = jest.fn();

        // On déclenche le rendu en donnant notre fausse fonction
        // à onTaskToggle
        render(<BrowserRouter>
            <TodoList tasks={tasks} onTaskToggle={mockFunction} />
        </BrowserRouter>);

        // On simule un click sur la tâche 1
        fireEvent.click(screen.getByText('MOCK_TASK_1'));

        // On s'attend à ce que la fausse fonction ait été appelée
        expect(mockFunction).toHaveBeenCalled();
    })
})
```

Notez dans le second test la présence d'une "*fausse fonction*" (appelée aussi fonction simulée ou plus généralement *mock function*). 

C'est un point particulièrement important dans nos tests car souvent, nos fonctions véritables appellent des API via HTTP ou font des traitements parfois complexes dont on n'a pas besoin pendant nos tests. On veut donc pouvoir "remplacer" ces fonctions par des fonctions simulées, qui n'auront pas de comportement particulier **mais qui auront un pouvoir très important : nous pourrons les *surveiller* et s'assurer que ces fonctions ont bien été appélées, et même *programmer à l'avance ce qu'elles pourraient retourner* !**

## Tester le composant TaskForm

Maintenant qu'on a compris le concept, on peut aussi tester que le *TaskForm* fonctionne correctement :

```jsx
// tests/TaskForm.test.js

import { fireEvent, render, screen } from "@testing-library/react";
import TaskForm from "../src/components/TaskForm";
import React from "react"

// On souhaite s'assurer que lorsque l'utilisateur soumet le formulaire
// La fonction qu'on a passé en props du composant TaskForm sera bien appelée  :
it("should call prop function on submit", async () => {
    // Créons une fausse fonction qui n'appelera donc pas l'API
    // via HTTP
    const mockFunction = jest.fn();

    // On déclenche le rendu du composant TaskForm en lui confiant
    // notre fausse fonction pour la props onTaskAdded
    render(<TaskForm onTaskAdded={mockFunction} />);

    // On simule un événement de changement sur l'<input> dont le placeholder est
    // "Ajouter une tâche" et on spécifie le texte qui aurait été tapé par le visiteur (MOCK_TEXT)
    fireEvent.change(screen.getByPlaceholderText("Ajouter une tâche"), {
        target: { value: "MOCK_TEXT" }
    });

    // On simule ensuite un click sur le bouton dont le texte est "Ajouter"
    // Ce qui devrait déclencher le submit du formulaire
    fireEvent.click(screen.getByText("Ajouter"));

    // On veut s'assurer alors que la fonction qui doit prendre en charge
    // Cette nouvelle tâche est bien appelée ET qu'elle est appelée
    // avec pour paramètre le texte présent dans l'<input>
    expect(mockFunction).toHaveBeenCalledWith("MOCK_TEXT");
})
```

## Enfin, on teste l'intégration entre *TodoList* et *TaskForm* !

On sait que nos composants se comportent bien comme on le souhaite. Par contre, ce qu'on ne sait pas, c'est si ils communiquent bien l'un avec l'autre via le state partagé qui se trouve dans le composant `<TodoListPage>`.

On va donc s'attarder un peu sur ce composant précis afin de s'assurer que tout fonctionne correctement.

**Attention : c'est ici que le fait d'avoir refactorisé nos appels d'API dans un module spécifique va PAYER :-) !**

```jsx
// tests/TodoListPage.test.js

import { fireEvent, render, screen } from "@testing-library/react";
import React from "react"
import { act } from "@testing-library/react";
import { addTaskToApi, loadTasksFromApi } from "../src/api/http";
import TodoListPage from "../src/pages/TodoListPage";
import { BrowserRouter } from "react-router-dom";

// Très important : nous décidons ici que TOUTES les fonctions du module http.js vont être MOCKEES (simulées)
// On va donc pouvoir les surveiller, savoir si elles ont été appelées, et même décider de ce qu'elles retourneront !
// On s'assurera donc que l'API réelle ne sera jamais appelée pendant nos tests
// C'est aussi pour ça qu'il est important de factoriser ce genre de méthodes dans un module à part, nous
// permettant ainsi de tout mocker d'un seul coup et de se substituer à ces fonctionnalités pendant nos tests
jest.mock("../src/api/http");

it("should add a task on form submit", async () => {
    // Avant de pouvoir appeler le composant App, il faut s'assurer que
    // les méthodes du module http.js soient définies correctement
    // Commençons par la méthode loadTasksFromApi() et faisons en sorte qu'elle retourne un tableau avec une tâche
    loadTasksFromApi.mockResolvedValue([
        { id: 1, text: "MOCK_TASK_1", done: false }
    ]);

    // On sait aussi que si l'on soumet le formulaire, la fonction addTaskToApi
    // sera appelée, il faut donc, elle aussi, la simuler de telle sorte qu'elle retourne 
    // une tâche similaire à celle qu'elle va recevoir, si ce n'est qu'elle y ajoutera
    // un identifiant, tel que l'aurait fait Supabase lors de cet appel API
    addTaskToApi.mockImplementation((task) => Promise.resolve({ ...task, id: 2 }))

    // Comme notre TodoListPage contient un hook useEffect, le fait même
    // de rendre le composant va occasionner un changement de state, il est recommandé
    // de l'encapsuler dans une fonction act().
    // Plus encore, comme notre useEffect fait appel à une fonction qui renvoi une Promesse (asynchrone)
    // il nous faut await le render(), et donc aussi await le act()
    await act(async () => {
        await render(
            <BrowserRouter>
                <TodoListPage />
            </BrowserRouter>
        );
    });

    // On peut d'ores et déjà confirmer qu'une fois notre composant rendu,
    // grâce à son useEffect, il aura appelé l'API et aura affiché la tâche
    // renvoyée :
    const itemsBeforeForm = screen.getByText("MOCK_TASK_1", { exact: false });
    expect(itemsBeforeForm).toBeTruthy();

    // On veut maintenant vérifier le bon fonctionnement du formulaire, celui-ci aussi est sensé
    // faire évoluer le state du composant, il est donc recommandé de l'encapsuler dans un act().
    // Plus encore, on sait que le state n'évoluera qu'à la fin de l'appel HTTP à l'API, donc de façon non synchrone,
    // Il faut donc attendre que ce soit fait en faisant un await de act()
    await act(async () => {
        // Le fait de changer le texte dans l'<input> du formulaire fera aussi évoluer
        // un state, on veut donc être sur que le state ait bien fini d'évoluer avant de continuer,
        // on va donc await la fin de l'événément change
        await fireEvent.change(screen.getByPlaceholderText("Ajouter une tâche"), {
            target: { value: "MOCK_TASK_2" }
        });

        // On pourra alors cliquer sur le bouton de soumission
        fireEvent.click(screen.getByText("Ajouter"));
    })

    // On peut désormais s'assurer qu'en cherchant tous les éléments qui contiennent
    // le terme "MOCK_TASK", on en trouvera bien 2 (la tâche MOCK_TASK_1, présente depuis le départ,
    // et la tâche MOCK_TASK_2 qu'on vient d'ajouter via le formulaire)
    const items = screen.getAllByText("MOCK_TASK", { exact: false });
    expect(items).toHaveLength(2);
})
```

Et voilà ! On a terminé notre petite application, et on a mis en place des tests qui permettent de s'assurer de la non régression lors de nos futures interventions !

A vous de créer des tests pour continuer à sécuriser encore plus notre application !