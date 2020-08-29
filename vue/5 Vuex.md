# Vuex (State Management)

- Contained in `Vuex.Store` instance
- Global `$store` object, with 4 properties:

**State** stores the data, like `data` in Components

```js
state: {
  todos: [];
}
```

**Getters** stores data getters, like `computed`

- Accessed like properties, not methods
- Receives `state` object as first parameter
- Revives `getters` object as second parameter
- Getters with arguments are declared by returning functions

```js
// declaration
getters: {
  // parameters
  getTodoCount: (state, getters) => {
    return state.todos.length;
  };

  // getter with parameter
  getTodoById: (state) => (id) => {
    return state.todos.find((todo) => todo.id === id);
  };
}

// usage
store.getters.getTodoCount; // without params
store.getters.getTodoById(2); // with
```

**Mutations** stores methods used to change state

- Cannot be directly called, executed with `store.commit` method
- Only way to modify state, allowing mutations to be rolled forwards/back with debugging tools
- Thus **must be synchronous**
- Accepts additional values can be added as parameters or as a `payload` object

```js
state: {
  count: 1
},

mutations: {
  // with argument
  increment(state, amount) {
    state += amount;
  }

  // with payload
  increment(state, payload) {
    state += payload.amount;
  }
}

// called with
store.commit('increment' 12);
store.commit('increment' { amount: 13 });
```

**Actions** stores methods

- Can be used to commit mutations
- Don't need to synchronous like mutations
- Returns `Promise` to execute multiple actions whist maintaining synchronisation
- Called using `store.dispatch()`
- Receives `context` object containing lots of `$store` [data](https://vuex.vuejs.org/api/#actions)
- Accepts `payload` object like mutations

```js
actions: {
  // full context object
  increment (context) {
    // calls mutation
    context.commit("increment", 10);
  },

  // destructuring
  increment ({commit}) {
    commit("increment", 7);
  }

  // payload passing
  increment({commit}, payload) {
    commit("increment", payload);
  }
}

// called with
store.dispatch("increment");
store.dispatch("increment" { amount: 15 });
```

## Usage

- Mutations track changes to state data, and can be rolled-back to see their effect
- Design pattern is actions are call mutations to change state

```js
const store = new Vuex.Store({
  // external data
  modules: {
    user,
  },

  state: {
    apiLoading = false,
    data = [],
  },

  mutations: {
    // accepts state object and status param
    SET_API_LOADING(state, status) {
      state.apiLoading = status;
    },

    SET_DATA(state, data) {
      state.data = data;
    },
  },

  actions: {
    // accepts Store instance
    fetchData(context) {
      // mutate the state
      context.commit("SET_API_LOADING", true);

      axios.get("/api/data")
        .then(res => {
          context.commit("SET_DATA", res.data);
          context.commit("SET_API_LOADING", false);
        });
    }
  }
});
```

## Mapping

- `mapX` functions provided by Vuex localise properties in components

```js
<template>
  <!-- default access or localised from mapState function -->
  <p>{{$store.state.description}} or {{ desc }}</p>
</template>

<script>
import { mapState, mapGetters } from "vuex";

export default {
  computed: mapState({
    // using arrow functions
    desc: state => state.description,

    // string value equates to `state => state.X`
    desc: 'description',

    // access local state in function using 'this'
    desc(state) {
      return state.desc + data.something
    }
  })

  // use spread operator to use in conjunction
  // with local computed properties
  ...mapState({...})
}
</script>
```

## Modules

- Modules prevent files from becoming to large
- Objects can be extracted as a whole, or each property from an object (state, actions) can be stored as constants

```js
// isolated as whole object
export default {
  state: {...},
  mutations: {...},
  actions: {...},
  getters: {...},
}

// imported with
import user from "user.js"

// isolated as individual properties
export const state {...}
export const mutations {...}

// imported with
import * as user from "user.js"
```
