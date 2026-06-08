# Hooks Avanzados

## El nuevo hook `use`: Consumir contextos y promesas de forma condicional.

El hook `use` representa uno de los cambios conceptuales más importantes del React moderno. Mientras que la mayoría de los Hooks (`useState`, `useReducer`, `useEffect`, etc.) están pensados para gestionar estado, efectos o lógica de componentes, `use` tiene un propósito mucho más específico: **leer recursos durante el renderizado**.

Conceptualmente, `use` le dice a React:

```text
Necesito el valor de este recurso para poder renderizar.
```

Y React se encarga de obtenerlo, esperarlo o recuperarlo según corresponda.

Actualmente esos recursos son principalmente:

```text
Contexts
Promesas
```
### Un pequeño cambio de paradigma

Durante años, React separó claramente dos momentos:

```text
Render
↓
Generar UI

Effect
↓
Obtener datos externos
```

Por eso era habitual ver:

```tsx
function UserProfile() {
  const [user, setUser] = useState<User | null>(null)

  useEffect(() => {
    fetchUser().then(setUser)
  }, [])

  if (!user) {
    return <p>Loading...</p>
  }

  return <p>{user.name}</p>
}
```

Aquí ocurre algo interesante:

1. React renderiza.
2. No hay datos.
3. Se muestra un loading.
4. Se ejecuta el effect.
5. Llega la respuesta.
6. React vuelve a renderizar.

El componente necesita atravesar varios ciclos para terminar mostrando la información.

Con `use`, React empieza a adoptar otra filosofía:

```text
Para renderizar necesito estos datos.
No renderices hasta tenerlos.
```
###  use con Promesas

Supongamos:

```tsx
const userPromise = fetchUser()
```

Podemos hacer:

```tsx
const user = use(userPromise)
```

A primera vista parece algo sencillo, pero internamente ocurre bastante magia.

### ¿Qué pasa si la promesa ya está resuelta?

```text
Promesa resuelta
↓
use devuelve el valor
↓
Render continúa
```

Todo funciona normalmente.

### ¿Qué pasa si la promesa sigue pendiente?

Aquí aparece _Suspense_.

React detecta:

```text
Todavía no tengo el valor.
```

Entonces:

```text
Suspende el render actual
↓
Muestra fallback de Suspense
↓
Espera la resolución
↓
Intenta renderizar nuevamente
```

Cuando la promesa termina:

```text
use devuelve el resultado
↓
El componente puede renderizarse
```

### El modelo mental correcto

No pienses:

```text
use espera la promesa
```

porque eso sugiere una espera bloqueante.

Más bien:

```text
use informa a React que
necesita ese recurso.

React suspende el render
y lo reintenta después.
```

Ese detalle es fundamental para comprender Suspense.

## use y Context

El otro uso importante es consumir contextos.

Tradicionalmente:

```tsx
const auth = useContext(AuthContext)
```

Con `use`:

```tsx
const auth = use(AuthContext)
```

El resultado es exactamente el mismo.

Sin embargo, aquí aparece una diferencia importante.

### El problema de las reglas de Hooks

Recordemos una de las reglas fundamentales:

```text
Los Hooks deben ejecutarse
siempre en el mismo orden.
```

Por eso esto es inválido:

```tsx
if (isLoggedIn) {
  const auth =  useContext(AuthContext)
}
```

Porque en algunos renders el Hook se ejecuta y en otros no.

React perdería la correspondencia interna entre Hooks y Fibers.

### use rompe esa limitación

`use` es un caso especial implementado directamente dentro del reconciliador de React.

Por eso esto sí es válido:

```tsx
if (isLoggedIn) {
  const auth = use(AuthContext)
}
```

e incluso:

```tsx
for (const context of contexts) {
  const value = use(context)
}
```

Algo completamente imposible con Hooks tradicionales.

### ¿Por qué React puede permitirse esto?

Porque `use` no almacena estado interno.

Cuando React procesa:

```tsx
useState(...)
useReducer(...)
useMemo(...)
```

necesita mantener una estructura interna de Hooks asociada al Fiber actual.

Por eso el orden es crítico.

Pero cuando procesa:

```tsx
use(Context)
```

o:

```tsx
use(Promise)
```

simplemente está leyendo un recurso.

No necesita reservar una posición dentro de la lista enlazada de Hooks del Fiber.

Esa es la razón técnica por la que puede utilizarse de forma condicional.

### Relación con Fiber

Si recuerdas lo que vimos sobre Fiber:

```text
Fiber
↓
Lista enlazada de Hooks
↓
useState
useReducer
useEffect
...
```

Todos esos Hooks ocupan una posición específica.

`use` es diferente.

No crea estado.

No crea efectos.

No crea memoización.

Simplemente consulta un recurso y devuelve un valor.

Por eso React no necesita rastrearlo como rastrea a los demás Hooks.

### ¿Reemplaza a useEffect?

No.

Y entender esto evita muchos errores.

`useEffect` responde:

```text
¿Cómo sincronizo mi componente
con algo externo?
```

Ejemplos:

```text
WebSockets
Timers
Eventos
Suscripciones
document.title
```

---

`use` responde:

```text
¿Qué dato necesito para renderizar?
```

Ejemplos:

```text
Context
Promesa
```

Son problemas completamente distintos.

### Dónde encaja en React moderno

El verdadero potencial de `use` aparece cuando se combina con:

* Suspense
* Server Components
* Streaming
* Next.js App Router

De hecho, muchas de las nuevas APIs de React y Next están diseñadas alrededor de este modelo:

```text
Declarar recursos
↓
Leer recursos con use
↓
Suspense coordina la espera
```

En lugar del viejo patrón:

```text
Render
↓
Effect
↓
Fetch
↓
Nuevo render
```

## Comparación conceptual

### React tradicional

```text
Render
↓
Loading
↓
useEffect
↓
Fetch
↓
Re-render
↓
Datos
```

### React moderno

```text
Render
↓
use(resource)
↓
Suspense
↓
Recurso disponible
↓
Render completo
```

### En resumen

`use` es un hook especial cuyo propósito es leer recursos durante el renderizado. A diferencia de Hooks como `useState` o `useEffect`, no gestiona estado ni sincroniza efectos externos, sino que obtiene valores de contextos o promesas. Su integración con Suspense permite que React pause y reintente renderizados cuando los datos aún no están disponibles, y su naturaleza especial le permite utilizarse de forma condicional porque no participa en el sistema tradicional de almacenamiento de Hooks dentro de Fiber. Por eso `use` representa uno de los pilares del nuevo modelo de datos de React basado en Suspense y Server Components.
