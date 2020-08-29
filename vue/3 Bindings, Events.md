# Binding

- Links attributes with live data
- Uses `v-bind:` prefix, or `:` shorthand

```html
<!-- only enabled if item in stock -->
<button id="#cartButton" v-bind:disabled="item.inStock">Add to cart</button>

<!-- eg: set code example theme to user preference -->
<code-sample :theme="user.preferredCodeTheme">...</code-sample>
```

# Events

- Do stuff on user input
- Uses `v-on` prefix or `@` shorthand
- Arguments can be specified

```html
<button v-on:click="addToCart"></button>
<!-- with args -->
<button v-on:click="addToCart(prodID)"></button>
```

- Modifiers add additional behaviour
- Can be keyboard, mouse, page behaviour, etc

```html
<!-- prevent default -->
<form @submit.prevent="addReview(user.ID, rating)">
  ...

  <!-- on ctrl+c -->
  <input @keyup.ctrl.c="onCopy" />
</form>
```

- Custom events can be passed up using props

```html
<!-- Child.vue -->
<template>
  <button @click="onClick">Go</button>
</template>
```

```js
/* Child.vue */
export default {
  methods: {
    onClick() {
      this.$emit("button-clicked", "someValue");
    },
  },
};
```

```html
<!-- Parent.vue -->
<Child @button-clicked="doJsStuff" />
```
