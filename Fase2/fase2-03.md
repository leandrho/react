# Hooks de optimización
*Rendimiento consciente*

## Introducción a la optimización en React

Antes de aprender herramientas de optimización hay que entender qué intenta optimizar React.

React funciona mediante un ciclo muy simple:

```text
Estado cambia
↓
React renderiza nuevamente
↓
Compara el resultado anterior con el nuevo
↓
Actualiza el DOM si es necesario
```

Cada vez que cambia un estado (`useState`), un contexto o una prop, React vuelve a ejecutar los componentes afectados para obtener una nueva representación de la UI. A este proceso lo llamamos **re-render**.

### ¿Un re-render es algo malo?

No. Este es probablemente el malentendido más común entre quienes empiezan React.

Muchos desarrolladores ven:

```text
Render #1
Render #2
Render #3
```

y automáticamente piensan: `"Mi aplicación es lenta"`

Pero React fue diseñado precisamente para renderizar componentes con frecuencia.

Un re-render no implica:

* modificar el DOM,
* repintar la pantalla,
* recalcular toda la aplicación.

Significa simplemente que React volvió a ejecutar una función para obtener una nueva descripción de la interfaz.


### Entonces, ¿qué intentan optimizar estas herramientas?

Existen tres costos potenciales:

#### 1. Cálculos costosos

```tsx
const result = expensiveCalculation(data)
```

Si ese cálculo tarda mucho tiempo y se ejecuta en cada render, puede afectar el rendimiento.

#### 2. Re-renderizados innecesarios

```tsx
<ExpensiveTable />
```

A veces un componente recibe exactamente las mismas props pero vuelve a renderizarse porque su padre se renderizó.


#### 3. Creación constante de referencias

```tsx
const handleClick = () => {
  ...
}
```

Cada render crea nuevas funciones y nuevos objetos.

Normalmente esto no es un problema, pero en ciertos escenarios puede provocar renders adicionales.

### La regla más importante

Antes de optimizar, hay que recordar:

> React es rápido por defecto.

La mayoría de las aplicaciones pequeñas y medianas no necesitan:

```tsx
useMemo(...)
useCallback(...)
React.memo(...)
```

Agregar optimizaciones innecesarias suele producir:

* código más complejo,
* bugs más difíciles de detectar,
* dependencias mal configuradas,
* peor mantenibilidad.

### Filosofía moderna de React

El enfoque actual es:

```text
Primero escribir código correcto
↓
Medir
↓
Identificar cuellos de botella reales
↓
Optimizar únicamente donde exista un problema
```

No:

```text
Escribir todo con useMemo
useCallback
React.memo
por las dudas
```

### En resumen

La optimización en React consiste en reducir trabajo innecesario, no en evitar renderizados a toda costa. Un re-render es una parte normal del funcionamiento de React y generalmente no representa un problema. Herramientas como `useRef`, `useMemo`, `useCallback` y `React.memo` existen para casos específicos donde el costo de ciertos cálculos, referencias o renderizados realmente afecta el rendimiento de la aplicación.

---

## `useRef`: referencias más allá del DOM

`useRef` es un Hook que permite almacenar un valor que **persiste entre renderizados sin provocar nuevos renderizados cuando cambia**.

Su sintaxis es:

```tsx
import { useRef } from "react"

function App() {
  const countRef = useRef(0)

  return <div>Hello</div>
}
```

A diferencia de `useState`, modificar un `ref` no hace que React vuelva a renderizar el componente.

### Modelo mental

Si `useState` representa información que afecta a la interfaz, `useRef` representa información que necesita sobrevivir entre renders pero que no requiere actualizar la UI.

```text
useState
↓
Cambia
↓
React renderiza nuevamente

useRef
↓
Cambia
↓
No hay render
```

Por eso suele decirse que un ref es una especie de "caja mutable" que React conserva entre renderizados.

### Anatomía de un ref

```tsx
const countRef = useRef(0)
```

React devuelve un objeto con una propiedad llamada `current`:

```ts
{
  current: 0
}
```

El valor se lee y modifica mediante `current`:

```tsx
countRef.current = countRef.current + 1
```

### Estado vs Ref

Supongamos:

```tsx
import { useRef, useState } from "react"

function App() {
  const [count, setCount] = useState(0)

  const countRef = useRef(0)

  return (
    <>
      <button
        onClick={ () => setCount(prev => prev + 1) }
      >
        State: {count}
      </button>

      <button
        onClick={() => {
          countRef.current += 1
          console.log(countRef.current)
        }}
      >
        Ref
      </button>
    </>
  )
}
```

Al pulsar:

```tsx
setCount(...)
```

React renderiza nuevamente.

Al modificar:

```tsx
countRef.current
```

React no renderiza nada.

### ¿Por qué existe useRef?

Porque hay datos que deben sobrevivir entre renders pero no forman parte de la interfaz.

Por ejemplo:

* IDs de timers.
* Referencias a elementos DOM.
* WebSockets.
* Instancias de librerías.
* Valores previos.
* Contadores internos.
* Flags de control.

### Caso clásico: acceder al DOM

Históricamente, este es el uso más conocido.

```tsx
import { useEffect, useRef } from "react"

function App() {
  const inputRef = useRef<HTMLInputElement>(null)

  useEffect(() => {
    inputRef.current?.focus()
  }, [])

  return (
    <input ref={inputRef} />
  )
}
```

Flujo:

```text
Render
↓
React crea el input
↓
React asigna el elemento a ref.current
↓
useEffect
↓
focus()
```

### Guardar un valor entre renders

También puede utilizarse sin relación con el DOM.

```tsx
import { useRef, useState } from "react"

function App() {
  const renderCount = useRef(0)

  const [count, setCount] = useState(0)

  renderCount.current += 1

  return (
    <>
      <p>Renders: {renderCount.current}</p>

      <button
        onClick={() => setCount(prev => prev + 1) }
      >
        {count}
      </button>
    </>
  )
}
```

El valor de `renderCount.current` persiste entre renders sin ser estado.


### Guardar IDs de timers

Un caso extremadamente común:

```tsx
import { useEffect, useRef } from "react"

function App() {
  const intervalRef = useRef<number | null>(null)

  useEffect(() => {
    intervalRef.current =
      window.setInterval(() => {
        console.log("tick")
      }, 1000)

    return () => {
      if (
        intervalRef.current !== null
      ) {
        clearInterval(
          intervalRef.current
        )
      }
    }
  }, [])

  return null
}
```


### useRef y Stale Closures

Recordemos este problema:

```tsx
useEffect(() => {
  const intervalId = setInterval(() => {
    console.log(count)
  }, 1000)

  return () => clearInterval(intervalId)
}, [])
```

La closure puede quedarse con valores antiguos.

En algunos escenarios avanzados se utiliza un ref para mantener acceso al valor más reciente:

```tsx
const countRef = useRef(count)

countRef.current = count
```

y luego:

```tsx
console.log(countRef.current)
```

Así el callback puede leer siempre el valor actualizado sin recrear el efecto.

### En resumen

`useRef` permite almacenar valores mutables que persisten entre renderizados sin provocar nuevos renders. Puede utilizarse para acceder a elementos del DOM, guardar referencias a recursos externos o conservar información interna del componente que no forma parte de la interfaz. Si un cambio debe reflejarse en la UI, debe utilizarse estado (`useState`); si no necesita renderizar, un ref suele ser la herramienta adecuada.

---

## `useMemo`: memoizar cálculos costosos

`useMemo` es un Hook que permite **memorizar (cachear) el resultado de un cálculo entre renderizados** para evitar volver a ejecutarlo cuando sus dependencias no cambiaron.

Su sintaxis es:

```tsx
const value = useMemo(() => {
  return expensiveCalculation(data)
}, [data])
```

La idea es simple:

```text
Primer render
↓
Ejecuta el cálculo
↓
Guarda el resultado

Render siguiente
↓
Las dependencias no cambiaron
↓
Reutiliza el resultado anterior
```

### El problema que intenta resolver

Recordemos que cada render vuelve a ejecutar la función del componente:

```tsx
function App() {
  const result = expensiveCalculation()

  return <div>{result}</div>
}
```

Si el componente renderiza diez veces:

```text
Render #1 → cálculo
Render #2 → cálculo
Render #3 → cálculo
...
```

el cálculo se ejecuta en cada render aunque el resultado sea siempre el mismo.

### Ejemplo básico

Sin `useMemo`:

```tsx
function ProductList({ products }: { products: Product[]}) {
  const sortedProducts =
    [...products].sort(
      (a, b) => a.price - b.price
    )

  return ...
}
```

Cada render vuelve a ordenar la lista.

Con `useMemo`:

```tsx
import { useMemo } from "react"

function ProductList({ products }: { products: Product[] }) {
  const sortedProducts =
    useMemo(() => {
      return [...products].sort(
        (a, b) => a.price - b.price
      )
    }, [products])

  return ...
}
```

Ahora el ordenamiento solo ocurre cuando cambia `products`.

### Cómo funciona internamente

Conceptualmente React hace algo parecido a:

```text
Render #1
↓
Ejecuta callback
↓
Guarda resultado
↓
Guarda dependencias

Render #2
↓
Compara dependencias
↓
No cambiaron
↓
Devuelve resultado guardado
```

Si alguna dependencia cambia:

```text
Render #3
↓
Dependencia cambió
↓
Ejecuta callback nuevamente
↓
Actualiza cache
```

### useMemo NO evita renders

Este es uno de los errores más comunes.

Mucha gente piensa:

```tsx
useMemo(...)
```

↓

```text
"Mi componente ya no renderiza"
```

Eso es falso.

El componente sigue renderizando normalmente.

Lo único que evita `useMemo` es volver a ejecutar un cálculo específico.

```text
Render
↓
Componente ejecutado
↓
useMemo reutiliza resultado
```

### ¿Qué debería ir dentro de useMemo?

Principalmente cálculos costosos.

Por ejemplo:

* filtros grandes,
* ordenamientos,
* agrupaciones,
* transformaciones complejas,
* cálculos matemáticos pesados.

Ejemplo:

```tsx
const filteredUsers =
  useMemo(() => {
    return users.filter(...)
  }, [users])
```

---

Ejemplo innecesario:

```tsx
const isAdmin = useMemo(() => {
  return role === "admin"
}, [role])
```

Más simple:

```tsx
const isAdmin =
  role === "admin"
```

### useMemo también memoiza referencias

Además de cálculos costosos, puede utilizarse para mantener estable una referencia.

Sin memoización:

```tsx
const options = {
  sortBy: "name"
}
```

Cada render crea un objeto nuevo.

Con memoización:

```tsx
const options = useMemo(() => {
  return {
    sortBy: "name"
  }
}, [])
```

La referencia permanece estable.

Esto suele aparecer combinado con:

* `React.memo`
* `useEffect`
* `useCallback`

aunque veremos esos casos más adelante.

### El costo de useMemo

`useMemo` no es gratuito.

React debe:

```text
Guardar dependencias
Compararlas
Mantener una cache
Gestionar memoria
```

Por eso:

> Memoizar algo barato puede costar más que recalcularlo.

---

# Regla práctica

Antes de usar `useMemo`, preguntate:

1. ¿El cálculo es realmente costoso?
2. ¿Se ejecuta frecuentemente?
3. ¿Existe un problema de rendimiento medible?

Si alguna respuesta es "no", probablemente no necesites `useMemo`.

Pensá en `useMemo` como una cache:


### En resumen

`useMemo` permite memorizar el resultado de un cálculo entre renderizados y volver a ejecutarlo únicamente cuando cambian sus dependencias. Su objetivo principal es evitar trabajo computacional innecesario en cálculos costosos o mantener referencias estables cuando realmente se necesita. No evita renderizados y no debe utilizarse de forma preventiva o indiscriminada.

---

## `useCallback`: referencias estables de funciones

`useCallback` es un Hook que permite **memorizar una función entre renderizados**, devolviendo la misma referencia mientras sus dependencias no cambien.

Su sintaxis es:

```tsx
const handleClick = useCallback(() => {
  console.log("click")
}, [])
```

Conceptualmente funciona de manera muy similar a `useMemo`:

```text
Primer render
↓
React guarda la función

Render siguiente
↓
Dependencias iguales
↓
React devuelve la misma función

Render siguiente
↓
Dependencias cambiaron
↓
React crea una nueva función
```

### El problema que intenta resolver

Recordemos que cada render ejecuta nuevamente la función del componente.

Por lo tanto:

```tsx
function App() {
  const handleClick = () => {
    console.log("click")
  }

  return null
}
```

En cada render se crea una función nueva.

```text
Render #1
↓
handleClick → referencia A

Render #2
↓
handleClick → referencia B

Render #3
↓
handleClick → referencia C
```

Aunque el código sea idéntico, las referencias son distintas.

### ¿Eso es un problema?

Normalmente No. La enorme mayoría de las funciones creadas durante un render no generan ningún inconveniente.

Por eso es importante entender:

> Crear funciones nuevas en React es completamente normal.


### Entonces, ¿para qué existe useCallback?

Principalmente para mantener una referencia estable cuando esa referencia es utilizada por otra optimización.

Por ejemplo:

```tsx
const handleClick = useCallback(() => {
  ...
}, [])
```

permite que React reutilice la misma función entre renders.

### Comparación con useMemo

Muchos desarrolladores los confunden.

```tsx
const value = useMemo(() => {
  return expensiveCalculation()
}, [])
```

Memoiza:

```text
El resultado de un cálculo
```

---

```tsx
const handleClick = useCallback(() => {
  ...
}, [])
```

Memoiza:

```text
La referencia de una función
```

De hecho, conceptualmente:

```tsx 
useCallback(fn, deps)
```

es muy parecido a:

```tsx 
useMemo(() => fn, deps)
```

### Caso real: Dependencias de efectos

Supongamos:

```tsx
function App() {
  const fetchUsers = () => {
    ...
  }

  useEffect(() => {
    fetchUsers()
  }, [fetchUsers])
}
```

Problema:

```text
Render
↓
Nueva función
↓
Dependencia cambió
↓
Effect se ejecuta otra vez
```

En algunos escenarios se utiliza:

```tsx
const fetchUsers =
  useCallback(() => {
    ...
  }, [])
```

para estabilizar la referencia.

Aunque muchas veces la mejor solución es reestructurar el efecto.

### useCallback NO evita renders

Este es probablemente el error más frecuente.

Mucha gente piensa:

```tsx
const handleClick =
  useCallback(...)
```

↓

```text
"Ahora mi componente ya no renderiza"
```

Eso es falso.

El componente sigue renderizando normalmente.

Lo único que React reutiliza es la referencia de la función.

```text
Render
↓
Componente ejecutado
↓
useCallback devuelve función guardada
```

### El costo de useCallback

Al igual que `useMemo`, no es gratuito.

React debe:

```text
Guardar dependencias
Compararlas
Mantener referencias
```

Por eso:

> Usar useCallback en todas las funciones suele empeorar la legibilidad sin aportar beneficios reales.


### Cuándo suele ser útil

Generalmente aparece junto a `React.memo`, `useEffect`, `Custom Hooks` donde la estabilidad de referencias sí puede influir en el comportamiento o el rendimiento.

### En resumen

`useCallback` permite memorizar funciones entre renderizados para conservar la misma referencia mientras sus dependencias no cambien. Su utilidad principal aparece cuando otras optimizaciones o efectos dependen de la identidad de esa función. No evita renderizados y no debería utilizarse de forma preventiva en todas las funciones de un componente.

---

## `React.memo`: evitar re-renderizados innecesarios

`React.memo` es una función de orden superior (*Higher-Order Component*) que permite a React **reutilizar el resultado de un componente cuando sus props no cambiaron**.

Su sintaxis es:

```tsx
import { memo } from "react"

type ButtonProps = {
  label: string
}

const Button = memo(
  function Button({
    label
  }: ButtonProps) {
    console.log("Button render")

    return <button>{label}</button>
  }
)
```

### El problema que intenta resolver

Recordemos que cuando un componente se renderiza, React también renderiza a sus hijos.

Ejemplo:

```tsx
function App() {
  const [count, setCount] =
    useState(0)

  return (
    <>
      <Child />

      <button
        onClick={() =>
          setCount(prev => prev + 1)
        }
      >
        {count}
      </button>
    </>
  )
}
```

```tsx
function Child() {
  console.log("Child render")

  return <h1>Child</h1>
}
```

Al cambiar `count`:

```text
App render
↓
Child render
```

aunque `Child` no use `count`.

### ¿Es un problema?

Normalmente No. React está diseñado para que los re-renderizados sean baratos.

La mayoría de los componentes pueden renderizar cientos o miles de veces sin inconvenientes.

### ¿Qué hace React.memo?

`React.memo` le dice a React:

> Si las props son iguales a las del render anterior, reutilizá el resultado anterior y no vuelvas a ejecutar el componente.

```tsx
const Child = memo(
  function Child() {
    console.log("Child render")

    return <h1>Child</h1>
  }
)
```

Ahora:

```text
App render
↓
Props de Child iguales
↓
React reutiliza resultado anterior
↓
Child no renderiza
```

### Cómo decide React si las props cambiaron

Por defecto React realiza una comparación superficial (*shallow comparison*).

Ejemplo:

```tsx
<Profile
  name="John"
  age={30}
/>
```

Comparación conceptual:

```text
name viejo === name nuevo
age viejo === age nuevo
```

Si todas las props son iguales:

```text
React reutiliza el render anterior
```

### El problema de las referencias

Veamos:

```tsx 
const Child = memo(
  function Child({ onClick }: { onClick: () => void }) {

    console.log("Child render")

    return (
      <button onClick={onClick}>
        Click
      </button>
    )
  }
)
```

Padre:

```tsx
function App() {
  const [count, setCount] =  useState(0)

  const handleClick = () => {
    console.log("clicked")
  }

  return (
    <>
      <Child
        onClick={handleClick}
      />

      <button
        onClick={() =>
          setCount(prev => prev + 1)
        }
      >
        {count}
      </button>
    </>
  )
}
```

Cada render crea:

```text
Nueva función
↓
Nueva referencia
↓
Prop distinta
↓
React.memo falla
↓
Child renderiza
```

Por eso suele combinarse con:

```tsx
const handleClick =
  useCallback(() => {
    console.log("clicked")
  }, [])
```

Ahora la referencia permanece estable.

### useMemo y React.memo

Otro caso frecuente:

```tsx
const options = {
  sortBy: "name"
}
```

Cada render crea:

```text
Nuevo objeto
↓
Nueva referencia
↓
Prop distinta
↓
React.memo falla
```

Muchas veces se utiliza:

```tsx id="r2f32h"
const options = useMemo(() => {
  return {
    sortBy: "name"
  }
}, [])
```

para estabilizar la referencia.

### React.memo NO impide todos los renders

Si el componente tiene estado propio:

```tsx
const Counter = memo(
  function Counter() {
    const [count, setCount] = useState(0)

    ...
  }
)
```

y ese estado cambia:

```text
Estado cambia
↓
Counter renderiza
```

`React.memo` solo compara props.

No bloquea:

* cambios de estado,
* cambios de contexto,
* renders provocados por el propio componente.

### Comparación personalizada

Opcionalmente puede pasarse una función de comparación:

```tsx
const UserCard = memo(
  UserCardComponent,
  (prevProps, nextProps) => {
    return (
      prevProps.id === nextProps.id
    )
  }
)
```

Sin embargo, esto es relativamente avanzado y suele utilizarse poco.

### El costo de React.memo

`React.memo` tampoco es gratuito.

React debe Guardar props anteriores, Comparar props, Decidir si renderizar

Por eso:

> Si el componente es pequeño y barato de renderizar, React.memo puede aportar más complejidad que beneficios.

### Cuándo suele ser útil

Generalmente cuando se cumplen dos condiciones:

#### El componente renderiza frecuentemente

```text
Tablas
Listas grandes
Gráficos
Dashboards
```

#### El render es costoso

```text
Muchos elementos
Muchos cálculos
Jerarquías profundas
```

### Relación entre useCallback, useMemo y React.memo

Estos tres conceptos suelen trabajar juntos:

```text
React.memo
↓
Evita renderizar un componente

useCallback
↓
Mantiene estables funciones

useMemo
↓
Mantiene estables valores y objetos
```

Por separado muchas veces no aportan nada.

Juntos pueden evitar renders innecesarios cuando realmente existe un problema de rendimiento.

### En resumen

`React.memo` permite que React reutilice el resultado de un componente cuando sus props no cambiaron. Su objetivo es evitar renderizados innecesarios en componentes costosos o que reciben las mismas props con frecuencia. Funciona mediante una comparación superficial de props y suele combinarse con `useCallback` y `useMemo` para mantener estables las referencias que participan en esa comparación.

---

## Cuándo optimizar (y cuándo es prematuro)

Después de aprender `useMemo`, `useCallback` y `React.memo`, es muy común caer en una trampa:

```tsx
const value = useMemo(...)
const handler = useCallback(...)
const Child = memo(...)
```

↓

```text
"Así mi aplicación será más rápida"
```

En realidad, muchas veces ocurre lo contrario.

### La optimización prematura

Existe una frase muy conocida en ingeniería de software:

> "Premature optimization is the root of all evil."

La idea no es que optimizar sea malo, sino que optimizar antes de tener un problema real suele generar más costos que beneficios.

Por ejemplo:

```tsx
const fullName = useMemo(() => {
  return `${firstName} ${lastName}`
}, [firstName, lastName])
```

o:

```tsx
const handleClick = useCallback(() => {
  console.log("click")
}, [])
```

En la mayoría de los casos esto no aporta ninguna mejora perceptible.

Sin embargo sí agrega:

* más código,
* más dependencias,
* más complejidad mental,
* más posibilidades de errores.

### React ya está optimizado

Un punto importante:

> React fue diseñado para renderizar con frecuencia.

Cuando un componente renderiza:

```text
Render
↓
Nueva descripción de UI
↓
Diffing
↓
Actualización mínima del DOM
```

El simple hecho de que un componente se renderice no significa que el navegador esté haciendo trabajo costoso.

Por eso:

```text
Re-render ≠ problema
```

### Qué sí debería preocuparnos

Normalmente optimizamos cuando existe alguno de estos síntomas:

#### Cálculos costosos

```tsx
const result =
  expensiveCalculation(data)
```

que tardan tiempo perceptible.

#### Componentes pesados

```tsx
<BigTable />
<LargeChart />
<Map />
```

que renderizan cientos o miles de elementos.


#### Re-renderizados excesivos

Cuando una interacción sencilla provoca que grandes partes de la aplicación vuelvan a renderizarse innecesariamente.

#### Problemas visibles para el usuario

Por ejemplo:

```text
Scroll entrecortado
Inputs lentos
Animaciones con saltos
Pantallas que se congelan
```

Si el usuario no percibe ningún problema, probablemente no haya nada que optimizar.

### React DevTools Profiler

Cuando realmente sospechamos un problema de rendimiento, la herramienta principal es:

[React DevTools Profiler](https://react.dev/learn/react-developer-tools?utm_source=chatgpt.com)

Permite ver:

* qué componentes renderizan,
* cuántas veces renderizan,
* cuánto tarda cada render,
* qué actualizaciones provocaron esos renders.

La regla es simple:

> No adivinar. Medir.

### El costo oculto de optimizar

Toda optimización tiene un costo.

Por ejemplo:

```tsx
useMemo(...)
```

implica:

```text
Guardar dependencias
Compararlas
Mantener una cache
```

---

```tsx
useCallback(...)
```

implica:

```text
Guardar referencias
Comparar dependencias
```

---

```tsx
React.memo(...)
```

implica:

```text
Comparar props
Guardar render previo
```

Por eso:

> Optimizar algo barato puede terminar siendo más caro que recalcularlo.

### Regla práctica

Antes de usar una herramienta de optimización preguntate:

1. ¿Existe un problema de rendimiento observable?
2. ¿Puedo medir ese problema?
3. ¿Sé exactamente qué trabajo estoy intentando evitar?

Si alguna respuesta es "no", probablemente todavía no sea momento de optimizar.

### En resumen

Las herramientas de optimización existen para resolver problemas concretos de rendimiento, no para usarse por defecto. Un re-render no es necesariamente costoso y React ya realiza muchas optimizaciones internamente. La mejor estrategia suele ser escribir código simple y correcto, medir cuando aparezcan problemas reales y optimizar únicamente las partes de la aplicación que demuestren ser un cuello de botella.


---

## React Compiler: el futuro de la optimización

Durante años, React ofreció herramientas como:

```tsx
useMemo(...)
useCallback(...)
React.memo(...)
```

para evitar cálculos o renderizados innecesarios.

El problema es que estas optimizaciones son **manuales**. El desarrollador debe identificar el problema, entender qué está ocurriendo y decidir dónde aplicar cada herramienta.

Por ejemplo:

```tsx
const filteredUsers = useMemo(() => {
  return users.filter(user => user.active)
}, [users])
```

o:

```tsx
const handleClick = useCallback(() => {
  saveUser()
}, [])
```

El código funciona, pero ahora el desarrollador también debe preocuparse por dependencias, referencias y memoización.


### El problema de las optimizaciones manuales

En la práctica ocurren dos situaciones muy comunes:

#### Se optimiza demasiado

```tsx
const fullName = useMemo(() => {
  return `${firstName} ${lastName}`
}, [firstName, lastName])
```

No aporta ningún beneficio real.

#### Se optimiza demasiado poco

```tsx
const expensiveData =
  processLargeDataset(data)
```

y nadie detecta el problema hasta que la aplicación empieza a volverse lenta.

### La idea detrás de React Compiler

React Compiler intenta mover gran parte de este trabajo desde el desarrollador hacia una herramienta de compilación.

La idea es:

```text
Código React
↓
React Compiler analiza el código
↓
Detecta optimizaciones seguras
↓
Genera código optimizado
```

de forma automática.

### Un ejemplo conceptual

Supongamos:

```tsx
function UserList({
  users
}: {
  users: User[]
}) {
  const activeUsers =
    users.filter(user => user.active)

  return ...
}
```

Hoy un desarrollador podría pensar:

```tsx
const activeUsers = useMemo(() => {
  return users.filter(
    user => user.active
  )
}, [users])
```

Con React Compiler, el objetivo es que el compilador pueda detectar automáticamente cuándo ese cálculo puede reutilizarse sin que el desarrollador tenga que escribir `useMemo`.

### ¿Qué intenta optimizar?

Principalmente:

#### Cálculos repetidos

```tsx
const result =
  expensiveCalculation(data)
```
#### Referencias estables

```tsx
const handleClick = () => {
  ...
}
```
#### Re-renderizados evitables

Casos donde React puede demostrar que una parte de la UI no necesita recalcularse.

### ¿Significa que useMemo desaparece?

No. Al menos por ahora.

Las APIs siguen existiendo y continúan siendo válidas.

Lo que cambia es que muchas optimizaciones que antes eran manuales podrían dejar de ser necesarias.

### La filosofía que React viene promoviendo

Durante varios años el equipo de React ha insistido en una idea:

> Es mejor escribir código simple y correcto que llenar la aplicación de optimizaciones preventivas.

Por ejemplo:

```tsx
const fullName =
  `${firstName} ${lastName}`
```

es preferible a:

```tsx
const fullName = useMemo(() => {
  return `${firstName} ${lastName}`
}, [firstName, lastName])
```

si no existe un problema real de rendimiento.

React Compiler lleva esa filosofía un paso más allá.

### Qué cambia para el desarrollador

Con el enfoque tradicional:

```text
Escribir código
↓
Detectar posibles optimizaciones
↓
Agregar useMemo
↓
Agregar useCallback
↓
Mantener dependencias
```

Con React Compiler:

```text
Escribir código simple
↓
El compilador analiza
↓
Aplica optimizaciones seguras
```

### ¿Entonces ya no necesito entender useMemo o useCallback?

Sí necesitás entenderlos.

Por varias razones:

1. Muchísimas aplicaciones seguirán utilizándolos durante años.
2. No todos los proyectos usarán React Compiler.
3. Comprenderlos ayuda a entender cómo funciona el renderizado de React.
4. Existen casos avanzados donde las optimizaciones manuales seguirán siendo útiles.

### Relación con todo lo que vimos

Podemos pensar la evolución así:

```text
React básico
↓
Renderiza libremente

Optimización manual
↓
useMemo
useCallback
React.memo

Optimización automática
↓
React Compiler
```

La dirección que está tomando React es reducir la necesidad de optimizaciones manuales y permitir que los desarrolladores se concentren más en la lógica de negocio y menos en detalles de rendimiento.

### En resumen

`React Compiler` es una tecnología que busca automatizar gran parte de las optimizaciones que históricamente se realizaban con `useMemo`, `useCallback` y `React.memo`. Su objetivo es permitir que los desarrolladores escriban código más simple y declarativo mientras el compilador identifica y aplica optimizaciones seguras durante la compilación. Esto no elimina la importancia de comprender las herramientas actuales, pero sí cambia la dirección hacia la que evoluciona React.


---

# Refs avanzadas

## `forwardRef`

Hasta ahora vimos que un `ref` puede apuntar a un elemento del DOM:

```tsx
function App() {
  const inputRef = useRef<HTMLInputElement>(null)

  return (
    <input ref={inputRef} />
  )
}
```

Esto funciona porque `input` es un elemento nativo del DOM y React sabe dónde asignar el `ref`.

Sin embargo, aparece un problema cuando intentamos hacer lo mismo con nuestros propios componentes.

### El problema

Supongamos:

```tsx
type InputProps = {
  label: string
}

function Input({
  label
}: InputProps) {
  return (
    <>
      <label>{label}</label>
      <input />
    </>
  )
}
```

Y luego:

```tsx
function App() {
  const inputRef = useRef<HTMLInputElement>(null)

  return (
    <Input
      ref={inputRef}
      label="Email"
    />
  )
}
```

Esto no funcionaba en React 18 y versiones anteriores.

¿Por qué?

Porque el `ref` no se enviaba automáticamente al elemento `<input>` interno.

React veía:

```text
App
↓
Input
↓
input
```

y el `ref` quedaba detenido en el componente `Input`.

### La solución: forwardRef

`forwardRef` permitía recibir el `ref` y reenviarlo manualmente al elemento deseado.

```tsx
import {
  forwardRef
} from "react"

type InputProps = {
  label: string
}

const Input = forwardRef<
  HTMLInputElement,
  InputProps
>(function Input(
  { label },
  ref
) {
  return (
    <>
      <label>{label}</label>

      <input ref={ref} />
    </>
  )
})
```

Ahora:

```tsx
function App() {
  const inputRef = useRef<HTMLInputElement>(null)

  return (
    <Input
      ref={inputRef}
      label="Email"
    />
  )
}
```

y `inputRef.current` apuntará al `<input>` real.

### ¿Por qué existe?

Los componentes suelen encapsular elementos internos.

Por ejemplo:

```tsx
<Input />
<DatePicker />
<SearchBox />
<Modal />
```

Pero a veces el componente padre necesita acceder a algún elemento interno para hacer foco, medir tamaño, controlar scroll, integrarse con librerías externas.

`forwardRef` permitía exponer esa referencia sin romper la encapsulación del componente.

### Ejemplo clásico: focus

```tsx
import {
  forwardRef,
  useRef
} from "react"

type InputProps = {
  label: string
}

const Input = forwardRef<
  HTMLInputElement,
  InputProps
>(function Input(
  { label },
  ref
) {
  return (
    <>
      <label>{label}</label>

      <input ref={ref} />
    </>
  )
})

function App() {
  const inputRef = useRef<HTMLInputElement>(null)

  return (
    <>
      <Input
        ref={inputRef}
        label="Email"
      />

      <button
        onClick={() =>
          inputRef.current?.focus()
        }
      >
        Focus
      </button>
    </>
  )
}
```

El padre no conoce el `<input>` interno, pero puede interactuar con él mediante el `ref`.

---

### ¿forwardRef comparte datos?

No. No comparte estado. No comparte props. No comparte lógica.

Únicamente comparte una referencia.

```text
Padre
↓
ref
↓
Elemento interno
```

### Limitaciones

Un detalle importante `inputRef.current` expone el elemento completo.

Eso significa que el componente padre puede hacer:

```tsx
inputRef.current?.focus()
inputRef.current?.blur()
inputRef.current?.select()
```

e incluso manipular directamente propiedades del DOM.

A veces esto es deseable.

A veces expone más de lo que realmente queremos mostrar.

Ese problema es precisamente lo que intenta resolver:

```tsx
useImperativeHandle()
```

que veremos a continuación.

---

### React 19

Es importante entender que `forwardRef` fue la solución tradicional en React 18 y versiones anteriores.

En React 19, los componentes pueden recibir `ref` como una prop normal, eliminando gran parte de la necesidad de utilizar `forwardRef`.

Por eso hoy se considera importante conocerlo porque existe muchísimo código que lo utiliza, pero el enfoque moderno apunta hacia un modelo más simple que veremos después.

### En resumen

`forwardRef` permite reenviar una referencia recibida por un componente hacia uno de sus elementos internos. Su principal utilidad es exponer elementos del DOM encapsulados dentro de componentes reutilizables, permitiendo operaciones como foco, medición o integración con librerías externas. En React 19 su importancia disminuye porque `ref` puede recibirse como una prop común, pero sigue siendo fundamental para comprender gran parte del ecosistema existente.

---

## `useImperativeHandle`: controlar qué expone un `ref`

`useImperativeHandle` permite personalizar el valor que un componente expone a través de un `ref`.

Normalmente, cuando un `ref` apunta a un elemento DOM:

```tsx
const inputRef = useRef<HTMLInputElement>(null)
```

el padre obtiene acceso al elemento completo:

```tsx
inputRef.current?.focus()
inputRef.current?.blur()
inputRef.current?.select()
```

Es decir:

```text
Padre
↓
Elemento DOM completo
```

A veces eso está bien.

Pero otras veces queremos exponer solamente una pequeña API pública y ocultar los detalles internos del componente.

Ahí aparece `useImperativeHandle`.


### El problema

Supongamos un componente:

```tsx
function Modal() {
  ...
}
```

Internamente puede tener:

```tsx
useState(...)
useEffect(...)
refs
handlers
```

Todo eso debería ser privado.

El componente padre quizás solo necesita:

```tsx
modalRef.current?.open()
modalRef.current?.close()
```

No debería conocer cómo está implementado el modal.

### La solución

`useImperativeHandle` permite definir exactamente qué recibe el padre.

Conceptualmente:

```tsx
useImperativeHandle(ref, () => ({
  open() {
    ...
  },

  close() {
    ...
  }
}))
```

El objeto retornado se convierte en el valor de:

```tsx
ref.current
```

### Ejemplo

```tsx
import {
  useImperativeHandle,
  useRef,
  useState
} from "react"

type ModalHandle = {
  open: () => void
  close: () => void
}

type ModalProps = {
  ref?: React.Ref<ModalHandle>
}

function Modal({
  ref
}: ModalProps) {
  const [isOpen, setIsOpen] =
    useState(false)

  useImperativeHandle(ref, () => ({
    open() {
      setIsOpen(true)
    },

    close() {
      setIsOpen(false)
    }
  }))

  return (
    isOpen && <div>Modal</div>
  )
}
```

Uso:

```tsx
function App() {
  const modalRef =
    useRef<ModalHandle>(null)

  return (
    <>
      <Modal ref={modalRef} />

      <button
        onClick={() =>
          modalRef.current?.open()
        }
      >
        Open
      </button>
    </>
  )
}
```

### ¿Qué recibe el padre?

No recibe:

```tsx
{
  isOpen,
  setIsOpen
}
```

ni tampoco:

```tsx
<div />
```

Recibe únicamente:

```tsx
{
  open(),
  close()
}
```

porque eso es exactamente lo que decidimos exponer.

### Encapsulación

La principal ventaja es la encapsulación.

Sin `useImperativeHandle`:

```text
Padre
↓
Acceso completo
↓
Implementación interna
```

Con `useImperativeHandle`:

```text
Padre
↓
API pública controlada
↓
Implementación privada
```

Es el mismo principio que usamos al diseñar componentes o clases:

> Exponer únicamente lo necesario.

### ¿Es declarativo o imperativo?

React favorece un modelo declarativo.

Por ejemplo:

```tsx
<Modal isOpen={isOpen} />
```

donde el estado controla la UI.

Sin embargo, algunos problemas son más naturales de resolver de forma imperativa:

```tsx
modalRef.current?.open()
videoRef.current?.play()
editorRef.current?.focus()
```

`useImperativeHandle` existe para esos casos excepcionales.

### Cuándo usarlo

Suele tener sentido en componentes reutilizables que exponen acciones concretas:

```text
Modal
Dialog
Drawer
DatePicker
Editor de texto
Video Player
Canvas
Mapas
```

### Cuándo NO usarlo

No debería convertirse en la forma principal de comunicación entre componentes.

Generalmente es mejor:

```tsx
<Modal isOpen={isOpen} />
```

que:

```tsx
modalRef.current?.open()
```

si el problema puede resolverse mediante props y estado.

### En resumen

`useImperativeHandle` permite controlar exactamente qué valor recibe un componente padre a través de un `ref`. Su principal objetivo es exponer una API pública pequeña y bien definida, ocultando los detalles internos del componente. Aunque React favorece enfoques declarativos basados en props y estado, `useImperativeHandle` resulta útil cuando es necesario ofrecer acciones imperativas como `focus`, `open`, `close` o `reset`.


---

## React 19: `ref` como prop común

Hasta React 18, `ref` era una prop especial que React trataba de manera distinta al resto. Los componentes funcionales no podían recibirla directamente.

Por ejemplo, esto no funcionaba:

```tsx
function Input() {
  return <input />
}

function App() {
  const inputRef =
    useRef<HTMLInputElement>(null)

  return (
    <Input ref={inputRef} />
  )
}
```

Para que funcionara era necesario utilizar `forwardRef`.

### El cambio en React 19

En React 19, `ref` pasa a comportarse como una prop normal.

Ahora un componente puede recibirla directamente:

```tsx
import { Ref } from "react"

type InputProps = {
  ref?: Ref<HTMLInputElement>
}

function Input({
  ref
}: InputProps) {
  return <input ref={ref} />
}
```

Y utilizarse así:

```tsx
function App() {
  const inputRef =
    useRef<HTMLInputElement>(null)

  return (
    <Input ref={inputRef} />
  )
}
```

Sin `forwardRef`.

### ¿Qué problema resuelve?

Principalmente elimina una capa de complejidad.

Antes:

```text
Padre
↓
ref
↓
forwardRef
↓
Componente
↓
Elemento interno
```

Ahora:

```text
Padre
↓
ref
↓
Componente
↓
Elemento interno
```

La intención es hacer que trabajar con refs sea tan natural como trabajar con cualquier otra prop.

### Relación con useImperativeHandle

Nada cambia aquí.

Si queremos exponer una API personalizada seguimos utilizando:

```tsx
useImperativeHandle(...)
```

Por ejemplo:

```tsx
import {
  Ref,
  useImperativeHandle,
  useRef
} from "react"

type ModalHandle = {
  open: () => void
}

type ModalProps = {
  ref?: Ref<ModalHandle>
}

function Modal({
  ref
}: ModalProps) {
  const dialogRef =
    useRef<HTMLDialogElement>(null)

  useImperativeHandle(ref, () => ({
    open() {
      dialogRef.current?.showModal()
    }
  }))

  return <dialog ref={dialogRef} />
}
```

La diferencia es que ya no necesitamos `forwardRef` para recibir el `ref`.


### ¿Desaparece forwardRef?

No inmediatamente.

Hay muchísimo código existente que utiliza `forwardRef(...)` y seguirá funcionando.

Pero para código nuevo en React 19, la recomendación general es preferir:

```tsx
function Component({
  ref
}: Props) {
  ...
}
```

cuando sea posible.

### Evolución histórica

```text
React 18 y anteriores
↓
ref no llega al componente
↓
forwardRef

React 19
↓
ref llega como prop normal
↓
forwardRef deja de ser necesario en muchos casos
```

## Resumen final de Refs

### `useRef`

Permite almacenar valores mutables que persisten entre renders sin provocar nuevos renderizados.

```tsx
const inputRef =
  useRef<HTMLInputElement>(null)
```

---

### `useImperativeHandle`

Permite controlar qué expone un componente a través de un `ref`.

```tsx
useImperativeHandle(ref, () => ({
  open,
  close
}))
```

---

### `forwardRef`

Mecanismo histórico utilizado para reenviar refs en React 18 y anteriores.

```tsx
forwardRef(...)
```

---

### React 19

`ref` puede recibirse como una prop común.

```tsx
function Input({
  ref
}: Props) {
  ...
}
```

### En resumen

React 19 simplifica el trabajo con referencias permitiendo que `ref` se reciba como una prop normal. Esto reduce la necesidad de utilizar `forwardRef` y hace que los componentes sean más sencillos de escribir y entender. Sin embargo, los conceptos fundamentales de refs (`useRef`) y la personalización de APIs mediante `useImperativeHandle` siguen siendo exactamente igual de importantes.


---

# Plus - Lectura Opcional pero Muy Recomendada

Este documento está diseñado con un enfoque de ingeniería moderno, eliminando explicaciones básicas, profundizando en la arquitectura interna y poniendo **el foco principal en los cambios drásticos de React 19**, pero con una estructura compacta y de rápida lectura.

## El Nuevo Paradigma de Optimización y Referencias en React 19

### 1. La Filosofía de Renderizado y el Fin de la Optimización Prematura

En React, un **re-render** es simplemente la ejecución de una función para generar una nueva descripción de la interfaz (Virtual DOM). No implica mutar el DOM real ni repintar la pantalla.

Históricamente, los desarrolladores abusaban de `useMemo` y `useCallback` de forma preventiva. Esto añade un costo oculto: React debe almacenar los arrays de dependencias en memoria, compararlos mediante `Object.is` en cada ejecución y gestionar el almacenamiento en caché. **Memoizar un cálculo barato es matemáticamente más costoso que recalcularlo.**

```
[Flujo Tradicional de Optimización]
Escribir código ──► Detectar lentitud ──► Añadir useMemo/useCallback manual ──► Mantener dependencias

[Flujo Moderno (React 19)]
Escribir código limpio y declarativo ──► React Compiler optimiza en tiempo de compilación automáticamente

```

### 2. React Compiler: La Automatización del Rendimiento

La llegada de **React Compiler** (el motor de optimización en tiempo de compilación) cambia las reglas del juego. Ya no necesitas adivinar dónde colocar optimizaciones manuales.

#### Cómo funciona bajo el capó

El compilador analiza el árbol de sintaxis abstracta (AST) de tu código TypeScript. Detecta qué valores y funciones dependen de qué variables y **reestructura el código inyectando una caché interna a nivel de bytecode**.

```tsx
// Código limpio escrito por el desarrollador
function UserDashboard({ users, filterType }) {
  const filtered = users.filter(u => u.type === filterType);
  const handleClick = () => console.log("Selected:", filterType);

  return <UserList data={filtered} onItemClick={handleClick} />;
}

```

El compilador transforma automáticamente este código para que `filtered` mantenga su referencia estable y `handleClick` no se recree visualmente, **sin que tengas que escribir `useMemo` o `useCallback**`.

### ¿Cuándo siguen siendo necesarios `useMemo` y `useCallback`?

1. **Librerías de terceros antiguas:** Si integras código que no pasa por tu compilador y requiere identidades referenciales estrictas.
2. **Cálculos pesados de CPU:** Algoritmos con complejidad computacional alta (ej. ordenamiento de miles de registros) donde necesitás asegurar la caché explícitamente si el compilador no puede deducir el impacto.


### 3. La Revolución de las Referencias en React 19: Adiós a `forwardRef`

En React 18 y anteriores, `ref` era una palabra clave tratada de forma exclusiva por el reconciliador. No se inyectaba en el objeto `props`, obligando al uso del Higher-Order Component `forwardRef`.

#### El Cambio Arquitectónico en React 19

En React 19, **`ref` es una prop común y corriente**. Desaparece la necesidad de envolver componentes. El motor de React unifica el tratamiento de los argumentos de la función del componente.

#### Comparativa de Implementación Profunda

##### En React 18 (Obsoleto pero común en código legado):

```tsx
// Doble indentación, tipado genérico complejo en TypeScript
type InputProps = { label: string };

const LegacyInput = forwardRef<HTMLInputElement, InputProps>((props, ref) => {
  return (
    <div>
      <label>{props.label}</label>
      <input ref={ref} />
    </div>
  );
});

```

##### En React 19 (Enfoque Moderno):

```tsx
// Limpio, ref entra directamente en la desestructuración de las props
type InputProps = {
  label: string;
  ref?: React.Ref<HTMLInputElement>; // Tipado estándar directo
};

function ModernInput({ label, ref }: InputProps) {
  return (
    <div>
      <label>{label}</label>
      <input ref={ref} />
    </div>
  );
}

```

### 4. `useImperativeHandle`: Blindaje y Encapsulación de Componentes

Cuando pasas una `ref` a un componente, por defecto le das al padre acceso total al nodo del DOM real. Esto rompe el principio de encapsulación de la programación orientada a componentes: el padre podría mutar estilos directamente, hacer `blur()` de forma inesperada o romper el ciclo de vida de React.

`useImperativeHandle` te permite **interceptar la referencia** y exponer una API pública restrictiva e imperativa.

#### Estructura de Memoria en Fiber

Cuando usas `useImperativeHandle`, el objeto que devuelve tu callback reemplaza directamente la propiedad `current` del objeto `Ref` en el nodo Fiber del componente. El padre ya no recibe el elemento del DOM, recibe tu objeto personalizado.

```
[Componente Padre] ──► ref.current ──► [ Objeto interceptado { open(), close() } ]
                                                   │
                                                   ▼ (Privado e inaccesible al padre)
                                       [ Elemento DOM real <dialog> ]

```

#### Implementación Avanzada en React 19 (Sin `forwardRef`)

A continuación, un ejemplo de un componente de reproducción de video que oculta el nodo del DOM y solo expone los métodos de control esenciales:

```tsx
import { useImperativeHandle, useRef } from "react";

// 1. Definimos el contrato público de la Ref
export type VideoPlayerHandle = {
  play: () => void;
  pause: () => void;
};

type VideoPlayerProps = {
  src: string;
  ref?: React.Ref<VideoPlayerHandle>;
};

export function VideoPlayer({ src, ref }: VideoPlayerProps) {
  const videoRef = useRef<HTMLVideoElement>(null);

  // 2. Interceptamos la ref y definimos qué exponer
  useImperativeHandle(ref, () => {
    return {
      play() {
        videoRef.current?.play();
      },
      pause() {
        videoRef.current?.pause();
      }
    };
  }, []); // Array de dependencias vacío garantiza que la API pública sea estática

  return (
    <div className="video-wrapper">
      {/* El DOM real queda confinado internamente */}
      <video ref={videoRef} src={src} width="100%" />
    </div>
  );
}

```

#### Uso desde el componente Padre:

```tsx
import { useRef } from "react";
import { VideoPlayer, VideoPlayerHandle } from "./VideoPlayer";

function App() {
  // El padre se tipa con la interfaz expuesta, no con el elemento HTML
  const playerRef = useRef<VideoPlayerHandle>(null);

  return (
    <>
      <VideoPlayer ref={playerRef} src="movie.mp4" />
      
      <button onClick={() => playerRef.current?.play()}>Reproducir</button>
      <button onClick={() => playerRef.current?.pause()}>Pausar</button>
    </>
  );
}

```
### Machete Técnico de Referencias (Cheat-sheet)

| Herramienta | Mecanismo en React 19 | ¿Cuándo usarla? |
| --- | --- | --- |
| **`useRef`** | Almacena un valor mutable en el nodo Fiber que persiste entre renders sin disparar ciclos de renderizado. | Guardar IDs de timers, instancias de WebSockets, valores previos o referencias directas a elementos HTML locales. |
| **`ref` como prop** | Pasa la referencia directamente de padre a hijo como cualquier otra propiedad del componente. | Siempre que necesites que un componente padre acceda a un nodo nativo que está dentro de un componente hijo. |
| **`useImperativeHandle`** | Reemplaza el valor de `ref.current` con un objeto personalizado, protegiendo la estructura interna del componente. | Creación de componentes reutilizables complejos (modales, reproductores, editores de texto) que necesitan exponer métodos de control (`open`, `focus`, `clear`). |
| **`forwardRef`** | **Obsoleto.** Reenviaba la referencia en versiones previas a React 19. | Únicamente para mantenimiento de bases de código heredadas (React 18 o inferiores) o compatibilidad hacia atrás en librerías. |
