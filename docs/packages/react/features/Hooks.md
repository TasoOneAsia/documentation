---
id: hooks
title: React Hooks
sidebar_label: React Hooks
slug: /react/hooks
---

:::warning

Be aware that [React Hooks](https://reactjs.org/docs/hooks-intro.html) are only supported in Functional Components!

:::


## `useAgile`

The `useAgile` React Hook, helps us to bind States to our React Component.
These binding ensures that the Component rerender, whenever the bound State mutates.
We can flexibly bind any State to our Component. 
It doesn't matter which State and how many States.
```ts
  const myCoolState = useAgile(MY_COOL_STATE); 
```
`useAgile` returns the current _output_ of the passed State.

```ts
  const [myCoolState1, myCoolStat2] = useAgile([MY_COOL_STATE1, MY_COOL_STATE2]);
```
It is also possible to bind more than one State to our component at once. 
This multiple binding has one advantage. It can lower the rerender count of our component,
because it allows AgileTs to determine whether we can combine two rerender triggered by different States and at same moment.
In this case `useAgile` returns the _output_ of the passed States, in the same order 
as they were entered.

```ts
  const [myCollection, myGroup] = useAgile([MY_COLLECTION, MY_GROUP]);
```
We are not limited to States, we can bind all Agile Instances that own
an `observer`.
These include:
- State
- Group
- Computed
- Item  
- Collection (_exception_ since it has no `observer`)

### 🔴 Example

```tsx live
  const App = new Agile();
  const MY_STATE = App.State("Hello Stranger!");
  
  const RandomComponent = () => {
      const myFirstState = useAgile(MY_STATE); // Returns "Hello Stranger!"
   
      return (
          <div>                                              
              <p>{myFirstState}</p>                          
              <button                                       
                  onClick={() => {
                      MY_STATE.set("Hello Friend!"); 
                  }}
              >
                  Update State
              </button>
          </div>
      );
  }
  
  render(<RandomComponent/>);
```

### 🟦 Typescript

`useAgile` is almost 100% typesafe.
There are a few side cases you probably won't run into.

### 📭 Props

| Prop              | Type                                            | Functionality                                                                | Required    | 
| ----------------- | ----------------------------------------------- | ---------------------------------------------------------------------------- | ------------|
| `dep`             | State \| Collection \| Observer \| undefined    | Agile Instances that get bound to the Component the useAgile Hook is in      | Yes         | 
| `key`             | string \| number                                | Key/Name of created Observer. Mainly thought for Debugging                   | No          | 
| `agileInstance`   | Agile                                           | To which main Agile Instance the State get bound. Gets autodetect!           | No          |

### 📄 Return

`useAgile` returns the current `output` of the passed Agile Instance/s.

```ts {6,9}
const MY_STATE = App.State(1);
const MY_STATE_2 = App.State(2);
const MY_STATE_3 = App.State(3);

// One passed Agile Instance
useAgile(MY_STATE); // Returns 3

// Multiple passed Agile Instances
useAgile([MY_STATE, MY_STATE_2, MY_STATE_3]); // Returns [1, 2, 3]
```

<br />

---

<br />

## `useEvent`

The `useEvent` React Hook, allows us to register a new callback function to the passed Event.
```ts
  useEvent(MY_EVENT, () => {
      // This is a 'callback function' which gets called when ever the EVENT gets triggered
  })
```
The advantage of using this Hook instead of the `on` function in a React Component, 
is that it automatically unregisters the callback function when the Component unmounts.

### 🔴 Example

```tsx live
const App = new Agile();
const MY_EVENT = App.Event();

const RandomComponent = () => {
    useEvent(MY_EVENT, () => {
        toast("You successfully triggered an Event!");
    })

    return (
        <div>
            <button
                onClick={() => {
                    MY_EVENT.trigger();
                }}
            >
                Trigger Event
            </button>
        </div>
    );
}

render(<RandomComponent/>);
```

### 🟦 Typescript

`useEvent` is almost 100% typesafe.

### 📭 Props

| Prop              | Type                                            | Functionality                                                                | Required    | 
| ----------------- | ----------------------------------------------- | ---------------------------------------------------------------------------- | ------------|
| `event`           | Event (E)                                       | Event to which the passed callback function gets applied                     | Yes         | 
| `callback`        | EventCallbackFunction<E['payload']>             | Callback Function which gets called whenever the event gets triggered        | Yes         | 
| `key`             | string \| number                                | Key/Name of created Observer. Mainly thought for Debugging                   | No          | 
| `agileInstance`   | Agile                                           | To which main Agile Instance the Event get bound. Gets autodetect!           | No          |

### 📄 Return

`useEvent` returns nothing.

<br />

---

<br />

## `useWatcher`

With the `useWatcher` React Hook we are able to create a callback function that gets called whenever
the passed State mutates.
```ts
  useWatcher(MY_STATE, () => {
    // This is a 'callback function' which gets called whenever MY_STATE mutates
});
```

### 🔴 Example

```tsx live
const App = new Agile();
const MY_STATE = App.State("hello");

const RandomComponent = () => {
    useWatcher(MY_STATE, (value) => {
        toast("New Value: " + value);
    });

    return (
        <div>
            <button
                onClick={() => {
                    MY_STATE.set("bye");
                }}
            >
                Change State
            </button>
        </div>
    );
}

render(<RandomComponent/>);
```

### 🟦 Typescript

`useWatcher` is almost 100% typesafe.

### 📭 Props

| Prop              | Type                                            | Functionality                                                                | Required    | 
| ----------------- | ----------------------------------------------- | ---------------------------------------------------------------------------- | ------------|
| `state`           | State                                           | State to which the passed watcher callback gets applied                      | Yes         | 
| `agileInstance`   | Agile                                           | To which main Agile Instance the Event get bound. Gets autodetect!           | No          |

### 📄 Return

`useWatcher` returns nothing.

