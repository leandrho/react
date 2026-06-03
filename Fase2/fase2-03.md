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
