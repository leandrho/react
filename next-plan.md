
## 🚀 Roadmap Definitivo: 75 Días de Sprints (Next.js 16 & React 19)

#### 🏁 FASE 1: El MVP del SaaS Moderno (Días 1 - 45)

*Objetivo: Construir desde cero la estructura híbrida de React 19, persistencia, validaciones estrictas, autenticación y despliegue del software core.*

**Sprint 1: Cimientos, UI y Enrutamiento Asíncrono (Días 1 - 10)**

* **Teoría (20%):** Arquitectura del App Router moderno. Server vs Client Components. El cambio fundamental de Next.js 16: por qué `params` y `searchParams` ahora son promesas y cómo manejarlas con `await` o el hook `use()`.
* **Práctica (80%):**
* Inicializar el proyecto (`create-next-app` con Tailwind CSS y TypeScript estricto).
* Configurar Shadcn UI e instalar componentes atómicos (Button, Input, Card, Dialog, Skeleton).
* Maquetar las vistas públicas (Landing Page) y privadas (Dashboard, perfiles y la vista dinámica `/dashboard/tasks/[id]` aplicando el desempaquetado asíncrono de parámetros).
* *Meta de commits:* Estructura de navegación, layouts anidados y maquetado responsivo completo del sitio.



**Sprint 2: Persistencia, Modelado y Seguridad Perimetral (Días 11 - 22)**

* **Teoría (20%):** Modelado de datos relacionales en PostgreSQL. Ciclo de vida de las migraciones con Prisma. Arquitectura de Auth.js (v5) para Next.js 16 y control de acceso en el Edge vía Middleware.
* **Práctica (80%):**
* Levantar PostgreSQL (local en Docker o remoto en Neon/Supabase).
* Diseñar el esquema de Prisma para un SaaS: tablas `User`, `Account`, `Session` y `Task` (relación 1 a muchos).
* Implementar Auth.js con proveedores OAuth (Google/GitHub) nativos.
* Escribir el `middleware.ts` para interceptar las peticiones y proteger de forma hermética el entorno de `/dashboard`.
* *Meta de commits:* Base de datos migrada, scripts de *seeding* automatizados y flujo de login/redirección blindado.



**Sprint 3: Mutaciones de React 19, Formularios y Acciones (Días 23 - 35)**

* **Teoría (20%):** Server Actions como funciones de primera clase. El nuevo hook **`useActionState`** (reemplazo oficial de `useFormState` en React 19) y manejo nativo de `isPending`. Integración con Zod y React Hook Form. Límites de `<Suspense>` y streaming.
* **Práctica (80%):**
* Escribir las Server Actions independientes para el CRUD (`createTask`, `updateTask`, `deleteTask`).
* Construir formularios de alta complejidad usando **React Hook Form + Zod** conectándolos al ciclo de vida del servidor mediante `useActionState`.
* Manejar los estados de error del servidor y renderizarlos en los inputs correspondientes del cliente sin perder el estado del formulario.
* *Meta de commits:* Mutaciones type-safe operacionales, formularios validados en ambos lados y control absoluto de estados de carga.



**Sprint 4: Optimistic UI, SEO y Deploy a Producción (Días 36 - 45)**

* **Teoría (20%):** Estados transicionales y el hook `useOptimistic` de React 19. Metadata API asíncrona de Next.js. Gestión segura de secretos en producción.
* **Práctica (80%):**
* Implementar **UI Optimista** en las acciones del listado de tareas (marcar como completada o borrar impacta instantáneamente en la interfaz).
* Configurar la función `generateMetadata` para optimizar dinámicamente el SEO y las etiquetas Open Graph del SaaS.
* Auditar variables de entorno y **realizar el deploy oficial a producción en Vercel**.
* *Meta de commits:* Experiencia de usuario instantánea (sin lag de red), SEO pulido y MVP corriendo en vivo en internet.



---

## 🧠 FASE 2: Profesionalización, Escalado y DevOps (Días 46 - 75)

*Objetivo: Elevar el MVP a un estándar de ingeniería Mid/Senior. Refactorizar para tolerar escala, agregar capas de optimización, testing y contenedores.*

**Sprint 5: Refactor Arquitectónico y Estado de UI (Días 46 - 55)**

* **Práctica (100%):**
* Migrar el código monolítico inicial de la raíz hacia una arquitectura modular basada en **Features**, aislando la lógica de negocio del enrutamiento:
```text
src/
  ├── app/            (Solo rutas, layouts y Suspense boundaries)
  ├── features/       (Módulos aislados: tasks, auth, users)
  │     ├── components/
  │     ├── actions.ts (Server Actions del módulo)
  │     └── schemas.ts (Validaciones Zod)
  ├── lib/            (Instancias de clientes globales: Prisma)
  └── store/          (Zustand exclusivamente para estado global de UI)

```


*   Integrar **Zustand** para manejar interacciones ricas de la interfaz cliente (como colapsar páneles laterales o manejar flujos paso a paso).
*   *Meta de commits:* Código modularizado, desacoplado y arquitectura escalable sin errores de TypeScript.

**Sprint 6: Caching Quirúrgico y Patrones de React 19 (Días 56 - 65)**
*   **Teoría (20%):** El renovado modelo de caché de Next.js 16, estrategias de revalidación bajo demanda (`revalidateTag`, `revalidatePath`). El hook **`use()`** de React 19 para resolver promesas directamente en componentes cliente.
*   **Práctica (80%):**
    *   Implementar políticas de caché óptimas e invalidación selectiva con tags al ejecutar mutaciones.
    *   Refactorizar componentes pesados para transmitir datos en streaming mediante promesas directo al cliente, resolviéndolas de manera elegante en el render con el hook `use()`.
    *   Optimizar recursos críticos del navegador mediante `next/image` y `next/font`.
    *   *Meta de commits:* Rendimiento extremo (puntuación Lighthouse óptima) y control absoluto sobre los datos en memoria.

**Sprint 7: Calidad, Automatización (CI/CD) y Dockerización (Días 66 - 75)**
*   **Teoría (20%):** Filosofía de testing para arquitecturas híbridas servidor/cliente. Contenerización standalone para Next.js.
*   **Práctica (80%):**
    *   Configurar **Vitest** y escribir pruebas unitarias sobre las funciones puras de utilidad y esquemas Zod.
    *   Configurar **Playwright** y programar un test de extremo a extremo (E2E) que cubra el flujo crítico del negocio: Autenticación exitosa -> Redirección -> Creación de Tarea -> Verificación de persistencia en la lista.
    *   Escribir el flujo de automatización en **GitHub Actions** (`ci.yml`) para ejecutar tests, linters y chequeos de tipos en cada pull request.
    *   Crear un `Dockerfile` multi-etapa optimizado para compilar en modo standalone.
    *   *Meta de commits:* Cobertura de tests esenciales, pipeline automatizado en verde y la aplicación completamente dockerizada.

---

### 📈 Plan de Consistencia para tus "Commits Diarios"

Para asegurar que cubras los **75 días sin excepciones**, aplicá la regla de la granularidad:
1.  **No commitees features enteras:** En lugar de hacer un commit que diga *"Terminé el sprint 2"*, tus commits diarios deben ser microscópicos e incrementales: *"Día 11: Instalación de prisma y setup de compose local"*, *"Día 12: Definición del modelo User y primera migración"*, *"Día 13: Configuración de los endpoints de Auth.js"*.
2.  **La hora del Push:** Usá los últimos 20 minutos de tus 4 horas diarias pura y exclusivamente para limpiar la consola, verificar que TypeScript compile sin errores con `npm run build` local, escribir el mensaje de commit bajo el estándar *Conventional Commits* (ej: `feat(tasks): implement useActionState in task form`) y subirlo.
