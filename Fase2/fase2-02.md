# Custom Hooks
*Reutilización real de lógica*

Un **Custom Hook** es una función de JavaScript/TypeScript que utiliza uno o más Hooks de React y permite **extraer, reutilizar y compartir lógica con estado entre componentes**.

Por convención, su nombre debe comenzar con `use`:

```tsx
function useCounter() {
  const [count, setCount] = useState(0)

  return {
    count,
    increment: () =>
      setCount(prev => prev + 1)
  }
}
```

### ¿Qué problema resuelven?

A medida que una aplicación crece, es común que varios componentes necesiten la misma lógica.

Por ejemplo:

* obtener datos de una API,
* sincronizar valores con `localStorage`,
* manejar formularios,
* escuchar eventos del navegador,
* controlar timers.

Sin Custom Hooks, esa lógica termina duplicada en múltiples componentes.

```tsx
function UserProfile() {
  const [user, setUser] = useState(...)
  const [loading, setLoading] = useState(...)

  useEffect(...)
}
```

```tsx
function UserDashboard() {
  const [user, setUser] = useState(...)
  const [loading, setLoading] = useState(...)

  useEffect(...)
}
```

La lógica se repite.

### La solución

Extraer esa lógica a un Hook reutilizable:

```tsx
function useUser(userId: number) {
  const [user, setUser] = useState<User | null>(null)
  const [loading, setLoading] = useState(true)

  useEffect(() => {
    // fetch...
  }, [userId])

  return {
    user,
    loading
  }
}
```

Y luego usarlo desde cualquier componente:

```tsx
function UserProfile() {
  const { user, loading } = useUser(1)

  return ...
}
```

### ¿Comparten estado?

Esta es una duda muy común. Pero **NO**. Cada vez que se llama a un Custom Hook, React crea un estado independiente.

```tsx
function App() {
  const counterA = useCounter()
  const counterB = useCounter()

  ...
}
```

Acá existen `Estado A, Estado B` completamente separados.

El Hook comparte la lógica, no los datos.

### Diferencia entre componente y Custom Hook

Un _Componente_ participa en el árbol de componentes, devuelve JSX, representa una parte de la interfaz.

Un _Custom Hook_ no devuelve JSX, no aparece en la UI, encapsula lógica reutilizable.

### ¿Son una característica especial de React?

No realmente.

Un Custom Hook es simplemente una función normal que sigue dos convenciones:

1. Su nombre empieza con `use`.
2. Puede llamar a otros Hooks.

Por eso React puede aplicar las reglas de Hooks y las herramientas (como ESLint) pueden detectar errores.

### En resumen

Un Custom Hook es una función reutilizable que encapsula lógica basada en Hooks. Su objetivo principal es extraer comportamiento repetido de los componentes, permitiendo compartir lógica sin duplicar código. Los Custom Hooks comparten lógica, pero cada llamada mantiene su propio estado independiente.

---

## Reglas de los Hooks y por qué existen

Los Hooks funcionan gracias a una serie de reglas muy estrictas. A primera vista pueden parecer arbitrarias, pero en realidad existen porque React necesita saber exactamente qué Hook corresponde a cada estado, efecto o referencia en cada render.

Las dos reglas principales son:

1. **Llamar Hooks únicamente en el nivel superior.**
2. **Llamar Hooks únicamente desde Function Components o Custom Hooks.**

### Regla 1: Solo en el nivel superior

Correcto:

```tsx
function Counter() {
  const [count, setCount] = useState(0)

  useEffect(() => {
    console.log(count)
  }, [count])

  return <button>{count}</button>
}
```
Incorrecto:

```tsx
function Counter() {
  if (someCondition) {
    useState(0)
  }

  return null
}
```

### ¿Por qué existe esta regla?

React no identifica los Hooks por nombre, los identifica por su **posición durante el render**.

Por ejemplo:

```tsx
function Component() {
  const [count] = useState(0)
  const [name] = useState("John")
}
```

React interpreta internamente algo parecido a:

```text
Hook #1 → count
Hook #2 → name
```

En el siguiente render espera encontrar exactamente el mismo orden.

### El problema de los condicionales

Supongamos:

```tsx
function Component({
  isLoggedIn
}: {
  isLoggedIn: boolean
}) {
  const [count] = useState(0)

  if (isLoggedIn) {
    useEffect(() => {
      ...
    }, [])
  }

  const [name] = useState("John")

  return null
}
```

Primer render:

```text
Hook #1 → useState(count)
Hook #2 → useEffect
Hook #3 → useState(name)
```

Segundo render:

```text
isLoggedIn = false
```

Ahora React ve:

```text
Hook #1 → useState(count)
Hook #2 → useState(name)
```

El orden cambió, por lo que React ya no sabe qué Hook corresponde a qué posición.

Por eso aparecen errores y comportamientos impredecibles.

### Regla 2: Solo desde componentes o Custom Hooks

Correcto:

```tsx
function App() {
  const [count] = useState(0)

  return null
}
```

✅ Correcto:

```tsx
function useCounter() {
  const [count] = useState(0)

  return count
}
```
---

# ¿Por qué existe esta regla?

Cuando React renderiza un componente sabe:

```text
Estoy renderizando App
↓
Voy ejecutando Hooks
↓
Guardo su estado
```

Lo mismo ocurre con un Custom Hook.

Pero una función común:

```tsx
function calculateSomething() {
  ...
}
```

puede ejecutarse:

* en cualquier momento,
* desde cualquier lugar,
* fuera de un render.

React no tendría forma de asociar esos Hooks a un componente.

### El papel del linter

La mayoría de estos errores son detectados automáticamente por:

```text
eslint-plugin-react-hooks
```

Por ejemplo:

```tsx
if (isOpen) {
  useEffect(...)
}
```

genera una advertencia inmediatamente.

El linter existe precisamente para garantizar que React siempre encuentre los Hooks en el mismo orden.

### En resumen

Las reglas de los Hooks existen porque React identifica cada Hook por el orden en que aparece durante el render, no por su nombre. Para que React pueda asociar correctamente el estado, los efectos y las referencias entre renderizados, los Hooks deben ejecutarse siempre en el mismo orden y únicamente dentro de Function Components o Custom Hooks.

---

## Extraer lógica a un Custom Hook

La principal razón para crear un Custom Hook es **reutilizar lógica con estado sin duplicar código**.

A medida que una aplicación crece, es común encontrar varios componentes que contienen la misma combinación de:

* `useState`,
* `useEffect`,
* event handlers,
* transformaciones de datos.

Cuando eso ocurre, suele ser una señal de que esa lógica puede extraerse a un Custom Hook.

### Antes: lógica duplicada

Supongamos que varios componentes necesitan cargar un usuario.

```tsx
import { useEffect, useState } from "react"

type User = {
  id: number
  name: string
}

function UserProfile({
  userId
}: {
  userId: number
}) {
  const [user, setUser] =
    useState<User | null>(null)

  const [loading, setLoading] =
    useState(true)

  useEffect(() => {
    async function loadUser() {
      setLoading(true)

      const response = await fetch(
        `/api/users/${userId}`
      )

      const data: User =
        await response.json()

      setUser(data)
      setLoading(false)
    }

    loadUser()
  }, [userId])

  return ...
}
```

Y luego aparece otro componente con exactamente la misma lógica.

```tsx
function UserCard({
  userId
}: {
  userId: number
}) {
  const [user, setUser] =
    useState<User | null>(null)

  const [loading, setLoading] =
    useState(true)

  useEffect(() => {
    ...
  }, [userId])

  return ...
}
```

La lógica está duplicada.

### Después: lógica reutilizable

Extraemos todo a un Hook:

```tsx
import {
  useEffect,
  useState
} from "react"

type User = {
  id: number
  name: string
}

function useUser(userId: number) {
  const [user, setUser] =
    useState<User | null>(null)

  const [loading, setLoading] =
    useState(true)

  useEffect(() => {
    async function loadUser() {
      setLoading(true)

      const response = await fetch(
        `/api/users/${userId}`
      )

      const data: User =
        await response.json()

      setUser(data)
      setLoading(false)
    }

    loadUser()
  }, [userId])

  return {
    user,
    loading
  }
}
```

Ahora los componentes son mucho más simples.

```tsx
function UserProfile({
  userId
}: {
  userId: number
}) {
  const { user, loading } =
    useUser(userId)

  return ...
}
```

```tsx
function UserCard({
  userId
}: {
  userId: number
}) {
  const { user, loading } =
    useUser(userId)

  return ...
}
```
### Qué debe extraerse

Una buena candidata para convertirse en Custom Hook suele cumplir alguna de estas condiciones:

#### La lógica se repite

```tsx
useState(...)
useEffect(...)
useEffect(...)
```

aparecen en varios componentes.

#### La lógica es compleja

Aunque se use una sola vez.

```tsx
function ProductPage() {
  // 200 líneas de estado y efectos
}
```

Extraer partes a Hooks suele mejorar la legibilidad.

#### Existe una responsabilidad clara

Por ejemplo:

```text
Gestión de usuarios
Gestión de formularios
Persistencia en localStorage
Autenticación
Scroll
Geolocalización
```

Cada responsabilidad puede encapsularse en un Hook.


### Qué NO debe extraerse

No todo merece un Custom Hook.

Por ejemplo:

```tsx
const fullName =
  `${firstName} ${lastName}`
```

o

```tsx
const isAdmin =
  role === "admin"
```

son cálculos simples que probablemente no justifican una abstracción.

### Un Custom Hook es una API

Cuando alguien utiliza:

```tsx
const {
  user,
  loading
} = useUser(userId)
```

No necesita saber qué estados existen, qué efectos hay, cómo funciona el fetch.

Solo necesita conocer la interfaz pública del Hook.

Por eso un buen Custom Hook oculta los detalles internos y expone únicamente lo necesario.

### En resumen

Extraer lógica a un Custom Hook permite reutilizar comportamiento basado en Hooks sin duplicar código. Un buen Custom Hook encapsula una responsabilidad específica, oculta los detalles de implementación y expone una API simple que los componentes pueden consumir para enfocarse únicamente en la interfaz.
