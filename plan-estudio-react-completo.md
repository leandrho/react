# Plan de Estudio React — Nivel Profesional

> **Duración estimada:** 12 semanas · ~80 horas totales  
> **Objetivo:** Dominar React desde sus fundamentos internos hasta el stack profesional, con proyecto integrador para portafolio y postulación laboral.

---

## Fase 1 — Fundamentos sólidos
**Semanas 1–2 · ~15 hs**

### ¿Cómo funciona React por dentro?
*El motor antes que el volante*

- Virtual DOM vs DOM real: qué problema resuelve
- Reconciliation y el algoritmo de diffing
- Fiber: qué es y por qué React lo usa
- Renders, commits y efectos: las tres fases del ciclo
- Por qué React re-renderiza y cuándo no debería

🔗 Ver: Pensamiento React (Core)

📖 Lectura obligatoria: https://react.dev/learn/render-and-commit

---

### JSX a fondo
*No es HTML, no es magia*

- Transpilación a `React.createElement()`
- Fragmentos: `<></>` y `Fragment`
- Expresiones, condicionales y listas en JSX
- Reglas de `key` y por qué importan

---

### Componentes
*La unidad de composición*

- Function components (solo estos, nada de class components)
- Props: paso por valor, desestructuración, defaults
- `children` y composición de componentes
- Componentes puros e impuros
- Cuándo dividir un componente

🔗 Ver: Pensamiento React (Core)

---

### Estado y eventos
*El núcleo de la reactividad*

- `useState`: estado local y actualizaciones
- Actualizaciones funcionales: `prev => ...`
- Estado vs variable local: la diferencia clave
- Manejo de formularios controlados
- Inmutabilidad: por qué no mutar el estado directamente

🔗 Ver: Pensamiento React (Core)

---

## Fase 2 — Hooks y patrones modernos
**Semanas 3–4 · ~20 hs**

### useEffect en profundidad
*El hook más mal entendido*

- Cuándo y cuándo NO usar `useEffect`
- Dependency array: reglas del linter (`eslint-plugin-hooks`)
- Cleanup functions y memory leaks
- Stale closures: qué son y cómo evitarlas
- Efectos vs eventos: la distinción clave

🔗 Ver: Pensamiento React (Core)

📖 Lectura obligatoria: https://react.dev/learn/you-might-not-need-an-effect

---

### Hooks de optimización
*Rendimiento consciente*

- `useRef`: referencias más allá del DOM
- `useMemo`: memoizar cálculos costosos
- `useCallback`: referencias estables de funciones
- `React.memo`: evitar re-renders innecesarios
- Cuándo optimizar (y cuándo es prematuro)

---

### useReducer y useContext
*Estado más complejo*

- `useReducer`: cuándo reemplaza a `useState`
- Context API: crear, proveer y consumir contexto
- Context + useReducer = mini Redux
- Problemas de performance del contexto
- Alternativas modernas: Zustand, Jotai

🔗 Ver: Arquitectura frontend (Core)

---

### Custom Hooks
*Reutilización real de lógica*

- Reglas de hooks y por qué existen
- Extraer lógica a un custom hook
- Hooks para fetching, formularios y local storage
- Testear hooks con React Testing Library

🔗 Ver: Arquitectura frontend (Core)

---

## Fase 3 — Ecosistema profesional
**Semanas 5–7 · ~25 hs**

### Routing con React Router v6
*Navegación en SPAs*

- `createBrowserRouter` y estructura de rutas
- Rutas anidadas y layouts compartidos
- Loaders, actions y manejo de errores
- Rutas dinámicas y params
- Rutas protegidas (auth guards)

🔗 Ver: Manejo de errores en UI (Protip)

📖 https://reactrouter.com

---

### Data fetching moderno
*Más allá del useEffect + fetch*

- TanStack Query (React Query): concepto de caché
- `useQuery`, `useMutation`, invalidación
- Manejo de loading, error y stale data
- Axios vs fetch: cuándo usar cada uno
- Optimistic updates

🔗 Ver: Manejo real de APIs (Core)

🛠 Herramienta: TanStack Query v5

---

### Gestión de estado global
*Cuándo escala el contexto*

- Zustand: store simple y escalable
- Redux Toolkit (si el trabajo lo requiere)
- Qué va al estado global vs local vs server state
- Slice de estado: separar concerns

🔗 Ver: Arquitectura frontend (Core)

---

### Formularios
*La pesadilla, domada*

- React Hook Form: registro, validación, errores
- Zod para validación de esquemas tipados
- Formularios controlados vs no controlados
- Integración con componentes UI externos

🔗 Ver: Accesibilidad (Protip)

🛠 Stack recomendado: React Hook Form + Zod

---

### TypeScript en React
*Requisito laboral hoy*

- Tipar props, estado y eventos
- `FC` vs función con tipo de retorno explícito
- Generics en componentes y hooks
- Utility types: `Partial`, `Pick`, `Omit`, `ReturnType`, etc.
- Tipar respuestas de API correctamente

---

### Styling profesional
*Opciones y criterios*

- Tailwind CSS: utilidades y design system
- CSS Modules: estilos con scope
- shadcn/ui: componentes accesibles y customizables
- Cuándo usar cada opción según el proyecto

🔗 Ver: Accesibilidad (Protip)

---

## Fase 4 — Nivel senior: patrones y arquitectura
**Semanas 8–10 · ~20 hs**

### Testing
*Código que confía en sí mismo*

- Vitest + React Testing Library
- Qué testear: comportamiento, no implementación
- Mocks de módulos, APIs y hooks
- Testing de formularios y rutas
- Coverage sin obsesionarse con el número

---

### Performance avanzada
*Medir antes de optimizar*

- React DevTools Profiler
- Code splitting con `lazy()` y `Suspense`
- Virtualización de listas (TanStack Virtual)
- Web Vitals: LCP, FID, CLS
- Bundle analysis con `vite-bundle-visualizer`

---

### Patrones de diseño
*Código mantenible*

- Compound components
- Render props (dónde todavía tiene sentido)
- HOC: Higher Order Components
- Container / Presentational
- Inversion of control en hooks

🔗 Ver: Arquitectura frontend (Core)

---

### Next.js *(opcional pero muy valioso)*
*React en producción real*

- App Router y Server Components
- SSR, SSG, ISR: cuándo usar cada uno
- Server Actions
- Metadata, SEO y Open Graph
- Deploy en Vercel

---

## Fase 5 — Proyecto integrador + portafolio
**Semanas 11–12 · construcción real**

### Proyecto final
*Todo junto, código real*

Construir una app completa que incluya:

- Autenticación (login/registro)
- CRUD completo
- Rutas con React Router
- Estado global (Zustand o Redux Toolkit)
- Fetching con TanStack Query
- TypeScript desde el inicio
- Tests para los flujos críticos
- Deploy en Vercel o Netlify

🔗 Ver: Manejo real de APIs (Core)  
🔗 Ver: Arquitectura frontend (Core)  
🔗 Ver: Git profesional (Core)

---

### Preparación laboral
*Entrar por la puerta grande*

- Estudiar preguntas frecuentes de entrevistas React
- Practicar code challenges: live coding + take-home
- Escribir un README claro, con descripción, tech stack y screenshots
- GitHub limpio: commits semánticos (feat, fix, chore...)
- Practicar explicar decisiones técnicas en voz alta

🔗 Ver: Git profesional (Core)

---

## 🔗 Módulos complementarios

---

### Core (obligatorio)

#### Arquitectura frontend
- Feature-based folder structure
- UI / domain / data
- Escalabilidad

---

#### Manejo real de APIs
- Interceptors
- Auth flow (JWT)
- Manejo de errores
- Retry strategies

---

#### Git profesional
- Branching
- PRs
- Commits semánticos

---

#### Pensamiento React
- UI = estado
- Derivar estado
- Evitar `useEffect` innecesario

---

### Protip (recomendado)

#### Accesibilidad (A11y)
- aria, focus, teclado

---

#### Manejo de errores en UI
- Error Boundaries
- fallback UI

---

#### Tooling profesional
- ESLint
- Prettier
- aliases
- env

---

### Extra (diferencial)

#### Storybook  
#### WebSockets  
#### PWA  
#### i18n  

---
