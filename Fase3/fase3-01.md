# Routing con React Router v6
*Navegación en SPAs*

Una aplicación React tradicional es una **SPA (Single Page Application)**.

Eso significa que, desde la perspectiva del navegador, normalmente existe una única página HTML:

```text
index.html
```

Cuando el usuario navega entre:

```text
/
about
products
contact
```

el navegador no descarga una página nueva en cada navegación. En su lugar, React actualiza la interfaz y React Router sincroniza la URL con los componentes que deben mostrarse.

Podemos pensar que React Router responde a la pregunta:
_**Dada esta URL, ¿qué componentes debo renderizar?**_


## El problema que resuelve

Sin un router tendríamos que hacer algo parecido a:

```tsx
if (pathname === "/") {
  return <Home />
}

if (pathname === "/about") {
  return <About />
}
```

Lo cual rápidamente se vuelve inmanejable.

React Router permite definir una estructura declarativa de rutas:

```text
/
├─ Home
├─ About
├─ Products
└─ Contact
```

y delega toda la navegación, coincidencia de rutas y renderizado al sistema de routing.

## El modelo mental moderno

Muchos desarrolladores piensan que React Router solamente hace:

```text
URL
↓
Componente
```

Pero desde React Router v6.4 el modelo es más amplio:

```text
URL
↓
Ruta
↓
Loader
↓
Componente
↓
Action
↓
Error Boundary
```

Es decir, una ruta puede definir:

* Qué componente renderizar.
* Qué datos cargar.
* Cómo procesar formularios.
* Cómo manejar errores.
* Qué rutas hijas contiene.

Por eso actualmente React Router se parece mucho más a un framework de routing que a una simple librería de navegación.

### En resumen

React Router ya no debe verse únicamente como una herramienta para cambiar de página dentro de una SPA. En su versión moderna, cada ruta se convierte en una unidad completa de navegación que puede definir componentes, carga de datos, acciones, manejo de errores y rutas hijas. Por eso el modelo mental correcto es pensar en un árbol de rutas que describe la estructura completa de la aplicación y no simplemente un conjunto de URLs conectadas a componentes.
