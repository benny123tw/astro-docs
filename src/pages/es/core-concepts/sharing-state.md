---
layout: ~/layouts/MainLayout.astro
title: Compartiendo Estado
i18nReady: true
setup: |
  import Tabs from '../../../components/tabs/Tabs'
  import UIFrameworkTabs from '~/components/tabs/UIFrameworkTabs.astro'
  import LoopingVideo from '~/components/LoopingVideo.astro'
---

Al construir tu proyecto usando la [arquitectura de islas / hidratación parcial](/es/concepts/islands/), puede que te hayas topado con este problema: **Quiero compartir estado entre mis componentes.**

Frameworks tales como React o Vue pueden alentar a usar [proveedores de "contexto"](https://reactjs.org/docs/context.html) ("context" providers) para que sea consumido por otros componentes. Pero al [hidratar componentes parcialmente](/es/core-concepts/framework-components/#hidratando-componentes-interactivos) dentro de Astro o en Markdown, no puedes usar esos contextos envolventes.

Astro recomienda una solución diferente para el almacenamiento compartido en el lado del cliente: [**Nano Stores**](https://github.com/nanostores/nanostores).

## ¿Por qué Nano Stores?

La librería [Nano Stores](https://github.com/nanostores/nanostores) te permite crear _stores_ con las que cualquier componente puede interactuar. Recomendamos Nano Stores porque:
- **Son livianas.** Nano Stores tiene la cantidad mínima de JS que puedas llegar a requerir (menos de 1 KB) con cero dependencias.
- **Son framework-agnósticas.** ¡Esto significa que puedes compartir estado entre _frameworks_ sin contratiempos! Astro está construido para ser flexible, así que amamos las soluciones que ofrecen una experiencia de desarrollo similar, sin importar tu preferencia.

Aun así, hay otras alternativas a explorar. Entre ellas puedes encontrar:
- [Svelte stores incorporadas](https://svelte.dev/tutorial/writable-stores)
- [Solid signals](https://www.solidjs.com/docs/latest) fuera de un elemento de contexto
- [Enviar eventos personalizados del navegador](https://developer.mozilla.org/es/docs/Web/Events/Creating_and_triggering_events) entre componentes

:::note[FAQ]

<details>
<summary>**🙋 ¿Puedo usar Nano Stores en archivos `.astro` u otros archivos del lado del servidor?**</summary>

Las Nano Stores _pueden_ ser importadas, escritas y leídas desde componentes del lado del servidor, **¡aunque no lo recomendamos!**. Esto se debe a ciertas restricciones:
- Escribir en una store desde un archivo `.astro` o un [componente no-hidratado](/es/core-concepts/framework-components/#hidratando-componentes-interactivos) _no_ afectará el valor recibido por un [componente del lado del cliente](/es/reference/directives-reference/#directivas-del-cliente).
- No puedes pasar una Nano Store como "prop" a componentes del lado del cliente.
- No puedes suscribirte a cambios en la store desde un archivo `.astro`, ya que los componentes de Astro no se re-renderizan.

¡Si entiendes estas restricciones y aun así encuentras un caso de uso, puedes darle una oportunidad a Nano Stores! Solamente recuerda que Nano Stores fue creado específicamente para reaccionar a cambios en el **cliente**.

</details>

<details>
<summary>**🙋 ¿Cómo se comparan las Svelte stores a Nano Stores?**</summary>

**¡Nano Stores y [Svelte stores](https://svelte.dev/tutorial/writable-stores) son muy similares!** De hecho, [nanostores te permite usar el mismo atajo `$`](https://github.com/nanostores/nanostores#svelte) para suscripciones que puedes utilizar con las Svelte stores.

Si quieres evitar usar una librería de terceros, [Svelte stores](https://svelte.dev/tutorial/writable-stores) es una gran herramienta para la comunicación entre islas. Aun así, puedes llegar a preferir Nano Stores si a) te gustaría añadir add-ons para ["objetos"](https://github.com/nanostores/nanostores#maps) y [estado asíncrono](https://github.com/nanostores/nanostores#lazy-stores), o b) quieres comunicarte entre Svelte y otros frameworks como Preact o Vue.
</details>

<details>
<summary>**🙋 ¿Cómo se comparan las Solid signals a Nano Stores?**</summary>

Si has usado Solid anteriormente, habrás intentado mover [signals](https://www.solidjs.com/docs/latest#createsignal) o [stores](https://www.solidjs.com/docs/latest#createstore) fuera de tus componentes. ¡Esta es una muy buena manera de compartir estado entre islas de Solid! Intenta exportar signals desde un archivo compartido:

```js
// sharedStore.js
import { createSignal } from 'solid-js';

export const sharedCount = createSignal(0);
```
...y todos los componentes que importen `sharedCount` compartirán el mismo estado. Aunque esto funciones bien, puedes llegar a preferir Nano Stores si a) te gustaría añadir add-ons para ["objetos"](https://github.com/nanostores/nanostores#maps) y [estado asíncrono](https://github.com/nanostores/nanostores#lazy-stores), o b) quieres comunicarte entre Solid y otros frameworks como Preact or Vue.
</details>
:::

## Instalando Nano Stores

Para empezar, instala Nano Stores junto al paquete helper para tu framework favorito:

<UIFrameworkTabs>
  <Fragment slot="preact">
  ```shell
  npm i nanostores @nanostores/preact
  ```
  </Fragment>
  <Fragment slot="react">
  ```shell
  npm i nanostores @nanostores/react
  ```
  </Fragment>
  <Fragment slot="solid">
  ```shell
  npm i nanostores @nanostores/solid
  ```
  </Fragment>
  <Fragment slot="svelte">
  ```shell
  npm i nanostores
  ```
  :::note
  ¡No se necesita paquete helper aquí! Nano Stores puede ser usado como una Svelte store estándar.
  :::
  </Fragment>
  <Fragment slot="vue">
  ```shell
  npm i nanostores @nanostores/vue
  ```
  </Fragment>
</UIFrameworkTabs>

¡Desde aquí, puedes saltar directamente a la [guía de uso de Nano Stores](https://github.com/nanostores/nanostores#guide), o seguir nuestra guía con ejemplos!

## Ejemplo de uso - menú desplegable con carrito de ecommerce

Digamos que queremos construir una interfaz de ecommerce simple con tres elementos interactivos:
- Un formulario para "agregar al carrito"
- Un menú desplegable con carrito para mostrar esos ítems agregados
- Un botón para desplegar el menú con carrito

<LoopingVideo sources={[{ src: '/videos/stores-example.mp4', type: 'video/mp4' }]} />

_[**Prueba el ejemplo terminado**](https://github.com/withastro/astro/tree/main/examples/with-nanostores) en tu máquina u online vía Stackblitz._

Tu archivo base de Astro podría verse así:

```astro
---
// Example: src/pages/index.astro
import CartFlyoutToggle from '../components/CartFlyoutToggle';
import CartFlyout from '../components/CartFlyout';
import AddToCartForm from '../components/AddToCartForm';
---

<!DOCTYPE html>
<html lang="en">
<head>...</head>
<body>
  <header>
    <nav>
      <a href="/">Tienda de Astro</a>
      <CartFlyoutToggle client:load />
    </nav>
  </header>
  <main>
    <AddToCartForm client:load>
    <!-- ... -->
    </AddToCartForm>
  </main>
  <CartFlyout client:load />
</body>
</html>
```

### Usando "atoms"

Empecemos por abrir nuestro `CartFlyout` cada vez que cliqueamos en `CartFlyoutToggle`. 

Primero, crea un nuevo archivo JS o TS para nuestra store. Usaremos un ["atom"](https://github.com/nanostores/nanostores#atoms) para esto:

```js
// src/cartStore.js
import { atom } from 'nanostores';

export const isCartOpen = atom(false);
```

Ahora, podemos importar esta store dentro de cualquier archivo que necesite leer o escribir en ella. Comenzaremos conectando nuestro `CartFlyoutToggle`:

<UIFrameworkTabs>
<Fragment slot="preact">
```jsx
// src/components/CartFlyoutToggle.jsx
import { useStore } from '@nanostores/preact';
import { isCartOpen } from '../cartStore';

export default function CartButton() {
  // lee el valor de la store con el hook `useStore`
  const $isCartOpen = useStore(isCartOpen);
  // escribe en la store importada usando `.set`
  return (
    <button onClick={() => isCartOpen.set(!$isCartOpen)}>Cart</button>
  )
}
```
</Fragment>
<Fragment slot="react">
```jsx
// src/components/CartFlyoutToggle.jsx
import { useStore } from '@nanostores/react';
import { isCartOpen } from '../cartStore';

export default function CartButton() {
  // lee el valor de la store con el hook `useStore`
  const $isCartOpen = useStore(isCartOpen);
  // escribe en la store importada usando `.set`
  return (
    <button onClick={() => isCartOpen.set(!$isCartOpen)}>Cart</button>
  )
}
```
</Fragment>
<Fragment slot="solid">
```jsx
// src/components/CartFlyoutToggle.jsx
import { useStore } from '@nanostores/solid';
import { isCartOpen } from '../cartStore';

export default function CartButton() {
  // lee el valor de la store con el hook `useStore`
  const $isCartOpen = useStore(isCartOpen);
  // escribe en la store importada usando `.set`
  return (
    <button onClick={() => isCartOpen.set(!$isCartOpen)}>Cart</button>
  )
}
```
</Fragment>
<Fragment slot="svelte">
```svelte
<!--src/components/CartFlyoutToggle.svelte-->
<script>
  import { isCartOpen } from '../cartStore';
</script>

<!--usa "$" para leer el valor de la store-->
<button on:click={() => isCartOpen.set(!$isCartOpen)}>Cart</button>
```
</Fragment>
<Fragment slot="vue">
```vue
<!--src/components/CartFlyoutToggle.vue-->
<template>
  <!--escribe en la store importada usando `.set`-->
  <button @click="isCartOpen.set(!$isCartOpen)">Cart</button>
</template>

<script setup>
  import { isCartOpen } from '../cartStore';
  import { useStore } from '@nanostores/vue';
  
  // lee el valor de la store con el hook `useStore`
  const $isCartOpen = useStore(isCartOpen);
</script>
```
</Fragment>
</UIFrameworkTabs>

Luego, podemos leer el valor de `isCartOpen` en nuestro componente `CartFlyout`:

<UIFrameworkTabs>
<Fragment slot="preact">
```jsx
// src/components/CartFlyout.jsx
import { useStore } from '@nanostores/preact';
import { isCartOpen } from '../cartStore';

export default function CartFlyout() {
  const $isCartOpen = useStore(isCartOpen);

  return $isCartOpen ? <aside>...</aside> : null;
}
```
</Fragment>
<Fragment slot="react">
```jsx
// src/components/CartFlyout.jsx
import { useStore } from '@nanostores/react';
import { isCartOpen } from '../cartStore';

export default function CartFlyout() {
  const $isCartOpen = useStore(isCartOpen);

  return $isCartOpen ? <aside>...</aside> : null;
}
```
</Fragment>
<Fragment slot="solid">
```jsx
// src/components/CartFlyout.jsx
import { useStore } from '@nanostores/solid';
import { isCartOpen } from '../cartStore';

export default function CartFlyout() {
  const $isCartOpen = useStore(isCartOpen);

  return $isCartOpen ? <aside>...</aside> : null;
}
```
</Fragment>
<Fragment slot="svelte">
```svelte
<!--src/components/CartFlyout.svelte-->
<script>
  import { isCartOpen } from '../cartStore';
</script>

{#if $isCartOpen}
<aside>...</aside>
{/if}
```
</Fragment>
<Fragment slot="vue">
```vue
<!--src/components/CartFlyout.vue-->
<template>
  <aside v-if="$isCartOpen">...</aside>
</template>

<script setup>
  import { isCartOpen } from '../cartStore';
  import { useStore } from '@nanostores/vue';

  const $isCartOpen = useStore(isCartOpen);
</script>
```
</Fragment>
</UIFrameworkTabs>

### Usando "maps"

:::tip
**¡Los [Maps](https://github.com/nanostores/nanostores#maps) son una muy buena opción para objetos que son escritos regularmente!** Junto a los helpers `get()` y `set()` estándar que provee un `atom`, también tendrás una función `.setKey()` para actualizar keys individuales en objetos.
:::

Ahora, llevemos la cuenta de los ítems que hay dentro de tu carrito. Para evitar duplicados y llevar el registro de la "cantidad", puedes guardar tu carrito como un objeto con el ID del ítem como key. Usaremos un [Map](https://github.com/nanostores/nanostores#maps) para lograr esto.

Agreguemos una store `cartItem` a nuestro `cartStore.js` anterior. También puedes utilizar un archivo TypeScript si deseas definir el tipo de dato.


<Tabs client:visible sharedStore="js-ts">
<Fragment slot="tab.js">JavaScript</Fragment>
<Fragment slot="tab.ts">TypeScript</Fragment>
<Fragment slot="panel.js">
```js
// src/cartStore.js
import { atom, map } from 'nanostores';

export const isCartOpen = atom(false);

/**
 * @typedef {Object} CartItem
 * @property {string} id
 * @property {string} name
 * @property {string} imageSrc
 * @property {number} quantity
 */

/** @type {import('nanostores').MapStore<Record<string, CartItem>>} */
export const cartItems = map({});

```
</Fragment>
<Fragment slot="panel.ts">
```ts
// src/cartStore.ts
import { atom, map } from 'nanostores';

export const isCartOpen = atom(false);

export type CartItem = {
  id: string;
  name: string;
  imageSrc: string;
  quantity: number;
}

export const cartItems = map<Record<string, CartItem>>({});
```
</Fragment>
</Tabs>

Ahora, exportemos una función helper `addCartItem` para que usen nuestros componentes.
- **Si el ítem no existe en el carrito**, añade el carrito con una cantidad inicial de 1.
- **Si el ítem _ya existe_**, aumenta la cantidad en 1.

<Tabs client:visible sharedStore="js-ts">
<Fragment slot="tab.js">JavaScript</Fragment>
<Fragment slot="tab.ts">TypeScript</Fragment>
<Fragment slot="panel.js">
```js
// src/cartStore.js
...
export function addCartItem({ id, name, imageSrc }) {
  const existingEntry = cartItems.get()[id];
  if (existingEntry) {
    cartItems.setKey(id, {
      ...existingEntry,
      quantity: existingEntry.quantity + 1,
    })
  } else {
    cartItems.setKey(
      id,
      { id, name, imageSrc, quantity: 1 }
    );
  }
}
```
</Fragment>
<Fragment slot="panel.ts">
```ts
// src/cartStore.ts
...
type ItemDisplayInfo = Pick<CartItem, 'id' | 'name' | 'imageSrc'>;
export function addCartItem({ id, name, imageSrc }: ItemDisplayInfo) {
  const existingEntry = cartItems.get()[id];
  if (existingEntry) {
    cartItems.setKey(id, {
      ...existingEntry,
      quantity: existingEntry.quantity + 1,
    });
  } else {
    cartItems.setKey(
      id,
      { id, name, imageSrc, quantity: 1 }
    );
  }
}
```
</Fragment>
</Tabs>

:::note
<details>

<summary>**🙋 ¿Por qué usamos `.get()` aquí en vez de un helper `useStore`?**</summary>

Habrás notado que estamos llamando a `cartItems.get()` aquí, en vez de usar el helper `useStore` de nuestros ejemplos de React / Preact / Solid / Vue. Esto es porque **useStore genera re-renderizados.** En otras palabras, `useStore` debe usarse cada vez que el valor de la store se renderice en la UI. Como estamos leyendo este valor cuando un **evento** es accionado (`addToCart` en este caso), y no estamos intentando renderizar ese valor, en este caso no necesitamos `useStore`.
</details>
:::

Con la store en su lugar, ahora podemos llamar esta función dentro de `AddToCartForm` cada vez que el formulario es enviado. También desplegaremos el menú con carrito para poder ver un resumen de lo que hay dentro.

<UIFrameworkTabs>
<Fragment slot="preact">
```jsx
// src/components/AddToCartForm.jsx
import { addCartItem, isCartOpen } from '../cartStore';

export default function AddToCartForm({ children }) {
  // ¡usaremos valores fijos por simplicidad!
  const hardcodedItemInfo = {
    id: 'astronaut-figurine',
    name: 'Astronaut Figurine',
    imageSrc: '/images/astronaut-figurine.png',
  }

  function addToCart(e) {
    e.preventDefault();
    isCartOpen.set(true);
    addCartItem(hardcodedItemInfo);
  }

  return (
    <form onSubmit={addToCart}>
      {children}
    </form>
  )
}
```
</Fragment>
<Fragment slot="react">
```jsx
// src/components/AddToCartForm.jsx
import { addCartItem, isCartOpen } from '../cartStore';

export default function AddToCartForm({ children }) {
  // ¡usaremos valores fijos por simplicidad!
  const hardcodedItemInfo = {
    id: 'astronaut-figurine',
    name: 'Astronaut Figurine',
    imageSrc: '/images/astronaut-figurine.png',
  }

  function addToCart(e) {
    e.preventDefault();
    isCartOpen.set(true);
    addCartItem(hardcodedItemInfo);
  }

  return (
    <form onSubmit={addToCart}>
      {children}
    </form>
  )
}
```
</Fragment>
<Fragment slot="solid">
```jsx
// src/components/AddToCartForm.jsx
import { addCartItem, isCartOpen } from '../cartStore';

export default function AddToCartForm({ children }) {
  // ¡usaremos valores fijos por simplicidad!
  const hardcodedItemInfo = {
    id: 'astronaut-figurine',
    name: 'Astronaut Figurine',
    imageSrc: '/images/astronaut-figurine.png',
  }

  function addToCart(e) {
    e.preventDefault();
    isCartOpen.set(true);
    addCartItem(hardcodedItemInfo);
  }

  return (
    <form onSubmit={addToCart}>
      {children}
    </form>
  )
}
```
</Fragment>
<Fragment slot="svelte">
```svelte
<!--src/components/AddToCartForm.svelte-->
<form on:submit|preventDefault={addToCart}>
  <slot></slot>
</form>

<script>
  import { addCartItem, isCartOpen } from '../cartStore';

  // ¡usaremos valores fijos por simplicidad!
  const hardcodedItemInfo = {
    id: 'astronaut-figurine',
    name: 'Astronaut Figurine',
    imageSrc: '/images/astronaut-figurine.png',
  }

  function addToCart() {
    isCartOpen.set(true);
    addCartItem(hardcodedItemInfo);
  }
</script>
```
</Fragment>
<Fragment slot="vue">
```vue
<!--src/components/AddToCartForm.vue-->
<template>
  <form @submit="addToCart">
    <slot></slot>
  </form>
</template>

<script setup>
  import { addCartItem, isCartOpen } from '../cartStore';

  // ¡usaremos valores fijos por simplicidad!
  const hardcodedItemInfo = {
    id: 'astronaut-figurine',
    name: 'Astronaut Figurine',
    imageSrc: '/images/astronaut-figurine.png',
  }

  function addToCart(e) {
    e.preventDefault();
    isCartOpen.set(true);
    addCartItem(hardcodedItemInfo);
  }
</script>
```
</Fragment>
</UIFrameworkTabs>

Finalmente, renderizamos los ítems dentro del carrito en nuestro componente `CartFlyout`:

<UIFrameworkTabs>
<Fragment slot="preact">
```jsx
// src/components/CartFlyout.jsx
import { useStore } from '@nanostores/preact';
import { isCartOpen, cartItems } from '../cartStore';

export default function CartFlyout() {
  const $isCartOpen = useStore(isCartOpen);
  const $cartItems = useStore(cartItems);

  return $isCartOpen ? (
    <aside>
      {Object.values($cartItems).length ? (
        <ul>
          {Object.values($cartItems).map(cartItem => (
            <li>
              <img src={cartItem.imageSrc} alt={cartItem.name} />
              <h3>{cartItem.name}</h3>
              <p>Cantidad: {cartItem.quantity}</p>
            </li>
          ))}
        </ul>
      ) : <p>¡Tu carrito está vacío!</p>}
    </aside>
  ) : null;
}
```
</Fragment>
<Fragment slot="react">
```jsx
// src/components/CartFlyout.jsx
import { useStore } from '@nanostores/react';
import { isCartOpen, cartItems } from '../cartStore';

export default function CartFlyout() {
  const $isCartOpen = useStore(isCartOpen);
  const $cartItems = useStore(cartItems);

  return $isCartOpen ? (
    <aside>
      {Object.values($cartItems).length ? (
        <ul>
          {Object.values($cartItems).map(cartItem => (
            <li>
              <img src={cartItem.imageSrc} alt={cartItem.name} />
              <h3>{cartItem.name}</h3>
              <p>Cantidad: {cartItem.quantity}</p>
            </li>
          ))}
        </ul>
      ) : <p>¡Tu carrito está vacío!</p>}
    </aside>
  ) : null;
}
```
</Fragment>
<Fragment slot="solid">
```jsx
// src/components/CartFlyout.jsx
import { useStore } from '@nanostores/solid';
import { isCartOpen, cartItems } from '../cartStore';

export default function CartFlyout() {
  const $isCartOpen = useStore(isCartOpen);
  const $cartItems = useStore(cartItems);

  return $isCartOpen ? (
    <aside>
      {Object.values($cartItems).length ? (
        <ul>
          {Object.values($cartItems).map(cartItem => (
            <li>
              <img src={cartItem.imageSrc} alt={cartItem.name} />
              <h3>{cartItem.name}</h3>
              <p>Cantidad: {cartItem.quantity}</p>
            </li>
          ))}
        </ul>
      ) : <p>¡Tu carrito está vacío!</p>}
    </aside>
  ) : null;
}
```
</Fragment>
<Fragment slot="svelte">
```svelte
<!--src/components/CartFlyout.svelte-->
<script>
  import { isCartOpen, cartItems } from '../cartStore';
</script>

{#if $isCartOpen}
  {#if Object.values($cartItems).length}
    <aside>
      {#each Object.values($cartItems) as cartItem}
      <li>
        <img src={cartItem.imageSrc} alt={cartItem.name} />
        <h3>{cartItem.name}</h3>
        <p>Cantidad: {cartItem.quantity}</p>
      </li>
      {/each}
    </aside>
  {#else}
    <p>¡Tu carrito está vacío!</p>
  {/if}
{/if}
```
</Fragment>
<Fragment slot="vue">
```vue
<!--src/components/CartFlyout.vue-->
<template>
  <aside v-if="$isCartOpen">
    <ul v-if="Object.values($cartItems).length">
      <li v-for="cartItem in Object.values($cartItems)" v-bind:key="cartItem.name">
        <img :src=cartItem.imageSrc :alt=cartItem.name />
        <h3>{{cartItem.name}}</h3>
        <p>Cantidad: {{cartItem.quantity}}</p>
      </li>
    </ul>
    <p v-else>¡Tu carrito está vacío!</p>
  </aside>
</template>

<script setup>
  import { cartItems, isCartOpen } from '../cartStore';
  import { useStore } from '@nanostores/vue';

  const $isCartOpen = useStore(isCartOpen);
  const $cartItems = useStore(cartItems);
</script>
```
</Fragment>
</UIFrameworkTabs>

Ya deberías tener un ejemplo de ecommerce totalmente interactivo con el menor paquete de JS de la galaxia 🚀

¡[**Prueba el ejemplo terminado**](https://github.com/withastro/astro/tree/main/examples/with-nanostores) en tu máquina u online vía Stackblitz!
