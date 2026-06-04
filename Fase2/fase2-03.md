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
  const result =
    expensiveCalculation()

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

# Ejemplo básico

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


