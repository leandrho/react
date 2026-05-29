# Estados y eventos
*El núcleo de la reactividad*

En React, el **estado** (state) representa la información dinámica que un componente puede almacenar y modificar a lo largo del tiempo. Cuando el estado cambia, React vuelve a renderizar el componente para recalcular la interfaz y mantener sincronizada la UI con los datos actuales.

Los **eventos** permiten que la interfaz reaccione a las interacciones del usuario o a determinadas acciones del sistema, como clicks, escritura, envíos de formularios o movimientos del mouse. En React, los eventos forman parte del flujo interactivo de la aplicación y normalmente son los que desencadenan cambios de estado y nuevas actualizaciones de la interfaz.

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

