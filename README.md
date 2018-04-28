
# Revault
> The state management library you were waiting for

## Table Of Contents
1. [Why Revault?](#why-revault)
1. [Usage](#usage)
1. [Debugging](#debugging)
1. [Docs](#docs)

## Why Revault?
[Redux](https://github.com/reactjs/redux) is great and without doubt has helped push the web forward by providing a strong mental model around global state. Lately however, a few things have started to frustrate me when using Redux:

1. Repetition. The boilerplate, just for the most simple task, has become wearisome.
1. Logic. Business logic is spread out. Some logic lives in the action, some lives in the reducer, and yet some more lives in the component.
1. Middleware. Apart from logging and handling promises, middleware is more of a hassle than a help.

A few libaries have been released recently that have attempted to solve these issues. Two to note, and mainly where inspiration for Revault was drawn, are [Unstated](https://github.com/jamiebuilds/unstated) and [Statty](https://github.com/vesparny/statty). Both libraries use a basic component with [render props](https://reactjs.org/docs/render-props.html) to access global state ❤️ - but each had shortcomings.

Unstated's `Container` works well to control business logic and encapsulate several pieces of state, all while feeling very familiar to Component local state. Containers, otherwise known as Stores in Revault, live on, only accessed differently on render.

Statty's approach to access is great - by using a `select` prop to pluck only the pieces of state you want, it's easy to inject derived state as a render prop, while also making it easy to check for referential equality to prevent unecessary renders. Statty was only missing a dedicated logic unit.

Thus, Revault was born - aiming to assume the best traits of Unstated and Statty. Hope ya like it 👍

## Usage

### Begin by creating your first store, which extends from our base `Store`
```js
import { Store } from 'revault';

export default class TodoStore extends Store {
  state = {
    input: '',
    entries: [],
  };

  updateInput = input => {
    this.setState({ input });
  }

  addTodo = todo => {
    // setState has the same functionality as Component.setState,
    // except that it's synchronous
    this.setState({
      entries: [...this.state.entries, todo],
    });
  }
}
```


### Sweet. Next, wrap your application with the `Provider` and pass it your stores.
```js
import { render } from 'react-dom';
import { Provider as VaultProvider } from 'revault';
import * as stores from './stores';

/*
  `stores` may look something like the following. The key's will be used to as the store identifier within the vault.
  {
    todos: TodoStore,
    ...etc
  }
*/

const App = () => (
  <VaultProvider vault={vault}>
    <Entry />
  </VaultProvider>
);

render(<App />, window.root);
```


### And finally, drum roll please 🥁, import the `Connect` component to access our vault on render:
```js
import { Connect } from 'revault';

export default () => (
  <Connect
    select={(stores) => ({
      todos: stores.todos.state.entries,
      input: stores.todos.state.input,
      updateInput: stores.todos.updateInput,
      addTodo: stores.todos.addTodo,
    })}
  >
    {({ todos, input, updateInput, addTodo }) => (
      <>
        <ul>
          {todo.map(todo => (
            <li>{todo}</li>
          ))}
        </ul>

        <form onSubmit={addTodo}>
          <input value={input} onChange={updateInput} />
          <button type="submit" >
            Submit
          </button>
        </form>
      <>
    )}
  </Connect>
)
```

You've done it! You have your first todo app up and running 3 simple steps.


## Debugging
Inspired by [unstated-debug](https://github.com/sindresorhus/unstated-debug), Vault also comes with a debugging module.

All you need to do is import the debug file, which will monkey patch both the Vault and Store classes, as well as add the vault instance to the window as `window.VAULT`:
```js
import 'revault/debug';
```
