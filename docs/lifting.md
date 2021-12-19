# Remontée d'état : communication entre composants

Notre application prend bonne forme ! Néanmoins, on trouve que le composant **TodoList** concentre trop de responsabilités :
* Il gère et affiche la liste des tâches ;
* Il gère le changement de statut des tâches ;
* Et il gère et affiche le formulaire d'ajout ;

Beaucoup de choses pour un simple composant. On peut augmenter encore l'organisation de notre code, sa maintenabilité et sa testabilité en fragmentant ce composant en deux composants distincts : un qui gère la liste, l'autre qui gère le formulaire !

## Sommaire
  * [But de l'exercice](#but-de-l-exercice)
  * [Extraction du code du formulaire vers un composant TaskForm](#extraction-du-code-du-formulaire-vers-un-composant-taskform)
  * [Le problème du partage d'état](#le-probl-me-du-partage-d--tat)
  * [(Re)Découverte des props](#-re-d-couverte-des-props)
  * [La limite des props](#la-limite-des-props)
  * [La solution : *lifting state up*](#la-solution----lifting-state-up-)
  * [Ce que vous avez appris](#ce-que-vous-avez-appris)

## But de l'exercice 

Nous souhaitons créer un nouveau composant qui sera en charge de l'ajout d'une tâche. Nous allons nous heurter à un problème : comment un composant qui souhaite ajouter une tâche peut-il le faire alors que l'état qui représente la liste des tâches est dans un AUTRE composant ?

**Ici, il sera vraiment important de comprendre la notion de props et de *remontée d'état***

## Extraction du code du formulaire vers un composant TaskForm

Commençons par créer un deuxième composant *TaskForm* dans le fichier *src/components/TaskForm.js*. Nous allons extraire du composant *TodoList* tout ce qui concerne le formulaire et l'ajout d'une tâche :

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

Vous le remarquez surement, à la fin de la fonction qui gère la soumission du formulaire, nous faisons appel à `setState()` qui n'existe pas dans notre composant : elle se trouve dans le composant *TodoList* !

Nous règlerons ce soucis un peu plus tard. Pour l'instant voyons comment on peut faire appel à ce composant à l'endroit où l'on souhaite effectivement faire apparaitre le formulaire :

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

Si vous ouvrez votre navigateur, vous constaterez que le formulaire apparait ! Par contre, la soumission de celui-ci causera une erreur, car comme on l'a dit plus tôt, la fonction `setState()` appelée dans le composant *TaskForm* n'existe pas, elle est dans le composant *TodoList* !

## Le problème du partage d'état

On arrive donc face à un problème connu et une limite évidente du développement d'applications avec React : le partage d'état entre différents composants et plus généralement **la communication entre composants**.

Nous utilisons pourtant un système depuis le tout premier composant que nous avons créé sans nous en rendre compte, et qui a tout le potentiel pour régler le soucis de la communication entre deux composants : **les props** !

## (Re)Découverte des props
Nous avions vu depuis le début que nous pouvions passer des informations à un composant :
```js
// Ici, nous créons un élément <p> et nous lui passons des informations (className et onClick) ainsi qu'un enfant ("Hello")
// L'objet passé en deuxième paramètre de la fonction createElement représente les PROPS du composant : des informations qu'on lui donne et qu'il pourra utiliser !
const Hello = () => React.createElement('p', { className: "rouge", onClick: () => alert("Hey!") }, "Hello");

// En JSX, c'est la même chose, si ce n'est que la syntaxe change,
// Les props sont représentées comme des attributs de la balise ouvrante, mais le système reste strictement identique
const Hello = () => <p className="rouge" onClick={() => alert("Hey!")}>Hello</p>;
```

Ici nous voyons comment donner des informations à un paragraphe `<p>`, mais nous pouvons faire exactement la même chose pour un composant personnalisé :

```js
// Imaginons qu'on appelle le composant Hello en lui donnant une props (sous forme d'attribut) :
<Hello firstName="Joseph" />

// On pourra utiliser ces informations dans le composant Hello : les props seront passées par React au composant en paramètres de la fonction :
const Hello = (props) => <p>Hello {props.firstName}</p>;
// Affichera donc <p>Hello Joseph</p>
```

Evidemment, les informations qu'on passe à un composant peuvent être de tout type : simple valeur scalaire (number, string, boolean), informations complexes (objets, tableaux) et même des fonctions !

## La limite des props 

Vous l'avez sans doute remarqué, pour pouvoir passer des informations à un composant, il faut qu'un composant parent (qui possède les informations) les passent à un composant enfant qui va s'en servir. Par exemple :

```js
// Imaginons encore notre composant Hello qui s'attend à recevoir des props et notamment un prop "firstName" :
const Hello = (props) => <p>Hello {props.firstName}</p>

// On pourrait alors depuis un composant parent lui passer cette props :
const Page = () => {
    const [name, setName] = useState("Joseph");

    return <div>
        <Hello firstName={name} />
    </div>
}
```

Mais comment faire si on a deux composants "frères" (l'un n'est pas contenu par l'autre, les deux sont utilisés en parallèle) qui doivent communiquer ? 

## La solution : *lifting state up*

La solution la plus simple pour résoudre ce problème, c'est de faire "remonter l'état" (*lifting state up*) : l'état qui doit être partagé entre deux composants frères devrait être confié à un composant PARENT des deux frères.

Ce composant parent pourra alors faire passer les informations de l'état aux deux composants enfants, qui pourront alors utiliser chacun ces informations / comportements.

Mettons en oeuvre cette solution en créant un nouveau fichier *src/pages/TodoListPage.js*, un composant qui contiendra les deux composants *TodoList* et *TaskForm* :

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

C'est désormais ce composant qu'on veut afficher dans notre application :

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

On peut désormais supprimer le TaskForm dans notre composant TodoList :

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

L'affichage revient au même ! Néanmoins, le formulaire ne fonctionne toujours pas. 

Ce qu'on va pouvoir faire, c'est faire remonter l'état dans le composant parent (*TodoListPage*), ainsi que toutes les fonctions qui permettent de le manipuler.

On pourra alors passer les informations et comportements nécessaires à chaque composant enfant (*TodoList* et *TaskForm) via leurs props !

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

Désormais, le composant *TodoList* ne contient plus ni état ni comportement : il ne fait que se servir des props qui lui ont été passées par son parent (à savoir la liste des tâches à afficher ET la fonction à appeler quand une case à cocher change) :

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

C'est la même chose pour le composant *TaskForm* à ceci près que celui ci gardera malgré tout un état *local* pour gérer le text de l'input, c'est de sa responsabilité !

Il recevra donc uniquement une fonction permettant d'ajouter une tâche via ses props. Cela lui permettra, lors de la soumission du formulaire, d'appeler cette fonction (définie dans le parent et qui aura donc accès à l'état pour le modifier) :

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

Et voilà ! On a deux composants bien distincts qui sont désormais capables de partager un état : c'est le composant parent qui fait passer par les props les informations et comportements dont auront besoin les deux composants pour fonctionner !

# Ce que vous avez appris
* Refactoriser votre code en plusieurs composants afin de séparer les responsabilités et augmenter la maintenabilité et la testabilité ;
* Passer des informations d'un composant parent à un composant enfant via les *props* ;
* Partager un état entre plusieurs composants non hiérarchiques grâce à la *remontée d'état* ;

[Revenir au sommaire](../README.md) ou [Requêtes HTTP et API REST](http.md)