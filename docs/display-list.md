# Afficher la liste des tâches

Maintenant qu'on a compris comment créer et imprimer un arbre d'éléments dans le DOM via les Composants, on va pouvoir s'attaquer au sujet réel de ce TP : La gestion d'une liste des tâches !

## Sommaire
  * [But de l'exercice](#but-de-l-exercice)
  * [Modéliser une liste de tâches dans le JS](#modéliser-une-liste-de-t-ches-dans-le-js)
  * [Impossibilité d'agir sur les checkbox](#impossibilité-d-agir-sur-les-checkbox)
  * [Dynamisme des composants : la notion d'état (state)](#dynamisme-des-composants---la-notion-d-état--state-)
  * [Flux de données dans un composant](#flux-de-données-dans-un-composant)
  * [Dynamisons la liste des tâches](#dynamisons-la-liste-des-tâches)
  * [Refactoring avant de continuer](#refactoring-avant-de-continuer)
  * [Ce que vous avez appris](#ce-que-vous-avez-appris)

## But de l'exercice

Nous allons créer un composant ***TodoList*** dont le but sera d'afficher dynamiquement une liste de tâches.

**Ici, on essayera de comprendre le flot des données dans un composant React**.

## Modéliser une liste de tâches dans le JS

Créons tout d'abord la liste des tâches (en dur pour l'instant) au sein de notre fichier *src/app.js*

A terme, ces tâches proviendront d'une base de données distante, mais pour l'instant, notre but est d'étudier le fonctionnement des composants.

```js
// src/app.js

// On créé ici un tableau TODO_ITEMS qui contient deux objets 
const TODO_ITEMS = [
  { id: 1, text: "Faire les courses", done: false },
  { id: 2, text: "Aller chercher les enfants", done: true },
];
```

Comme dans toute application JS (et donc React ne fera pas exception), l'architecture est telle que :
* Les données et les traitements sont gérés en JS ;
* L'affichage est géré en HTML ;

Le but sera donc d'utiliser ce tableau de tâches afin de créer un arbre d'éléments qu'on ira ensuite imprimer dans l'interface (le DOM).

On va créer un composant **TodoList** de telle sorte qu'il créé un `<ul>` et autant de `<li>` qu'il y'a de tâches :

```jsx
// src/app.js

// React va permettre de dessiner notre arbre d'éléments HTML
import React from "react";
// ReactDOM va permettre de créer le rendu correspondant dans le DOM HTML
import ReactDOM from "react-dom";

// On créé ici un tableau TODO_ITEMS qui contient deux objets 
const TODO_ITEMS = [
  { id: 1, text: "Faire les courses", done: false },
  { id: 2, text: "Aller chercher les enfants", done: true },
];

const TodoList = () => {
    // On retourne une list <ul> qui contient un tableau d'éléments React
    // Chaque objet de TODO_ITEMS sera transformé en un <li> contenant les détails de la tâche
    // Les éléments React générés par une boucle doivent avoir une "key" unique
    return <ul>
        {TODO_ITEMS.map(item => <li key={item.id}>
            <label>
                <input type="checkbox" id={"todo-" + item.id} checked={item.done} />
                {item.text}
            </label>
        </li>)}
    </ul>
}

// Imprime l'arbre renvoyé par TodoList() dans l'élément <main> du DOM HTML
ReactDOM.render(<TodoList />, document.querySelector('main'));
```

Et voilà ! On  a une liste de tâches qui s'affiche sur la page !

## Impossibilité d'agir sur les checkbox
Vous avez peut-être eu la tentation humaine de cliquer sur les checkbox que nous avons généré .. Et vous avez sans doute remarqué que vous n'avez pas le droit d'y toucher !

En effet, le fonctionnement de React est tel que si la valeur (ou l'état) d'un élément HTML (ici notre `<input type="checkbox" />`) est liée à une *donnée* (variable) du composant, l'utilisateur ne peut la changer.

**Rappelez vous le fameux flux de l'application JS :** Les données sont gérées dans le JS, pas dans le HTML. Il n'y a donc pas de raison que le HTML puisse influencer sur les données (ici le fait que la tâche soit faite ou pas) SAUF si le HTML appelle un traitement disponible dans le JS.

## Dynamisme des composants : la notion d'état (state)

Alors comment allons nous permettre à l'utilisateur de modifier le statut d'une checkbox ? **Grâce à la notion d'Etat (state)** !

L'état d'un composant représente les données sur lesquelles il s'appuie pour générer un rendu visuel. **Le but ultime de React est que l'arbre d'éléments qu'il génère soit toujours fidèle aux données présentes dans cet arbre**.

L'idée est la suivante :
```js

// En imaginant une donnée quelconque :
let name = "Joseph";

// Et un élément React (affichera <p>Hello Joseph</p>)
<p>Hello {name}</p>

// Le but de React est de faire en sorte que si la donnée change, alors l'élément se mette à jour : l'élément devra réagir (React ;-)) au changement de la donnée

// Modification de la donnée :
name = "Elise"; // Devrait afficher <p>Hello Elise</p>
```

Vous l'avez compris, l'idée est que si un élément repose sur une variable pour afficher quelque chose, cet élément devra être mis à jour dynamiquement si la donnée change.

React, au contraire d'autres technologies (comme Svelte ou Angular) ne peut pas suivre chaque variable et ses changements afin de réagir à chaque fois qu'une d'entre elle change (on appelle cela un *mécanisme de détection de changement*). 

Il nous donne donc un moyen de représenter des variables un peu spéciales qu'il pourra suivre lors de leurs évolutions : **le state (état).**

Voyons en exemple comment créer un état dans un composant avec la fonction (hook) `useState()` :

```js
// Reprenons l'exemple précédent en utilisant un state plutôt qu'une variable classique
// La fonction useState() permet de créer une variable ET une fonction de mutation
// de cette variable : il ne faudra jamais essayer de modifier un state sans passer par cette fonction de mutation. 
// La fonction prend en paramètre la valeur par défaut que prendra la variable :
const [name, setName] = useState("Joseph")

// Affichera <p>Hello Joseph</p>
<p>Hello {name}</p>

// Dès que la fonction setName() sera appelée, le composant sera "actualisé" dans notre page, en tenant compte de la nouvelle valeur de la variable :
setName("Elise"); 

// Le composant va dynamiquement changer l'affichage pour <p>Hello Elise</p>
```

## Flux de données dans un composant 

On comprend donc que pour espérer obtenir un dynamisme de la part de React, il nous faut nous appuyer sur la notion de **state**. Les données qui pourront changer et dont le changement aura des répercutions sur l'affichage devront être des états et pas de simples variables.

Plus encore, lorsque l'utilisateur voudra avoir un impact sur ces données, il devra appeler un traitement que le composant met à disposition et qui mutera l'état. React fera ensuite son travail : il actualisera l'arbre des éléments à imprimer dans le DOM avec la nouvelle valeur de la variable !

## Dynamisons la liste des tâches

On va donc mettre en oeuvre ces notions dans notre composant *TodoList* et en permettant à l'utilisateur de modifier l'état des tâches (notamment leur statut "fait" ou "pas fait") :

```diff
// src/app.js

+ import React, { useState } from "react";
import ReactDOM from "react-dom";

// On créé ici un tableau TODO_ITEMS qui contient deux objets 
const TODO_ITEMS = [
  { id: 1, text: "Faire les courses", done: false },
  { id: 2, text: "Aller chercher les enfants", done: true },
];

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

En testant ces modifications sur le navigateur, vous constaterez qu'on a réussi à faire en sorte que notre composant soit réellement dynamique !

Il se base sur un état pour créer un arbre d'éléments. Si l'on change l'état, il va regénérer un nouvel arbre d'éléments tenant compte de cette modification et va imprimer ce nouvel arbre sur l'interface (dans le DOM) !

## Refactoring avant de continuer

Comme on compte créer une application assez large dans ce TP, essayons d'organiser notre code au mieux en refactorisant. Déplaçons le code du composant TodoList dans son propre fichier *src/components/TodoList.js* :

```jsx
// src/components/TodoList.js

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

Et faisons appel à ce composant dans le fichier principal *src/app.js* :

```jsx
// src/app.js

// React va permettre de dessiner notre arbre d'éléments HTML
import React from "react";
// ReactDOM va permettre de créer le rendu correspondant dans le DOM HTML
import ReactDOM from "react-dom";
// On importe la fonction TodoList
import TodoList from "./components/TodoList";


// Imprime l'arbre renvoyé par TodoList() dans l'élément <main> du DOM HTML
ReactDOM.render(<TodoList />, document.querySelector('main'));
```

# Ce que vous avez appris
* Boucler sur un tableau grâce à `.map()` pour afficher ses éléments ;
* Notion de flux de données dans un composant ;
* Créer un *state* grâce au Hook `useState()` ;
* Générer du dynamisme dans un composant en s'appuyant sur un état et en le modifiant ;

[Revenir au sommaire](../README.md) ou [Passer à la suite : Ajouter une tâche](add-item.md)
