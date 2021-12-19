# Créer une application React JS (Todo List)

Dans ce TP, vous allez créer une Single Page Application (SPA) avec React dans le but de découvrir différentes notions telles que les composants, les states et props, la gestion des modules et du bundling, les appels HTTP et les promesses, le routage et bien sur les tests unitaires qui assureront la qualité et la non régression de vos codes.

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
    * [Initialiser la gestion de dépendances dans le projet](docs/setup.md#initialiser-la-gestion-de-dépendances-dans-le-projet)
    * [Installer le module bundler Webpack](docs/setup.md#installer-le-module-bundler-webpack)
    * [Lancer l'application dans le navigateur](docs/setup.md#lancer-l-application-dans-le-navigateur)
    * [Vérification de l'outillage et des liens](docs/setup.md#vérification-de-l-outillage-et-des-liens)
    * [L'avantage du watch avec Wepback](docs/setup.md#l-avantage-du-watch-avec-wepback)
    * [Ce que vous avez appris](docs/setup.md#ce-que-vous-avez-appris--)
* [**Premier composant React**](docs/component.md)
  * [But de l'exercice](docs/component.md#but-de-l-exercice)
  * [Dépendances à installer](docs/component.md#dépendances-à-installer)
  * [Vérifions le fonctionnement avec un premier composant React](docs/component.md#vérifions-le-fonctionnement-avec-un-premier-composant-react)
  * [A la découverte du JSX](docs/component.md#a-la-découverte-du-jsx)
  * [Babel, l'acteur obligé](docs/component.md#babel--l-acteur-obligé)
  * [Ce que vous avez appris](docs/component.md#ce-que-vous-avez-appris)
* [**Afficher une liste dynamique**](docs/display-list.md)
  * [But de l'exercice](docs/display-list.md#but-de-l-exercice)
  * [Modéliser une liste de tâches dans le JS](docs/display-list.md#modéliser-une-liste-de-t-ches-dans-le-js)
  * [Impossibilité d'agir sur les checkbox](docs/display-list.md#impossibilité-d-agir-sur-les-checkbox)
  * [Dynamisme des composants : la notion d'état (state)](docs/display-list.md#dynamisme-des-composants---la-notion-d-état--state-)
  * [Flux de données dans un composant](docs/display-list.md#flux-de-données-dans-un-composant)
  * [Dynamisons la liste des tâches](docs/display-list.md#dynamisons-la-liste-des-tâches)
  * [Refactoring avant de continuer](docs/display-list.md#refactoring-avant-de-continuer)
  * [Ce que vous avez appris](docs/display-list.md#ce-que-vous-avez-appris)
* [**Ajout d'une nouvelle tâche**](docs/add-item.md)
  * [But de l'exercice](docs/add-item.md#but-de-l-exercice)
  * [Afficher le formulaire](docs/add-item.md#afficher-le-formulaire)
  * [Suivre la valeur de l'input à tout moment grâce à un état](docs/add-item.md#suivre-la-valeur-de-l-input-à-tout-moment-grâce-à-un-état)
  * [Gestion de la soumission et ajout de la tâche](docs/add-item.md#gestion-de-la-soumission-et-ajout-de-la-tâche)
  * [Ce que vous avez appris](docs/add-item.md#ce-que-vous-avez-appris)
* [**Remontée d'état : communication entre composants**](docs/lifting.md)
  * [But de l'exercice](docs/lifting.md#but-de-l-exercice)
  * [Extraction du code du formulaire vers un composant TaskForm](docs/lifting.md#extraction-du-code-du-formulaire-vers-un-composant-taskform)
  * [Le problème du partage d'état](docs/lifting.md#le-problème-du-partage-d-état)
  * [(Re)Découverte des props](docs/lifting.md#-re-découverte-des-props)
  * [La limite des props](docs/lifting.md#la-limite-des-props)
  * [La solution : *lifting state up*](docs/lifting.md#la-solution----lifting-state-up-)
  * [Ce que vous avez appris](docs/lifting.md#ce-que-vous-avez-appris)
* [**Appels HTTP et API REST**](docs/http.md)
  * [But de l'exercice :](docs/http.md#but-de-l-exercice--)
  * [Créer un projet sur Supabase :](docs/http.md#créer-un-projet-sur-supabase--)
  * [Comprendre comment fonctionne l'API de Supabase](docs/http.md#comprendre-comment-fonctionne-l-api-de-supabase)
  * [Insérer une nouvelle tâche (Requêtes HTTP et Promesses)](docs/http.md#insérer-une-nouvelle-tâche--requêtes-http-et-promesses-)
    + [Système de Promesses](docs/http.md#système-de-promesses)
    + [Enchaînement de comportements](docs/http.md#enchaînement-de-comportements)
    + [La fonction fetch()](docs/http.md#la-fonction-fetch--)
  * [Afficher la liste lors du chargement de la page](docs/http.md#afficher-la-liste-lors-du-chargement-de-la-page)
  * [Passer les éléments à "fait" ou "pas fait"](docs/http.md#passer-les-éléments-à--fait--ou--pas-fait-)
  * [Ce que vous avez appris](docs/http.md#ce-que-vous-avez-appris--)
* [**Refactoring des appels HTTP et système de modules**](docs/refactoring.md)
  * [But de l'exercice](docs/refactoring.md#but-de-l-exercice--)
  * [Créer un fichier http.js qui contiendra les appels HTTP](docs/refactoring.md#créer-un-fichier-httpjs-qui-contiendra-les-appels-http)
  * [Ce que vous avez appris](docs/refactoring.md#ce-que-vous-avez-appris)
* [**Routing et affichage dynamique**](docs/routing.md)
  * [But de l'exercice](docs/routing.md#but-de-l-exercice)
  * [Installation des dépendances (react-router-dom)](docs/routing.md#installation-des-dépendances--react-router-dom-)
  * [Un composant pour afficher le détail d'une tâche](docs/routing.md#un-composant-pour-afficher-le-détail-d-une-tâche)
  * [Mise en place du routeur](docs/routing.md#mise-en-place-du-routeur)
  * [Récupérer un paramètre dans l'URL](docs/routing.md#récupérer-un-paramètre-dans-l-url)
  * [Récupérer la tâche dans l'API et l'afficher](docs/routing.md#récupérer-la-tâche-dans-l-api-et-l-afficher)
  * [Le problème du rechargement de page](docs/routing.md#le-problème-du-rechargement-de-page)
  * [Ce que vous avez appris](docs/routing.md#ce-que-vous-avez-appris)
* [**Tester ses composants React**](docs/tests.md)
  * [But de l'exercice](docs/tests.md#but-de-l-exercice)
  * [Installation de Jest](docs/tests.md#installation-de-jest)
  * [Premier test avec Jest](docs/tests.md#premier-test-avec-jest)
  * [Installation de React Testing Library](docs/tests.md#installation-de-react-testing-library)
  * [Tester le composant TodoList](docs/tests.md#tester-le-composant-todolist)
  * [Tester le composant TaskForm](docs/tests.md#tester-le-composant-taskform)
  * [Enfin, on teste l'intégration entre *TodoList* et *TaskForm* !](docs/tests.md#enfin--on-teste-l-intégration-entre--todolist--et--taskform---)

