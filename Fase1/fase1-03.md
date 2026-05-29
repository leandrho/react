# Componentes
*La unidad de composición*

Un componente es una unidad reutilizable e independiente de la interfaz de usuario. Los componentes permiten dividir la UI en pequeñas piezas encapsuladas que pueden combinarse para construir interfaces más grandes y complejas.

Un componente puede contener:

* estructura visual (JSX),
* lógica,
* estado,
* eventos,
* y comportamiento propio.

React construye toda la interfaz ejecutando componentes y combinando el resultado que estos retornan.

Conceptualmente, un componente funciona de manera muy similar a una función: recibe entradas (props) y devuelve una representación de la interfaz de usuario.

## Function Components

Los Function Components son la forma moderna y principal de crear componentes en React. Son simplemente funciones de JavaScript que retornan JSX.

Ejemplo:

```tsx
function Button() {
  return <button>Click</button>
}
```

Cuando React renderiza un componente, ejecuta esa función para obtener la representación de la interfaz.

Un Function Component devuelve elementos de React (JSX transpílado) que React utiliza para construir el Virtual DOM y luego sincronizar con el DOM real.

### Características principales

Un Function Component:

* recibe props como argumentos,
* puede manejar estado mediante hooks,
* puede utilizar efectos (`useEffect`),
* y puede renderizar otros componentes.

Ejemplo:

```tsx
function Welcome({ name }) {
  return <h1>Hello {name}</h1>
}
```

## Props: paso por valor, desestructuración, defaults

Las props (properties) son datos que un componente padre le pasa a un componente hijo. Funcionan de manera similar a los parámetros de una función.

Las props permiten que los componentes sean dinámicos y reutilizables, pasando información desde componentes padres hacia componentes hijos de manera declarativa.

Ejemplo:

```tsx
function Welcome(props) {
  return <h1>Hello {props.name}</h1>
}
````
```
<Welcome name="John" />
```

En este caso:

* el componente padre envía la prop `name`,
* y el componente hijo la recibe dentro del objeto `props`.


### Paso por valor

Las props son solo datos que React pasa al componente cuando lo ejecuta.

Conceptualmente:

```tsx
Welcome({ name: "John" })
```

Es decir, React llama al componente como una función y le entrega un objeto con las props.

### Props son read-only

Un componente nunca debería modificar sus props.

Esto está mal:

```tsx
props.name = "Jane"
```

Porque las props deben considerarse inmutables.

El componente recibe datos y los utiliza, pero no debería alterarlos.

### Desestructuración de props

Es muy común desestructurar las props directamente en los parámetros:

```tsx
function Welcome({ name }) {
  return <h1>Hello {name}</h1>
}
```

En lugar de:

```tsx
function Welcome(props) {
  return <h1>{props.name}</h1>
}
```

La desestructuración mejora:

* legibilidad,
* acceso a props,
* y evita repetir `props.` constantemente.

### Default values

Las props pueden tener valores por defecto utilizando parámetros default de JavaScript.

Ejemplo:

```tsx
function Button({ text = "Click" }) {
  return <button>{text}</button>
}
```

Si no se envía `text`, el componente utilizará `"Click"`.

### Tipos de props

Las props pueden ser:

* strings,
* números,
* booleanos,
* arrays,
* objetos,
* funciones,
* JSX,
* e incluso otros componentes.

Ejemplo:

```tsx
<Button disabled={true} />
```

o:

```tsx
<Button onClick={handleClick} />
```
### Props Typescript

En React es muy común utilizar TypeScript para tipar las `props` de los componentes. Esto permite definir explícitamente qué datos puede recibir un componente, mejorando la seguridad, el autocompletado y la detección temprana de errores durante el desarrollo.

Normalmente se define un `type` o `interface` para describir la estructura de las `props` y luego se utiliza ese tipo en los parámetros del componente. Además, TypeScript permite marcar `props` opcionales, tipar funciones, objetos, arrays y cualquier otro valor que el componente necesite recibir.

Ejemplo:

```tsx
type ButtonProps = {
  text: string
  disabled?: boolean
}

function Button({
  text,
  disabled = false
}: ButtonProps) {
  return (
    <button disabled={disabled}>
      {text}
    </button>
  )
}
```

Uso:

```tsx
<Button text="Save" />
<Button text="Delete" disabled />
```









