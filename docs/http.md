# Appels HTTP vers une API REST

Pour l'instant, les données de notre applications sont **particulièrement éphémères** ! En effet, à chaque réactualisation du navigateur, les tâches reviennent au départ. Il existe plusieurs possibilités pour pallier à ce problème :

1. Le stockage local : on peut tout à fait décider de stocker les données de notre application sur le navigateur via le *LocalStorage* (dont la durée de vie est illimitée sauf intervention de l'utilisateur) ou le *SessionStorage* (dont la durée de vie est relativement courte) ;
2. Le stockage distant : on peut décider au contraire de stocker les données à distance, de telle sorte que la durée de vie du stockage soit réellement illimitée et **surtout que les données restent accessibles y compris sur d'autres navigateurs** ;


**Notez que Javascript n'a aucun moyen de se connecter à une base de données, qu'elle soit distante ou locale !**

Par contre, ce que Javascript sait très bien faire, c'est appeler un serveur web distant via des requêtes HTTP.

Il faut donc créer une application web backend qui aura accès à la base de données, et que l'on pourra appeler depuis notre javascript via des requêtes HTTP (le concept même d'API ;-)).

## Sommaire :
  * [But de l'exercice :](#but-de-l-exercice--)
  * [Créer un projet sur Supabase :](#créer-un-projet-sur-supabase--)
  * [Comprendre comment fonctionne l'API de Supabase](#comprendre-comment-fonctionne-l-api-de-supabase)
  * [Insérer une nouvelle tâche (Requêtes HTTP et Promesses)](#insérer-une-nouvelle-tâche--requêtes-http-et-promesses-)
    + [Système de Promesses](#système-de-promesses)
    + [Enchaînement de comportements](#enchaînement-de-comportements)
    + [La fonction fetch()](#la-fonction-fetch--)
  * [Afficher la liste lors du chargement de la page](#afficher-la-liste-lors-du-chargement-de-la-page)
  * [Passer les éléments à "fait" ou "pas fait"](#passer-les-éléments-à--fait--ou--pas-fait-)
  * [Ce que vous avez appris :](#ce-que-vous-avez-appris--)

## But de l'exercice :
Nous allons créer une base de données PostgreSQL sur un serveur distant avec une table qui permettra de stocker les données de nos tâches. 

Pour ce faire nous utiliserons un service sur mesure : Supabase (une alternative aux services Firebase de Google).

Cette solution nous permet non seulement de créer des bases de données, mais intègre aussi par défaut une API qui permet d'intéroger ces bases de données via des requêtes HTTP.

Nous allons donc :
1. Créer un compte sur le service Supabase (nécessite un compte GitHub)
2. Créer un projet et une base de données adéquate
3. Connecter notre application front à Supabase afin de stocker les données à distance

## Créer un projet sur Supabase :
[Rendez vous sur le service Supabase](https://supabase.com) et cliquez sur ***"Start your project"*** afin de vous y connecter.

Une fois authentifié via GitHub, vous pourrez créer un projet en cliquant sur ***New Project***.

La création du projet peut durer quelques minutes et vous pourrez ensuite aller sur le ***Table Editor*** de votre base de données et cliquer sur ***New Table*** afin de créer une table que vous appellerez ***todos*** et dont les champs seront :

| Nom du champ | Type | Defaut |
| --- | --- | --- |
| id | Laissez tel quel
| created_at | Laissez tel quel
| text | varchar | null |
| done | bool | false |

Vous pouvez enfin insérer des lignes selon vos choix afin d'avoir déjà des données d'exemple.

## Comprendre comment fonctionne l'API de Supabase

Avant de continuer, vous devrez tout de même essayer de comprendre l'API en vous rendant dans la documentation (icône de fichier sur la gauche) et en sélectionnant ***Bash*** sur l'onglet de droite pour véritablement voir les requêtes HTTP qui sont acceptables ainsi que les options.

Une fois dans la documentation de l'API, vous pourrez même sélectionner le menu "***todos***" dans la barre de gauche et voir les requêtes HTTP spécifiques à la table *todos*.

Vous allez avoir besoin de votre identifiant de projet ainsi que de la clé de sécurité. Vous trouverez ces informations dans la partie *Authentication* de la documentation de l'API.

## Insérer une nouvelle tâche (Requêtes HTTP et Promesses)

Nous allons modifier le composant *TodoListPage* afin que la création d'une tâche implique une requête HTTP vers Supabase :

```diff
// src/pages/TodoListPage.js 

+ const SUPABASE_URL = "https://IDENTIFIANT_SUPABASE.supabase.co/rest/v1/todos";
+ const SUPABASE_API_KEY = "CLE_API_SUPABASE";

const TodoListPage = () => {
    // ...

  const addNewTask = (text) => {
        const task = {
-         id: Date.now(),
          text: text,
          done: false
        };

+       // Appel HTTP vers Supabase en method POST
+       fetch(SUPABASE_URL, {
+           method: "POST",
+           body: JSON.stringify(task),
+           headers: {
+               "Content-Type": "application/json",
+               apiKey: SUPABASE_API_KEY,
+               Prefer: "return=representation",
+           },
+       })
+           .then((response) => response.json())
+           .then((items) => {
+               setState([...state, items[0]]);
+           });
    }

  // ...
}
```
Vous remarquez qu'on attend le retour de la requête HTTP avant de toucher à l'interface. On s'assure par là qu'on n'ajoute pas une tâche dans l'interface alors que par ailleurs on aurait eu un soucis avec l'API et la base de données (on appelle cela *l'approche pessimiste*).

Comme vous pouvez vous en douter, une requête HTTP n'est pas instantannée, et vous ne pouvez donc pas compter sur le résultat immédiatement dans votre code.

**C'est pourquoi Javascript nous fourni un système de code asynchrone : les Promesses**

### Système de Promesses
Une Promesse Javascript est un objet dont le but est de lancer un travail puis de vous permettre de décider quoi faire du résultat de ce travail.

On peut rattacher à une Promesse un comportement à exécuter en fin de travail grâce à la méthode `.then()` à qui nous devons confier une fonction.

La Promesse exécutera cette fonction en lui passant en paramètre le résultat de son travail.

### Enchaînement de comportements
Parfois, on a envie non pas d'attacher un unique comportement à la fin d'un Promesse, mais bien un enchaînement de comportements qui permettront de raffiner les données.

On peut donc enchaîner les `.then()` après une Promesse. Le premier de la chaîne recevra le résultat du travail de la Promesse, le deuxième récupèrera la valeur retournée par le `.then()` d'avant, et ainsi de suite.

### La fonction fetch()

Ici nous utilisons la fonction `fetch()` qui permet d'envoyer une requête HTTP en tâche de fond. La fonction renvoie donc une promesse qui contient ce comportement et qui nous permet de décider quoi faire lorsque la requête HTTP aura été exécutée et que la réponse HTTP du seveur sera disponible.

Vous remarquez que nous n'avons pas rattaché un unique comportement pour la fin de la Promesse `fetch()`. Nous avons :
1. Un premier `.then()` qui pourra travailler sur la réponse http et qui retournera le JSON contenu dans la réponse ;
2. Un deuxième `.then()` qui recevra la valeur retournée par le premier `.then()` (à savoir le JSON retourné par le serveur) et qui fera un nouveau traitement là dessus ;

## Afficher la liste lors du chargement de la page

Nous ne souhaitons plus utiliser le tableau `TODO_ITEMS`, nous pouvons donc supprimer toute référence à celui ci.

A la place, nous comptons récupérer les tâches via une requête HTTP et les afficher :

```jsx
// src/pages/TodoListPage.js

const TodoListPage = () => {
  // Désormais, on prend un tableau vide comme valeur par défaut
  // pour la liste des tâches. Ce tableau évoluera lors du chargement
  // du composant grâce au useEffect ci-dessous
  const [state, setState] = useState([]);

  // On utilise le hook useEffect, qui permet de créer un comportement
  // qui aura lieu lors de CHAQUE rendu du composant React
  // mais en passant un tableau de dépendances vide en deuxième paramètres, on explique à React que ce comportement 
  // ne devra avoir lieu qu'une seule fois, au chargement du composant
  useEffect(() => {
      // Appel HTTP vers Supabase
      fetch(`${SUPABASE_URL}?order=created_at`, {
          headers: {
              apiKey: SUPABASE_API_KEY,
          },
      })
          .then((response) => response.json())
          .then((items) => {
              // On remplace la valeur actuel de state
              // par le tableau d'items venant de l'API
              setState(items);
          });
  }, []);

  // ...
}
```

Vous remarquez ici l'utilisation du hook `useEffect` qui permet de lancer un comportement à chaque fois que React affiche quelque chose dans le DOM. On peut néanmoins mitiger (ou piloter) ce comportement en passant en second paramètre un tableau des *dépendances* : cela explique à React que le comportement ne devra être exécuté que si l'une des variables listées dans le tableau change. En donnant un tableau vide `[]`, on fait en sorte que le comportement ne se déclenche qu'au premier rendu du composant, car quelque soit les rendus suivants, le comportement ne sera rejoué que si une des variables listées a changé. Et comme on ne lui liste aucune variable, évidemment, elles n'ont pas changé.

## Passer les éléments à "fait" ou "pas fait"

On veut désormais aller un peu plus loin, de telle sorte qu'on permettre à l'utilisateur de décider si une tâche a été faite ou pas :

```js
// src/pages/TodoListPage.js

const TodoListPage = () => {

  const toggle = (id) => {
      // Récupérons l'index de la tâche concernée
      const idx = state.findIndex(task => task.id === id);

      // Créons une copie de la tâche concernée 
      const item = { ...state[idx] };

      // Appel HTTP en PATCH pour modifier la tâche
      fetch(`${SUPABASE_URL}?id=eq.${id}`, {
          method: "PATCH",
          headers: {
              "Content-Type": "application/json",
              apiKey: SUPABASE_API_KEY,
              Prefer: "return=representation",
          },
          body: JSON.stringify({ done: !item.done }),
      }).then(() => {
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

  //..
}
```

Et voilà, normalement, vous devriez désormais pouvoir afficher les tâches de la base de données, y ajouter une tâche et même modifier les tâches afin de faire varier leur statut !

# Ce que vous avez appris :
* La notion d'asynchronicité en Javascript avec les Promesses, qui permettent de lancer un travail tout en précisant quoi faire lorsque ce travail sera terminé ;
* L'enchaînement de `.then()` à la suite d'une promesse afin d'enchaîner les travaux en raffinant les données ;
* Découverte du service Supabase qui vous permet en quelques secondes de créer une base de données distante accessible par une API (mais pas que ;-)) ;
* Découverte de la fonction `fetch()` permettant d'envoyer des requêtes HTTP ;
* Découverte du hook `useEffect()` qui permet d'exécuter un comportement à un moment précis de la vie du composant ;

[Revenir au sommaire](../README.md) ou [Passer à la suite : Refactoring et modules](refactoring.md)
