Aquí tienes el **Roadmap Definitivo Blindado**. He integrado todos los micro-ajustes, dividido el Sprint 3 para evitar el "muro de la frustración" y añadido las reglas de oro para que tu portafolio grite "nivel Mid/Senior".

Este documento está listo para ser tu guía diaria. Cópialo, pégalo en tu Notion/Obsidian y empieza.

---

# 🚀 Roadmap Definitivo: 75 Días de Sprints (Next.js 16 & React 19) - Versión Blindada

#### 🏁 FASE 1: El MVP del SaaS Moderno (Días 1 - 45)
*Objetivo: Construir desde cero la estructura híbrida, persistencia, validaciones estrictas, autenticación y despliegue del software core.*

**Sprint 1: Cimientos, UI y Enrutamiento Asíncrono (Días 1 - 10)**
* **Teoría (20%):** Arquitectura del App Router. Server vs Client Components. El cambio fundamental de Next.js 16/React 19: por qué `params` y `searchParams` ahora son promesas y cómo manejarlas con `await` o el hook `use()`.
* **Práctica (80%):**
  * Inicializar el proyecto (`create-next-app` con Tailwind CSS y TypeScript estricto).
  * Configurar Shadcn UI e instalar componentes atómicos (Button, Input, Card, Dialog, Skeleton).
  * Maquetar las vistas públicas (Landing Page) y privadas (Dashboard).
  * Crear la vista dinámica `/dashboard/tasks/[id]` aplicando el desempaquetado asíncrono de parámetros (`const { id } = await params`).
* **Meta de commits:** Estructura de navegación, layouts anidados y maquetado responsivo completo del sitio.

**Sprint 2: Persistencia, Modelado y Seguridad Perimetral (Días 11 - 22)**
* **Teoría (20%):** Modelado de datos relacionales en PostgreSQL. Ciclo de vida de las migraciones con Prisma. **Ojo:** Auth.js v5 usa `auth()` (no `getServerSession`) y requiere *TypeScript Module Augmentation* para tipar la sesión correctamente.
* **Práctica (80%):**
  * Levantar PostgreSQL (local en Docker o remoto en Neon/Supabase).
  * Diseñar el esquema de Prisma: `User`, `Account`, `Session` y `Task` (relación 1 a muchos).
  * Implementar Auth.js v5 con proveedores OAuth (Google/GitHub) siguiendo **exclusivamente** la documentación oficial de `authjs.dev`.
  * Escribir el `middleware.ts` para interceptar peticiones y proteger el entorno de `/dashboard`.
* **Meta de commits:** Base de datos migrada, scripts de *seeding* automatizados y flujo de login/redirección blindado y tipado.

**Sprint 3: Mutaciones, Formularios y Acciones (Días 23 - 35) ⚠️ *Dividido para evitar frustración***
* **Fase A (Días 23-28): Fundamentos puros.**
  * Escribir Server Actions independientes para el CRUD (`createTask`, `updateTask`, `deleteTask`).
  * Construir formularios con **HTML nativo** conectados al servidor mediante el hook `useActionState` (React 19).
  * Aprender a devolver y mostrar errores del servidor en la UI sin perder el estado del formulario.
* **Fase B (Días 29-35): Profesionalización.**
  * Refactorizar los formularios nativos usando **React Hook Form + Zod**.
  * Conectar la validación de Zod en el cliente con la validación de la Server Action en el servidor.
  * Implementar límites de `<Suspense>` y estados de `loading.tsx` durante las mutaciones.
* **Meta de commits:** Mutaciones type-safe operacionales, formularios validados en ambos lados y control absoluto de estados de carga y error.

**Sprint 4: Optimistic UI, SEO y Deploy a Producción (Días 36 - 45)**
* **Teoría (20%):** Estados transicionales y el hook `useOptimistic` de React 19. Metadata API asíncrona. Gestión segura de secretos.
* **Práctica (80%):**
  * Implementar **UI Optimista** en el listado de tareas (ej: al borrar, la tarea desaparece de la UI instantáneamente antes de la respuesta del servidor).
  * Configurar `generateMetadata` para optimizar dinámicamente el SEO y las etiquetas Open Graph.
  * Auditar variables de entorno y **realizar el deploy oficial a producción en Vercel**.
* **Meta de commits:** Experiencia de usuario instantánea (sin lag de red), SEO pulido y MVP corriendo en vivo en internet.

---

#### 🧠 FASE 2: Profesionalización, Escalado y DevOps (Días 46 - 75)
*Objetivo: Elevar el MVP a un estándar de ingeniería Mid/Senior. Refactorizar para tolerar escala, agregar capas de optimización, testing y contenedores.*

**Sprint 5: Refactor Arquitectónico y Estado de UI (Días 46 - 55)**
* **Práctica (100%):**
  * Migrar el código monolítico hacia una arquitectura modular basada en **Features**, aislando la lógica de negocio del enrutamiento:
    ```text
    src/
      ├── app/            (Solo rutas, layouts y Suspense boundaries)
      ├── features/       (Módulos aislados: tasks, auth, users)
      │     ├── components/
      │     ├── actions.ts (Server Actions del módulo)
      │     └── schemas.ts (Validaciones Zod)
      ├── db/             (Instancia global de Prisma y tipos exportados)
      ├── lib/            (Utilidades y funciones puras)
      └── store/          (Zustand exclusivamente para estado global de UI)
    ```
  * Integrar **Zustand** solo para interacciones ricas de la interfaz cliente (ej: colapsar paneles laterales, wizards de varios pasos).
* **Meta de commits:** Código modularizado, desacoplado, sin dependencias circulares y arquitectura escalable sin errores de TypeScript.

**Sprint 6: Caching Quirúrgico y Patrones de React 19 (Días 56 - 65)**
* **Teoría (20%):** Modelo de caché de Next.js, revalidación bajo demanda (`revalidateTag`, `revalidatePath`). Uso del hook `use()` para resolver promesas en componentes cliente.
* **Práctica (80%):**
  * Implementar invalidación selectiva con tags al ejecutar mutaciones (ej: `revalidateTag('tasks')`).
  * **Regla de oro:** Para rutas 100% privadas (como el Dashboard), usar `export const dynamic = 'force-dynamic'` (o `unstable_noStore`) para evitar dolores de cabeza con la caché innecesaria.
  * Refactorizar componentes pesados para transmitir datos en streaming y resolverlos con `use()`.
  * Optimizar recursos con `next/image` y `next/font`.
* **Meta de commits:** Rendimiento extremo (puntuación Lighthouse óptima) y control absoluto sobre los datos en memoria.

**Sprint 7: Calidad, Automatización (CI/CD) y Dockerización (Días 66 - 75)**
* **Teoría (20%):** Filosofía de testing para arquitecturas híbridas. Contenerización *standalone* para Next.js.
* **Práctica (80%):**
  * Configurar **Vitest** y escribir pruebas unitarias sobre funciones puras de utilidad y esquemas Zod.
  * Configurar **Playwright** y programar un test E2E del flujo crítico: Login -> Creación de Tarea -> Verificación en lista.
  * Escribir el flujo de automatización en **GitHub Actions** (`ci.yml`) para ejecutar tests, linters y chequeos de tipos en cada PR.
  * Crear un `Dockerfile` multi-etapa optimizado. **Clave:** Configurar `output: 'standalone'` en `next.config.js` para que la imagen Docker sea mínima y profesional.
* **Meta de commits:** Cobertura de tests esenciales, pipeline automatizado en verde y la aplicación completamente dockerizada.

---

### 📈 Plan de Consistencia: La Regla de los Commits Diarios

Para asegurar que cubras los **75 días sin excepciones**, aplica estas 3 reglas innegociables:

1. **Granularidad Microscópica:** No commitees features enteras. Tus commits diarios deben ser incrementales:
   * ✅ `chore: install prisma and setup local docker compose`
   * ✅ `feat(db): define User and Task models with 1:N relation`
   * ✅ `feat(auth): configure auth.js v5 with google provider`
2. **La Regla de Oro del Build:** **Nunca hagas push a `main` con el build roto.** Tu ritual de los últimos 20 minutos debe ser siempre:
   * `npm run build` (Si falla, se arregla, no se commitea).
   * `git add .`
   * `git commit -m "feat(tasks): implement useActionState in task form"`
   * `git push`
3. **Documenta mientras avanzas:** Cada vez que resuelvas un error difícil (ej: tipar la sesión de Auth.js), escribe 2 líneas en el `README.md` de tu proyecto explicando cómo lo solucionaste. Eso es oro puro para las entrevistas.

