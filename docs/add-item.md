# Ajouter une tâche

Continuons notre plongée dans React et avançons sur notre gestion des tâches en permettant à l'utilisateur de créer une nouvelle tâche !

## Sommaire
  * [But de l'exercice](#but-de-l-exercice)
  * [Afficher le formulaire](#afficher-le-formulaire)
  * [Suivre la valeur de l'input à tout moment grâce à un état](#suivre-la-valeur-de-l-input-à-tout-moment-grâce-à-un-état)
  * [Gestion de la soumission et ajout de la tâche](#gestion-de-la-soumission-et-ajout-de-la-tâche)
  * [Ce que vous avez appris](#ce-que-vous-avez-appris)

## But de l'exercice
Nous allons créer un formulaire dans le composant *TodoList* et apprendre comment réagir à sa soumission pour ajouter une nouvelle tâche à la liste !

**Ici, on se concentrera sur la façon dont on peut intercepter la soumission d'un formulaire et impacter le state afin de déclencher un nouveau rendu du composant**

## Afficher le formulaire 

Commençons donc par ajouter un formulaire dans l'arbre de notre composant *TodoList* :

```jsx
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
        <form>
            <input 
              type="text" 
              name="todo-text" 
              placeholder="Ajouter une tâche" 
            />
            <button>Ajouter</button>
        </form>
    </>;
}
```

Voilà, la structure HTML étant faite, on veut maintenant s'assurer qu'on pourra à tout moment connaître la valeur de l'input afin de s'en servir.

## Suivre la valeur de l'input à tout moment grâce à un état

Encore une fois, rappelez vous toujours le flux de données dans un composant React (comme dans toute application JS) : les données sont gérée dans le JS, pas dans l'interface HTML.

Nous souhaitons donc que quoique l'utilisateur tape dans l' `<input>`, nous sachions toujours ce qu'il en est.

Et dans React, les données dont on souhaite suivre l'évolution sont des **états** !

Créons donc un état dont le but sera de représenter la chaîne tapée par l'utilisateur dans l'input :

```diff
const TodoList = () => {
  // ...
  
+ // Créons un état (qui pourra évoluer dynamiquement) qui représentera
+ // la chaîne tapée par l'utilisateur dans l'<input>
+ const [text, setText] = useState('');

  return <>
        <ul>
            {state.map(item => <li key={item.id}>
                <label>
                    <input type="checkbox" id="todo-${item.id}" checked={item.done} onChange={() => toggle(item.id)} />
                    {item.text}
                </label>
            </li>)}
        </ul>
        <form>
            <input 
              type="text" 
              name="todo-text" 
              placeholder="Ajouter une tâche" 
+             value={text}
            />
            <button>Ajouter</button>
        </form>
    </>;
}
```

Si vous ouvrez votre navigateur, vous constaterez surement que tout fonctionne, si ce n'est que vous n'avez plus le droit de taper quoique ce soit dans l'input ! C'est le même problème que la checkbox de l'exercice précédent !

React a eu pour ordre que le rendu visuel corresponde à son **état**. Or la variable *text* (notre état) ne contient rien. Quoique vous tapiez dans l'input, React le refusera car l'arbre d'éléments qu'il affiche DOIT SE CONFORMER aux données de l'état du composant.

Comme dans l'exercice précédent, il nous suffit de permettre à l'utilisateur d'appeler depuis l'interface HTML un comportement du composant qui aura pour but de mettre à jour l'état. React fera ensuite sa part du travail en réaffichant le composant :

```diff
const TodoList = () => {
  // ...
  
  // Créons un état (qui pourra évoluer dynamiquement) qui représentera
  // la chaîne tapée par l'utilisateur dans l'<input>
  const [text, setText] = useState('');

+ // La fonction sera appelée à chaque changement sur l'<input>
+ // Elle recevra les détails de l'événement
+ const updateText = (event) => {
+     // On fait évoluer le state "text" en remplaçant la valeur
+     // acutelle par la valeur mise à jour de l'<input>
+     setText(event.target.value);
+ }

  return <>
        <ul>
            {state.map(item => <li key={item.id}>
                <label>
                    <input type="checkbox" id="todo-${item.id}" checked={item.done} onChange={() => toggle(item.id)} />
                    {item.text}
                </label>
            </li>)}
        </ul>
        <form>
            <input 
              type="text" 
              name="todo-text" 
              placeholder="Ajouter une tâche" 
              value={text}
+             onChange={updateText}
            />
            <button>Ajouter</button>
        </form>
    </>;
}
```
Et voilà ! On respecte le flux des données : 
* React s'appuie sur le state *text* pour afficher le composant ;
* Lorsque l'utilisateur tape dans l'input, on modifie le state *text* ; 
* React réagit (:D :D) en réaffichant une nouvelle version de l'arbre s'appuyant sur la nouvelle donnée ;

## Gestion de la soumission et ajout de la tâche

Désormais, on veut réagir à la soumission du formulaire afin d'afficher une nouvelle tâche. Et si vous avez compris le système des *states*, c'est super simple :
1. React s'appuie sur le tableau *state* pour afficher une liste ;
2. Nous ajoutons une entrée dans le tableau *state* représentant la nouvelle tâche ;
3. React réagit et affiche un nouvel arbre tenant compte de la mutation de l'état ;

Mettons en place ce code dans le composant *TodoList* :

```jsx
const TodoList = () => {
  // ...

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

  return <>
      <ul>
          {state.map(item => <li key={item.id}>
              <label>
                  <input type="checkbox" id="todo-${item.id}" checked={item.done} onChange={() => toggle(item.id)} />
                  {item.text}
              </label>
          </li>)}
      </ul>
      <form onSubmit={handleSubmit}>
          <input 
            type="text" 
            name="todo-text" 
            placeholder="Ajouter une tâche" 
            onChange={updateText} 
            value={text} 
          />
          <button>Ajouter</button>
      </form>
  </>
}
```
Et voilà ! Nous nous sommes appuyé sur l'événement `onSubmit` du formulaire pour déclencher une action dont le but est d'ajouter une tâche au tableau `state`, faisant ainsi réagir le composant.

Rendez-vous sur le navigateur pour tester que tout fonctionne bien !

# Ce que vous avez appris
* Suivre la valeur d'un input grâce à un state ;
* Intercépter la soumission d'un formulaire grâce à l'événement `onSubmit` ;
* Voir et revoir le flux des données dans un composant ;


[Revenir au sommaire](../README.md) ou [Passer à la suite : Refactoring et problème de partage d'état](lifting.md)