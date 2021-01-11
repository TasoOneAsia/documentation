---
id: style-guide
title: Style Guide
sidebar_label: Style Guide
slug: /style-guide
---

AgileTs isn't bound to any specific Style-Guide. 
But of course there are some you can get inspiration from.

## 🚀 Suggestion

To give you an idea, I will show you the style guide which worked well for me. 
Feel free to tweak it to your needs

In this Style-Guide we have a so called `core` at the top level of our `src` folder.
This core is thought to be the brain of our application and should contain all 
business logic, that isn't bound to a specific Component. 
It holds and manages our _Routes_, _States_, _Collections_, ..
The outsourcing of the Logic makes our code more decoupled, portable, and above all, easily testable.

```js {3} title="MyApp"
my-app
├── src
│   └── core
│   └── render
.
```

To visually demonstrate the structure of a `core`, 
I show you the core from a simple [TODO-List Application](TODO).

```js title="TodoList-Core"
core
├── src
│── api
│   ├── index.ts
│── controllers
│  └── todo
│       ├── index.ts
│       └── todo.actions.ts
|       └── todo.controller.ts
|       └── todo.interface.ts
|       └── todo.routes.ts
│  └── user
│       ├── index.ts
│       └── user.actions.ts
|       └── user.controller.ts
|       └── user.interface.ts
|       └── user.routes.ts
.
```

A basic Todo-List should have two Instances that should get handled by a State-Management Framework.
The **User**'s and of course the **TODO**'s.

## 📁 api

### 📝 index.ts

Our Todo-List needs to communicate to a **Backend**, so we need a package that makes this possible.
I am using the [AgileTs API](../packages/api/introduction.md) but of course you can use another package.
Well in the `api folder` we initialize the API-Package. So that we can use it properly in the rest of the `core`.

```ts title="index.ts"
import { API } from "@agile-ts/api";

const API = API({
    baseURL: 'http://localhost:5000',
    timeout: 10000,
    options: {
        credentials: undefined
    }
});

export default API;
```

## 📁 controllers

### 📝 index.ts

```ts
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

```ts
import {ITodo} from './todo.interface';
import {ADD_TODO} from './todo.routes';

export const addTodo = async (userId: string, description: string): Promise<IError | null> => {

    // Api
    const response = await ADD_TODO({description: description, userId: userId});
    if ('error' in response) return response;
    const todo = response;

    // Core
    core.todos.collection.todos.collect(todo, userId);

    return null;
};

// ..

```

### 📝 .controller.ts

```ts
import {App} from '../../app';
import {ITodo} from './todo.interface';

// Collectiont that holds all _TODO'S
export const TODOS_COLLECTION = App.Collection<ITodo>()();

// Computed that only olds the _TODO's of the currently logged in User
export const USER_TODOS = App.Computed(() => {
    return TodosCollection.getGroup(user.state.value.id).output;
});

```

### 📝 .interface.ts

```ts
export interface ITodo {
	id: string
	userId: string
	description: string
	creationDate: string
}

```

### 📝 .routes.ts

```ts
import {ITodo} from "./todo.interface";
import API from "../../api";

interface IAddTodoPayload {
    description: string,
    userId: string
}

type TAddTodoResponse = ITodo | IError;

export const ADD_TODO = async (payload: IAddTodoPayload): Promise<ITodo> => {
    const response = await api.post('todos', payload);
    return response.data.body.todo;
}
```

## 📝 app.ts

```ts
import {Agile} from "@agile-ts/core";
import reactIntegration from "@agile-ts/react";

export const App = new Agile({ logJobs: true }).use(reactIntegration);
```

## 📝 index.ts

```ts
import todo from "./controllers/todo";
import user from "./controllers/user";
import {globalBind} from "@agile-ts/core";

const core = {
  todo: todo,
  user: user,
};

// If you want to see your core in the console (Don't do that in PRODUCTION!!)
globalBind("__core__", core);

export default core;
```



