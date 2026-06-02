# useEffect
*El hook más mal entendido*

## ¿Qué es useEffect y para qué sirve?

Antes de hablar de sintaxis, dependencias o cleanup, quiero que se entienda el propósito.

### El trabajo principal de React

Recordemos cuál es el trabajo principal de React:

```text
State + Props
↓
Calcular UI
↓
Actualizar DOM
```

Por ejemplo:

```tsx
type GreetingProps = {
  name: string
}

function Greeting({ name }: GreetingProps) {
  return <h1>Hola {name}</h1>
}
```

React ejecuta la función:

```tsx
Greeting({ name: "Leandro" })
```

y obtiene:

```tsx
<h1>Hola Leandro</h1>
```

Eso es puro cálculo.

No hay efectos secundarios.

### El mundo fuera de React

Ahora imaginemos algo diferente.

```tsx
function Counter() {
  const [count, setCount] = useState<number>(0)

  document.title = `Count: ${count}`

  return (
    <button onClick={() => setCount(count + 1)}>
      {count}
    </button>
  )
}
```

Acá ocurre algo interesante.

Tenemos dos cosas distintas:

### Dentro de React

```tsx
<button>{count}</button>
```

Es UI.

React sabe manejarla.

### Fuera de React

```tsx
document.title = `Count: ${count}`
```

Esto modifica algo externo.

React no controla el título de la pestaña.


## Definición de efecto

Un efecto es cualquier operación que interactúa con algo fuera del proceso de renderizado de React.

Ejemplos:

```tsx
document.title = "Hola"
```

```tsx
localStorage.setItem("theme", "dark")
```

```tsx
fetch("/api/users")
```

```tsx
window.addEventListener(...)
```

```tsx
setInterval(...)
```

```tsx
socket.connect()
```

Todos tienen algo en común:

No calculan UI e interactúan con sistemas externos.


### Entonces, ¿qué es useEffect?
`useEffect` es un Hook que permite ejecutar efectos secundarios (side effects) después de que React haya renderizado y confirmado los cambios en el DOM.

Un _efecto secundario_ es cualquier operación que interactúa con algo externo al proceso de renderizado.

`useEffect` le permite a un componente _**sincronizarse**_ con sistemas externos después de que React complete el render y el commit.

### ¿Por qué React no ejecuta los efectos durante el render?

Imaginemos esto:

```tsx
function Example() {
  fetch("/api/users")

  return <h1>Hola</h1>
}
```

Si React permitiera esto libremente:

cada render podría disparar:

```text
fetch
fetch
fetch
fetch
fetch
```

Además React podría:

```text
empezar un render
cancelarlo
volver a renderizar
```

(recordá Fiber y Concurrent Rendering).

Por eso React intenta mantener el render como una fase pura.

### Modelo mental correcto

No pienses:

```text
useEffect es para ejecutar código después del render
```

Porque es cierto, pero incompleto.

Pensá:

```text
React renderiza UI
↓
React actualiza DOM
↓
React sincroniza sistemas externos
```

Y esa sincronización ocurre dentro de:

```tsx
useEffect(...)
```

## Ejemplos típicos donde useEffect tiene sentido

### Actualizar el título

```tsx
useEffect(() => {
  document.title = `Count: ${count}`
}, [count])
```

### Consultar una API

```tsx
useEffect(() => {
  async function loadUsers(): Promise<void> {
    const response = await fetch("/api/users")
    const data = await response.json()

    setUsers(data)
  }

  void loadUsers()
}, [])
```

### Frase para recordar

Si tuviera que resumir todo este tema en una sola frase sería:

> `useEffect` existe para sincronizar un componente React con sistemas externos después de que React termine de actualizar la interfaz.

## Cuándo usar useEffect

Regla práctica:

Usá `useEffect` cuando necesites sincronizar tu componente con algo que existe fuera de React.

Por ejemplo:
- APIs
- Eventos del navegador
- Local Storage
- Librerías externas
- WebSockets

WebSockets ejemplo:

```tsx
useEffect(() => {
  const socket = createSocket()

  socket.connect()

  return () => socket.disconnect()
}, [])
```

Porque React no administra esa conexión.

## Cuándo NO usar useEffect

Acá es donde empieza el React moderno.

La documentación actual de React insiste muchísimo en esto.

No usar useEffect para:
* calcular valores derivados
* sincronizar estado con estado
* reaccionar a eventos de usuario
* transformar datos para renderizar

### Ejemplo para calcular datos derivados


```tsx
type FullNameProps = {
  firstName: string
  lastName: string
}

function FullName({
  firstName,
  lastName,
}: FullNameProps) {
  const [fullName, setFullName] = useState("")

  useEffect(() => {
    setFullName(`${firstName} ${lastName}`)
  }, [firstName, lastName])

  return <p>{fullName}</p>
}
```

Muchos desarrolladores hacen esto.

Pero es innecesario, ya que `fullName` no es un sistema externo.
Se puede calcular directamente.

La versión correcta:

```tsx
type FullNameProps = {
  firstName: string
  lastName: string
}

function FullName({
  firstName,
  lastName,
}: FullNameProps) {
  const fullName = `${firstName} ${lastName}`

  return <p>{fullName}</p>
}
```
Más simple. Más rápida. Menos renders. Menos bugs.

## Una pregunta útil

Cuando escribís un Effect preguntate:

> ¿Qué sistema externo estoy sincronizando?

### Resumen

Usá `useEffect` para sincronizar React con sistemas externos:

* APIs
* WebSockets
* Eventos del navegador
* Local Storage
* Librerías externas
* document.title

No uses `useEffect` para:

* calcular valores derivados
* sincronizar estado con estado
* reaccionar a eventos de usuario
* transformar datos para renderizar

La pregunta clave es:

> **¿Qué sistema externo estoy sincronizando?**

Si no podés responder esa pregunta, es una señal muy fuerte de que ese `useEffect` probablemente no debería existir.

---

## Dependency Array

El **dependency array** es el segundo argumento de `useEffect` y le indica a React **cuándo debe volver a ejecutar un efecto**.

Sintaxis general:

```tsx
useEffect(() => {
  // efecto
}, [/* dependencias */])
```

React compara las dependencias del render actual con las del render anterior. Si alguna cambió, vuelve a ejecutar el efecto.

### Sin dependency array

```tsx
useEffect(() => {
  console.log("effect")
})
```

Se ejecuta después de **cada render**.

Es poco frecuente necesitar este comportamiento.

### Dependency array vacío

```tsx
useEffect(() => {
  console.log("effect")
}, [])
```

Se ejecuta solo una sola vez después del primer render.

Flujo:

```text
Primer render
↓
Effect

Render posteriores
↓
No se ejecuta
```

Suele utilizarse para:

* inicializaciones,
* listeners,
* conexiones,
* carga inicial de datos.

### Una dependencia

```tsx
useEffect(() => {
  console.log(userId)
}, [userId])
```

React vuelve a ejecutar el efecto únicamente cuando cambia `userId`.

```text
userId cambia
↓
Render
↓
Effect
```
### Múltiples dependencias

```tsx
useEffect(() => {
  console.log(userId, filter)
}, [userId, filter])
```

El efecto se ejecuta si cambia cualquiera de ellas.

### ¿Cómo compara React las dependencias?

React realiza una comparación por referencia usando algo equivalente a:

```ts
Object.is()
```

Ejemplo:

```tsx
const name = "John"
```

Si el valor no cambia:

```text
John === John
```

el efecto no se vuelve a ejecutar.

Pero con objetos:

```tsx
const user = {
  name: "John"
}
```

cada nueva referencia cuenta como un cambio:

```text
{} !== {}
```

aunque el contenido sea idéntico.


## Reglas del linter (`eslint-plugin-react-hooks`)

Cuando usás algo dentro del efecto:

```tsx
useEffect(() => {
  console.log(userId)
}, [])
```

el linter suele mostrar:

```text
Missing dependency: userId
```

porque el efecto utiliza `userId` pero no lo declaró como dependencia.

### Regla fundamental

> Todo valor utilizado dentro del efecto y proveniente del componente debe aparecer en el dependency array.

Ejemplo correcto:

```tsx
useEffect(() => {
  console.log(userId)
}, [userId])
```

### ¿Por qué existe esta regla?

Porque React crea una nueva ejecución del componente en cada render.

Si el efecto sigue utilizando valores antiguos porque faltan dependencias, aparecen errores muy difíciles de detectar, conocidos como **stale closures**, que veremos más adelante.

### ¿Debo ignorar el linter?

Generalmente NO, la gran mayoría de las veces el linter tiene razón.

Cuando aparece una advertencia suele indicar _una dependencia faltante, una lógica mal estructurada o un efecto innecesario_.
Por eso React recomienda corregir el código antes que desactivar la regla.


### En resumen

El dependency array le indica a React cuándo volver a ejecutar un efecto. Como regla general, todos los valores utilizados dentro del efecto y que provienen del componente deben declararse como dependencias. El linter ayuda a detectar omisiones que podrían provocar comportamientos inconsistentes o stale closures.

---

## Cleanup Functions y Memory Leaks

Un efecto no solo puede ejecutar código cuando comienza, sino también limpiar recursos cuando deja de ser necesario. Para eso, `useEffect` puede devolver una función conocida como **cleanup function**.

Ejemplo:

```tsx
import { useEffect } from "react"

function App() {
  useEffect(() => {
    console.log("setup")

    return () => {
      console.log("cleanup")
    }
  }, [])

  return <h1>Hello</h1>
}
```

## ¿Cuándo se ejecuta el cleanup?

Hay dos situaciones principales.

### 1. Antes de volver a ejecutar el efecto

```tsx
useEffect(() => {
  console.log("effect", userId)

  return () => {
    console.log("cleanup", userId)
  }
}, [userId])
```

Si `userId` cambia:

```text
effect(1)

userId cambia

cleanup(1)
effect(2)
```

React primero limpia el efecto anterior y luego ejecuta el nuevo.

### 2. Cuando el componente se desmonta

Si el componente desaparece de la interfaz:

```text
Component mounted
↓
Effect
↓
Component unmounted
↓
Cleanup
```

React ejecuta el cleanup para liberar recursos.

### ¿Por qué existe el cleanup?

Porque muchos efectos crean recursos externos que deben eliminarse cuando ya no se usan.

Ejemplos:

* listeners,
* timers,
* websockets,
* suscripciones,
* conexiones,
* observadores (`IntersectionObserver`, `MutationObserver`, etc.).

### Ejemplo: Event Listener

```tsx
import { useEffect } from "react"

function App() {
  useEffect(() => {
    function handleResize() {
      console.log(window.innerWidth)
    }

    window.addEventListener(
      "resize",
      handleResize
    )

    return () => {
      window.removeEventListener(
        "resize",
        handleResize
      )
    }
  }, [])

  return <h1>Hello</h1>
}
```

Si no eliminamos el listener, seguirá existiendo aunque el componente desaparezca.

## ¿Qué es un Memory Leak?

Un **memory leak** ocurre cuando recursos que ya no deberían existir continúan ocupando memoria o ejecutándose porque nunca fueron liberados correctamente.

Ejemplos comunes:

```text
Listener olvidado
Timer olvidado
WebSocket abierto
Suscripción activa
Observer activo
```

Aunque el componente ya no exista, esos recursos pueden seguir funcionando.

## ¿Siempre necesito cleanup?

Respuesta corta No.

Por ejemplo:

```tsx
useEffect(() => {
  document.title = "Dashboard"
}, [])
```

No hay nada que liberar.

## Regla práctica

Preguntate:

> ¿Este efecto crea algo que debe detenerse, desconectarse o eliminarse más adelante?

Si la respuesta es sí, probablemente necesite un cleanup.

### En resumen

El cleanup permite deshacer el trabajo realizado por un efecto. React lo ejecuta antes de re-ejecutar el efecto y cuando el componente se desmonta. Su función principal es evitar recursos huérfanos, comportamientos inesperados y memory leaks.

