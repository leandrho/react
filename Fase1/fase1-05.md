# Eventos
*La conexión entre el usuario y la UI*

Los **eventos** permiten que la interfaz reaccione a las interacciones del usuario o a determinadas acciones del sistema, como clicks, escritura, envíos de formularios o movimientos del mouse. En React, los eventos forman parte del flujo interactivo de la aplicación y normalmente son los que desencadenan cambios de estado y nuevas actualizaciones de la interfaz.

# Eventos en profundidad
## Synthetic Events

En React, los eventos no son exactamente los eventos nativos del navegador. React utiliza un sistema propio llamado **Synthetic Events**.

Un **Synthetic Event** es un objeto envoltorio (wrapper) que React crea alrededor de los eventos nativos del DOM para ofrecer:

* una API consistente entre navegadores,
* mejor compatibilidad,
* y un sistema de eventos unificado.

Ejemplo:

```tsx
import { MouseEvent } from "react"

function Button() {
  function handleClick(
    event: MouseEvent<HTMLButtonElement>
  ) {
    console.log(event)
  }

  return (
    <button onClick={handleClick}>
      Click
    </button>
  )
}
```

Aunque el evento proviene del navegador, React entrega un Synthetic Event.

## Diferencias con DOM nativo

En HTML/JavaScript tradicional:

```js
button.addEventListener("click", handler)
```

En React:

```tsx
<button onClick={handleClick}>
```

React:

* registra eventos internamente,
* escucha eventos globalmente,
* y luego distribuye los handlers correspondientes.

### Diferencias importantes
#### Nombres camelCase

En React:

```tsx
onClick
onChange
onMouseEnter
```

En HTML tradicional:

```html
onclick
onchange
```
#### Se pasan funciones, no strings

Incorrecto:

```tsx
<button onClick="handleClick()">
```

Correcto:

```tsx
<button onClick={handleClick}>
```

#### JSX usa JavaScript real

Los handlers reciben funciones reales y pueden trabajar directamente con:

* estado,
* props,
* closures,
* hooks,
* etc.

## Event Bubbling

La mayoría de los eventos en React utilizan bubbling (propagación hacia arriba).

Esto significa que un evento disparado en un elemento hijo también se propaga hacia sus ancestros.

Ejemplo:

```tsx
function App() {
  return (
    <div onClick={() => console.log("div")}>
      <button onClick={() => console.log("button")}>
        Click
      </button>
    </div>
  )
}
```

Al hacer click en el botón:

```text
button
div
```

porque el evento:

1. ocurre en el botón,
2. y luego sube hacia el padre.

## preventDefault

`preventDefault()` evita el comportamiento por defecto del navegador.

Ejemplo:

```tsx
import { FormEvent } from "react"

function Form() {
  function handleSubmit(
    event: FormEvent<HTMLFormElement>
  ) {
    event.preventDefault()

    console.log("submitted")
  }

  return (
    <form onSubmit={handleSubmit}>
      <button type="submit">
        Send
      </button>
    </form>
  )
}
```

Sin `preventDefault`, el formulario recargaría la página.

## stopPropagation

`stopPropagation()` detiene la propagación del evento hacia elementos padres.

Ejemplo:

```tsx
import { MouseEvent } from "react"

function App() {
  function handleDivClick() {
    console.log("div")
  }

  function handleButtonClick(
    event: MouseEvent<HTMLButtonElement>
  ) {
    event.stopPropagation()

    console.log("button")
  }

  return (
    <div onClick={handleDivClick}>
      <button onClick={handleButtonClick}>
        Click
      </button>
    </div>
  )
}
```

Resultado:

```text
button
```
El click ya no llega al `div`.

## Event Handlers

Los event handlers son funciones que React ejecuta cuando ocurre un determinado evento en la interfaz, como un click, un cambio en un input o el envío de un formulario. Son el mecanismo que conecta las interacciones del usuario con la lógica de la aplicación.

En React, los handlers se pasan mediante props de eventos como `onClick`, `onChange` o `onSubmit`. Cuando el evento ocurre, React ejecuta la función asociada y permite realizar acciones como actualizar estado, validar datos o interactuar con otros componentes.

### Pasar funciones

Los handlers deben pasarse como funciones.

```tsx
function Button() {
  function handleClick() {
    console.log("clicked")
  }

  return (
    <button onClick={handleClick}>
      Click
    </button>
  )
}
```

React guarda la referencia de la función y la ejecuta cuando ocurre el evento.

### Referencia vs ejecución inmediata

Esta es una de las diferencias más importantes.

Correcto:

```tsx
<button onClick={handleClick}>
```

Acá React recibe una referencia a la función.

Incorrecto:

```tsx
<button onClick={handleClick()}>
```

Acá la función se ejecuta inmediatamente durante el render.

Equivale aproximadamente a:

```tsx
const result = handleClick()

<button onClick={result}>
```

Por eso normalmente genera comportamientos incorrectos.

### Handlers inline

También es posible definir handlers directamente dentro del JSX.

Ejemplo:

```tsx
<button
  onClick={() => {
    console.log("clicked")
  }}
>
  Click
</button>
```

Esto resulta útil cuando la lógica es pequeña, se necesitan argumentos o el handler se usa una sola vez.

Ejemplo con argumentos:

```tsx
<button
  onClick={() => handleDelete(userId)}
>
  Delete
</button>
```

### Handlers reutilizables

Cuando la lógica crece o se utiliza en varios lugares, suele ser mejor extraerla a una función separada.

```tsx
function UserCard() {
  function handleDelete() {
    console.log("delete user")
  }

  return (
    <button onClick={handleDelete}>
      Delete
    </button>
  )
}
```

Ventajas:

* código más legible,
* más fácil de probar,
* más fácil de reutilizar,
* JSX más limpio.

### ¿Inline o reutilizable?

No existe una regla absoluta.

Generalmente:

* lógica simple → inline,
* lógica compleja o reutilizable → función separada.

Lo importante es mantener el código claro y fácil de mantener.

### En resumen

Un event handler es una función que React ejecuta cuando ocurre una interacción. Lo importante es pasar una referencia a la función, no ejecutarla durante el render, y elegir entre handlers inline o reutilizables según la complejidad y reutilización de la lógica.
