# useReducer y useContext
*Estado más complejo*

## Estado complejo y estado compartido

Hasta ahora trabajamos principalmente con estado local usando:

```tsx
const [count, setCount] = useState(0)
```

Este enfoque funciona perfectamente cuando el estado es pequeño y afecta únicamente a un componente o a unos pocos componentes cercanos.

Sin embargo, a medida que una aplicación crece suelen aparecer dos problemas:

### Estado complejo

Cuando una actualización ya no es una simple asignación:

```tsx
setCount(count + 1)
```

sino que implica múltiples reglas de negocio:

```tsx
setState(...)
```

con validaciones, múltiples propiedades relacionadas, acciones distintas y transiciones de estado más sofisticadas.

### Estado compartido

Cuando varios componentes necesitan acceder o modificar la misma información.

Por ejemplo:

```text
Navbar
Cart
ProductList
Checkout
```

todos necesitan conocer:

```text
Usuario actual
Tema
Idioma
Carrito
Permisos
```

y comenzar a pasar props manualmente por múltiples niveles puede volverse incómodo.

### Dos herramientas para dos problemas distintos

React proporciona dos herramientas fundamentales:

#### `useReducer`

Para gestionar estados complejos y centralizar la lógica de actualización.

```tsx
const [state, dispatch] =
  useReducer(reducer, initialState)
```

#### Context API

Para compartir información entre componentes sin necesidad de prop drilling.

```tsx
<AuthContext>
  <App />
</AuthContext>
```

### Una combinación muy común

En aplicaciones medianas y grandes suelen utilizarse juntos:

```text
useReducer
↓
Gestiona el estado

Context
↓
Distribuye el estado
```

Por eso muchas veces escucharás la frase:

> Context + useReducer = "mini Redux"

aunque veremos más adelante que existen diferencias importantes.

### En resumen

`useState` resuelve gran parte de los problemas cotidianos, pero cuando el estado se vuelve complejo o necesita compartirse entre muchos componentes, React ofrece herramientas más especializadas. `useReducer` ayuda a organizar la lógica de actualización del estado, mientras que Context permite distribuir información a través del árbol de componentes. Juntas forman la base de muchas arquitecturas modernas en React.

---

## `useReducer`: cuándo reemplaza a `useState`

`useReducer` es un Hook para gestionar estado mediante un modelo basado en **acciones y transiciones de estado**. Aunque puede resolver los mismos problemas que `useState`, su principal objetivo es organizar estados complejos y centralizar la lógica de actualización.

Su sintaxis básica es:

```tsx
const [state, dispatch] = useReducer(
  reducer,
  initialState
)
```

donde:

```text
state
↓
Estado actual

dispatch
↓
Envía acciones

reducer
↓
Decide cómo cambia el estado
```

### El modelo mental de useState

Con `useState` solemos pensar así:

```tsx
setCount(prev => prev + 1)
```

o:

```tsx
setUser(...)
```

Es decir:

```text
Tengo estado
↓
Lo modifico directamente
```

Este modelo es simple y funciona muy bien para estados pequeños.

### Cuándo empieza a complicarse

Supongamos un formulario:

```tsx
type FormState = {
  name: string
  email: string
  isSubmitting: boolean
  error: string | null
}
```

Con `useState` podríamos tener:

```tsx
const [form, setForm] =
  useState<FormState>(...)
```

Y luego:

```tsx
setForm(...)
setForm(...)
setForm(...)
```

para distintos casos.

A medida que el componente crece aparecen:

* múltiples actualizaciones,
* lógica repetida,
* validaciones,
* distintos flujos de negocio.

La lógica de actualización comienza a dispersarse por todo el componente.

### El modelo mental de useReducer

`useReducer` propone otro enfoque.

En lugar de preguntar:

```text
¿Cómo modifico el estado?
```

pregunta:

```text
¿Qué ocurrió?
```

Por ejemplo:

```text
USER_CREATED
USER_UPDATED
FORM_SUBMITTED
LOGIN_SUCCESS
LOGIN_FAILED
```

El componente envía eventos (acciones):

```tsx
dispatch({
  type: "LOGIN_SUCCESS"
})
```

y el reducer decide cómo cambia el estado.

### Un ejemplo simple

Estado:

```tsx
type CounterState = {
  count: number
}
```

Acciones:

```tsx
type CounterAction =
  | { type: "increment" }
  | { type: "decrement" }
```

Reducer:

```tsx
function reducer(
  state: CounterState,
  action: CounterAction
): CounterState {
  switch (action.type) {
    case "increment":
      return {
        count: state.count + 1
      }

    case "decrement":
      return {
        count: state.count - 1
      }

    default:
      return state
  }
}
```

Uso:

```tsx
const [state, dispatch] =
  useReducer(reducer, {
    count: 0
  })
```

Actualizar:

```tsx
dispatch({
  type: "increment"
})
```

### ¿Qué es un reducer?

Un reducer es simplemente una función pura.

Recibe:

```text
Estado actual + Acción
```
y devuelve:

```text
Nuevo estado
```

Conceptualmente:

```tsx
function reducer(
  state,
  action
) {
  return newState
}
```

Nada más.

### Ventajas frente a useState

#### Centraliza la lógica

Con `useState`:

```tsx
setState(...)
setState(...)
setState(...)
```

repartidos por el componente.

Con `useReducer`:

```tsx
dispatch(...)
```

y toda la lógica vive en un único lugar.

#### Hace explícitas las transiciones

En lugar de:

```tsx
setUser(...)
```

vemos:

```tsx
dispatch({
  type: "LOGIN_SUCCESS"
})
```

que suele ser mucho más descriptivo.

#### Escala mejor

Cuando el estado tiene muchas propiedades relacionadas:

```tsx
user
permissions
loading
error
token
```

un reducer suele ser más fácil de mantener que múltiples llamadas a `setState`.


## Regla práctica

Usá `useState` cuando:

```text
Estado pequeño
Actualizaciones simples
Poca lógica
```

Considerá `useReducer` cuando:

```text
Muchas propiedades relacionadas
Muchas transiciones posibles
Lógica compleja
Reglas de negocio importantes
```

## Relación con Redux

Si ya escuchaste hablar de Redux, probablemente notes similitudes:

```text
State
Actions
Reducer
Dispatch
```

No es casualidad.
Redux está inspirado en exactamente el mismo patrón que utiliza `useReducer`.
Por eso aprender `useReducer` ayuda enormemente a comprender Redux y otras librerías de gestión de estado.

### En resumen

`useReducer` es una alternativa a `useState` diseñada para manejar estados complejos y centralizar la lógica de actualización. En lugar de modificar el estado directamente, los componentes envían acciones mediante `dispatch`, y un reducer determina cómo debe cambiar el estado. Esto hace que las transiciones sean más explícitas, facilita el mantenimiento y permite escalar mejor cuando la lógica de negocio comienza a crecer.

---

## Context API: crear, proveer y consumir contexto

La **Context API** es un mecanismo que permite compartir información entre componentes sin necesidad de pasar props manualmente a través de múltiples niveles del árbol.

Su objetivo principal es resolver el problema conocido como:

```text
Prop Drilling
```

es decir, tener que pasar props por componentes intermedios que ni siquiera las utilizan.

### El problema

Supongamos la siguiente estructura:

```text
App
└─ Layout
   └─ Header
      └─ UserMenu
```

Y que `UserMenu` necesita conocer el usuario autenticado.

Sin Context:

```tsx
<App user={user} />
```
↓
```tsx
<Layout user={user} />
```
↓
```tsx
<Header user={user} />
```
↓
```tsx
<UserMenu user={user} />
```

Aunque:

```text
Layout
Header
```

no utilicen esa información.

Esto es prop drilling.

### La idea de Context

Context permite colocar información en una especie de "canal compartido" para que cualquier componente descendiente pueda acceder a ella.

Conceptualmente:

```text
Provider
↓
Árbol de componentes
↓
Consumer
```

Sin importar cuántos niveles existan entre ambos.


### Paso 1: Crear el contexto

Todo comienza con:

```tsx
import { createContext } from "react"

type AuthContextValue = {
  user: string | null
}

const AuthContext =
  createContext<AuthContextValue | null>(
    null
  )
```

`createContext` crea un objeto que representa el contexto.

Podemos imaginarlo como:

```text
Canal de comunicación
```

que luego será utilizado por Providers y Consumers.

### Paso 2: Proveer el contexto

Crear el contexto no alcanza.

También debemos proporcionar un valor.

```tsx
function App() {
  const user = "Leandro"

  return (
    <AuthContext.Provider
      value={{ user }}
    >
      <Layout />
    </AuthContext.Provider>
  )
}
```

El Provider establece:

```text
Qué valor compartir
```
con todos los componentes descendientes.

### Paso 3: Consumir el contexto

Para leer el valor utilizamos:

```tsx
import { useContext } from "react"
```

```tsx
function UserMenu() {
  const auth =
    useContext(AuthContext)

  return (
    <p>{auth?.user}</p>
  )
}
```

React busca el Provider más cercano y devuelve su valor.

### ¿Cómo encuentra React el valor?

Conceptualmente:

```text
UserMenu
↓
Busca Provider
↑
Header
↑
Layout
↑
AuthContext.Provider
```

Cuando lo encuentra: `Devuelve value`

### Qué ocurre cuando cambia el contexto

Supongamos:

```tsx
<AuthContext.Provider
  value={{ user }}
>
```

y luego:

```tsx
setUser("Juan")
```

El valor del contexto cambia.

React volverá a renderizar todos los consumidores que utilicen ese contexto.

```text
Provider cambia
↓
Consumers renderizan nuevamente
```

Este detalle será muy importante cuando hablemos de performance.

### Context no es estado

Este es un error muy común.

Mucha gente piensa:

```text
Context = State Management
```

pero no es exactamente así.

Context solamente responde a la pregunta:

```text
¿Cómo comparto datos?
```

No responde:

```text
¿Cómo actualizo esos datos?
¿Cómo organizo la lógica?
```

Por eso normalmente aparece junto a:

```tsx
useState
```

o:

```tsx
useReducer
```

### Ejemplo típico

```tsx
type ThemeContextValue = {
  theme: "light" | "dark"
}
```

```tsx
const ThemeContext =
  createContext<
    ThemeContextValue | null
  >(null)
```

```tsx
<ThemeContext.Provider
  value={{ theme }}
>
  <App />
</ThemeContext.Provider>
```

Consumir:

```tsx
const theme =
  useContext(ThemeContext)
```

### Casos de uso comunes

Context suele utilizarse para información global o semiglobal:

```text
Usuario autenticado
Tema visual
Idioma
Permisos
Configuración
Carrito de compras
```

## Context en React moderno

Hoy es muy común ver una estructura como:

```tsx
<AuthProvider>
  <ThemeProvider>
    <CartProvider>
      <App />
    </CartProvider>
  </ThemeProvider>
</AuthProvider>
```

Cada Provider comparte una responsabilidad específica.

### En resumen

La Context API permite compartir información entre componentes sin necesidad de prop drilling. Un contexto se crea mediante `createContext`, se distribuye usando un Provider y se consume con `useContext`. Context no reemplaza al estado ni gestiona la lógica de actualización; simplemente proporciona un mecanismo para hacer que ciertos datos estén disponibles en cualquier parte de una sección del árbol de componentes.
