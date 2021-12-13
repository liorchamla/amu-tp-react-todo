# Premier composant React

## Sommaire

## But de l'exercice

## Dépendances à installer

npm install react react-dom

## Vérifions le fonctionnement avec un premier composant React

```js
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

npm run dev

## A la découverte du JSX

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

## Babel, l'acteur obligé

npm install --save-dev @babel/core @babel/cli @babel/preset-env @babel/preset-react babel-loader

```js
// babel.config.js
module.exports = {
    presets: ["@babel/env", "@babel/preset-react"]
}
```

# Ce que vous avez appris