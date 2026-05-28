# JSX a fondo
No es HTML, no es magia.

## ¿Qué es JSX?

JSX (JavaScript XML) es una extensión de sintaxis de JavaScript utilizada por React para describir la interfaz de usuario de una forma más declarativa y visual, usando una sintaxis similar a HTML dentro de JavaScript.

JSX sirve para definir la estructura de la UI de manera más clara y legible. En lugar de crear elementos manualmente con funciones como `React.createElement`, JSX permite escribir componentes de una forma mucho más cercana a cómo se verá finalmente la interfaz.

Aunque se parezca a HTML, JSX no es HTML real. El código JSX es transformado durante la compilación en llamadas JavaScript que React utiliza para crear los elementos del Virtual DOM.

## Transpilación a `React.createElement()`

El código JSX que escribimos no es entendido directamente por el navegador. Antes de ejecutarse, debe ser transformado (transpilado) a JavaScript puro.

Por ejemplo:

```tsx id="yngoxh"
const element = <h1>Hello</h1>
```

se transpila aproximadamente a:

```js id="sqjotv"
const element = React.createElement(
  "h1",
  null,
  "Hello"
)
```

`React.createElement()` es la función que React utiliza para crear los objetos que representan elementos del Virtual DOM.

---

### ¿Qué hace `React.createElement()`?

Recibe principalmente, el tipo de elemento, las props y los children.

Ejemplo:

```js
React.createElement(
  "button",
  { className: "btn" },
  "Click"
)
```

Eso genera un objeto JavaScript similar a:

```js
{
  type: "button",
  props: {
    className: "btn",
    children: "Click"
  }
}
```
**En Resumen:** JSX se transpila, se convierte en llamadas JavaScript y React utiliza esos objetos para construir el Virtual DOM.

## Fragmentos en JSX

En React, un componente debe retornar un único elemento raíz.

Esto NO es válido:

```tsx
return (
  <h1>Hello</h1>
  <p>World</p>
)
```

Porque hay múltiples elementos al mismo nivel.

Los fragmentos permiten agrupar múltiples elementos **sin agregar nodos extra** al DOM.

Ejemplo:

```tsx
return (
  <>
    <h1>Hello</h1>
    <p>World</p>
  </>
)
```

React agrupa ambos elementos, pero no crea un `<div>` adicional en el DOM real.

#### Sintaxis corta

```tsx
<>
  ...
</>
```

#### `Fragment`

También puede escribirse explícitamente:

```tsx
import { Fragment } from "react"

return (
  <Fragment>
    <h1>Hello</h1>
    <p>World</p>
  </Fragment>
)
```

> La sintaxis corta no permite props.
> Mientras que `Fragment` sí puede recibir algunas props especiales como `key`.
>
> `<Fragment key={id}>`


