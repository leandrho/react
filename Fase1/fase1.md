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

   