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

Si estas utilizando `typescript` podemos tipar las `props` de los componentes. Esto permite definir explícitamente qué datos puede recibir un componente, mejorando la seguridad, el autocompletado y la detección temprana de errores durante el desarrollo.

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

## `children` y composición de componentes

`children` es una prop especial que representa todo el contenido colocado entre la etiqueta de apertura y cierre de un componente.

Ejemplo:

```tsx
type CardProps = {
  children: React.ReactNode
}

function Card({ children }: CardProps) {
  return (
    <div>
      {children}
    </div>
  )
}
```

Uso:

```tsx
<Card>
  <h1>Hello</h1>
  <p>Welcome</p>
</Card>
```

Internamente, React transforma ese contenido en la prop `children` y se la pasa al componente.

Conceptualmente:

```tsx
Card({
  children: (
    <>
      <h1>Hello</h1>
      <p>Welcome</p>
    </>
  )
})
```

### ¿Qué puede contener `children`?

`children` puede contener prácticamente cualquier cosa que React pueda renderizar:

* elementos JSX,
* strings,
* números,
* fragments,
* arrays de elementos,
* otros componentes,
* e incluso expresiones dinámicas.

Ejemplo:

```tsx
<Card>
  <Button />
  {"Hello"}
  {isAdmin && <AdminPanel />}
</Card>
```

Por eso normalmente se tipa como:

```tsx
React.ReactNode
```

### ¿Qué problema resuelve?

Sin `children`, muchos componentes serían extremadamente rígidos.

Ejemplo rígido:

```tsx
type CardProps = {
  title: string
  text: string
}
```

Ese componente solo serviría para una estructura específica.

Con `children`, el componente se vuelve flexible y reutilizable:

```tsx
<Card>
  <CustomHeader />
  <UserInfo />
  <Footer />
</Card>
```

El componente ya no necesita conocer qué contenido renderizará.

### Composición de componentes

La composición es uno de los principios fundamentales de React.

Consiste en construir interfaces complejas combinando componentes más pequeños y reutilizables.

Ejemplo:

```tsx
<Page>
  <Sidebar />
  <Content>
    <Article />
  </Content>
</Page>
```

Cada componente:

* encapsula responsabilidad,
* puede reutilizarse,
* y se combina con otros componentes para formar la UI completa.


### `children` como slot dinámico

Mentalmente, `children` funciona como un espacio dinámico donde el componente padre puede insertar contenido.

Por ejemplo:

```tsx
function Modal({ children }: ModalProps) {
  return (
    <div className="modal">
      {children}
    </div>
  )
}
```

El modal controla:

* estructura,
* estilos,
* comportamiento,

pero el contenido interno queda completamente libre.

#### En resumen:
`children` permite pasar interfaz dentro de otros componentes, haciendo posible la composición, que es el mecanismo principal que React utiliza para construir interfaces reutilizables, flexibles y desacopladas.








