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

---

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

## Dos herramientas para dos problemas distintos

React proporciona dos herramientas fundamentales:

### `useReducer`

Para gestionar estados complejos y centralizar la lógica de actualización.

```tsx
const [state, dispatch] =
  useReducer(reducer, initialState)
```

### Context API

Para compartir información entre componentes sin necesidad de prop drilling.

```tsx
<AuthContext>
  <App />
</AuthContext>
```

## Una combinación muy común

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
