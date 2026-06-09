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

---

# `createBrowserRouter` y estructura de rutas

A partir de React Router v6.4, la forma recomendada de configurar una aplicación es mediante los llamados **Data Routers**.

La pieza central de este sistema es:

```tsx
createBrowserRouter(...)
```

que permite definir toda la estructura de navegación de la aplicación en un único árbol de rutas.

## El modelo mental

Antes de React Router v6.4 era común ver:

```tsx
<BrowserRouter>
  <Routes>
    <Route />
    <Route />
    <Route />
  </Routes>
</BrowserRouter>
```

El router estaba distribuido dentro del árbol de componentes.

Con Data Router la filosofía cambia:

```text
Definir árbol de rutas
↓
Crear router
↓
Entregar router a React
```

Es decir, primero definimos la estructura de navegación y luego React Router se encarga de ejecutarla.

## Crear el router

Supongamos una aplicación sencilla:

```text
/
about
contact
```

Podemos definirla así:

```tsx
import { createBrowserRouter } from "react-router-dom"

const router =
  createBrowserRouter([
    {
      path: "/",
      element: <HomePage />
    },
    {
      path: "/about",
      element: <AboutPage />
    },
    {
      path: "/contact",
      element: <ContactPage />
    }
  ])
```

Cada objeto representa una ruta.

## ¿Qué contiene una ruta?

La forma mínima es:

```tsx
{
  path: "/about",
  element: <AboutPage />
}
```

donde:

```text
path
↓
URL

element
↓
Componente a renderizar
```

## RouterProvider

Una vez creado el router debemos conectarlo con React:

```tsx
import { RouterProvider } from "react-router-dom"

function App() {
  return (
    <RouterProvider router={router} />
  )
}
```

Normalmente este componente reemplaza al antiguo:

```tsx
<BrowserRouter>
```

## ¿Qué ocurre internamente?

Cuando el usuario visita:

```text
/about
```

React Router:

```text
Lee URL actual
↓
Busca coincidencia
↓
Encuentra ruta "/about"
↓
Renderiza AboutPage
```

Conceptualmente:

```text
URL
↓
Matching
↓
Route
↓
Element
```

## El árbol de rutas

La verdadera potencia aparece cuando dejamos de pensar en rutas aisladas y empezamos a pensar en un árbol.

Por ejemplo:

```text
/
├─ about
├─ products
│  ├─ 1
│  ├─ 2
│  └─ 3
└─ contact
```

React Router modela exactamente esa estructura.

## Rutas anidadas

Podemos definir:

```tsx
const router =
  createBrowserRouter([
    {
      path: "/",
      element: <RootLayout />,
      children: [
        {
          index: true,
          element: <HomePage />
        },
        {
          path: "about",
          element: <AboutPage />
        },
        {
          path: "contact",
          element: <ContactPage />
        }
      ]
    }
  ])
```

Observa:

```text
RootLayout
├─ HomePage
├─ AboutPage
└─ ContactPage
```

Ya estamos describiendo una jerarquía.

## ¿Por qué es importante?

Porque la mayoría de las aplicaciones tienen algo así:

```text
Navbar
Sidebar
Footer
```

que permanece visible en todas las páginas.

No queremos repetir:

```tsx
<Navbar />
<Footer />
```

en cada componente.

Queremos un layout compartido.

## El árbol representa la UI

Una idea muy importante:

El árbol de rutas suele reflejar el árbol visual de la aplicación.

Por ejemplo:

```text
App
├─ Layout
│  ├─ Home
│  ├─ Products
│  └─ Contact
```

se traduce naturalmente a:

```tsx
{
  element: <Layout />,
  children: [...]
}
```

## index routes

Una duda muy frecuente.

Si tenemos:

```tsx
{
  path: "/",
  element: <Layout />,
  children: [
    {
      index: true,
      element: <HomePage />
    }
  ]
}
```

¿Qué significa?

Significa:

Cuando estoy exactamente en `"/"`

renderiza: `<HomePage />`

Es equivalente a la página principal de esa sección.

## Ventajas de createBrowserRouter

### Centralización

Toda la navegación vive en un único lugar.

```text
Router
↓
Rutas
↓
Jerarquía completa
```

### Escalabilidad

Es fácil agregar:

```text
Loaders
Actions
Error Boundaries
```

porque todo pertenece a la definición de la ruta.

### Mejor modelo mental

En lugar de pensar:

```text
Tengo componentes con rutas
```

pensamos:

```text
Tengo una aplicación estructurada como un árbol de navegación
```

## Comparación con Context

Hay una analogía interesante.

Context:

```text
Provider
↓
Árbol React
```

React Router:

```text
Route
↓
Árbol de navegación
```

En ambos casos existe una jerarquía donde los nodos superiores proporcionan comportamiento a los inferiores.

### En resumen

`createBrowserRouter` es el punto de entrada del sistema moderno de routing de React Router. Su función es definir un árbol completo de rutas que describe cómo está organizada la navegación de la aplicación. Cada ruta puede contener componentes, rutas hijas y posteriormente loaders, actions y manejo de errores. El modelo mental correcto no es pensar en páginas aisladas, sino en una jerarquía de rutas que refleja la estructura visual y funcional de la aplicación.
