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
