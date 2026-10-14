# LIFE OS

Tu sistema operativo personal: metas, hábitos, tareas, salud, finanzas, aprendizaje y reflexión en un solo lugar.

LIFE OS es **local-first**. Todos tus datos viven en tu navegador (IndexedDB). No hay servidor, cuentas, sincronización ni tracking. GitHub Pages solo aloja los archivos estáticos de la app; tus datos nunca llegan a GitHub.

> **Advertencia sobre tus datos.** Borrar los datos del sitio, usar navegación privada o desinstalar el navegador elimina la información de forma permanente. La única forma de recuperarla, o de llevarla a otro dispositivo, es un backup exportado desde **Settings → Backup**. IndexedDB no está cifrado.

## Estado del proyecto

| Fase | Contenido | Estado |
|---|---|---|
| 1. Arquitectura | Diseño, modelo de datos, roadmap | ✅ |
| 2. Foundation | Proyecto, design system, layout, navegación, dark mode, IndexedDB, Settings, backup, dashboard base, command palette, CI/CD | ✅ |
| 3. Core | Life Areas, Goals, Habits, Tasks (3A) · Calendar (3B) | ⏳ |
| 4. Health | Nutrition, Fitness | ⏳ |
| 5. Finance | Cuentas, transacciones, presupuestos, gráficas | ⏳ |
| 6. Growth | Journal, Learning, Weekly Review | ⏳ |
| 7. Analytics | Estadísticas globales | ⏳ |
| 8. Polish | Accesibilidad, rendimiento, PWA, backups cifrados | ⏳ |

Los módulos que aún no existen muestran una página que explica qué harán y en qué fase llegan. No hay pantallas que simulen funcionar.

## Desarrollo local

Requisitos: **Node.js 22 o superior** y npm.

```bash
npm install
npm run dev        # http://localhost:5173
```

| Script | Qué hace |
|---|---|
| `npm run dev` | Servidor de desarrollo con recarga en caliente |
| `npm run typecheck` | Verifica tipos (TypeScript estricto) |
| `npm run lint` | ESLint |
| `npm test` | Tests de Vitest (cálculos, base de datos, backups) |
| `npm run build` | Build de producción en `dist/` |
| `npm run preview` | Sirve `dist/` localmente para revisarlo |

Para probar el build con la misma ruta que tendrá en GitHub Pages:

```bash
BASE_PATH=/life-os/ npm run build && npm run preview
# abre http://localhost:4173/life-os/
```

## Desplegar en GitHub Pages

1. Crea un repositorio en GitHub, por ejemplo `life-os`. Puede ser privado si tu plan lo permite con Pages; si no, que sea público. El código no contiene datos personales.
2. Sube el proyecto:
   ```bash
   git init
   git add .
   git commit -m "LIFE OS: Fase 2 (Foundation)"
   git branch -M main
   git remote add origin https://github.com/<tu-usuario>/life-os.git
   git push -u origin main
   ```
3. En GitHub ve a **Settings → Pages → Build and deployment → Source** y elige **GitHub Actions**.
4. Abre la pestaña **Actions**. El workflow *CI & Deploy* corre typecheck, lint, tests y build. **Si algo falla, no se despliega.**
5. Al terminar, tu app estará en `https://<tu-usuario>.github.io/life-os/`.

La ruta base se toma del nombre del repositorio, así que puedes llamarlo como quieras. Las rutas usan hash (`/life-os/#/settings`), por lo que recargar cualquier página funciona sin trucos de `404.html`.

### Advertencia: origen compartido en github.io

El navegador aísla IndexedDB por *origen*, y **todos** tus proyectos en `<tu-usuario>.github.io/*` comparten el mismo origen. Cualquier otra página que publiques en ese dominio podría leer la base de LIFE OS. Para evitarlo, no publiques código de terceros en tu user site, o usa un dominio propio para LIFE OS (GitHub Pages lo soporta gratis en *Settings → Pages → Custom domain*).

Ten en cuenta también que cambiar de dominio o de nombre de repositorio cambia el origen, y el navegador no verá tus datos anteriores. Exporta un backup antes de hacerlo e impórtalo después.

## Backups

- **Exportar:** *Settings → Backup → Export backup* descarga `life-os-backup-AAAA-MM-DD-HHmm.json` con todos tus datos.
- **Restaurar:** *Restore from backup…* valida el archivo campo por campo, te muestra un resumen y pide confirmación. Antes de reemplazar, descarga automáticamente un backup de lo que tienes (`…-before-import.json`). La escritura es atómica: si algo falla, no cambia nada.
- **Versiones:** cada backup guarda su `schemaVersion`. Los backups de versiones anteriores se migran al importarlos; los de versiones más nuevas se rechazan con un mensaje claro.
- **Recordatorio:** si pasan 14 días sin backup, el dashboard te lo recuerda.
- Restaurar **reemplaza** todo. Fusionar dos backups todavía no está soportado.
- Los backups contienen información personal (finanzas, reflexiones). Guárdalos en un lugar seguro y **nunca los subas al repositorio**; el `.gitignore` ya excluye `life-os-backup-*.json`.

## Atajos de teclado

| Atajo | Acción |
|---|---|
| `⌘K` / `Ctrl+K` | Buscar módulos y acciones |
| `N` | Quick Add |
| `G` y luego una letra | Ir a un módulo: `O` Overview, `C` Calendar, `G` Goals, `T` Tasks, `A` Life Areas, `H` Habits, `E` Nutrition, `W` Fitness, `F` Finance, `J` Journal, `L` Learning, `R` Weekly Review, `Y` Analytics, `S` Settings |
| `?` | Lista de atajos |
| `Esc` | Cerrar cualquier panel |

Los atajos se pausan mientras escribes en un campo.

## Arquitectura

```
src/
  app/            router, providers, layout raíz, navegación, pantallas de error
  components/
    ui/           componentes base (button, dialog, sheet, inputs, tabs…)
    layout/       sidebar, navegación móvil, tema
    shared/       EmptyState, StatCard, PageHeader, ConfirmDialog
  features/
    dashboard/    saludo, widgets, personalización
    settings/     perfil, preferencias, almacenamiento, backup
    command/      command palette y atajos
    quick-add/    esqueleto de Quick Add
    placeholder/  páginas de módulos aún no construidos
  lib/
    database/     Dexie (db.ts), registro de tablas, migraciones, siembra, repositorios
    calculations/ funciones puras y testeadas (dinero, …)
    validation/   schemas Zod de todas las entidades
    export/       exportación e importación de backups
    dates.ts      fechas locales sin desfase UTC
  types/          tipos derivados de los schemas Zod
  styles/         tokens y estilos globales
```

**Reglas que sigue el código:**

- Ningún componente habla directamente con Dexie para escribir: lo hacen los repositorios, que validan con Zod.
- Ningún cálculo vive dentro de un componente: está en `lib/` como función pura con tests.
- Un mismo schema Zod valida formularios, escrituras y backups importados.
- **Dinero:** enteros en unidades menores (`12050` = $120.50). Nunca floats.
- **Fechas de calendario:** `"AAAA-MM-DD"` en hora local. **Instantes:** ISO 8601.
- **Schema:** cada cambio es una migración nueva en `lib/database/migrations.ts` (con su versión para backups) y sube `SCHEMA_VERSION`. Nunca se edita una versión publicada.
- **localStorage** solo guarda la preferencia de tema (`life-os:theme`), para aplicarla antes del primer render. Ningún dato personal.

**Stack:** React 19, TypeScript, Vite 8, Tailwind CSS 4, componentes estilo shadcn/ui sobre Radix, Lucide, Dexie 4, React Router 8 (HashRouter), Zod 4, React Hook Form, date-fns, cmdk, sonner, dnd-kit, Vitest. Fuentes Geist autoalojadas: la app no hace peticiones a servicios externos.

## Cómo probar la Fase 2

1. **Persistencia:** cambia tu nombre en Settings y recarga; el saludo lo conserva.
2. **Tema:** cambia entre claro, oscuro y sistema y recarga; no debe haber destello blanco.
3. **Dashboard:** en *Customize dashboard*, oculta un widget y arrastra otro (o usa el teclado: `Espacio` y flechas). Recarga: el orden y la visibilidad se conservan. *Reset to default* lo restaura.
4. **Backup en otro navegador:** exporta un backup, ábrelo en otro navegador (o borra los datos del sitio) y restáuralo desde Settings. Todo vuelve, incluidos nombre, tema y layout.
5. **Archivo inválido:** intenta restaurar cualquier otro JSON; se rechaza sin tocar tus datos y explica por qué.
6. **Atajos:** `⌘K`/`Ctrl+K`, `N`, `?`, `G H`, etc.
7. **Móvil:** abre la app en el teléfono; usa la barra inferior y *More*.
8. **Rutas:** recarga en `#/areas` o `#/settings`; no da 404.
9. **CI:** haz push a `main` y revisa que el workflow pase y despliegue.
