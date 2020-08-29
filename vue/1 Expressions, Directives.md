# Expressions

- Access data inside HTML using `{{ }}`
- Uses standard Javascript syntax

```html
<p>{{ user.dob }}</p>
<p>{{ user.username.toUpperCase() }}</p>
```

# Directives

- Attributes identified with `v-` prefix

* `if, else-if, else`: added and removed from DOM based on conditions
* `show`: CSS 'display' property toggled
* `model`: two-way data binding for form inputs
* `for`: repeats the element

```html
<p v-if="condition">show this text</p>
<p v-else-if="condition">show this text</p>
<p v-else>show this text</p>

<img v-show="user.isPremium" src="path/to/premium-icon.png" />

<!-- will change if user types, or value is change in code -->
<input v-model="firstName" />

<!-- key recommended, /sometimes required -->
<li v-for="tweet in tweets" :key="tweet.id">{{ tweet.body }}</li>
```
