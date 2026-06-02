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

---

## Stale Closures: qué son y cómo evitarlas

Los **stale closures** son uno de los problemas más comunes y difíciles de detectar cuando se empieza a trabajar con `useEffect`.

La idea fundamental es:

> Un efecto "recuerda" los valores que existían en el render donde fue creado.

Si más adelante esos valores cambian pero el efecto no se vuelve a crear, seguirá utilizando los valores antiguos.

### El problema en React

Supongamos:

```tsx
import { useEffect, useState } from "react"

function App() {
  const [count, setCount] = useState<number>(0)

  useEffect(() => {
    const intervalId = setInterval(() => {
      console.log(count)
    }, 1000)

    return () => {
      clearInterval(intervalId)
    }
  }, [])

  return (
    <button
      onClick={() =>
        setCount(prev => prev + 1)
      }
    >
      {count}
    </button>
  )
}
```

¿Qué imprime?

Muchos esperan:

```text
0 1 2 3 ...
```

Pero en realidad imprime:

```text
0 0 0 0 ...
```

### ¿Por qué ocurre?

Porque el efecto se ejecutó una sola vez:

```tsx
[], // dependency array vacío
```

Durante ese primer render:

```text
count = 0
```

El callback del `setInterval` quedó asociado a ese valor.

Aunque el componente vuelva a renderizar y `count` cambie, el intervalo sigue utilizando la closure original.


### ¿Cómo solucionarlo?

### Opción 1: Dependencias correctas

```tsx
useEffect(() => {
  const intervalId = setInterval(() => {
    console.log(count)
  }, 1000)

  return () => {
    clearInterval(intervalId)
  }
}, [count])
```

Ahora React:

* limpia el efecto anterior,
* crea uno nuevo,
* y la closure recibe el valor actualizado.

### Opción 2: Actualizaciones funcionales

Muchas veces el problema aparece al actualizar estado.

Incorrecto:

```tsx
setCount(count + 1)
```

Mejor:

```tsx
setCount(prev => prev + 1)
```

Porque no depende del valor capturado por la closure.

### Opción 3: useRef (más adelante)

En algunos casos avanzados se utiliza:

```tsx
const countRef = useRef(count)
```

para acceder siempre al valor más reciente sin recrear efectos constantemente.

Lo veremos cuando estudiemos `useRef`.

### ¿Por qué el linter insiste tanto?

Porque la mayoría de los stale closures aparecen cuando faltan dependencias.

Ejemplo:

```tsx
useEffect(() => {
  console.log(userId)
}, [])
```

El efecto usa:

```tsx
userId
```

pero no lo declara como dependencia.

Por eso el linter avisa:

```text
Missing dependency: userId
```

Está intentando evitar que el efecto se quede con una versión antigua de esa variable.

### En resumen

Un stale closure ocurre cuando un efecto o callback continúa utilizando valores de un render anterior porque fue creado con una closure antigua. La forma principal de evitarlo es declarar correctamente las dependencias del efecto para que React pueda recrearlo cuando esos valores cambien.

---

## Efectos vs Eventos: la distinción clave

La diferencia fundamental es:

> **Los eventos ocurren porque el usuario hizo algo.**
>
> **Los efectos ocurren porque el componente se renderizó.**

Los **eventos** representan interacciones explícitas del usuario, como clicks, cambios en inputs o envíos de formularios. Por eso, la lógica que responde directamente a esas acciones suele vivir en handlers como `onClick`, `onChange` o `onSubmit`.

```tsx
function handleSubmit() {
  sendForm()
}

<form onSubmit={handleSubmit}>
```

En cambio, los **efectos** existen para sincronizar el componente con sistemas externos después de que React terminó un render. Algunos ejemplos son realizar peticiones HTTP, registrar listeners, iniciar timers, actualizar `document.title` o conectar un websocket.

```tsx
useEffect(() => {
  document.title = title
}, [title])
```

La pregunta práctica es:

* **¿Esto ocurre porque el usuario hizo algo?** → probablemente pertenece a un evento.
* **¿Esto ocurre porque el componente apareció, desapareció o cambió?** → probablemente pertenece a un efecto.

Esta distinción es una de las claves para evitar el uso innecesario de `useEffect` y escribir componentes más simples y predecibles.

## Anatomía del useEffect

Si el `useState` es el mecanismo para capturar fotografías del estado de tu aplicación, el `useEffect` es el mecanismo de React para **sincronizar esas fotografías con el mundo exterior**.

Muchos desarrolladores cometen el error de pensar en `useEffect` como un sustituto de los ciclos de vida de los componentes de clase (`componentDidMount`, `componentDidUpdate`, `componentWillUnmount`). **Ese modelo mental es incorrecto y peligroso en el React moderno.** Para entender `useEffect` a nivel experto, tenemos que cambiar el chip: `useEffect` no piensa en "momentos del tiempo", piensa en **Sincronización y Dependencias**.

Vamos a desarmarlo a fondo, desde su anatomía interna en el árbol Fiber hasta cómo maneja las closures y la limpieza de memoria.

---

## 1. El Modelo Mental: Sincronización, no Ciclo de Vida

Un componente de React puro toma datos (props y estado) y devuelve una interfaz (JSX). Es una función matemática pura. Pero las aplicaciones reales necesitan hacer cosas que escapan a este flujo limpio: conectarse a un WebSocket, hacer un `fetch` a una API, modificar el título de la pestaña del navegador (`document.title`) o configurar un temporizador (`setInterval`).

A estas acciones que afectan a cosas fuera del control de React las llamamos **Efectos Secundarios** (*Side Effects*).

El trabajo de `useEffect` es asegurar que el mundo exterior esté perfectamente sincronizado con el estado actual de tu componente.

## 2. La Anatomía Interna: ¿Cómo vive un Effect en Fiber?

Al igual que `useState`, `useEffect` no es mágico; es una estructura de datos que se almacena en la **lista enlazada de hooks** dentro del nodo Fiber de tu componente.

Cuando declarás un `useEffect`, React crea un objeto de tipo `Effect` y lo guarda en la propiedad `memoizedState` de ese hook específico.

```typescript
interface Effect {
  tag: HookFlags;       // Define el tipo de efecto (ej: si es un efecto de layout o normal)
  create: () => (() => void) | void; // Tu función del efecto (lo que querés que corra)
  destroy: (() => void) | void;     // Tu función de limpieza (Cleanup), si devolviste una
  deps: Array<any> | null;          // El array de dependencias que pasaste
  next: Effect | null;  // Puntero al siguiente efecto del componente (cola circular)
}

```

### El secreto del array de dependencias (`deps`)

React utiliza un algoritmo de comparación superficial llamado `Object.is` para verificar si las dependencias cambiaron entre un render y el siguiente.

* **Sin array de dependencias (`useEffect(fn)`):** React asume que el efecto está sincronizado con *todo* el componente. Se ejecuta en **cada uno** de los renders.
* **Array vacío (`useEffect(fn, [])`):** React compara el array actual con el anterior. Como ambos están vacíos, `Object.is` dice "son iguales". React asume que el efecto ya está sincronizado y **no lo vuelve a ejecutar**.
* **Con elementos (`useEffect(fn, [a, b])`):** React compara `a` viejo con `a` nuevo, y `b` viejo con `b` nuevo. Si al menos uno cambió, marca el efecto para ser ejecutado.

## 3. El Ciclo de Ejecución Sincrónico y Asíncrono

Para no bloquear la pantalla del usuario, React maneja los efectos de forma **asíncrona y pasiva**. Esto significa que el código de tu `useEffect` no se ejecuta mientras el componente se está renderizando, ni tampoco inmediatamente cuando se modifica el DOM.

El viaje exacto de un render con `useEffect` es el siguiente:

1. **Fase de Render:** Se ejecuta la función del componente. `useEffect` registra su función `create` y sus `deps` en el nodo Fiber.
2. **Fase de Commit:** React muta el DOM real. La pantalla del navegador se actualiza físicamente y el usuario ve los cambios visuales.
3. **Pausa de Pintado:** React le cede el control al navegador para que "pinte" los píxeles en la pantalla (*Paint*). Esto garantiza que la app se sienta fluida.
4. **Fase de Efecto (Macro-tarea/Scheduler):** El Scheduler de React se activa inmediatamente después del pintado y ejecuta las funciones `create` de los efectos que quedaron agendados porque sus dependencias cambiaron.

## 4. Las Closures atacan de nuevo: El peligro de los efectos congelados

Al igual que vimos con el `useState`, las funciones dentro de un `useEffect` son **closures** que capturan las variables del render en el que fueron creadas. Si no manejás bien las dependencias, tu efecto sufrirá de "estados viejos" (*stale closures*).

Analicemos este temporizador roto:

```typescript
function Timer() {
  const [seconds, setSeconds] = useState<number>(0);

  useEffect(() => {
    const interval = setInterval(() => {
      // Alerta de peligro: Esta función captura 'seconds' del Render #1 (vale 0)
      console.log("Segundos leídos por el intervalo:", seconds); 
      setSeconds(seconds + 1); 
    }, 1000);

    return () => clearInterval(interval);
  }, []); // Array vacío: El efecto solo se ejecuta en el Render #1

  return <div>Segundos: {seconds}</div>;
}

```

### ¿Qué pasa en la memoria aquí?

1. **Render #1:** `seconds` vale `0`. Se ejecuta el `useEffect`. La closure del `setInterval` captura la variable `seconds` con el valor `0`.
2. **Segundo 1:** El intervalo se dispara. Lee `seconds` (que está congelado en `0` por la closure), hace `setSeconds(0 + 1)`. React agenda un nuevo render.
3. **Render #2:** `seconds` ahora vale `1`. Pero como pasaste un array de dependencias vacío `[]`, **React no vuelve a ejecutar el efecto**. El viejo intervalo del Render #1 sigue corriendo en el fondo.
4. **Segundo 2:** El intervalo se vuelve a disparar. Vuelve a leer la variable `seconds` de su propia closure (que sigue siendo `0`). Hace `setSeconds(0 + 1)`.
5. **Resultado:** El contador se clava en `1` para siempre y el console log imprime `0` infinitamente.

### ¿Cómo se soluciona? Dos caminos muy diferentes:

**Opción A: Añadir la dependencia (Sincronización total)**

```typescript
useEffect(() => {
  const interval = setInterval(() => {
    setSeconds(seconds + 1);
  }, 1000);
  return () => clearInterval(interval);
}, [seconds]); // Cada vez que seconds cambia, el efecto se destruye y se vuelve a crear

```

*Costo:* Cada segundo destruimos un intervalo y creamos uno nuevo. Funciona, pero es ineficiente para este caso.

**Opción B: Usar la función actualizadora (Romper la closure)**

```typescript
useEffect(() => {
  const interval = setInterval(() => {
    // No leemos la variable externa 'seconds', usamos el estado vivo que nos da React
    setSeconds(prev => prev + 1); 
  }, 1000);
  return () => clearInterval(interval);
}, []); // No depende de variables locales, el efecto es seguro y corre una sola vez

```

## 5. La función de Limpieza (Cleanup): Cerrando el portal

La función que devolvés dentro del `useEffect` (el `return () => { ... }`) se llama **Cleanup**. Su objetivo es limpiar cualquier desastre o conexión abierta antes de que el componente muera o antes de que el efecto se vuelva a ejecutar con nuevos datos.

Hay un error conceptual masivo aquí: la mayoría cree que el cleanup solo corre cuando el componente se desmonta (desaparece de la pantalla). **Falso.** El cleanup corre **antes de cada ejecución del efecto** para limpiar el render anterior.

Mirá este flujo de sincronización de un chat:

```typescript
function ChatRoom({ roomId }: { roomId: string }) {
  useEffect(() => {
    const connection = connectToRoom(roomId);
    console.log(`🟢 Conectado a la sala: ${roomId}`);

    return () => {
      connection.disconnect();
      console.log(`🔴 Desconectado de la sala: ${roomId}`);
    };
  }, [roomId]); // Depende de roomId
}

```

Si el usuario está en la sala `"general"` y cambia a la sala `"gaming"`, el orden exacto de eventos en la memoria de React es:

1. **Render #1:** `roomId` es `"general"`. Se ejecuta el efecto $\rightarrow$ *Log: 🟢 Conectado a la sala: general*. El cleanup queda guardado en la propiedad `destroy` de Fiber.
2. **Render #2:** `roomId` cambia a `"gaming"`. React hace el Diffing, ve que la dependencia cambió.
3. Antes de ejecutar el nuevo efecto, React va al Fiber viejo, busca la función `destroy` guardada y la ejecuta $\rightarrow$ *Log: 🔴 Desconectado de la sala: general* (La closure del cleanup del Render #1 todavía recordaba que su `roomId` era `"general"`).
4. Ahora que el pasado está limpio, React ejecuta el `create` del Render #2 $\rightarrow$ *Log: 🟢 Conectado a la sala: gaming*.

### Resumen de Reglas de Oro para un Senior

* **No pienses en tiempos (mount/update):** Pensá en qué entidades externas deben estar sincronizadas con cuáles variables locales de tu render.
* **Mentir en las dependencias es el peor pecado:** Si usás una variable dentro del efecto, **debe** ir en el array de dependencias. Si no la ponés, tu closure se romperá y leerá datos fantasmas del pasado. Si ponerla causa bucles infinitos, el problema no es el array, es que necesitás reestructurar tu lógica (usando funciones actualizadoras de `useState` o abstrayendo con `useCallback`).
* **Cada render tiene sus propios Efectos:** Al igual que el estado, las funciones de tus efectos pertenecen a una fotografía específica en el tiempo. El cleanup limpia la fotografía anterior justo antes de que se tome la nueva.