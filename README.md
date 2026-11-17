# StudyGoAbroad — Frontend

Sistema de gestión interna de StudyGoAbroad, agencia de viajes de estudio al exterior. Lo usan dos personas del equipo (Adriana y Coni), desde la computadora y el celular, para administrar pasajeros, programas, viajes, vuelos, pagos, instituciones y pasaportes.

El frontend consume la API REST de `StudyGoAbroadBackend` (NestJS).

> **Estado:** en etapa de diseño. El repo todavía conserva código de la plantilla original (Books, Turnos/Pedidos, Miembros, SignUp), que se va a quitar al implementar el diseño (ver [Hoja de ruta](#hoja-de-ruta)).

## Diseño (Figma = fuente de verdad)

- **Archivo de Figma:** https://www.figma.com/design/hRVqF2oLap3K5bQ5XT9KEI
- **Seguimiento en ClickUp:**
  - [[F1] Diseñar el prototipo en Figma](https://app.clickup.com/t/86e3rvh9n)
  - [[F2] Preparar el frontend para implementar el diseño](https://app.clickup.com/t/86e3rvhdx)

Las pantallas se diseñan y aprueban primero en Figma, y después se implementan en este repo **tal cual están en Figma**. Para que eso sea posible:

- Colores, tipografía y radios salen solo de [`src/theme.ts`](src/theme.ts). Si el diseño necesita un valor nuevo, primero se agrega al tema.
- Cada componente de Figma corresponde a un componente React:

  | Figma | Código |
  |---|---|
  | Button (Primary, Primary icon, Outlined, Text, Danger) | MUI `Button` |
  | Chip (Success, Warning, Error, Info, Primary, Neutral) | MUI `Chip` |
  | TextField (Default, Focused, Error) | MUI `TextField` outlined |
  | StatCard | `components/StatCard` *(a crear)* |
  | Sidebar / SidebarItem | `layouts/components/DrawerList` + `MenuItem` |
  | AppBar | `layouts/components/AppBar` |

- Íconos: Material Icons (`@mui/icons-material`).
- Pantallas en dos anchos: escritorio (1440 px) y celular (390 px, breakpoint `md` de MUI).
- Los textos de Figma son los definitivos, en español de Argentina.
- Los nombres de campos siguen los de la base de datos en camelCase (`nombres`, `apellido`, `dni`, `fechaNacimiento`, `montoAbonado`, etc.).

## Stack

| Herramienta | Versión | Rol |
|---|---|---|
| React | 18 | UI |
| Vite | 6 | Bundler / dev server |
| TypeScript | 5.6 (modo estricto) | Tipado |
| MUI | 6.4 | Componentes UI + tema |
| MUI X Date Pickers | 7 | Selectores de fecha |
| TanStack Query | 5 | Estado del servidor / caché |
| React Router | 7 | Ruteo (importar desde `react-router`) |
| React Hook Form | 7 | Formularios |
| Axios | 1.7 | Cliente HTTP |
| dayjs | 1.11 | Fechas |

## Primeros pasos

Requisitos: Node.js 20 o superior y el backend corriendo localmente.

```bash
# 1. Crear el archivo de entorno con la URL del backend
echo "VITE_SERVICE_URL=http://localhost:3001" > .env

# 2. Instalar dependencias
npm install

# 3. Iniciar el servidor de desarrollo
npm run dev
```

## Scripts

| Comando | Descripción |
|---|---|
| `npm run dev` | Servidor de desarrollo con HMR |
| `npm run build` | Build de producción (incluye chequeo de tipos) |
| `npm run preview` | Sirve el build localmente |
| `npm run lint` | ESLint |
| `npm run type-check` | Chequeo de tipos de TypeScript |

## Variables de entorno

| Variable | Descripción | Ejemplo |
|---|---|---|
| `VITE_SERVICE_URL` | URL base de la API de NestJS | `http://localhost:3001` |

El archivo `.env` está en `.gitignore`; no subir valores reales al repo.

## Estructura de carpetas

```
src/
├── components/            # Componentes reutilizables (Modal, Notification, Table)
├── contexts/
│   ├── AuthContext/       # Usuario logueado, login y logout
│   └── ViewContext/       # Notificaciones, modales y elección de layout
├── layouts/
│   ├── app/               # Layout con sesión iniciada (AppBar + sidebar)
│   ├── base/              # Layout de páginas públicas (login)
│   ├── components/        # AppBar, SideDrawer, DrawerList, MenuItem, Footer
│   └── constants/         # menuList.ts (ítems del menú lateral)
├── lib/
│   ├── constants/         # routes.ts (ROUTES)
│   ├── dto/               # Cuerpos de las requests (Create*/Update*DTO)
│   ├── hooks/             # Hooks de datos con TanStack Query (useX)
│   │   └── contextHooks/  # useAuthContext, useViewContext
│   ├── responses/         # Tipos de las respuestas del backend (I*Response)
│   ├── services/          # Llamadas con Axios (httpGET*, httpPOST*, …)
│   ├── types/forms/       # Tipos de los formularios de react-hook-form
│   └── utils/             # format.ts (fechas y montos)
├── pages/                 # Una carpeta por pantalla: pages/<Pantalla>/index.tsx
├── providers/             # Árbol de providers (Query, tema, router, auth, vista, fechas)
├── theme.ts               # Tokens de diseño y tema de MUI
└── main.tsx               # Punto de entrada
```

## Cómo fluyen los datos

Las páginas nunca llaman a Axios directamente:

```
pages/Pasajeros  →  lib/hooks/usePasajeros.ts  →  lib/services/pasajeros.ts  →  API NestJS
                     (useQuery / useMutation)      (axios.get('/pasajeros'))
```

- **Servicios** (`lib/services`): una función por endpoint, llamada `httpGETPasajeros`, `httpPOSTPasajero`, `httpPATCHPasajero`, `httpDELETEPasajero`, etc.
- **Hooks** (`lib/hooks`): envuelven los servicios con `useQuery` / `useMutation` y, después de cada alta, edición o baja, invalidan la `queryKey` correspondiente.
- **Tipos:** `lib/dto` para lo que se envía, `lib/responses` para lo que se recibe.

## Autenticación

JWT guardado en `localStorage`:

1. `POST /auth/login` con `{ usuario, password }` devuelve el token, que se guarda en `localStorage`.
2. Un interceptor de Axios agrega `Authorization: Bearer <token>` a cada request ([`src/providers/index.tsx`](src/providers/index.tsx)).
3. `GET /auth` valida el token y devuelve el usuario actual.
4. Si el backend responde `401`, se borra el token y se redirige a `/login`.

Las rutas privadas se envuelven con `PrivateRoute` en [`src/App.tsx`](src/App.tsx). Los ítems del menú se pueden restringir con `adminOnly` y `activeOnly` en `menuList.ts`.

## Agregar una pantalla

1. Crear `src/pages/MiPantalla/index.tsx`.
2. Agregar la ruta en `src/lib/constants/routes.ts`.
3. Agregar el `<Route>` (dentro de `PrivateRoute` si requiere sesión) en `src/App.tsx`.
4. Agregar el ítem en `src/layouts/constants/menuList.ts`, si va en el menú.

## Notificaciones y modales

Desde cualquier componente:

```tsx
const { notification, modal } = useViewContext();

notification.show({ content: 'Pasajero guardado', severity: 'success' });
modal.show({ title: 'Eliminar pasajero', content: <ConfirmarBaja /> });
modal.hide();
```

## Tema

Todos los valores visuales están en [`src/theme.ts`](src/theme.ts). Para cambiar la apariencia, editá solo la sección de tokens al inicio del archivo; los overrides de MUI se actualizan solos.

| Token | Valor | Uso |
|---|---|---|
| `sidebar.background` | `#0F172A` | Sidebar y AppBar |
| `primary.main` | `#4F46E5` | Botones, acentos, ítem activo del menú |
| `page.background` | `#F8FAFC` | Fondo de la app |
| `page.surface` | `#FFFFFF` | Cards, tablas, diálogos |
| `text.primary` | `#1E293B` | Texto principal |
| `success` / `warning` / `error` / `info` | `#16A34A` / `#D97706` / `#DC2626` / `#0284C7` | Estados (al día, por vencer, vencido, información) |

Tipografía: **Inter**. Tamaños permitidos: 12, 14, 16, 20, 24 y 32 px. Pesos: 400, 500, 600 y 700.

> La fuente Inter todavía no se carga en `index.html`. Hasta que se agregue, la app usa la fuente del sistema.

## Módulos

| Módulo | Contenido |
|---|---|
| Inicio | Bienvenida, buscador de pasajeros y accesos rápidos |
| Pasajeros | Alta, baja y edición; ficha con datos, pasaporte, viajes y pagos |
| Programas | Programas de viaje: tipo, destino, duración y monto total |
| Viajes | Asignación de pasajero + programa + vuelo internacional (obligatorio) + vuelo nacional (opcional) |
| Vuelos | Vuelos nacionales e internacionales |
| Pagos | Cuotas abonadas por pasajero. El saldo pendiente se calcula (total del programa − pagos), no se guarda |
| Instituciones | Colegios e institutos de origen |
| Pasaportes | Vencimientos, con aviso de pasaportes por vencer y vencidos |
| Administración | Usuarios, roles, países, provincias, ciudades, aeropuertos y tipos de programa (solo administradores) |

## Hoja de ruta

1. **Diseño en Figma** ([F1](https://app.clickup.com/t/86e3rvh9n)): en curso.
2. **Preparar el frontend** ([F2](https://app.clickup.com/t/86e3rvhdx)):
   - Quitar el código de la plantilla.
   - Cargar Inter.
   - Localizar MUI y dayjs a español.
   - Implementar el layout de Figma (sidebar en escritorio, menú hamburguesa en celular).
   - Crear los componentes base y `.env.example`.
3. **Implementar cada módulo** según su pantalla aprobada en Figma.
