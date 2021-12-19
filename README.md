# Créer une application Vanilla JS (Todo List)

Dans ce TP, vous allez créer une Single Page Application (SPA) en Vanilla JS dans le but de découvrir différentes notions telles que les affichages dynamiques, la gestion des modules et du bundling, les appels HTTP et les promesses, le routage et bien sur les tests unitaires qui assureront la qualité et la non régression de vos codes.

## Mettre en place le projet en local
Clonez ce dépôt où bon vous semble :
```bash
git clone https://github.com/liorchamla/amu-tp-react-todo.git
```
Ouvrez le dossier dans votre éditeur de code favoris !

## Travailler directement sur le navigateur
Plutôt que de cloner le dépôt, vous pouvez aussi travailler directement en ligne via Gitpod.

Pour cela il vous faudra *forker* le dépôt sur votre propre compte (bouton *Fork* en haut à droite de GitHub). Une fois sur la page du fork (la copie de ce dépôt sur votre propre compte), vous pourrez simplement ajouter `gitpod.io/#` juste devant l'URL du dépôt.

Par exemple, le lien suivant ouvrira **ce dépôt** dans Gitpod : https://gitpod.io/#https://github.com/liorchamla/amu-tp-react-todo

**Attention : le travail ne sera pas sauvegardé d'une session de travail à l'autre, vous devrez systématiquement commit et push vos modifications si vous espérez les retrouver ensuite !**



## Attention au départ !

Une fois que le projet est mis en place, vous pouvez tout simplement suivre le TP étape par étape :

* [**Installation des outils de développement**](docs/setup.md)
    * [Initialiser la gestion de dépendances dans le projet](#initialiser-la-gestion-de-dépendances-dans-le-projet)
    * [Installer le module bundler Webpack](#installer-le-module-bundler-webpack)
    * [Lancer l'application dans le navigateur](#lancer-l-application-dans-le-navigateur)
    * [Vérification de l'outillage et des liens](#vérification-de-l-outillage-et-des-liens)
    * [L'avantage du watch avec Wepback](#l-avantage-du-watch-avec-wepback)
    * [Ce que vous avez appris](#ce-que-vous-avez-appris--)
* [**Premier composant React**](docs/component.md)
  * [But de l'exercice](#but-de-l-exercice)
  * [Dépendances à installer](#dépendances-à-installer)
  * [Vérifions le fonctionnement avec un premier composant React](#vérifions-le-fonctionnement-avec-un-premier-composant-react)
  * [A la découverte du JSX](#a-la-découverte-du-jsx)
  * [Babel, l'acteur obligé](#babel--l-acteur-obligé)
  * [Ce que vous avez appris](#ce-que-vous-avez-appris)
* [**Afficher une liste dynamique**](docs/display-list.md)
  * [But de l'exercice](#but-de-l-exercice)
  * [Modéliser une liste de tâches dans le JS](#modéliser-une-liste-de-t-ches-dans-le-js)
  * [Impossibilité d'agir sur les checkbox](#impossibilité-d-agir-sur-les-checkbox)
  * [Dynamisme des composants : la notion d'état (state)](#dynamisme-des-composants---la-notion-d-état--state-)
  * [Flux de données dans un composant](#flux-de-données-dans-un-composant)
  * [Dynamisons la liste des tâches](#dynamisons-la-liste-des-tâches)
  * [Refactoring avant de continer](#refactoring-avant-de-continer)
  * [Ce que vous avez appris](#ce-que-vous-avez-appris)
* [**Ajout d'une nouvelle tâche**](docs/add-item.md)
  * [But de l'exercice](#but-de-l-exercice)
  * [Afficher le formulaire](#afficher-le-formulaire)
  * [Suivre la valeur de l'input à tout moment grâce à un état](#suivre-la-valeur-de-l-input-à-tout-moment-grâce-à-un-état)
  * [Gestion de la soumission et ajout de la tâche](#gestion-de-la-soumission-et-ajout-de-la-tâche)
  * [Ce que vous avez appris](#ce-que-vous-avez-appris)
* [**Remontée d'état : communication entre composants**](docs/lifting.md)
  * [But de l'exercice](#but-de-l-exercice)
  * [Extraction du code du formulaire vers un composant TaskForm](#extraction-du-code-du-formulaire-vers-un-composant-taskform)
  * [Le problème du partage d'état](#le-problème-du-partage-d-état)
  * [(Re)Découverte des props](#-re-découverte-des-props)
  * [La limite des props](#la-limite-des-props)
  * [La solution : *lifting state up*](#la-solution----lifting-state-up-)
  * [Ce que vous avez appris](#ce-que-vous-avez-appris)
* [**Appels HTTP et API REST**](docs/http.md)
  * [But de l'exercice :](#but-de-l-exercice--)
  * [Créer un projet sur Supabase :](#créer-un-projet-sur-supabase--)
  * [Comprendre comment fonctionne l'API de Supabase](#comprendre-comment-fonctionne-l-api-de-supabase)
  * [Insérer une nouvelle tâche (Requêtes HTTP et Promesses)](#insérer-une-nouvelle-tâche--requêtes-http-et-promesses-)
    + [Système de Promesses](#système-de-promesses)
    + [Enchaînement de comportements](#enchaînement-de-comportements)
    + [La fonction fetch()](#la-fonction-fetch--)
  * [Afficher la liste lors du chargement de la page](#afficher-la-liste-lors-du-chargement-de-la-page)
  * [Passer les éléments à "fait" ou "pas fait"](#passer-les-éléments-à--fait--ou--pas-fait-)
  * [Ce que vous avez appris](#ce-que-vous-avez-appris--)
* [**Refactoring des appels HTTP et système de modules**](#refactoring-des-appels-http-et-système-de-modules)
  * [But de l'exercice](#but-de-l-exercice--)
  * [Créer un fichier http.js qui contiendra les appels HTTP](#créer-un-fichier-httpjs-qui-contiendra-les-appels-http)
  * [Ce que vous avez appris](#ce-que-vous-avez-appris)
* [**Routing et affichage dynamique**](docs/routing.md)
  * [But de l'exercice](#but-de-l-exercice)
  * [Installation des dépendances (react-router-dom)](#installation-des-dépendances--react-router-dom-)
  * [Un composant pour afficher le détail d'une tâche](#un-composant-pour-afficher-le-détail-d-une-tâche)
  * [Mise en place du routeur](#mise-en-place-du-routeur)
  * [Récupérer un paramètre dans l'URL](#récupérer-un-paramètre-dans-l-url)
  * [Récupérer la tâche dans l'API et l'afficher](#récupérer-la-tâche-dans-l-api-et-l-afficher)
  * [Le problème du rechargement de page](#le-problème-du-rechargement-de-page)
  * [Ce que vous avez appris](#ce-que-vous-avez-appris)
* [**Tester ses composants React**](#tester-ses-composants-react)
  * [But de l'exercice](#but-de-l-exercice)
  * [Installation de Jest](#installation-de-jest)
  * [Premier test avec Jest](#premier-test-avec-jest)
  * [Installation de React Testing Library](#installation-de-react-testing-library)
  * [Tester le composant TodoList](#tester-le-composant-todolist)
  * [Tester le composant TaskForm](#tester-le-composant-taskform)
  * [Enfin, on teste l'intégration entre *TodoList* et *TaskForm* !](#enfin--on-teste-l-intégration-entre--todolist--et--taskform---)

