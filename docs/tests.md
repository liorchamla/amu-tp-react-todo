# Tester ses composants React

## Sommaire

## But de l'exercice

## Installation des dépendances

`npm install --save-dev jest @testing-library/react @types/jest babel-jest`

```diff
// package.json
"scripts": {
+   "test": "jest --watch",
    "dev": "webpack --mode development --watch",
    "serve": "live-server --entry-file=./index.html"
},
```

## Tester le composant TodoList

```jsx
// tests/todolist.test.js

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

## Tester le composant TaskForm