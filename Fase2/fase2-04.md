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

---

## Context + useReducer = mini Redux

Después de aprender `useReducer` y Context por separado, es natural combinarlos.

De hecho, esta combinación es probablemente el patrón de gestión de estado global más común construido únicamente con herramientas nativas de React.

La idea es muy simple:

```text
useReducer
↓
Gestiona el estado

Context
↓
Distribuye el estado
```

Juntos permiten construir una pequeña arquitectura centralizada muy similar a Redux.

### El problema

Supongamos que tenemos un carrito de compras.

Muchos componentes necesitan acceder a él:

```text
Navbar
ProductList
ProductCard
CartSidebar
Checkout
```

Necesitamos:

* leer productos,
* agregar productos,
* eliminar productos,
* vaciar carrito.

Si usamos únicamente props:

```text
App
↓
Navbar
↓
ProductList
↓
ProductCard
```

rápidamente aparece prop drilling.

### Primera pieza: useReducer

Definimos un estado:

```tsx
type CartState = {
  items: string[]
}
```

Acciones:

```tsx
type CartAction =
  | {
      type: "ADD_ITEM"
      payload: string
    }
  | {
      type: "REMOVE_ITEM"
      payload: string
    }
```

Reducer:

```tsx
function cartReducer(
  state: CartState,
  action: CartAction
): CartState {
  switch (action.type) {
    case "ADD_ITEM":
      return {
        items: [
          ...state.items,
          action.payload
        ]
      }

    case "REMOVE_ITEM":
      return {
        items: state.items.filter(
          item =>
            item !== action.payload
        )
      }

    default:
      return state
  }
}
```

Hasta aquí solo resolvimos la lógica.

Todavía no compartimos nada.

### Segunda pieza: Context

Creamos un contexto:

```tsx
import {
  createContext
} from "react"

type CartContextValue = {
  state: CartState
  dispatch:
    React.Dispatch<CartAction>
}

const CartContext =
  createContext<
    CartContextValue | null
  >(null)
```

### El Provider

Ahora combinamos ambas herramientas:

```tsx
import {
  useReducer
} from "react"

function CartProvider({
  children
}: {
  children: React.ReactNode
}) {
  const [state, dispatch] =
    useReducer(
      cartReducer,
      {
        items: []
      }
    )

  return (
    <CartContext.Provider
      value={{
        state,
        dispatch
      }}
    >
      {children}
    </CartContext.Provider>
  )
}
```
### Consumir el estado

Cualquier componente descendiente puede acceder:

```tsx
import {
  useContext
} from "react"

function CartSummary() {
  const cart =
    useContext(CartContext)

  if (!cart) {
    return null
  }

  return (
    <p>
      Items:
      {cart.state.items.length}
    </p>
  )
}
```

### Actualizar el estado

También puede despachar acciones:

```tsx
function AddButton() {
  const cart =
    useContext(CartContext)

  if (!cart) {
    return null
  }

  return (
    <button
      onClick={() =>
        cart.dispatch({
          type: "ADD_ITEM",
          payload: "Laptop"
        })
      }
    >
      Add
    </button>
  )
}
```

### ¿Por qué se parece a Redux?

Porque aparecen exactamente los mismos conceptos:

```text
State
↓
Reducer
↓
Actions
↓
Dispatch
```

Por ejemplo:

```tsx
dispatch({
  type: "ADD_ITEM",
  payload: "Laptop"
})
```

es prácticamente idéntico a Redux.

### La diferencia con Redux

Redux agrega muchas capacidades extra:

* DevTools.
* Middleware.
* Time travel debugging.
* Selectores.
* Ecosistema enorme.
* Herramientas avanzadas de rendimiento.

Mientras que Context + useReducer es una solución mucho más simple y liviana.

### Arquitectura típica

Es muy común ver algo así:

```tsx
<AuthProvider>
  <CartProvider>
    <ThemeProvider>
      <App />
    </ThemeProvider>
  </CartProvider>
</AuthProvider>
```

Cada Provider encapsula:

```text
Estado
+
Reducer
+
Context
```

para una responsabilidad específica.

### ¿Es una buena práctica?

Sí, especialmente para aplicaciones pequeñas y medianas.

Permite:

* evitar prop drilling,
* centralizar la lógica,
* mantener dependencias mínimas,
* no agregar librerías externas.

### ¿Es un reemplazo completo de Redux?

No necesariamente.

Cuando una aplicación crece mucho:

```text
Decenas de contextos
Miles de componentes
Actualizaciones frecuentes
```

comienzan a aparecer problemas de rendimiento y escalabilidad que veremos en el siguiente tema.

Por eso muchas aplicaciones modernas terminan utilizando herramientas especializadas como:

* Zustand
* Jotai
* Redux Toolkit

### En resumen

La combinación de Context y `useReducer` permite construir una solución de estado global utilizando únicamente herramientas nativas de React. `useReducer` centraliza la lógica de actualización mediante acciones y reducers, mientras que Context distribuye el estado y el `dispatch` a cualquier componente del árbol. Este patrón comparte muchos conceptos con Redux y suele ser suficiente para aplicaciones pequeñas y medianas sin necesidad de incorporar librerías externas.

---

## Problemas de performance del Context

Después de aprender Context, es muy común pensar:

> "Perfecto, entonces voy a poner todo mi estado global dentro de Context."

Y ahí es donde suelen empezar los problemas.

Context resuelve muy bien el **prop drilling**, pero no fue diseñado para ser una solución completa de gestión de estado de alta performance.

### El problema fundamental

Supongamos este contexto:

```tsx
type AppContextValue = {
  user: User | null
  theme: "light" | "dark"
  notifications: Notification[]
}
```

```tsx
<AppContext.Provider value={value}>
  <App />
</AppContext.Provider>
```

Y tres consumidores:

```text
Navbar        → usa user
ThemeToggle   → usa theme
Notifications → usa notifications
```

### ¿Qué ocurre si cambia `theme`?

```tsx
setTheme("dark")
```

React ve:

```text
Nuevo value
↓
Context cambió
↓
Notificar consumidores
```

Entonces:

```text
Navbar render
ThemeToggle render
Notifications render
```

Aunque:

```text
Navbar no usa theme
Notifications no usa theme
```

### Context no tiene selectores

Este es el punto clave.

Cuando hacemos:

```tsx
const context =
  useContext(AppContext)
```

React no sabe qué parte del objeto estás usando.

Solo sabe:

```text
Este componente consume AppContext
```

Por lo tanto:

```text
Context cambia
↓
Consumer renderiza
```

### Ejemplo visual

```text
Context
├─ user
├─ theme
└─ notifications
```

Cambia:

```text
theme
```

React no puede pensar:

```text
Solo renderizo ThemeToggle
```

Piensa:

```text
Cambió AppContext
↓
Renderizar todos los consumidores
```

### El problema escala

Imaginemos:

```text
50 componentes
↓
usan el mismo contexto
```

Cada actualización provoca:

```text
50 renders potenciales
```

Aunque solo uno de ellos necesite el dato actualizado.

### Primer consejo: dividir contextos

En lugar de:

```tsx
<AppContext>
```

es preferible:

```tsx
<AuthProvider>
  <ThemeProvider>
    <NotificationProvider>
      <App />
    </NotificationProvider>
  </ThemeProvider>
</AuthProvider>
```

Ahora:

```text
Theme cambia
↓
Solo consumidores de ThemeContext
```

### Segundo consejo: estabilizar referencias

Veamos esto:

```tsx
<AuthContext.Provider
  value={{
    user,
    logout
  }}
>
```

Problema:

Cada render crea un objeto nuevo.

```text
Render
↓
Nuevo objeto
↓
Nuevo value
↓
Context cambia
```

Incluso aunque:

```text
user no cambió
logout no cambió
```

Por eso suele verse:

```tsx
const value = useMemo(() => {
  return {
    user,
    logout
  }
}, [user, logout])
```

para mantener estable la referencia.

### Context + useReducer tampoco escapa

Supongamos:

```tsx
const [state, dispatch] =
  useReducer(...)
```

y luego:

```tsx
<CartContext.Provider
  value={{
    state,
    dispatch
  }}
>
```

Cuando cambia cualquier parte de:

```tsx
state
```

todos los consumidores del contexto pueden volver a renderizar.

Por ejemplo:

```text
CartCount
CartSummary
CartTotal
CartItems
```

aunque solo cambie una propiedad específica.

### Por qué nacieron Zustand y Jotai

Precisamente para resolver este problema.

Estas librerías permiten:

```text
Suscribirse a una parte específica
del estado
```

Por ejemplo:

```tsx
const count =
  useStore(state => state.count)
```

Si cambia:

```text
theme
```

el componente que usa:

```text
count
```

no renderiza.

### ¿Es Context lento?

No.

Este es otro error muy común.

Context es perfectamente válido para:

* usuario autenticado,
* tema,
* idioma,
* configuración,
* datos que cambian poco.

El problema aparece cuando intentamos usar Context para:

```text
Estado enorme
+
Muchas actualizaciones
+
Muchos consumidores
```

### Regla práctica

Context suele ser excelente para:

```text
Auth
Theme
Locale
Settings
Feature flags
```

Empieza a mostrar limitaciones cuando tenemos:

```text
Estados complejos
Actualizaciones frecuentes
Grandes árboles de componentes
```

### En resumen

El principal problema de performance de Context es que cuando el valor proporcionado por un Provider cambia, React vuelve a renderizar todos los componentes que consumen ese contexto, incluso si solo utilizan una pequeña parte de la información. Esto no suele ser un problema para datos globales relativamente estables como autenticación o temas visuales, pero puede convertirse en una limitación cuando el estado es grande, cambia con frecuencia o tiene muchos consumidores. Por eso es habitual dividir contextos o recurrir a soluciones como Zustand y Jotai para escenarios más complejos.

---

## Alternativas modernas: Zustand y Jotai

Después de entender las limitaciones de Context, surge una pregunta natural:

> Si Context empieza a sufrir cuando el estado crece o cambia con frecuencia, ¿qué se utiliza en aplicaciones modernas?

Actualmente las alternativas más populares y recomendadas dentro del ecosistema React son:

* [Zustand](https://zustand.docs.pmnd.rs?utm_source=chatgpt.com)
* [Jotai](https://jotai.org?utm_source=chatgpt.com)

Ambas buscan resolver problemas que aparecen cuando Context comienza a escalar.

### El problema que intentan resolver

Recordemos el principal problema de Context:

```text
Context cambia
↓
Todos los consumidores
pueden renderizar nuevamente
```

Aunque un componente solo necesite una pequeña parte del estado.

Por ejemplo:

```text
State
├─ user
├─ theme
├─ notifications
└─ cart
```

Si cambia:

```text
theme
```

muchos consumidores pueden volver a renderizar.

### La idea moderna: suscripciones selectivas

En lugar de decir:

```text
Consumo todo el contexto
```

las librerías modernas permiten decir:

```text
Solo me interesa esta parte
del estado
```

Por ejemplo:

```tsx
const count =
  useStore(state => state.count)
```

Ahora:

```text
theme cambia
↓
count no cambió
↓
Este componente no renderiza
```


## Zustand

Zustand probablemente sea la librería de estado global más popular y simple del ecosistema React actual.

Su filosofía es:

```text
Store central
+
Hooks
+
Poca configuración
```

### Crear un store

```tsx
import { create } from "zustand"

type CounterStore = {
  count: number

  increment: () => void
}

const useCounterStore =
  create<CounterStore>(
    set => ({
      count: 0,

      increment: () =>
        set(state => ({
          count:
            state.count + 1
        }))
    })
  )
```

### Consumir estado

```tsx
function Counter() {
  const count =
    useCounterStore(
      state => state.count
    )

  return <p>{count}</p>
}
```
### Actualizar estado

```tsx
function IncrementButton() {
  const increment =
    useCounterStore(
      state => state.increment
    )

  return (
    <button
      onClick={increment}
    >
      +
    </button>
  )
}
```
### Lo importante

Cuando cambia:

```text
count
```

solo renderizan los componentes que usan:

```text
count
```

No existe el problema clásico del Context global.

## Jotai

Jotai toma una filosofía completamente distinta.

En lugar de tener un store central, divide el estado en pequeñas unidades llamadas:

```text
Atoms
```

(atomos).

#### Crear un atom

```tsx
import { atom } from "jotai"

const countAtom =
  atom(0)
```


#### Consumirlo

```tsx
import { useAtom }
  from "jotai"

function Counter() {
  const [count] =
    useAtom(countAtom)

  return <p>{count}</p>
}
```

#### Actualizarlo

```tsx
function IncrementButton() {
  const [
    ,
    setCount
  ] = useAtom(countAtom)

  return (
    <button
      onClick={() =>
        setCount(
          prev => prev + 1
        )
      }
    >
      +
    </button>
  )
}
```

### Diferencia conceptual

#### Zustand

Piensa:

```text
Tengo un store
global
```

muy parecido a Redux.

```text
Store
├─ user
├─ theme
├─ cart
└─ notifications
```

#### Jotai

Piensa:

```text
Tengo muchas piezas
pequeñas de estado
```

```text
userAtom
themeAtom
cartAtom
notificationAtom
```

Cada una independiente.

#### Comparación rápida

| Característica           | Context | Zustand | Jotai |
| ------------------------ | ------- | ------- | ----- |
| Incluido en React        | ✅       | ❌       | ❌     |
| Evita prop drilling      | ✅       | ✅       | ✅     |
| Suscripciones selectivas | ❌       | ✅       | ✅     |
| Muy simple               | ✅       | ✅       | ✅     |
| Escala mejor             | ⚠️      | ✅       | ✅     |
| Estado global            | ⚠️      | ✅       | ✅     |

#### ¿Y Redux?

Históricamente Redux fue la solución dominante.

Hoy, para muchas aplicaciones nuevas:

```text
Redux
↓
Zustand
```

se ha vuelto una migración bastante común porque Zustand ofrece:

* menos código,
* menos configuración,
* curva de aprendizaje menor.

Sin embargo Redux sigue siendo muy utilizado en empresas grandes y proyectos existentes.

### ¿Cuál deberías aprender?

Mi recomendación para una mentoría profunda de React sería:

#### Obligatorio

* `useState`
* `useReducer`
* Context API

Porque son parte de React y aparecen en todos lados.

#### Muy recomendable

* Zustand

Porque actualmente es una de las soluciones más adoptadas para estado global moderno.

#### Interesante conocer

* Jotai

Porque introduce un modelo diferente basado en atoms que influenció muchas herramientas modernas.

### Evolución histórica

Podemos pensar la evolución así:

```text
Estado local
↓
useState

Estado complejo
↓
useReducer

Estado compartido
↓
Context

Estado global escalable
↓
Zustand / Jotai

Grandes ecosistemas empresariales
↓
Redux Toolkit
```

### En resumen

Context es una excelente solución para compartir información en React, pero cuando el estado crece y las actualizaciones se vuelven frecuentes pueden aparecer problemas de escalabilidad y rendimiento. Librerías como Zustand y Jotai resuelven este problema mediante suscripciones más precisas, permitiendo que los componentes reaccionen únicamente a la parte del estado que realmente utilizan. Actualmente, Zustand se ha convertido en una de las alternativas más populares por su simplicidad, mientras que Jotai propone un enfoque más granular basado en atoms.
