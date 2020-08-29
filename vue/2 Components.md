# Components

- Named `like-this` or `LikeThis`
- Processing languages can be specified in tags
- Other components can be imported

```js
import Tweet from "/some/path/Tweet.vue";
```

## Structure

- One root element, like `<div>` or `<nav>`

```
<template lang="html">
  <!-- good practice to add class to root element -->
  <nav class="nav-bar">
    <h1>3Squared</h1>
  </nav>
</template>
```

## Data Properties

`data()`

- Stores internal data used by the component
- Returns a new object for each component

`methods: { }`

- Stores internally-used methods

`computed`

- Stores functions accessed like data members in the DOM
- E.g., `fullName() {}` would be used as `{{ fullName }}`

`watched`

- Methods automatically called when properties are modified
- Use the same name as the data property

## Lifecycle hooks

- Methods executed at a particular stage of Vue's lifecycle
- Full list in [Vue docs](https://vuejs.org/v2/guide/instance.html#Instance-Lifecycle-Hooks)

```js
export default {
  data() {
    return {
      animal: {
        species: "Elephant",
        firstName: "Dave",
        lastName: "The Elephant",
        color: "grey",
        weight: "a lot",
      }
    };
  },

  methods: {
    doSomething() {}
  },

  computed: {
    fullName() {
      return `${this.firstName} ${this.lastName}`
    }
  },

  watch: {
    firstName(val, oldVal) {...}
  }

  // executed at creation phase of cycle
  created() {...}
}
```

## Props

- Props specify external data to be passed in

```js
props: ["id", "values"],

// more specific with type and default
props: {
  id: Number,
  values: [],
  premium: {
    type: Boolean,
    default: false,
  }
}
```

## Slots

- Slots create locations to insert external content
- Enable HTML/component injection, instead to plain data using props

```html
<!-- specification -->
<template>
  <div class="blog-post">
    <slot name="title"></slot>
    <slot name="subtitle"></slot>

    <!-- slot without name is default  -->
    <slot></slot>

    <!-- default content -->
    <slot name="signature">Harper Lee</slot>
  </div>
</template>

<!-- usage -->
<blog-post>
  <h1 slot="header">A Blog Post.</h1>
  <!-- no subtitle, element will not be used -->
  <p>Default slot will be used here</p>
  <p slot="signature">Overwrite default signature :)</p>
</blog-post>
```

## Styling

- Can be scoped to the file/component: `<style scoped>`
- Component named **cannot** be styled, e.g., `NavBar`, use the HTML class name instead

## Mixins

- Share functionality between components to reduce code reuse (similar concept to inheritance)
- Mixins are loaded before the component (super class)
- Component declarations overwrite the mixins (override)

```js
// requirements for custom form fields
export const formFieldMixin = {
  props: {
    label: String,
  },
};

// CustomSelect.vue
import { formFieldMixin } from "some/path/";
export default {
  // use the mixin
  mixins: [formFieldMixin],

  // additional props for the specific component
  props: {
    options: {
      type: Array,
      required: true,
    },
  },
};
```

## Filters

- Methods usable in expressions
- Chain-able

```js
export default {
  data() {
    return {
      url: "http://google.co.uk",
      comment: "this is shit",
    };
  },

  filters: {
    shout(str) {
      return str.toUpperCase();
    },

    append(str, extra) {
      return str + extra;
    },
  },
};
```

```html
// usage in templates
<p>{{ comment | shout }}</p>

// in directives, chained
<a :href="url | shout | extra('/search')">Google</a>
```
