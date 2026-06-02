# Estados
*El núcleo de la reactividad*

En React, el **estado** (state) representa la información dinámica que un componente puede almacenar y modificar a lo largo del tiempo. Cuando el estado cambia, React vuelve a renderizar el componente para recalcular la interfaz y mantener sincronizada la UI con los datos actuales.

## Hooks

Antes de continuar, debemos entender qué son los hooks.

En React, los Hooks son funciones especiales que permiten utilizar características internas de React dentro de los Function Components, como estado, efectos, referencias, contexto y otras capacidades relacionadas con el ciclo de vida y el renderizado.

Los hooks permiten reutilizar lógica y agregar comportamiento a los componentes sin necesidad de utilizar Class Components. React los identifica por convención porque sus nombres comienzan con `use`, como por ejemplo: `useState`.

## `useState`: estado local y actualizaciones

`useState` es un hook que permite agregar estado local a los Function Components. El estado representa información dinámica que puede cambiar con el tiempo y afectar lo que el componente renderiza.

`useState` permite que un componente recuerde información entre renders y que React vuelva a renderizar automáticamente la interfaz cuando esa información cambia.

Ejemplo:

```tsx
import { useState } from "react"

function Counter() {
  const [count, setCount] = useState(0)

  return <h1>{count}</h1>
}
```

En este caso:

* `count` contiene el estado actual,
* y `setCount` es la función utilizada para actualizarlo.

### ¿Qué significa “estado local”?

El estado es “local” porque pertenece específicamente a ese componente.

Cada instancia del componente mantiene su propio estado independiente.

Ejemplo:

```tsx
<Counter />
<Counter />
```

Cada `Counter` tendrá su propio valor de `count`.

### ¿Qué ocurre cuando el estado cambia?

Cuando se ejecuta:

```tsx
setCount(1)
```

React:

1. registra la actualización,
2. vuelve a renderizar el componente,
3. recalcula la UI,
4. y luego sincroniza los cambios necesarios con el DOM real.

### `useState` no modifica la variable directamente

Esto es importante:

```tsx
count = 10
```

NO actualiza correctamente el estado en React.

El estado debe actualizarse mediante:

```tsx
setCount(10)
```

Esto se debe a que React necesita saber cuándo ocurrió el cambio para iniciar el proceso de actualización.

### Estado y persistencia entre renders

Las variables normales se reinician en cada render:

```tsx
let count = 0
```

Pero el estado de `useState` persiste entre renderizados.

React almacena internamente ese valor y lo conserva aunque el componente vuelva a ejecutarse.

## Actualizaciones funcionales: `prev => ...`

El setter de `useState` puede recibir no solo un valor nuevo, sino también una función. Esa función recibe el estado anterior (`prev`) y debe devolver el siguiente estado.

`prev => ...` garantiza que el nuevo estado se calcule usando el valor más actualizado disponible.

Ejemplo:

```tsx
const [count, setCount] = useState(0)

setCount(prev => prev + 1)
```

En este caso:

* `prev` representa el valor anterior del estado,
* y el nuevo estado se calcula a partir de él.

### ¿Por qué es importante?

Las actualizaciones de estado en React pueden agruparse y ejecutarse de manera asincrónica. Por eso, depender directamente de la variable actual puede producir resultados incorrectos en ciertas situaciones.

Ejemplo problemático:

```tsx
setCount(count + 1)
setCount(count + 1)
```

Ambas líneas usan el mismo valor de `count`.

En cambio:

```tsx
setCount(prev => prev + 1)
setCount(prev => prev + 1)
```

Cada actualización recibe el estado más reciente.

### Cuándo usarlo

Las actualizaciones funcionales son recomendadas cuando el *nuevo estado depende del estado anterior*, cuando *hay múltiples actualizaciones consecutivas* o cuando *existe posibilidad de concurrencia/asincronía*.

## Estado vs Variable local: la diferencia clave

Una variable local y el estado (`state`) pueden almacenar datos, pero React los trata de manera completamente diferente.

Una variable local existe solo durante la ejecución actual del componente. Cada vez que el componente se vuelve a renderizar, la función se ejecuta nuevamente y las variables locales se recrean desde cero.

Ejemplo:

```tsx
function Counter() {
  let count = 0

  count++

  return <h1>{count}</h1>
}
```

En cada render, `count` vuelve a comenzar en `0`.

### El estado persiste entre renders

El estado creado con `useState` sí se conserva entre renderizados porque React lo almacena internamente.

Ejemplo:

```tsx
function Counter() {
  const [count, setCount] = useState(0)

  return <h1>{count}</h1>
}
```

Aunque el componente vuelva a ejecutarse muchas veces, React mantiene el valor actual de `count`.

### Diferencia fundamental

Cambiar una variable local:

* no persiste entre renders,
* y no provoca re-renderizados.

Cambiar el estado:

* persiste entre renders,
* y le indica a React que debe actualizar la interfaz.

## Manejo de formularios controlados

En React, un formulario controlado es aquel donde los valores de los inputs son controlados completamente por el estado del componente. Esto significa que React se convierte en la “fuente de verdad” de los datos del formulario.

Es decir, el estado del componente es la fuente de verdad del formulario, y los inputs simplemente reflejan ese estado

Ejemplo:

```tsx
import { ChangeEvent, useState } from "react"

function Form() {
  const [name, setName] = useState<string>("")

  function handleChange(
    event: ChangeEvent<HTMLInputElement>
  ) {
    setName(event.target.value)
  }

  return (
    <input
      value={name}
      onChange={handleChange}
    />
  )
}
```

En este caso:

* el valor visible del input proviene del estado (`name`),
* y cada cambio del usuario actualiza ese estado mediante `onChange`.

---

# ¿Qué ocurre internamente?

El flujo normalmente es:

```text
Usuario escribe
↓
Se dispara onChange
↓
Se actualiza el estado
↓
React re-renderiza
↓
El input recibe el nuevo value
```

La interfaz siempre refleja el estado actual del componente.

### ¿Por qué se llaman “controlados”?

Porque el valor del input no queda gestionado por el DOM de forma independiente, sino por React mediante el estado.

React controla el valor, la sincronización y el renderizado del input.

### Ventajas

Los formularios controlados permiten:

* validar datos fácilmente,
* sincronizar UI y estado,
* controlar inputs dinámicamente,
* deshabilitar botones,
* mostrar errores,
* transformar datos,
* y manejar formularios complejos de manera predecible.

## Inmutabilidad: por qué no mutar el estado directamente

En React, el estado debe tratarse como inmutable. Esto significa que no debemos modificar directamente los objetos o arrays almacenados en el estado, sino crear nuevas versiones con los cambios aplicados.

React no analiza profundamente cada objeto para detectar cambios.
En cambio, espera recibir nuevas referencias para saber que el estado cambió y que debe actualizar la interfaz.

Ejemplo incorrecto:

```tsx
type User = {
  name: string
}

function App() {
  const [user, setUser] = useState<User>({
    name: "John"
  })

  function handleChange() {
    user.name = "Jane"
  }

  return <h1>{user.name}</h1>
}
```

Acá el objeto se está modificando directamente.

### ¿Por qué es un problema?

React detecta cambios principalmente mediante referencias.

Cuando hacemos `setUser(newUser)` React compara la referencia anterior, con la referencia nueva.
Si mutamos el mismo objeto `user.name = "Jane"` la referencia sigue siendo la misma.

Entonces React puede:

* no detectar correctamente el cambio,
* no re-renderizar,
* romper optimizaciones,
* y generar bugs difíciles de detectar.

### Forma correcta

Debemos crear un nuevo objeto:

```tsx
function handleChange() {
  setUser({
    ...user,
    name: "Jane"
  })
}
```

Ahora existe una nueva referencia, React detecta el cambio y puede actualizar correctamente la UI.

### Lo mismo aplica a arrays

Incorrecto:

```tsx
items.push("New")
```

Correcto:

```tsx
setItems([
  ...items,
  "New"
])
```

#### Nota
La inmutabilidad en React no solo aplica al objeto o array principal del estado, sino también a cualquier estructura anidada que cambie. Si el estado contiene subobjetos o arrays de objetos, no alcanza con copiar solo el nivel superior: cada nivel modificado necesita una nueva referencia.

Por eso, cuando se actualizan estructuras profundas, normalmente se utilizan spreads (`...`) u otros métodos que crean nuevas copias en cada nivel necesario. Esto permite que React detecte correctamente los cambios mediante comparación de referencias y actualice la interfaz de forma consistente.

## Anatomía completa de un setState

Para entender realmente cómo funciona el estado en React (`useState`), es necesario derribar la ilusión que la biblioteca nos presenta en la superficie. Cuando escribís `const [state, setState] = useState(0)`, parece que estás declarando una variable normal que cambia de valor con el tiempo. **Pero en JavaScript, eso es físicamente imposible.**

Para dominar React tenemos que abrir el motor, mirar los engranajes y entender el flujo desde tres niveles: **JavaScript puro (Contextos de Ejecución), la estructura de datos interna (React Fiber) y el ciclo de vida del renderizado.**


## 1. La paradoja de la variable inmutable

Empecemos con el síntoma más común que confunde a los desarrolladores. Analicemos este componente que maneja un contador de likes:

```typescript
import { useState } from 'react';

function LikeButton() {
  const [likes, setLikes] = useState<number>(0);

  function handleLike(): void {
    console.log('1. Valor inicial en el click:', likes); // Imprime 0

    setLikes(likes + 1);

    console.log('2. Valor inmediatamente después:', likes); // ¡Sigue imprimiendo 0!
  }

  return (
    <button onClick={handleLike}>
      Likes: {likes}
    </button>
  );
}

```

Si le das click al botón por primera vez, verás que ambos `console.log` muestran `0`, pero la pantalla mágicamente se actualiza a `1`. ¿Por qué la variable no cambió si acabamos de ejecutar `setLikes`?

### El secreto: `likes` es una constante, no una variable viva

Cuando React renderiza `LikeButton`, ejecuta la función de arriba a abajo. Al desestructurar el array, obtenés una variable local llamada `likes`.

En JavaScript, los números son **valores primitivos** y se pasan por valor, no por referencia. Además, la declarás con `const`. React no tiene superpoderes para alterar las reglas de JavaScript: no puede forzar a una constante local a mutar su valor en la línea siguiente.

Lo que realmente está ocurriendo es que `likes` está atrapado dentro de una **Closure** (clausura). La función `handleLike` captura el entorno léxico del momento exacto en que fue creada. En ese momento (Render #1), `likes` valía `0`. Por lo tanto, para esa ejecución específica de `handleLike`, `likes` será `0` de principio a fin.

>Una **clausura(Closure)** es la combinación de una función y el entorno léxico (el ámbito/scope) dentro del cual fue creada. En la práctica, esto significa que una función interna siempre va a recordar y tener acceso a las variables de su función externa, incluso después de que la función externa haya terminado de ejecutarse y su contexto haya desaparecido de la pila de ejecución (Call Stack).

## 2. La arquitectura interna: ¿Dónde vive el estado si la función se destruye?

Si las funciones de React se ejecutan de principio a fin y sus variables locales desaparecen de la memoria al terminar, ¿cómo es que React "recuerda" que tus likes iban por el número 5?

La respuesta es **React Fiber**.

Fuera de tus componentes de JavaScript, React mantiene una estructura de datos en memoria llamada el **Árbol Fiber**. Cada componente en tu interfaz tiene un nodo Fiber asignado (un objeto gigante que representa el estado real y persistente del componente).

Cuando llamás a un Hook como `useState`, React busca el nodo Fiber de ese componente. Dentro de ese nodo, hay una propiedad llamada `memoizedState` que almacena una **lista enlazada** (Linked List) con todos los hooks que declaraste, en el orden exacto en que los pusiste.

```
FiberNode (LikeButton)
   │
   └─── memoizedState ───► [ Hook 1 (useState) ] ───► [ Hook 2 (useEffect) ] ───► null

```

Si hacemos un zoom microscópico a la estructura interna de ese objeto `Hook` de la lista enlazada, nos encontramos con esto:

```typescript
interface Hook {
  memoizedState: any;       // El valor del estado que se usó en el último render exitoso (ej: 0)
  baseState: any;           // El estado base para calcular las actualizaciones pendientes
  queue: UpdateQueue | null;// Una cola circular de cambios que solicitaste pero no se han procesado
  next: Hook | null;        // Puntero al siguiente hook en el componente
}

```

### ¿Qué hace realmente `setLikes`?

Cuando ejecutás `setLikes(1)`, no estás modificando el estado. Lo que estás haciendo es despachar un **Update** (un mensaje de actualización) a la cola (`queue`) de ese hook específico dentro de Fiber.

Ese objeto de actualización se ve tipicamente así:

```typescript
interface Update {
  action: any;         // El nuevo valor (1) o la función reductora (prev => prev + 1)
  next: Update | null; // Puntero al siguiente cambio (React agrupa múltiples setStates)
}

```

Entonces, cuando llamás a `setLikes(likes + 1)`, React crea este objeto de actualización, lo encola en la estructura interna de Fiber, y deja tu variable local `likes` completamente intacta. Tu función termina de ejecutarse, el segundo `console.log` lee el `likes` local (que sigue siendo 0) y el hilo principal de JavaScript queda libre.


## 3. El ciclo dinámico: El viaje de Render y Commit

Una vez que tu función manejadora de eventos (`handleLike`) termina de ejecutarse por completo, el Call Stack de JavaScript se vacía. Ahí es cuando React toma el control del escenario para procesar el trabajo pendiente a través de dos fases fundamentales: **Render** y **Commit**.

```
[Click del Usuario]
       │
       ▼
1. Fase de Evento (JS Puro)
   - Se ejecuta handleLike()
   - setLikes() añade la Update a la cola del Hook
   - handleLike() termina (Call Stack vacío)
       │
       ▼
2. Fase de Render (React - Asíncrona)
   - React vuelve a ejecutar LikeButton()
   - useState lee la cola de updates y calcula el nuevo valor
   - Se genera el nuevo árbol de elementos (Virtual DOM)
       │
       ▼
3. Fase de Commit (React - Síncrona)
   - React compara el nuevo Virtual DOM con el viejo
   - Aplica los cambios estrictamente necesarios en el DOM Real del navegador

```

### Detalle de la Fase de Render (El procesamiento de la cola)

React nota que el componente quedó marcado como "sucio" porque tiene actualizaciones pendientes. Entonces, **vuelve a invocar la función `LikeButton()` desde cero**. Es una ejecución completamente nueva e independiente de la anterior.

Cuando la ejecución pasa por la línea del hook:

```typescript
const [likes, setLikes] = useState<number>(0);

```

React va al nodo Fiber, abre la `queue` del hook, ve que hay una actualización pendiente `{ action: 1 }`, toma el `memoizedState` viejo (`0`), le aplica la acción y calcula que el nuevo estado es `1`.

Luego, guarda de forma permanente ese `1` en el `memoizedState` del Fiber y te lo devuelve en la desestructuración. Ahora, en esta nueva ejecución (Render #2), la constante `likes` vale `1`. Tu componente genera un nuevo bloque de código de interfaz (Virtual DOM): `<button>Likes: 1</button>`.

### Detalle de la Fase de Commit (Pintando la pantalla)

React agarra ese nuevo fragmento de interfaz y lo compara con el que generó en el Render #1 (`<button>Likes: 0</button>`) (*Reconciliation*).

Al notar que la única diferencia es el texto interno del botón, React entra a la fase de **Commit** y modifica de forma síncrona el nodo del DOM del navegador utilizando propiedades nativas ultrarrápidas como `textContent`. Ahora, el usuario finalmente ve el número `1` en su monitor.

## 4. El caso avanzado: Funciones actualizadoras (`prev => prev + 1`)

Para terminar analicemos qué pasa cuando mandamos llamadas consecutivas. Si entendiste lo anterior, podrás deducir por qué este código falla en hacer un incremento triple:

```typescript
function handleTripleLike(): void {
  setLikes(likes + 1); // likes es 0 -> encola "quiero que valga 1"
  setLikes(likes + 1); // likes sigue siendo 0 -> encola "quiero que valga 1"
  setLikes(likes + 1); // likes sigue siendo 0 -> encola "quiero que valga 1"
}

```

Al final del evento, la cola del hook tiene tres actualizaciones idénticas: `[1, 1, 1]`. Cuando React corre el Render #2, procesa la cola, pisa los valores uno tras otro y el resultado final es simplemente `1`.

Para solucionar esto, React nos permite pasar una **función actualizadora**:

```typescript
function handleTripleLikeAnatomia(): void {
  setLikes(prev => prev + 1);
  setLikes(prev => prev + 1);
  setLikes(prev => prev + 1);
}

```

¿Por qué este sí funciona y sube de `0` a `3` si `likes` sigue congelado en `0` durante todo el bloque?

Porque en la estructura de datos interna del hook de Fiber, la `queue` ahora almacena funciones puras (instrucciones de cálculo) en lugar de valores estáticos:

```typescript
// La cola interna de actualizaciones en Fiber se ve así:
queue: [
  { action: prev => prev + 1 },
  { action: prev => prev + 1 },
  { action: prev => prev + 1 }
]

```

Cuando el evento termina y React inicia la **Fase de Render**, entra al hook y ejecuta un proceso idéntico a un `.reduce()` de JavaScript sobre esa cola, usando el último estado guardado (`0`) como valor inicial:

1. Agarra el estado base `0`, lo pasa a la primera función: $0 + 1 = \mathbf{1}$.
2. Agarra ese resultado acumulado `1`, lo pasa a la segunda función: $1 + 1 = \mathbf{2}$.
3. Agarra ese resultado acumulado `2`, lo pasa a la tercera función: $2 + 1 = \mathbf{3}$.

Guarda el `3` final en el nodo Fiber y arranca el renderizado devolviéndote un `3` limpio.

### Resumen para llevar en la cabeza

Nunca pienses que `setState` busca tu variable y la modifica en el acto. El modelo mental correcto es:

> **`useState` te da una fotografía instantánea del estado fija para el render actual. Cuando usás el modificador (`setLikes`), no cambiás el presente; estás firmando un formulario de solicitud para que React planifique un render en el futuro con un dato completamente nuevo.**



