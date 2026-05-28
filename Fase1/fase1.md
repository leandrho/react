# Fase 1 — Fundamentos sólidos

## ¿Qué es React?

React es una biblioteca de JavaScript que permite construir interfaces de usuario mediante componentes reutilizables utilizando un enfoque declarativo. React sincroniza el estado de los componentes con la interfaz de usuario mediante un sistema de renderizado basado en un Virtual DOM, actualizando eficientemente solo las partes necesarias del DOM real.

* **Declarativo:** Significa describir cómo debería verse la interfaz según un estado, en lugar de manipular manualmente el DOM paso a paso.

* **Componentes:** Son piezas reutilizables e independientes de la interfaz. Un componente puede contener estructura (JSX), lógica, estado y comportamiento. React construye toda la UI combinando componentes.

* **JSX:** Es una sintaxis que parece HTML pero en realidad es JavaScript. React la transforma en objetos que representan la UI.

* **Virtual DOM:** Es una representación liviana del DOM en memoria hecha con objetos JavaScript. React la usa para comparar cambios antes de actualizar el DOM real.


## Virtual DOM vs DOM real: ¿Qué problema resuelve?

### DOM real

El DOM real (Real DOM) es la estructura que el navegador crea a partir del HTML.

Por ejemplo:
```html
<h1>Hello</h1>
```

El navegador lo transforma en un árbol de nodos que JavaScript puede modificar.

### Virtual DOM

El Virtual DOM es una representación en memoria del DOM real que React utiliza para comparar cambios y actualizar eficientemente solo las partes necesarias de la interfaz.

El Virtual DOM es una representación liviana del DOM hecha con objetos JavaScript.

React genera una copia virtual de la interfaz en memoria.

Ejemplo conceptual:
```js
{
  type: "h1",
  props: {
    children: "Hello"
  }
}
```
### ¿Qué problema resuelve el Virtual DOM?

El Virtual DOM permite que React compare el estado anterior y el nuevo, detecte exactamente qué cambió y actualice solo las partes necesarias del DOM real.

Ejemplo simple

Supongamos:

```html
<h1>0</h1>
```
y luego cambia a:

```html
<h1>1</h1>
```
React detecta:

el ```<h1>``` sigue existiendo,
solo cambió el texto.

Entonces actualiza únicamente el contenido del texto, no todo el nodo.

#### La idea clave

El Virtual DOM NO existe para “hacer magia”.

Existe para abstraer la manipulación manual del DOM,
optimizar actualizaciones,
y permitir un modelo declarativo.
Importante: el Virtual DOM no es automáticamente más rápido

Mucha gente cree:

“Virtual DOM = más velocidad”

No exactamente.
React sigue tocando el DOM real eventualmente.
La ventaja es que reduce operaciones innecesarias, agrupa cambios y automatiza optimizaciones complejas.

## Reconciliation y el Algoritmo de Diffing 

### Reconciliation (Reconciliación) 

La reconciliación es el proceso mediante el cual React compara el Virtual DOM anterior con el nuevo para determinar qué cambió y qué debe actualizarse en el DOM real.

Cada vez que cambia el estado, cambian props o un componente se vuelve a renderizar, React genera un nuevo árbol de elementos(Virtual DOM) y lo compara con el anterior.

El objetivo es actualizar la menor cantidad posible del DOM real.

La reconciliación permite:

* reutilizar nodos existentes,
* evitar renders innecesarios,
* minimizar operaciones sobre el DOM real.

### Diffing Algorithm

El “diffing algorithm” es el algoritmo que React utiliza durante la reconciliación para detectar diferencias entre árboles de elementos de forma eficiente.

React NO compara absolutamente todo de forma profunda porque sería demasiado costoso.

En cambio, utiliza heurísticas y reglas optimizadas para hacer la comparación mucho más rápida.


### Reglas principales del algoritmo de reconciliación de React

1. **Elementos de distinto tipo → React destruye y recrea**

```tsx id="ns3rxg"
<div />
```

→

```tsx id="6q4slj"
<span />
```

React desmonta el anterior y crea uno nuevo.

---

2. **Elementos del mismo tipo → React reutiliza el nodo**

```tsx id="u35k78"
<h1 className="red" />
```

→

```tsx id="ftt2e0"
<h1 className="blue" />
```

React mantiene el mismo nodo y actualiza solo las props necesarias.

---

3. **Los componentes también se comparan por tipo**

```tsx id="hmql5h"
<UserCard />
```

→

```tsx id="3u2qk1"
<ProductCard />
```

React desmonta uno y monta el otro.

---

4. **Las keys identifican elementos en listas**

```tsx id="71x9d9"
items.map(item => (
  <li key={item.id}>{item.name}</li>
))
```

Las `key` ayudan a React a:

* reutilizar elementos,
* moverlos correctamente,
* preservar estado,
* evitar recreaciones innecesarias.

---

5. **React intenta actualizar lo mínimo posible del DOM**

React compara:

* tipo,
* props,
* children,
* keys,

y modifica únicamente lo necesario en el DOM real.


## Fiber: ¿Qué es y por qué React lo usa?

Fiber es la arquitectura interna moderna de React encargada de gestionar el renderizado y la reconciliación.

React usa Fiber para dividir el trabajo en pequeñas tareas, priorizar actualizaciones, pausar y reanudar renders y evitar bloquear la interfaz.

Fiber permite que React renderice de forma más eficiente y flexible.

Gracias a Fiber, React puede implementar características modernas como:

* Concurrent Rendering,
* Suspense,
* y Transitions.

<span style="background-color:red; color:white">TODO</span> Esto lo veremos mas adelante..

**Nota:**
> Cuando decimos que Fiber es una “arquitectura”, nos referimos a la forma interna en la que React está organizado para resolver el renderizado y las actualizaciones. Es decir, las estructuras de datos que usa, cómo recorre componentes, cómo divide tareas, cómo prioriza trabajo y cómo procesa renders.

> Ademas "Gestionar el renderizado” significa controlar todo el proceso mediante el cual React, detecta cambios, vuelve a ejecutar componentes, compara la UI anterior con la nueva, decide qué actualizar y aplica cambios al DOM real.

#### En Resumen: Virtual DOM, Reconciliation, Diffing y Fiber

React utiliza un Virtual DOM, que es una representación liviana del DOM real hecha con objetos JavaScript. Cuando el estado o las props cambian, React genera un nuevo árbol virtual y realiza un proceso llamado reconciliación (Reconciliation), donde compara el Virtual DOM anterior con el nuevo para determinar qué partes de la interfaz cambiaron.

Para realizar esa comparación, React utiliza un Diffing Algorithm, un algoritmo optimizado que detecta diferencias entre ambos árboles virtuales y permite actualizar únicamente las partes necesarias del DOM real de manera eficiente.

Todo este proceso es gestionado internamente por Fiber, la arquitectura moderna de React encargada de administrar el renderizado y la reconciliación, permitiendo dividir, priorizar y optimizar las actualizaciones de la interfaz de usuario.

## Renders, Commits y Efectos: _las tres fases del ciclo_

En React, una actualización normalmente pasa por tres fases principales: **Render, Commit, Effects**.
Nota: 
> Con "Actualización" nos referimos a cualquier cambio que obliga a React a volver a renderizar parte de la interfaz.

### Render Phase

La Render Phase es la fase donde React calcula cómo debería verse la interfaz según el estado y las props actuales.

En esta fase React:

* ejecuta componentes,
* genera el nuevo Virtual DOM,
* compara cambios mediante reconciliación,
* y determina qué debe actualizarse.

React todavía NO modifica el DOM real.

### Commit Phase

La Commit Phase es la fase donde React aplica al DOM real los cambios calculados durante el render.

En esta fase React:

* crea nodos,
* elimina nodos,
* actualiza atributos,
* modifica texto,
* y sincroniza finalmente la UI visible.

Acá React sí toca el DOM real.

### Effects Phase

La Effects Phase es la fase donde React ejecuta efectos secundarios después de actualizar la interfaz en el DOM real. 

En esta fase React ejecuta principalmente el código contenido dentro de los `useEffect`.

Acá suelen realizarse operaciones como:
* fetch de datos,
* timers,
* subscriptions,
* listeners,
* y otras operaciones externas o side effects.

Ocurre después del commit porque la UI ya está actualizada.

### En Resumen: *Las 3 fases del ciclo*
En React, una actualización normalmente pasa por tres fases principales: Render, Commit y Effects. La **Render Phase** es donde React ejecuta los componentes, genera el nuevo Virtual DOM y compara cambios mediante reconciliación para calcular cómo debería verse la interfaz. En esta fase React todavía no modifica el DOM real.

Luego ocurre la **Commit Phase**, donde React aplica al DOM real los cambios calculados durante el render, actualizando finalmente la interfaz visible. Después llega la **Effects Phase**, donde React ejecuta los efectos secundarios, principalmente el código dentro de los `useEffect`, como fetch de datos, subscriptions, listeners, timers y otras operaciones externas.

### Esto es de la doc oficial de React, cabe aclarar que le falta la fase de efecto, pero es interesante leerlo
El proceso de mostrar componentes en pantalla en React se divide en tres pasos fundamentales que garantizan que la interfaz de usuario sea consistente y eficiente: **disparar (trigger)**, **renderizar (render)** y **confirmar (commit)**.

### Paso 1: Disparar un renderizado
Existen dos motivos principales por los cuales un componente inicia este proceso:
*   **Renderizado inicial:** Ocurre cuando la aplicación arranca por primera vez. Se realiza llamando a `createRoot` con el nodo de destino del DOM y ejecutando su método `render` con el componente raíz.
*   **Actualizaciones de estado:** Una vez que la aplicación ya se ha renderizado inicialmente, cualquier actualización en el **estado** del componente (o de uno de sus ancestros) mediante una función de actualización (`set`) dispara automáticamente un nuevo renderizado. Estas actualizaciones se imaginan como pedidos adicionales que un cliente hace en un restaurante después del primer pedido.

### Paso 2: React renderiza los componentes
En esta etapa, React llama a tus componentes para determinar qué debe mostrarse en la pantalla. **"Renderizar" es, básicamente, React llamando a las funciones de tus componentes**.
*   **Proceso recursivo:** Si un componente devuelve otros componentes, React renderizará esos componentes a continuación. Este proceso continúa de forma recursiva hasta que no queden más componentes anidados y React sepa exactamente qué debería mostrarse.
*   **Cálculo puro:** El renderizado debe ser siempre una **operación pura**. Esto significa que:
    *   Debe devolver el **mismo JSX** ante las mismas entradas.
    *   No debe modificar objetos o variables que existieran antes del renderizado (debe "ocuparse de sus propios asuntos").
*   **Optimización:** Durante un re-renderizado, React calcula cuáles de las propiedades han cambiado desde el renderizado anterior, pero no hace nada con esa información hasta el siguiente paso.

### Paso 3: React confirma los cambios en el DOM
Después de realizar los cálculos del paso anterior, React procede a modificar el DOM real:
*   **Para el renderizado inicial:** React utiliza la API `appendChild()` del DOM para insertar todos los nodos que ha creado en la pantalla.
*   **Para los re-renderizados:** React aplica únicamente las **operaciones mínimas necesarias** (calculadas durante el renderizado) para que el DOM coincida con la salida más reciente.
*   **Eficiencia:** React **solo cambia los nodos del DOM si existe una diferencia** entre el renderizado actual y el anterior. Por ejemplo, si un componente muestra la hora y tiene un campo de texto, React solo actualizará el texto de la hora y no tocará el contenido del campo de texto si este no ha cambiado en el código.

### Epílogo: Pintado del navegador
Una vez que React ha terminado de actualizar el DOM y el ciclo de "renderizado y confirmación" concluye, el navegador realiza el proceso de **"pintado" (painting)** de la pantalla para que el usuario pueda ver los cambios visualmente.


## ¿Por qué React re-renderiza y cuándo no debería?

### ¿Por qué React re-renderiza?

React re-renderiza un componente cuando detecta un cambio que podría afectar la interfaz.

Las causas principales son:

* cambio de estado (`useState`),
* cambio de props,
* cambio de contexto (`Context API`),
* re-render del componente padre,
* o actualización forzada.

Cuando eso ocurre, React vuelve a ejecutar el componente para calcular nuevamente la UI.

### Importante

Re-renderizar NO significa recrear todo el DOM, ni volver a pintar toda la página. Significa volver a ejecutar el componente para obtener una nueva representación virtual de la UI.
Después React decide mediante reconciliación si realmente necesita actualizar el DOM real.

### ¿Cuándo NO debería re-renderizar?

Idealmente, un componente no debería re-renderizar si:

* sus props no cambiaron,
* su estado no cambió,
* su contexto no cambió,
* y su UI resultante sería exactamente la misma.

Porque renderizar innecesariamente consume CPU, genera trabajo extra y puede afectar rendimiento en aplicaciones grandes.

React prefiere renderizar de más antes que arriesgar inconsistencias en la interfaz.
Por eso el re-render es algo normal en React, no un error.
