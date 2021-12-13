# Ajouter une tâche

## Sommaire

## But de l'exercice

## Afficher le formulaire 

```jsx
// src/pages/TodoList.js

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

## Impossibilité de modifier l'input
```diff
const TodoList = () => {
  // ...
  
+ // Créons un état (qui pourra évoluer dynamiquement) qui représentera
+ // la chaîne tapée par l'utilisateur dans l'<input>
+ const [text, setText] = useState('');

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
+             value={text}
+             onChange={updateText}
            />
            <button>Ajouter</button>
        </form>
    </>;
}
```

## Gestion de la soumission et ajout de la tâche
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

# Ce que vous avez appris