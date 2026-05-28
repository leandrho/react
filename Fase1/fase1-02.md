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

## Expresiones, condicionales y listas en JSX

JSX permite insertar JavaScript dentro de la interfaz utilizando llaves `{}`. Todo lo que se coloque dentro de ellas es interpretado como una expresión JavaScript.

Ejemplo:

```tsx
const name = "John"

return <h1>Hello {name}</h1>
```

Dentro de JSX pueden utilizarse:

* variables,
* operaciones,
* llamadas a funciones,
* métodos de arrays,
* operadores,
* y expresiones en general.

Ejemplo:

```tsx
return <h1>{2 + 2}</h1>
```

o:

```tsx
return <h1>{formatName(user)}</h1>
```

### JSX espera expresiones, no instrucciones

Dentro de JSX pueden colocarse expresiones que produzcan un valor, pero no instrucciones como:

```js
if
for
switch
```

Por eso las condiciones suelen resolverse utilizando:

* operadores ternarios,
* `&&`,
* y las listas mediante `.map()`.


### Condicionales en JSX

React utiliza JavaScript normal para renderizado condicional.

#### Operador ternario

```tsx
return (
  <h1>
    {isLogged ? "Welcome" : "Login"}
  </h1>
)
```

El ternario es útil cuando existen dos posibles resultados.

#### Operador `&&`

```tsx
return (
  <>
    {isAdmin && <button>Delete</button>}
  </>
)
```

Si `isAdmin` es `true`, React renderiza el botón.
Si es `false`, no renderiza nada.

#### Importante sobre `&&`

El operador `&&` devuelve el valor real de la expresión JavaScript.

Ejemplo:

```js
0 && <Modal />
```

devuelve:

```js
0
```

Y React sí renderiza números, por lo que aparecería:

```text id="k63d7o"
0
```

En cambio:

```js
false && <Modal />
```

devuelve `false`, y React ignora:

* `false`,
* `null`,
* `undefined`,
* `true`.

Por eso normalmente no se renderiza nada visualmente.

### Listas en JSX

Para renderizar múltiples elementos dinámicamente normalmente se utiliza `.map()`.

Ejemplo:

```tsx
const users = ["John", "Jane"]

return (
  <ul>
    {users.map(user => (
      <li>{user}</li>
    ))}
  </ul>
)
```

`.map()`: Recorre el array, transforma cada elemento y devuelve un nuevo array de JSX que React puede renderizar.

## Reglas de `key` y por qué importan

Cuando React renderiza listas de elementos, necesita una forma de identificar cada elemento de manera única entre renderizados. Para eso existen las `key`.

Ejemplo:

```tsx
users.map(user => (
  <li key={user.id}>
    {user.name}
  </li>
))
```

La `key` permite que React relacione el elemento anterior, con el nuevo elemento renderizado.

### ¿Por qué son importantes?

Durante la reconciliación, React compara listas para detectar:

* elementos agregados,
* eliminados,
* movidos,
* o actualizados.

Las `key` ayudan a React a identificar correctamente cada elemento y reutilizar los nodos adecuados en lugar de recrearlos innecesariamente.

### Qué pasa sin keys correctas

Si React no puede identificar correctamente los elementos:

* puede recrear componentes innecesariamente,
* perder estado interno,
* mezclar datos visuales,
* o generar renders incorrectos.

### Regla principal

Las `key` deben ser únicas entre hermanos, estables y persistentes entre renderizados.

La mejor opción normalmente es un ID real:

```tsx
key={user.id}
```

### Problema de usar índices

Esto:

```tsx
key={index}
```

puede funcionar en listas estáticas, pero en listas dinámicas puede causar problemas si:

* se reordenan elementos,
* se eliminan,
* o se insertan nuevos.

Porque los índices cambian y React puede pensar que un elemento es otro distinto.

