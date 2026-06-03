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
