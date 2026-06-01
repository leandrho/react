# Estados
*El núcleo de la reactividad*

En React, el **estado** (state) representa la información dinámica que un componente puede almacenar y modificar a lo largo del tiempo. Cuando el estado cambia, React vuelve a renderizar el componente para recalcular la interfaz y mantener sincronizada la UI con los datos actuales.

## Hooks

Antes de continuar, debemos entender qué son los hooks.

En React, los Hooks son funciones especiales que permiten utilizar características internas de React dentro de los Function Components, como estado, efectos, referencias, contexto y otras capacidades relacionadas con el ciclo de vida y el renderizado.

Los hooks permiten reutilizar lógica y agregar comportamiento a los componentes sin necesidad de utilizar Class Components. React los identifica por convención porque sus nombres comienzan con `use`, como por ejemplo: `useState`.

## `useState`: estado local y actualizaciones

`useState` es un hook que permite agregar estado local a los Function Components. El estado representa información dinámica que puede cambiar con el tiempo y afectar lo que el componente renderiza.

`useState` permite que un componente recuerde información entre renders y que React vuelva a renderizar automáticamente la interfaz cuando esa información cambia.

Ejemplo:

```tsx
import { useState } from "react"

function Counter() {
  const [count, setCount] = useState(0)

  return <h1>{count}</h1>
}
```

En este caso:

* `count` contiene el estado actual,
* y `setCount` es la función utilizada para actualizarlo.

### ¿Qué significa “estado local”?

El estado es “local” porque pertenece específicamente a ese componente.

Cada instancia del componente mantiene su propio estado independiente.

Ejemplo:

```tsx
<Counter />
<Counter />
```

Cada `Counter` tendrá su propio valor de `count`.

### ¿Qué ocurre cuando el estado cambia?

Cuando se ejecuta:

```tsx
setCount(1)
```

React:

1. registra la actualización,
2. vuelve a renderizar el componente,
3. recalcula la UI,
4. y luego sincroniza los cambios necesarios con el DOM real.

### `useState` no modifica la variable directamente

Esto es importante:

```tsx
count = 10
```

NO actualiza correctamente el estado en React.

El estado debe actualizarse mediante:

```tsx
setCount(10)
```

Esto se debe a que React necesita saber cuándo ocurrió el cambio para iniciar el proceso de actualización.

### Estado y persistencia entre renders

Las variables normales se reinician en cada render:

```tsx
let count = 0
```

Pero el estado de `useState` persiste entre renderizados.

React almacena internamente ese valor y lo conserva aunque el componente vuelva a ejecutarse.

## Actualizaciones funcionales: `prev => ...`

El setter de `useState` puede recibir no solo un valor nuevo, sino también una función. Esa función recibe el estado anterior (`prev`) y debe devolver el siguiente estado.

`prev => ...` garantiza que el nuevo estado se calcule usando el valor más actualizado disponible.

Ejemplo:

```tsx
const [count, setCount] = useState(0)

setCount(prev => prev + 1)
```

En este caso:

* `prev` representa el valor anterior del estado,
* y el nuevo estado se calcula a partir de él.

### ¿Por qué es importante?

Las actualizaciones de estado en React pueden agruparse y ejecutarse de manera asincrónica. Por eso, depender directamente de la variable actual puede producir resultados incorrectos en ciertas situaciones.

Ejemplo problemático:

```tsx
setCount(count + 1)
setCount(count + 1)
```

Ambas líneas usan el mismo valor de `count`.

En cambio:

```tsx
setCount(prev => prev + 1)
setCount(prev => prev + 1)
```

Cada actualización recibe el estado más reciente.

### Cuándo usarlo

Las actualizaciones funcionales son recomendadas cuando el *nuevo estado depende del estado anterior*, cuando *hay múltiples actualizaciones consecutivas* o cuando *existe posibilidad de concurrencia/asincronía*.

## Estado vs Variable local: la diferencia clave

Una variable local y el estado (`state`) pueden almacenar datos, pero React los trata de manera completamente diferente.

Una variable local existe solo durante la ejecución actual del componente. Cada vez que el componente se vuelve a renderizar, la función se ejecuta nuevamente y las variables locales se recrean desde cero.

Ejemplo:

```tsx
function Counter() {
  let count = 0

  count++

  return <h1>{count}</h1>
}
```

En cada render, `count` vuelve a comenzar en `0`.

### El estado persiste entre renders

El estado creado con `useState` sí se conserva entre renderizados porque React lo almacena internamente.

Ejemplo:

```tsx
function Counter() {
  const [count, setCount] = useState(0)

  return <h1>{count}</h1>
}
```

Aunque el componente vuelva a ejecutarse muchas veces, React mantiene el valor actual de `count`.

### Diferencia fundamental

Cambiar una variable local:

* no persiste entre renders,
* y no provoca re-renderizados.

Cambiar el estado:

* persiste entre renders,
* y le indica a React que debe actualizar la interfaz.

## Manejo de formularios controlados

En React, un formulario controlado es aquel donde los valores de los inputs son controlados completamente por el estado del componente. Esto significa que React se convierte en la “fuente de verdad” de los datos del formulario.

Es decir, el estado del componente es la fuente de verdad del formulario, y los inputs simplemente reflejan ese estado

Ejemplo:

```tsx
import { ChangeEvent, useState } from "react"

function Form() {
  const [name, setName] = useState<string>("")

  function handleChange(
    event: ChangeEvent<HTMLInputElement>
  ) {
    setName(event.target.value)
  }

  return (
    <input
      value={name}
      onChange={handleChange}
    />
  )
}
```

En este caso:

* el valor visible del input proviene del estado (`name`),
* y cada cambio del usuario actualiza ese estado mediante `onChange`.

---

# ¿Qué ocurre internamente?

El flujo normalmente es:

```text
Usuario escribe
↓
Se dispara onChange
↓
Se actualiza el estado
↓
React re-renderiza
↓
El input recibe el nuevo value
```

La interfaz siempre refleja el estado actual del componente.

### ¿Por qué se llaman “controlados”?

Porque el valor del input no queda gestionado por el DOM de forma independiente, sino por React mediante el estado.

React controla el valor, la sincronización y el renderizado del input.

### Ventajas

Los formularios controlados permiten:

* validar datos fácilmente,
* sincronizar UI y estado,
* controlar inputs dinámicamente,
* deshabilitar botones,
* mostrar errores,
* transformar datos,
* y manejar formularios complejos de manera predecible.

## Inmutabilidad: por qué no mutar el estado directamente

En React, el estado debe tratarse como inmutable. Esto significa que no debemos modificar directamente los objetos o arrays almacenados en el estado, sino crear nuevas versiones con los cambios aplicados.

React no analiza profundamente cada objeto para detectar cambios.
En cambio, espera recibir nuevas referencias para saber que el estado cambió y que debe actualizar la interfaz.

Ejemplo incorrecto:

```tsx
type User = {
  name: string
}

function App() {
  const [user, setUser] = useState<User>({
    name: "John"
  })

  function handleChange() {
    user.name = "Jane"
  }

  return <h1>{user.name}</h1>
}
```

Acá el objeto se está modificando directamente.

### ¿Por qué es un problema?

React detecta cambios principalmente mediante referencias.

Cuando hacemos `setUser(newUser)` React compara la referencia anterior, con la referencia nueva.
Si mutamos el mismo objeto `user.name = "Jane"` la referencia sigue siendo la misma.

Entonces React puede:

* no detectar correctamente el cambio,
* no re-renderizar,
* romper optimizaciones,
* y generar bugs difíciles de detectar.

### Forma correcta

Debemos crear un nuevo objeto:

```tsx
function handleChange() {
  setUser({
    ...user,
    name: "Jane"
  })
}
```

Ahora existe una nueva referencia, React detecta el cambio y puede actualizar correctamente la UI.

### Lo mismo aplica a arrays

Incorrecto:

```tsx
items.push("New")
```

Correcto:

```tsx
setItems([
  ...items,
  "New"
])
```

#### Nota
La inmutabilidad en React no solo aplica al objeto o array principal del estado, sino también a cualquier estructura anidada que cambie. Si el estado contiene subobjetos o arrays de objetos, no alcanza con copiar solo el nivel superior: cada nivel modificado necesita una nueva referencia.

Por eso, cuando se actualizan estructuras profundas, normalmente se utilizan spreads (`...`) u otros métodos que crean nuevas copias en cada nivel necesario. Esto permite que React detecte correctamente los cambios mediante comparación de referencias y actualice la interfaz de forma consistente.




