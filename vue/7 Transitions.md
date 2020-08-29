# Transitions

- Implemented with the `transition` component, which wraps an element
- Triggered by changes to elements in following contexts:

  - Conditional rendering (using `v-if`)
  - Conditional display (using `v-show`)
  - Dynamic components
  - Component root nodes

## Styling Syntax

- `v-enter` and `v-leave` phases allow transitions to apply transitions in both directions
- Non-postfixed and `-to` suffix allow for start and end styles respectively
- Additional `-active` class applies to the whole transition duration, used to declare `transition` property

* `v-enter-to` and `v-leave` can usually be omitted, as transitions usually apply to an element's default style

## Naming

- Can be named with `name` attribute
- Names replace the 'v' in the `v-` prefix with the transition name
- Hence transitions should be named after their effect, e.g., 'fade'

## Options

- `:duration` prop can specify durations

```html
<transition :duration="1000">...</transition>
<transition :duration="{ enter: 500, leave: 800 }">...</transition>
```

- `appear` attribute can trigger the transitions on render
- Custom classes can specified

## Example

```css
/* fade effect */

.fade-enter {
  /* start transparent */
  opacity: 0;
}

.fade-enter-active {
  transition: opacity 0.5s ease-in;
}

.fade-enter-to {
  /* -to not required */
  /* opacity: 1 */
}

.fade-leave {
  /* not required */
  /* opacity: 0; */
}

.fade-leave-active {
  /* different transition on leave */
  transition: opacity 0.2s ease-out;
}

.fade-enter-to {
  /* finish transparent */
  opacity: 0;
}
```

Fade transition applied to component:

```html
<!-- applied to component -->

<button @click="show = !show">
  Toggle render
</button>

<!-- named transition -->
<transition name="fade">
  <!-- triggered by v-if -->
  <p v-if="show">hello</p>
</transition>
```
