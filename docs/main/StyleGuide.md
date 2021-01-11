---
id: style-guide 
title: Style Guide 
sidebar_label: Style Guide 
slug: /style-guide
---

:::note

**AgileTs isn't bound to any specific Style-Guide** but there are some you may get inspiration from.


:::

## 🚀 Suggestion

To give you an idea of a possible organisation of Agile Instances, 
I will show you the style guide which worked well for me. Feel free to tweak it to your needs

In this Style-Guide we have a so called `core` at the top level of our `src` folder beside our UI-Components. 
This so called `core` is thought to be the brain of our application and should contain all business logic
and logic in general that isn't specifically bound to a Component like Animation Calculations, ..
This outsourcing of our Logic makes the code more decoupled,
portable, and above all, easily testable.

Below you can see where our `core` might be located.

```js {3} title="MyApp"
my - app
├── src
│   └── core
│   └── render
    .
```

To represent the `core` visually, I use one of a simple **TODO-Application**.
Which has two [Entities](#📁-entities) that are handled by AgileTs.
The **User** and of course the **TODO**. These 2 main parts are mapped in our `core`.

```js title="TodoList-Core"
core
├── src
│── api
│   ├── index.ts
│── entities
│  └── todo
│  |    ├── index.ts
│  |    └── todo.actions.ts
|  |    └── todo.controller.ts
|  |    └── todo.interface.ts
|  |    └── todo.routes.ts
│  └── user
│       ├── index.ts
│       └── user.actions.ts
|       └── user.controller.ts
|       └── user.interface.ts
|       └── user.routes.ts
|── app.ts
|── index.ts
    .
```

Below you can find a similar structure to the Todo-List `core`
where I explain each part in detail. 

:::info

On the right side you can see a structure that should represent the structure above.

:::

## 📁 api

Our Todo-List has to communicate to a **Backend**, that's why we need something that handles http/s requests for us. I
am using the [AgileTs API](../packages/api/Introduction.md) but of course you can use whatever you want. 

### 📝 index.ts

To make simple Rest-Calls possible, we initialize our API-Class here.
The defined API-Instance gets mainly used by the [Routes](#📝-.routes.ts) of an Entity.

```ts title="index.ts"
import {API} from "@agile-ts/api";

const API = API({
    baseURL: 'http://localhost:5000',
    timeout: 10000,
    options: {
        credentials: undefined
    }
});

export default API;
```

## 📁 entities

Our core consists of several entities, which exists apart from each other, having its own independent existence. Each
Entity manages its Data separately by doing rest calls or mutating States. This separation makes our `core` more
structured, readable and improves the maintainability.

**For instance:** <br />
A _User Entity_ should only treat the whole logic of the User and shouldn't do rest calls for the _Todo Entity_.

### 📝 index.ts

Here we just export all [actions](#📝-.action.ts), [routes](#📝-.routes.ts), [interfaces](#📝-.interface.ts) and
the [controller](#📝-.controller.ts). To properly import them in a UI-Components later.

```ts title="index.ts in 📁todo"
import * as actions from "./todo.actions";
import * as controller from "./todo.controller";
import * as routes from "./todo.routes";
import * as interfaces from "./todo.interface";

export default {
    ...actions,
    ...controller,
    ...routes,
    ...interfaces,
};
```

### 📝 .action.ts

An action is any piece of code that modifies our `Agile Instances`(State, Collection, ..).
In principle, actions always happen in response to an event. For example, a button got clicked.

**For instance:** <br />
The creation of an Event in the UI-Layer, triggers the `addTodo` action which than mutates our State.

```ts title="todo.action.ts in 📁todo"
import {TodoInterface} from './todo.interface';
import {ADD_TODO} from './todo.routes';
import {TODOS} from './todo.controller';

export const addTodo = async (userId: string, description: string): Promise<void> => {
    // Rest Call
    const response = await ADD_TODO({description: description, userId: userId});

    // Add Todo to Collection
    TODOS.collect(todo, userId);
};

// ..

```

### 📝 .controller.ts

Each Entity should have a Controller, that initialises and holds Agile Instances, like States, Collections, for it.
These Agile Instances can than get modified by [Actions](#📝-.action.ts) or bound to a Component in the UI-Layer.

```ts title="todo.controller.ts in 📁todo"
import {App} from '../../app';
import {TodoInterface} from './todo.interface';
import {CURRENT_USER} from '../user'

// Holds all existing TODO's
export const TODOS = App.Collection<TodoInterface>()();

// Holds all TODO's that belong to the current logged in USER
export const USER_TODOS = App.Computed(() => {
    return TodosCollection.getGroup(CURRENT_USER.value.id).output;
});
```

### 📝 .interface.ts

The Interface-File holds the main Interfaces that represent our Entity.
**For instance** The .interface in the TODO-Entity should contain a TodoInterface.

:::info

Not necessary if you aren't using [Typescript](https://www.typescriptlang.org/).

:::

```ts title="todo.interface.ts in 📁todo"
export interface TodoInterface {
    id: string
    userId: string
    description: string
    creationDate: string
}

```

### 📝 .routes.ts

Is like the interface to the outside world. 
Here Rest-Calls in Function Form live, to easily call them in the [Actions](#📝-.action.ts).
It's not recommended calling them inside the UI-Layer.

```ts title="todo.routes.ts in 📁todo"
import {TodoInterface} from "./todo.interface";
import API from "../../api";

interface AddTodoPayloadInterface {
    description: string,
    userId: string
}

export const ADD_TODO = async (payload: AddTodoPayloadInterface): Promise<TodoInterface> => {
    const response = await api.post('todos', payload);
    return response.data.body.todo;
}

// ..

```

## 📝 app.ts

Is like the brain of our `core`. 
It contains the main Agile Instance from which other Agile Instances(States, Collection) emerge.

```ts title="app.ts"
import {Agile} from "@agile-ts/core";
import reactIntegration from "@agile-ts/react";

export const App = new Agile({logJobs: true}).use(reactIntegration);
```

## 📝 index.ts

Here we just export our `core`, to import it in our Application UI-Layer.

```ts title="index.ts"
import todo from "./controllers/todo";
import user from "./controllers/user";
import {globalBind} from "@agile-ts/core";

const core = {
    todo: todo,
    user: user,
};

// If you want to have your core global (Don't do that in PRODUCTION!!)
globalBind("__core__", core);

export default core;
```



