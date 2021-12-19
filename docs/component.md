# Premier composant React

Puisque notre but est de mettre en pratique React, il nous faut absolument avoir une idée de ce que sont les **composants**, briques essentielles de toute application ou même script React !

## Sommaire
  * [But de l'exercice](#but-de-l-exercice)
  * [Dépendances à installer](#dépendances-à-installer)
  * [Vérifions le fonctionnement avec un premier composant React](#vérifions-le-fonctionnement-avec-un-premier-composant-react)
  * [A la découverte du JSX](#a-la-découverte-du-jsx)
  * [Babel, l'acteur obligé](#babel--l-acteur-obligé)
  * [Ce que vous avez appris](#ce-que-vous-avez-appris)

## But de l'exercice

Nous allons installer la librairie React et l'utiliser afin de créer notre premier composant. Nous constaterons que pour écrire des interfaces complexes, React seul arrive à une limite, et on utilisera alors la syntaxe JSX qui nous facilitera grandement l'écriture et la lecture de notre code !

## Dépendances à installer

Commençons par installer la librairie React au sein de notre projet grâce à NPM :

```bash
npm install react react-dom
```

Comme vous le remarquez, nous n'installons pas simplement React mais aussi une autre librairie : React DOM.

* React permet de représenter un arbre d'éléments et de gérer le dynamisme au sein de cet arbre (tel élément doit apparaitre, tel autre va changer, tel autre disparait, etc.) en fonction des changements qui impactent les "états" dans les éléments de cet arbre ;
* React DOM, elle, permet d'imprimer l'arbre d'éléments géré grâce à React dans l'interface HTML du navigateur (le DOM) ;

Vous l'aurez compris : React et ses concepts sont en réalité déclinables dans divers environnements (web ou mobile par exemple), et React DOM est la librairie qui permet d'utiliser React dans le navigateur.

## Vérifions le fonctionnement avec un premier composant React

Afin de vérifier que tout fonctionne bien pour la suite du projet, on va créer un premier **composant React** extrêmement simple : le composant Hello !

Son but sera de représenter sur la page HTML un arbre d'éléments. Il devra donc construire cet arbre d'éléments, puis ReactDOM imprimera cet arbre dans le DOM.

```js
// src/app.js

// React va permettre de dessiner notre arbre d'éléments HTML
import React from "react";
// ReactDOM va permettre de créer le rendu correspondant dans le DOM HTML
import ReactDOM from "react-dom";

/**
 * Créé et retourne un arbre d'éléments React
 * @returns React.DetailedReactHTMLElement
 */
const Hello = () => {
    // On créé un élément <div> qui aura les classes "example-class" et "example-class-2", ainsi que l'id "lorem-id"
    return React.createElement('div', { className: "example-class example-class-2", id: "lorem-id" }, [
        // Contenant un élement <p>
        React.createElement('p', {}, [
            // Contenant lui même un élément <strong> qui contient la string "Hello "
            React.createElement('strong', {}, "Hello "),
            // Et un élément TEXT "World"
            "World"
        ])
    ]);
}

// Imprime l'arbre renvoyé par hello() dans l'élément <main> du DOM HTML
ReactDOM.render(React.createElement(Hello), document.querySelector('main'));
```

Vous remarquez que le composant Hello n'est en fait qu'une fonction. Elle a tout de même une spécificité : elle retourne un élément React (qui lui même peut en contenir une infinité d'autres). Cet élément React qui est retourné est la racine de l'arbre d'éléments qui constitue notre composant.

L'avantage de cette configuration est de permettre d'afficher plusieurs fois, à différents moments ou endroits dans notre application web, le même arbre d'éléments, simplement en appelant la fonction `Hello()`.

**Attention : tous les composants React (donc les fonctions qui retournent un élément React) doivent commencer par une majuscule, c'est pourquoi la fonction se nomme `Hello()` et pas `hello()`**

Normalement, Webpack doit être capable de comprendre et d'opérer la construction de notre fichier final  *dist/app.js*. Lançons donc le bundling avec :

```bash
npm run dev
```

En vérifiant sur notre navigateur, on devrait constater que les éléments de l'arbre React ont bien été appliqués au DOM, et qu'ils apparaissent !

## A la découverte du JSX

Notez tout de même la difficulté qu'a été la notre à écrire cet arbre d'éléments pourtant extrêmement simple. Difficulté d'écriture mais aussi de lecture. Le code manque de clarté, et imaginez ce que ce serait avec un arbre plus complexe !

**C'est là qu'entre en jeu la syntaxe JSX** : C'est une syntaxe hétérodoxe, qui n'existe pas dans le standard Javascript et qui sera absolument incompréhensible pour les navigateurs.

Elle permet d'écrire les appels à `React.createElement()` sous la forme de **balises** ouvrantes et fermantes.

Voici un exemple :

```jsx
// Code JSX :
<div className="rouge" id="app" onClick={() => alert("Hello !")}>
    <p>Cliquez ici !</p>
</div>

// Equivalent JS Vanilla :
React.createElement('div', { className: "rouge", id: "app", onClick: () => alert("Hello !") }, [
    React.createElement('p', {}, "Cliquez ici")
]);
```

On remarque évidemment que le code JSX est bien plus lisible que le code vanilla (natif / classique).

Voyons ce que ça donnerait pour notre composant Hello :

```diff
/**
 * Créé et retourne un arbre d'éléments React
 * @returns React.DetailedReactHTMLElement
 */
const Hello = () => {
+    return <div className="example-class example-class-2" id="lorem-id">
+        <p>
+            <strong>Hello </strong>
+            World
+        </p>
+    </div>;
-    // On créé un élément <div> qui aura les classes "example-class" et "example-class-2", ainsi que l'id "lorem-id"
-    return React.createElement('div', { className: "example-class example-class-2", id: "lorem-id" }, [
-        // Contenant un élement <p>
-        React.createElement('p', {}, [
-            // Contenant lui même un élément <strong> qui contient la string "Hello "
-            React.createElement('strong', {}, "Hello "),
-            // Et un élément TEXT "World"
-            "World"
-        ])
-    ]);
}


// Imprime l'arbre renvoyé par hello() dans l'élément <main> du DOM HTML
-ReactDOM.render(React.createElement(Hello), document.querySelector('main'));
+ReactDOM.render(<Hello />, document.querySelector('main'));
```

C'est beaucoup plus propre, néanmoins, n'oubliez pas que ce code est absolument incrompréhensible pour un navigateur !

Il y'a fort à parier que même Webpack est complètement perdu à la lecture de cette syntaxe.

## Babel, l'acteur obligé

C'est pourquoi tout projet React qui souhaite utiliser la syntaxe JSX se repose forcément sur **Babel** !

Babel est une librairie qui est capable de lire un code Javascript non standard afin de le traduire en code Javascript classique, et donc lisible et compréhensible par n'importe quel navigateur.

Nous allons donc installer la librairie ainsi que certaines de ses dépendances et plugins :

```bash
npm install --save-dev @babel/core @babel/cli @babel/preset-env @babel/preset-react babel-loader
```

Une fois les dépendances installées, nous devrons modifier la configuration de Webpack afin de lui faire comprendre que certains fichiers devront passer par Babel avant d'être intégrés à notre fichier final *dist/app.js* :

```js
// webpack.config.js

// Permet de résoudre des chemins de fichiers
const path = require("path");

module.exports = {
    // Le fichier qui sert de point d'entrée et qui importe les différentes dépendances de l'application
    entry: "./src/app.js",
    // Le dossier et nom du fichier qui sera généré par Webpack après le build
    // Webpack va donc intégrer l'ensemble des modules (dépendances) nécessaires dans un seul fichier dist/app.js
    output: {
        filename: "app.js",
        path: path.resolve(__dirname, "dist"),
    },
    // Explique la façon dont nos modules JS devront être traités
    module: {
        // Permet d'expliciter les règles à suivre en fonction des fichiers
        rules: [
            {
                // Tout fichier qui aura l'extension .js ou .jsx (aussi utilisé pour marquer les fichiers contenant du JSX)
                test: /\.(js|jsx)$/,
                // A l'exception du dossier node_modules
                exclude: /node_modules/,
                // Devra être traité via babel
                loader: "babel-loader",
                options: { presets: ["@babel/env"] }
            },
        ]
    }
};
```

Par ailleurs, il faut configurer aussi Babel afin qu'il soit capable de comprendre le JSX et de le traduire :

```js
// babel.config.js
module.exports = {
    presets: [["@babel/preset-env", { targets: { node: 'current' } }], "@babel/preset-react"]
}
```

Afin de vérifier si tout fonctionne bien, vous devriez arrêter la commande `npm run dev` et la relancer. Normalement, le bundling se passe bien, votre navigateur n'affiche aucune erreur, et le composant Hello s'affiche bien à l'écran !

# Ce que vous avez appris
* Utiliser React pour créer un arbre d'éléments en Vanilla JS ;
* Utiliser la syntaxe JSX pour écrire et lire plus facilement le code relatif à React ;
* Installer Babel et configurer Webpack afin qu'il s'appui sur lui pour traduire le code JSX en code vanilla compréhensible par les navigateurs ;

[Revenir au sommaire](../README.md) ou [Passer à la suite : Afficher une liste des tâches](display-list.md)