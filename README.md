# Entrega: la web de FitVal con React

## Objetivo

Vas a hacer **la web de FitVal**: una aplicación React donde los socios ven los planes y el horario, y el personal del gimnasio crea y borra clases. Al final la conectarás con **tu API de NestJS** y la publicarás en **Netlify**.

Es **una sola entrega** con **dos partes**:

1. **Primera parte (apartados PARTE 0 a 4): solo React.** Los datos están en un `db.json` que te damos y **json-server** los sirve como una API, igual que en los apuntes 03 y 04. Así te centras en React sin pelearte con el backend. Al terminarla, **la cierras con una etiqueta** en GitHub (`v1-json-server`) para que se vea todo tu trabajo de React antes de conectar nada.
2. **Segunda parte (PARTE 5): conecta con tu API.** Cambias json-server por tu API de NestJS. Van a salir errores: es lo normal cuando se juntan un frontend y un backend hechos por separado. Los resuelves tú, con ayuda de la IA, y publicas la web.

Al terminar deberías poder:

- Dividir una página en **componentes** con **props** y darles estilo con **CSS Modules**.
- Pedir datos a una API con `useEffect` + `fetch` y con **RTK Query**.
- Tener varias páginas con **React Router**: rutas con parámetros (`/classes/:id`) y filtros en la URL (`?activity=yoga`).
- Hacer un **login**, guardar la sesión en **Redux** y proteger una página para que solo entre el **staff**.
- Crear y borrar datos con **mutations** y enseñar los errores de la API.
- Escribir **tests** con Vitest y Testing Library y pasarlos en el CI.
- **Conectar** tu web con una API de verdad y resolver los errores de integración (CORS, rutas, formatos, `400`, `401`, `409`), usando la IA con cabeza.
- Publicar la web en **Netlify** y conectarla con tu API de **Render**.

**Tiempo orientativo:** primera parte, 6 horas; segunda parte, 2 o 3 horas.

**Qué vas a entregar:** un repositorio `fitval-web` en GitHub con una PR por cada parte y la etiqueta `v1-json-server`, un archivo `respuestas.md` y la dirección de Netlify. Se entrega **una vez**, al final (detalles en [Entrega](#entrega)).

**Cómo trabajarla:**

- Haz las partes **en orden**. Cada parte termina con una PR que fusionas antes de empezar la siguiente.
- Ten a mano los apuntes de React: [01](../sesiones/frontend/01_react_componentes_props_estado.md), [02](../sesiones/frontend/02_react_formularios_comunicacion.md), [03](../sesiones/frontend/03_react_efectos_api_rutas.md), [04](../sesiones/frontend/04_react_redux_toolkit_login.md) y [05](../sesiones/frontend/05_react_proyecto_profesional.md). Esta entrega usa lo mismo, con otros datos.
- **Los estilos ya están hechos.** Te damos todos los `.css`: cópialos tal cual. Tu trabajo es el código React.
- Verás dos tipos de bloque de código:
  - **Cópialo tal cual**: código de apoyo (datos, estilos, tipos, formatos). No es lo importante de la entrega.
  - **Complétalo**: un esqueleto con comentarios `// TODO`. Ahí está lo que tienes que aprender. Si te atascas, busca el mismo caso en el apunte que se indica.
- En la primera parte, **no uses la IA para escribir el código**: es tu práctica de React. En la segunda sí, con las normas de la Parte 5.
- Después de cada paso, **mira el navegador**. Cada parte tiene capturas de cómo debe quedar.
- Las preguntas son pocas y cortas: respóndelas en `respuestas.md` con una o dos frases.

### Todo en inglés

Igual que en la API: componentes, variables, funciones, tipos, rutas (`/classes`, `/staff`) y nombres de fichero, en **inglés**. Lo que lee la persona que usa la web (textos de los botones, títulos, mensajes) va en **español**.

---

## Lo que vas a construir

| Ruta | Página | Quién entra | Qué pide a json-server (primera parte) |
|---|---|---|---|
| `/` | Planes | todos | `GET /plans` |
| `/classes` | Horario, con buscador | todos | `GET /classes?activity:contains=...` |
| `/classes/:id` | Detalle de una clase | todos | `GET /classes/:id` |
| `/login` | Entrar | todos | `GET /users?email=...` (login simulado) |
| `/register` | Crear cuenta | todos | `POST /users` |
| `/staff` | Panel del personal | solo **staff** | `POST /classes` y `DELETE /classes/:id` |
| cualquier otra | Página no encontrada | todos | — |

En la segunda parte, **tu API** decide las rutas. Algunas no coinciden con las de json-server: encontrarlas y arreglarlas es parte del trabajo.

Así quedará el horario:

![Horario de clases](img/05_react_fitval/02_clases.png)

Y el panel del personal:

![Panel del personal](img/05_react_fitval/09_panel.png)

---

# PARTE 0 — El proyecto, el CI y Netlify

Antes de programar, dejas el proyecto listo: repositorio con CI, `main` protegida y la web publicada en Netlify. Así, cada vez que fusiones una PR, Netlify publicará la nueva versión sola.

## 0.1 Crea el proyecto

```bash
npm create vite@latest fitval-web -- --template react-ts
cd fitval-web
npm install
npm install @reduxjs/toolkit react-redux react-router react-hook-form zod @hookform/resolvers
npm install -D json-server
npm run dev
```

Abre `http://localhost:5173`: verás la página de ejemplo de Vite.

Limpia la plantilla, como en el apunte 01:

- [ ] Borra `src/App.css`, la carpeta `src/assets` y `public/icons.svg`.
- [ ] En `index.html`: `<html lang="es">` y `<title>FitVal</title>`.
- [ ] Deja `src/App.tsx` así, de momento:

```tsx
function App() {
  return <h1>Hola, FitVal</h1>;
}

export default App;
```

- [ ] Sustituye `src/index.css` por los estilos generales de FitVal. **Cópialo tal cual:**

```css
/* ===== Estilos generales de FitVal (se copian tal cual) ===== */
:root {
  --ink: #111827; /* texto */
  --muted: #4b5563; /* texto secundario */
  --line: #e5e7eb; /* bordes */
  --bg: #f8f7f4; /* fondo */
  --accent: #c2410c; /* naranja oscuro: botones y enlaces */
  --accent-light: #fb923c; /* naranja claro: logo y menú */
  --danger: #991b1b;
}

* {
  box-sizing: border-box;
}

body {
  margin: 0;
  font-family: system-ui, sans-serif;
  background: var(--bg);
  color: var(--ink);
}

main {
  max-width: 1120px;
  margin: 0 auto;
  padding: 32px 24px;
}

h1 {
  margin: 0 0 16px;
  font-size: 32px;
}

a {
  color: var(--accent);
}

/* Formularios */
form {
  display: flex;
  flex-direction: column;
  gap: 8px;
  max-width: 420px;
  padding: 24px;
  background: white;
  border: 1px solid var(--line);
  border-radius: 14px;
}

label {
  font-size: 14px;
  font-weight: 600;
}

input,
select {
  font: inherit;
  padding: 10px 12px;
  border: 1px solid #9ca3af;
  border-radius: 8px;
  background: white;
}

button {
  font: inherit;
  font-weight: 600;
  padding: 10px 18px;
  border: 0;
  border-radius: 8px;
  background: var(--accent);
  color: white;
  cursor: pointer;
}

button:disabled {
  opacity: 0.6;
  cursor: default;
}

/* Mensajes */
.error {
  padding: 10px 12px;
  border: 1px solid #fca5a5;
  border-radius: 8px;
  background: #fef2f2;
  color: #7f1d1d;
}

.field-error {
  margin: 0;
  font-size: 13px;
  color: var(--danger);
}

.loading {
  color: var(--muted);
}

/* Rejilla de tarjetas */
.grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(220px, 1fr));
  gap: 20px;
}

/* Tablas */
table {
  width: 100%;
  border-collapse: collapse;
  background: white;
}

th,
td {
  padding: 10px;
  border-bottom: 1px solid var(--line);
  text-align: left;
}
```

Comprueba que `npm run lint` y `npm run build` terminan sin errores.

## 0.2 Los datos: `db.json` y json-server

Igual que en el apunte 03, apartado 5. Crea `db.json` en la **raíz** del proyecto (al lado de `package.json`). Son los mismos datos que el `schema.sql` de tu API. **Cópialo tal cual:**

```json
{
  "plans": [
    {"id": "1", "name": "Básico", "monthlyPrice": 29.9},
    {"id": "2", "name": "Completo", "monthlyPrice": 44.9},
    {"id": "3", "name": "Estudiante", "monthlyPrice": 24.9},
    {"id": "4", "name": "Mañanas", "monthlyPrice": 19.9}
  ],
  "classes": [
    {"id": "1", "activity": "Spinning", "room": "Sala 1", "startsAt": "2026-11-02T09:00:00.000Z", "spots": 12, "trainerId": 1, "trainerName": "Carla Navarro"},
    {"id": "2", "activity": "Yoga", "room": "Sala 2", "startsAt": "2026-11-02T18:00:00.000Z", "spots": 10, "trainerId": 2, "trainerName": "Javier Ortega"},
    {"id": "3", "activity": "Pilates", "room": "Sala 2", "startsAt": "2026-11-03T10:00:00.000Z", "spots": 8, "trainerId": 3, "trainerName": "Elena Romero"},
    {"id": "4", "activity": "Spinning", "room": "Sala 1", "startsAt": "2026-11-03T19:00:00.000Z", "spots": 12, "trainerId": 1, "trainerName": "Carla Navarro"},
    {"id": "5", "activity": "Zumba", "room": "Sala 3", "startsAt": "2026-11-04T18:30:00.000Z", "spots": 20, "trainerId": 3, "trainerName": "Elena Romero"},
    {"id": "6", "activity": "Yoga", "room": "Sala 2", "startsAt": "2026-11-05T09:30:00.000Z", "spots": 10, "trainerId": 2, "trainerName": "Javier Ortega"},
    {"id": "7", "activity": "Crossfit", "room": "Sala 3", "startsAt": "2026-11-05T20:00:00.000Z", "spots": 3, "trainerId": 1, "trainerName": "Carla Navarro"}
  ],
  "users": [
    {"id": "1", "email": "staff@fitval.es", "password": "staff1234", "role": "staff"},
    {"id": "2", "email": "lucia@gym.com", "password": "lucia1234", "role": "member"}
  ]
}
```

> Una API real **nunca** guarda ni devuelve contraseñas así. Es una API de juguete para practicar React.

Haz una copia: `db.initial.json`, con el mismo contenido. Si llenas `db.json` de pruebas, cópiale encima `db.initial.json` y vuelves a empezar.

En `package.json`, añade el script `api`:

```json
"scripts": {
  "dev": "vite",
  "api": "json-server db.json --port 3001",
  ...
}
```

Arráncalo **en otra terminal** (la de `npm run dev` sigue abierta):

```bash
npm run api
```

Prueba en el navegador:

| Abre | Resultado esperado |
|---|---|
| `http://localhost:3001/plans` | Los 4 planes |
| `http://localhost:3001/classes/7` | El Crossfit |
| `http://localhost:3001/classes/99` | `Not Found` (un `404`) |
| `http://localhost:3001/classes?activity:contains=yo` | Las 2 clases de Yoga. `:contains` busca un trozo del texto, sin importar mayúsculas |
| `http://localhost:3001/users?email=lucia@gym.com` | Un array con Lucía |

> Fíjate: los `id` de json-server son **textos** (`"1"`), y el filtro se escribe `activity:contains`. Tu API de NestJS no tiene por qué hacerlo igual. Lo verás en la Parte 5.

## 0.3 Las rutas en Netlify: `_redirects`

Crea `public/_redirects` (sin extensión) con esta línea:

```text
/*  /index.html  200
```

Sin este archivo, al recargar en `/classes` Netlify daría un `404`. El porqué está en el apartado 6 del apunte 05.

## 0.4 Añade el `ci.yml`

Crea `.github/workflows/ci.yml`. Es el CI mínimo de siempre, con el **build** activado (React con TypeScript):

```yaml
# CI mínimo: revisa el código en cada push a main y en cada PR hacia main.
name: ci

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

permissions:
  contents: read

jobs:
  # Check 1: revisa los .html (aquí, index.html)
  htmlhint:
    runs-on: ubuntu-latest
    steps:
      - name: Descargar el código
        uses: actions/checkout@v7
      - name: Preparar Node.js
        uses: actions/setup-node@v7
        with:
          node-version: 24
      - name: Revisar el HTML con HTMLHint
        run: npx --yes htmlhint@1 "**/*.html" --ignore "dist/**,build/**,coverage/**"

  # Check 2: lint (oxlint) y build (comprueba los tipos de TypeScript)
  lint:
    runs-on: ubuntu-latest
    steps:
      - name: Descargar el código
        uses: actions/checkout@v7
      - name: Preparar Node.js
        uses: actions/setup-node@v7
        with:
          node-version: 24
          cache: npm
      - name: Instalar dependencias
        run: npm ci
      - name: Lint
        run: npm run lint
      - name: Build
        run: npm run build
```

## 0.5 Súbelo a GitHub y protege `main`

1. Crea en GitHub un repositorio **público** `fitval-web`, **vacío**.
2. Súbelo:

```bash
git add .
git commit -m "Proyecto inicial con CI mínimo"
git branch -M main
git remote add origin https://github.com/TU_USUARIO/fitval-web.git
git push -u origin main
```

3. Espera a que `htmlhint` y `lint` salgan en verde ✅ en **Actions**.
4. Crea el ruleset `proteger-main` **igual que en la entrega de NestJS** (Parte 0.4): pull request obligatoria y los checks `htmlhint` y `lint`.

## 0.6 Publica en Netlify

Sigue el apartado 6 del apunte 05:

1. En Netlify: **Add new project** › **Import an existing project** › **GitHub** › `fitval-web`.
2. Comprueba: **Build command** `npm run build`, **Publish directory** `dist`.
3. **Deploy**. Cuando termine, cambia el nombre en **Project configuration** › **Change project name**: `fitval-tunombre`.

**Resultado esperado:** `https://fitval-tunombre.netlify.app` enseña "Hola, FitVal".

> Durante la primera parte, la web de Netlify **no** tendrá datos: json-server solo existe en tu ordenador. No pasa nada. En la Parte 5 la conectarás con tu API de Render.

### Ejercicio 0

Abre `https://fitval-tunombre.netlify.app/lo-que-sea` y recarga.

**Pregunta 1:** ¿qué ves? Borra mentalmente el `_redirects`: ¿qué verías y por qué?

## 0.7 Cómo trabajarás a partir de ahora

Igual que en la API: cada parte es **una rama `feature/...` y una PR**.

```bash
git switch main
git pull
git switch -c feature/plans-page
# ...con npm run api y npm run dev arrancados, programas y pruebas en el navegador...
npm run lint
npm run build
git add .
git commit -m "Página de planes"
git push -u origin feature/plans-page
```

En la **descripción de cada PR**: qué has hecho, la **tabla de pruebas** de la parte con lo que te ha salido, y **una captura** de la página.

Al fusionar, Netlify vuelve a publicar la web sola en uno o dos minutos.

---

# PARTE 1 — Los planes

**Rama:** `feature/plans-page`. **Apuntes:** 01 (componentes, props, listas, CSS Modules) y 03 (efectos y `fetch`).

En esta parte pides los planes a json-server **a mano**, con `useEffect` y `fetch`. En la Parte 2 verás cómo RTK Query te ahorra casi todo este trabajo.

![Página de planes](img/05_react_fitval/01_planes.png)

## 1.1 La dirección de la API: `.env`

Crea `.env` en la raíz de `fitval-web`:

```env
VITE_API_URL=http://localhost:3001
```

Es la dirección de json-server. En la Parte 5 la cambiarás por la de tu API: será **lo único** que cambie de sitio, porque en el código nunca escribirás la dirección a mano.

Y `.env.example`, con lo mismo (este **sí** se sube, sin secretos). Añade `.env` al final de `.gitignore`.

Crea `src/vite-env.d.ts` para que TypeScript conozca la variable. **Cópialo tal cual:**

```ts
/// <reference types="vite/client" />

interface ImportMetaEnv {
  readonly VITE_API_URL: string;
}

interface ImportMeta {
  readonly env: ImportMetaEnv;
}
```

> Las variables de Vite **tienen** que empezar por `VITE_`. Y si cambias el `.env`, para y vuelve a arrancar `npm run dev`.

## 1.2 Código de apoyo

Crea la carpeta `src/app` y dentro estos dos ficheros. **Cópialos tal cual.**

`src/app/types.ts`, cómo llegan los datos de `db.json`:

```ts
// Cómo llegan los datos (de momento, de json-server)

export type Plan = {
  id: string; // json-server guarda los id como texto
  name: string;
  monthlyPrice: number;
};

export type GymClass = {
  id: string;
  activity: string;
  room: string;
  startsAt: string; // fecha en texto ISO: "2026-11-02T09:00:00.000Z"
  spots: number;
  trainerName: string;
};

// Lo que se envía para crear una clase (sin id: lo pone json-server)
export type NewGymClass = {
  activity: string;
  room: string;
  startsAt: string;
  spots: number;
  trainerId: number;
  trainerName: string;
};

export type User = {
  id: string;
  email: string;
  role: "member" | "staff";
};
```

`src/app/format.ts`, precios y fechas en español:

```ts
// Formatos en español (se copian tal cual)

// 29.9 → "29,90 €"
export function formatPrice(price: number): string {
  return new Intl.NumberFormat("es-ES", { style: "currency", currency: "EUR" }).format(price);
}

// "2026-11-02T09:00:00.000Z" → "lun, 2 nov, 10:00" (en la hora de tu ordenador)
export function formatDate(iso: string): string {
  return new Date(iso).toLocaleString("es-ES", {
    weekday: "short",
    day: "numeric",
    month: "short",
    hour: "2-digit",
    minute: "2-digit",
  });
}
```

> ¿Por qué la clase de las `09:00` sale a las 10:00? La `Z` del final de `"2026-11-02T09:00:00.000Z"` significa hora **UTC** (la de Londres). Tu navegador la enseña en hora de España, una hora más. Tu API hará lo mismo: en el `schema.sql` las horas se guardaron sin zona y PostgreSQL las tomó como UTC.

## 1.3 El mensaje de carga: `Loading`

Con json-server todo va rapidísimo. Pero al final tu web hablará con tu API de Render, y Render gratis **se duerme** a los 15 minutos sin uso: la primera petición puede tardar un minuto. La web no puede quedarse en blanco: tiene que decir que está cargando y, si tarda, por qué.

Crea `src/components/ui/Loading.tsx`. **Complétalo** (apunte 03, apartados 1 y 2: efectos y limpieza):

```tsx
import { useEffect, useState } from "react";

type LoadingProps = {
  text: string;
};

// "Cargando...". Si pasan 5 segundos, avisa de que el servidor se está despertando
export function Loading({ text }: LoadingProps) {
  const [slow, setSlow] = useState(false);

  useEffect(() => {
    // TODO: un setTimeout que, a los 5000 ms, ponga slow a true
    // TODO: devuelve la limpieza: clearTimeout(...)
  }, []);

  return (
    <p className="loading">
      {text}
      {/* TODO: si slow es true, añade " El servidor se está despertando: puede tardar hasta un minuto." */}
    </p>
  );
}
```

**Pregunta 2:** ¿qué pasaría si no devolvieras la limpieza y el usuario cambiara de página antes de los 5 segundos?

## 1.4 Pedir los planes: `usePlans`

Crea `src/features/plans/usePlans.ts`. Es un **custom hook**: una función que usa hooks y devuelve datos. **Complétalo** (apunte 03, apartado 7):

```ts
import { useEffect, useState } from "react";
import type { Plan } from "../../app/types";

// Pide los planes al montar el componente
export function usePlans() {
  const [plans, setPlans] = useState<Plan[]>([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);

  useEffect(() => {
    fetch(`${import.meta.env.VITE_API_URL}/plans`)
      .then((response) => {
        // TODO: si !response.ok, lanza un Error con el código: `Error ${response.status}`
        return response.json();
      })
      .then((data: Plan[]) => {
        // TODO: guarda los planes
      })
      .catch(() => {
        // TODO: guarda el error "No se han podido cargar los planes."
      })
      .finally(() => {
        // TODO: ya no está cargando
      });
  }, []); // [] = solo una vez, al abrir la página

  return { plans, loading, error };
}
```

## 1.5 La tarjeta: `PlanCard`

Crea `src/features/plans/PlanCard.module.css`. **Cópialo tal cual:**

```css
.card {
  display: flex;
  flex-direction: column;
  gap: 10px;
  padding: 24px;
  background: white;
  border: 1px solid var(--line);
  border-radius: 14px;
}

.name {
  font-size: 14px;
  font-weight: 600;
  color: var(--muted);
  text-transform: uppercase;
  letter-spacing: 0.06em;
}

.price {
  font-size: 36px;
  font-weight: 700;
}

.month {
  font-size: 16px;
  font-weight: 500;
  color: var(--muted);
}
```

Crea `src/features/plans/PlanCard.tsx`. **Complétalo** (apunte 01, apartados 5 y 11):

```tsx
import { formatPrice } from "../../app/format";
import type { Plan } from "../../app/types";
import styles from "./PlanCard.module.css";

type PlanCardProps = {
  plan: Plan;
};

export function PlanCard({ plan }: PlanCardProps) {
  return (
    <article className={styles.card}>
      {/* TODO: un <span> con la clase styles.name y el nombre del plan */}
      {/* TODO: un <span> con la clase styles.price y el precio con formatPrice(...).
                Dentro, al final, otro <span className={styles.month}> /mes</span> */}
    </article>
  );
}
```

> Más adelante, en la Parte 2, añadirás un enlace "Ver clases". Ahora aún no hay rutas.

## 1.6 La página: `PlansPage`

Crea `src/features/plans/PlansPage.tsx`. **Complétalo** (apunte 01, apartado 6, y apunte 03, apartado 7):

```tsx
import { Loading } from "../../components/ui/Loading";
import { PlanCard } from "./PlanCard";
import { usePlans } from "./usePlans";

export function PlansPage() {
  const { plans, loading, error } = usePlans();

  // TODO: si loading, devuelve <Loading text="Cargando planes..." />
  // TODO: si error, devuelve <p className="error">{error}</p>

  // Del más barato al más caro. [...plans] hace una copia: el estado no se toca
  const sortedPlans = [...plans].sort((a, b) => a.monthlyPrice - b.monthlyPrice);

  return (
    <>
      <h1>Entrena a tu ritmo</h1>
      <p>Elige el plan que mejor te encaje. Sin permanencia.</p>
      <div className="grid">
        {/* TODO: un <PlanCard> por cada plan de sortedPlans. No olvides la key */}
      </div>
    </>
  );
}
```

En `App.tsx`, cambia el `<h1>` por `<main><PlansPage /></main>`.

**Resultado esperado:** la captura del principio de esta parte. Los planes van de 19,90 € a 44,90 €.

### Ejercicio 1.1. Rómpelo a propósito: la API apagada

1. Para json-server (Ctrl + C) y recarga la web.
2. Vuelve a arrancarlo.

**Resultado esperado:** "Cargando planes..." un momento y después el mensaje de error en rojo.

### Ejercicio 1.2. Rómpelo a propósito: la ruta mal

1. En `usePlans`, cambia `/plans` por `/plan` y recarga.
2. Abre la pestaña **Red** (Network) de las herramientas del navegador (F12) y mira la petición.
3. Déjalo como estaba.

**Resultado esperado:** el mensaje de error en la web y, en Red, la petición a `plan` con un `404`. Así es como buscarás los errores de la Parte 5: **primero la pestaña Red**.

## Pruebas de la Parte 1

| Prueba | Resultado esperado |
|---|---|
| Abrir `/` con json-server arrancado | 4 tarjetas, de la más barata a la más cara |
| Abrir `/` con json-server parado | Mensaje de error en rojo |
| Pedir `/plan` en vez de `/plans` | Mensaje de error y un `404` en la pestaña Red |

---

# PARTE 2 — Rutas y horario de clases

**Rama:** `feature/classes`. **Apuntes:** 03 (React Router) y 04 (RTK Query).

## 2.1 El store y RTK Query

Primero, el store de Redux con RTK Query. Es lo mismo que en el apunte 04, apartados 3 y 5.

`src/app/api.ts`. **Complétalo**: `getClasses` está hecho, `getClass` lo haces tú.

```ts
import { createApi, fetchBaseQuery } from "@reduxjs/toolkit/query/react";
import type { GymClass } from "./types";

export const api = createApi({
  reducerPath: "api", // dónde se guarda la caché en el store
  baseQuery: fetchBaseQuery({
    baseUrl: import.meta.env.VITE_API_URL, // de momento, http://localhost:3001
  }),
  tagTypes: ["Class"],
  endpoints: (builder) => ({
    // GET /classes o GET /classes?activity:contains=yoga
    // <lo que devuelve, lo que recibe>: devuelve un array de clases, recibe el texto a buscar
    getClasses: builder.query<GymClass[], string>({
      query: (activity) =>
        activity ? `/classes?activity:contains=${encodeURIComponent(activity)}` : "/classes",
      providesTags: ["Class"],
    }),

    // TODO: getClass. GET /classes/2
    //   devuelve UNA GymClass y recibe el id como string (así llega de useParams)
    //   providesTags: ["Class"]
  }),
});

// TODO: exporta también useGetClassQuery
export const { useGetClassesQuery } = api;
```

`src/app/store.ts`. **Cópialo tal cual** (en la Parte 3 le añadirás la sesión):

```ts
import { configureStore } from "@reduxjs/toolkit";
import { api } from "./api";

export const store = configureStore({
  reducer: {
    [api.reducerPath]: api.reducer, // la caché de RTK Query
  },
  // Necesario para que RTK Query funcione
  middleware: (getDefaultMiddleware) => getDefaultMiddleware().concat(api.middleware),
});

export type RootState = ReturnType<typeof store.getState>;
export type AppDispatch = typeof store.dispatch;
```

`src/app/hooks.ts`. **Cópialo tal cual:**

```ts
import { useDispatch, useSelector } from "react-redux";
import type { AppDispatch, RootState } from "./store";

// Como useDispatch y useSelector, pero con los tipos de nuestro store
export const useAppDispatch = useDispatch.withTypes<AppDispatch>();
export const useAppSelector = useSelector.withTypes<RootState>();
```

`src/main.tsx`: envuelve `<App />` con el `<Provider store={store}>` de `react-redux`, como en el apunte 04.

## 2.2 La cabecera y el layout

`src/components/layout/Header.module.css`. **Cópialo tal cual:**

```css
.header {
  background: var(--ink);
}

.nav {
  display: flex;
  flex-wrap: wrap;
  align-items: center;
  gap: 24px;
  max-width: 1120px;
  min-height: 64px;
  margin: 0 auto;
  padding: 0 24px;
}

.logo {
  font-size: 22px;
  font-weight: 700;
  color: var(--accent-light);
  text-decoration: none;
}

/* Los enlaces del menú. NavLink añade la clase "active" al de la página actual */
.nav a:not(.logo) {
  color: #e5e7eb;
  text-decoration: none;
  padding: 8px 0;
}

.nav a:global(.active) {
  color: white;
  border-bottom: 2px solid var(--accent-light);
}

.spacer {
  flex: 1;
}

.user {
  color: #e5e7eb;
  font-size: 14px;
}

.logout {
  background: transparent;
  border: 1px solid #6b7280;
  padding: 6px 14px;
}
```

> `:global(.active)`: en un CSS Module los nombres de clase se cambian solos. `active` la pone React Router, no nosotros, así que le decimos al módulo que **no** la cambie.

`src/components/layout/Header.tsx`. **Complétalo** (apunte 03, apartado 8):

```tsx
import { Link, NavLink } from "react-router";
import styles from "./Header.module.css";

export function Header() {
  return (
    <header className={styles.header}>
      <nav className={styles.nav}>
        <Link to="/" className={styles.logo}>
          FitVal
        </Link>
        {/* TODO: un NavLink a "/" con el texto "Planes". Ojo: necesita la prop end. ¿Por qué? */}
        {/* TODO: un NavLink a "/classes" con el texto "Clases" */}
        <span className={styles.spacer} />
        {/* TODO: un NavLink a "/login" con el texto "Entrar" */}
      </nav>
    </header>
  );
}
```

Crea también `src/components/layout/Layout.tsx` (el `Header` + `<main><Outlet /></main>`) y `src/components/layout/NotFoundPage.tsx` (un título "Página no encontrada" y un `Link` al inicio). Los dos son **iguales que en el apunte 03**.

Ahora que hay rutas, añade a `PlanCard` el enlace del final: `<Link to="/classes">Ver clases</Link>`.

## 2.3 Las rutas

`src/app/router.tsx`. **Complétalo**:

```tsx
import { createBrowserRouter } from "react-router";
import { Layout } from "../components/layout/Layout";
import { NotFoundPage } from "../components/layout/NotFoundPage";
import { PlansPage } from "../features/plans/PlansPage";
import { ClassesPage } from "../features/classes/ClassesPage";
import { ClassDetailPage } from "../features/classes/ClassDetailPage";

export const router = createBrowserRouter([
  {
    path: "/",
    element: <Layout />,
    children: [
      { index: true, element: <PlansPage /> },
      // TODO: "classes" → <ClassesPage />
      // TODO: "classes/:id" → <ClassDetailPage />
      // TODO: "*" → <NotFoundPage />  (siempre la última)
    ],
  },
]);
```

`src/App.tsx` queda así:

```tsx
import { RouterProvider } from "react-router";
import { router } from "./app/router";

function App() {
  return <RouterProvider router={router} />;
}

export default App;
```

> Hasta que crees `ClassesPage` y `ClassDetailPage` (apartados 2.4 a 2.6), TypeScript se quejará. Es normal.

## 2.4 Las plazas: `SpotsBadge`

Una etiqueta verde con las plazas. Si quedan **3 o menos**, naranja.

`src/features/classes/SpotsBadge.module.css`. **Cópialo tal cual:**

```css
.badge {
  padding: 4px 10px;
  border-radius: 999px;
  font-size: 13px;
  font-weight: 600;
  background: #dcfce7;
  color: #166534;
}

/* Se añade cuando quedan pocas plazas */
.few {
  background: #ffedd5;
  color: #9a3412;
}
```

`src/features/classes/SpotsBadge.tsx`. **Complétalo**:

```tsx
import styles from "./SpotsBadge.module.css";

type SpotsBadgeProps = {
  spots: number;
};

export function SpotsBadge({ spots }: SpotsBadgeProps) {
  // TODO: si spots <= 3, className = las dos clases: `${styles.badge} ${styles.few}`
  //       si no, solo styles.badge
  const className = styles.badge;
  return <span className={className}>{spots} plazas</span>;
}
```

## 2.5 La tarjeta de una clase: `ClassCard`

`src/features/classes/ClassCard.module.css`. **Cópialo tal cual:**

```css
.card {
  display: flex;
  flex-wrap: wrap;
  align-items: center;
  gap: 16px;
  padding: 16px 20px;
  background: white;
  border: 1px solid var(--line);
  border-radius: 12px;
  color: var(--ink);
  text-decoration: none;
}

.card:hover {
  border-color: var(--accent-light);
}

.when {
  width: 160px;
  font-weight: 600;
}

.info {
  display: flex;
  flex: 1;
  flex-direction: column;
  gap: 2px;
  min-width: 200px;
}

.activity {
  font-size: 18px;
  font-weight: 700;
}

.details {
  color: var(--muted);
}
```

`src/features/classes/ClassCard.tsx`. **Complétalo** mirando la captura del horario:

```tsx
import { Link } from "react-router";
import { formatDate } from "../../app/format";
import type { GymClass } from "../../app/types";
import { SpotsBadge } from "./SpotsBadge";
import styles from "./ClassCard.module.css";

type ClassCardProps = {
  gymClass: GymClass; // "class" es una palabra reservada de JavaScript: por eso gymClass
};

export function ClassCard({ gymClass }: ClassCardProps) {
  return (
    // Toda la tarjeta es un enlace a /classes/:id
    <Link to={`/classes/${gymClass.id}`} className={styles.card}>
      {/* TODO: <span className={styles.when}> con la fecha formateada (formatDate) */}
      <div className={styles.info}>
        {/* TODO: <span className={styles.activity}> con la actividad */}
        {/* TODO: <span className={styles.details}> con "Sala 1 · con Carla Navarro" */}
      </div>
      {/* TODO: <SpotsBadge> con las plazas */}
    </Link>
  );
}
```

## 2.6 El horario con buscador: `ClassesPage`

El texto del buscador se guarda **en la URL** (`/classes?activity=yoga`), no en un `useState`. Así, si recargas o compartes el enlace, la búsqueda sigue ahí. Es el reto 2 del apunte 03.

`src/features/classes/ClassesPage.module.css`. **Cópialo tal cual:**

```css
.toolbar {
  display: flex;
  flex-wrap: wrap;
  align-items: flex-end;
  justify-content: space-between;
  gap: 16px;
  margin-bottom: 20px;
}

.search {
  display: flex;
  flex-direction: column;
  gap: 6px;
  min-width: 280px;
}

.list {
  display: flex;
  flex-direction: column;
  gap: 10px;
}
```

`src/features/classes/ClassesPage.tsx`. **Complétalo**:

```tsx
import { useSearchParams } from "react-router";
import { useGetClassesQuery } from "../../app/api";
import { Loading } from "../../components/ui/Loading";
import { ClassCard } from "./ClassCard";
import styles from "./ClassesPage.module.css";

export function ClassesPage() {
  // Como useState, pero guardado en la URL
  const [searchParams, setSearchParams] = useSearchParams();
  const activity = searchParams.get("activity") ?? ""; // null si no está → ""

  // TODO: llama a useGetClassesQuery(activity) y saca data (con [] por defecto), isLoading e isError

  function changeActivity(text: string) {
    // Si hay texto: ?activity=texto. Si no: sin parámetros
    setSearchParams(text ? { activity: text } : {});
  }

  return (
    <>
      <div className={styles.toolbar}>
        <h1>Horario de clases</h1>
        <div className={styles.search}>
          <label htmlFor="activity">Buscar actividad</label>
          {/* TODO: input controlado: value={activity} y onChange que llama a changeActivity */}
        </div>
      </div>

      {/* TODO: si isLoading, <Loading text="Cargando clases..." /> */}
      {/* TODO: si isError, un <p className="error"> */}
      {/* TODO: si no carga, no hay error y no hay clases: "No hay clases de «...». Prueba con otra actividad." */}

      <div className={styles.list}>{/* TODO: un <ClassCard> por clase */}</div>
    </>
  );
}
```

**Resultado esperado:** al escribir `yoga`, la URL cambia a `/classes?activity=yoga` y salen 2 clases.

![Buscando yoga](img/05_react_fitval/03_clases_filtro.png)

### Ejercicio 2.1

Busca `yoga`, recarga la página y después pulsa **Atrás** en el navegador varias veces.

**Pregunta 3:** ¿qué pasa con el texto del buscador al recargar y al ir atrás? ¿Pasaría lo mismo si `activity` fuera un `useState`?

### Ejercicio 2.2

Abre la pestaña **Red** (Network), filtra por `classes` y escribe `spin` letra a letra. Cuenta las peticiones. Ahora borra y vuelve a escribir `spin`.

**Pregunta 4:** ¿cuántas peticiones salen la segunda vez? ¿Por qué? (Apunte 04, apartado 5, "La caché".)

## 2.7 Ahora tú: el detalle de una clase

Crea `src/features/classes/ClassDetailPage.tsx`. Esta vez, **sin esqueleto**: es igual que `CoursePage` del apunte 04, con una clase en vez de un curso.

Tiene que:

- [ ] Leer el `id` de la URL con `useParams` y pedir la clase con `useGetClassQuery(id)`.
- [ ] Mientras carga: `<Loading text="Cargando clase..." />`.
- [ ] Si hay error (json-server da `404` si no existe): "Esa clase no existe." y un enlace al horario.
- [ ] Si todo va bien: enlace "← Volver al horario", la actividad en un `<h1>`, el `SpotsBadge`, y tres párrafos: cuándo (`formatDate`), sala y monitor.

![Detalle de una clase](img/05_react_fitval/04_detalle.png)

## Pruebas de la Parte 2

| Prueba | Resultado esperado |
|---|---|
| Menú: pulsar "Planes" y "Clases" | Cambia de página sin recargar. El enlace activo, subrayado en naranja |
| `/classes` | 7 clases. El Crossfit (3 plazas) en naranja |
| Buscar `yoga` | 2 clases y la URL `/classes?activity=yoga` |
| Buscar `zzz` | "No hay clases de «zzz»..." |
| Pulsar una clase | Su detalle en `/classes/:id` |
| `/classes/99` | "Esa clase no existe." |
| `/classes/abc` | "Esa clase no existe." |
| `/no-existe` | Página no encontrada |

---

# PARTE 3 — Entrar y crear cuenta

**Rama:** `feature/auth`. **Apuntes:** 02 (formularios, React Hook Form y Zod) y 04 (login y slice).

json-server no sabe hacer login. Igual que en el apunte 04, apartado 7, lo **simulamos**: buscamos el usuario por su correo en `GET /users?email=...`, comprobamos la contraseña en el navegador y nos inventamos un token.

Todo lo demás (el slice, la sesión guardada, la cabecera, la ruta protegida) es **igual** que con una API de verdad. En la Parte 5 solo cambiarás el `authService` por el login real de tu API, con JWT.

Los usuarios de `db.json`:

| Correo | Contraseña | Rol |
|---|---|---|
| `staff@fitval.es` | `staff1234` | staff |
| `lucia@gym.com` | `lucia1234` | member (socia) |

## 3.1 La sesión en Redux: `authSlice`

`src/features/auth/authSlice.ts` es **el mismo que en el apunte 04**, apartado 7: guarda **solo el token** y sus reducers devuelven un objeto nuevo (`({ ...state, token: action.payload })`). El usuario no va en el slice.

Añádelo al store y guarda en `localStorage` **solo el token**, como en el apunte 04. Nada del usuario (ni correo, ni rol). Tu `store.ts` queda así:

```ts
import { configureStore } from "@reduxjs/toolkit";
import authReducer from "../features/auth/authSlice";
import { api } from "./api";

export const store = configureStore({
  reducer: {
    auth: authReducer,
    [api.reducerPath]: api.reducer,
  },
  middleware: (getDefaultMiddleware) => getDefaultMiddleware().concat(api.middleware),
  preloadedState: {
    auth: { token: localStorage.getItem("token") }, // el token de la última vez (o null)
  },
});

// Después de cada acción, guarda SOLO el token. Al cerrar sesión, se borra
store.subscribe(() => {
  const { token } = store.getState().auth;
  if (token) {
    localStorage.setItem("token", token);
  } else {
    localStorage.removeItem("token");
  }
});

export type RootState = ReturnType<typeof store.getState>;
export type AppDispatch = typeof store.dispatch;
```

## 3.2 El login simulado: `authService`

`src/features/auth/authService.ts`. **Complétalo** (es el `authService` del apunte 04, con la dirección del `.env`):

```ts
import type { User } from "../../app/types";

type StoredUser = User & { password: string }; // como está en db.json

// Login FALSO: json-server no sabe hacer login.
// En la Parte 5 lo cambiarás por el de tu API (POST /auth/login).
// Solo devuelve el token: el usuario se pide después a "/me"
export async function login(email: string, password: string): Promise<string> {
  const response = await fetch(
    `${import.meta.env.VITE_API_URL}/users?email=${encodeURIComponent(email)}`,
  );
  if (!response.ok) {
    throw new Error(`Error ${response.status}`);
  }
  const users: StoredUser[] = await response.json();

  // TODO: busca en users el que tenga esa password (find)
  // TODO: si no hay ninguno, lanza un Error "Correo o contraseña incorrectos."
  // TODO: devuelve el token inventado: `token-fake-${found.id}`
}
```

### ¿Quién soy? `getMe` y `useCurrentUser`

Igual que en el apunte 04 ("¿Quién soy? El endpoint `/me`"). En `api.ts`, añade el endpoint y exporta `useGetMeQuery` y `useLazyGetMeQuery`:

```ts
// ¿Quién soy? Recibe el token. Con tu API será: query: () => "/auth/me"
// json-server no tiene /me: sacamos el id del token falso ("token-fake-1" -> "1")
getMe: builder.query<User, string>({
  query: (token) => `/users/${token.replace("token-fake-", "")}`,
  // json-server devuelve también la contraseña: la quitamos
  transformResponse: (response: User & { password: string }) => {
    const { password: _password, ...user } = response;
    return user;
  },
}),
```

Y copia el hook `useCurrentUser` del apunte 04 en `src/features/auth/useCurrentUser.ts` (importa `useGetMeQuery` de `../../app/api`). Lo usarán la cabecera y `ProtectedRoute`.

## 3.3 La página de login

Crea `src/features/auth/LoginPage.tsx` a partir del `LoginPage` del apunte 04. Cambios:

- [ ] Textos: "Correo" en vez de "Email".
- [ ] Antes de llamar a la API, si el correo o la contraseña están vacíos: "Escribe el correo y la contraseña." (sin llamar a la API).
- [ ] Al entrar, el **staff** va a `/staff` y los **socios** a `/classes`. Pista: el login ya no te da el usuario. Después de `dispatch(loggedIn(token))`, pídelo con `const [getMe] = useLazyGetMeQuery();` y `const user = await getMe(token).unwrap();`. Mira `user.role`.
- [ ] Debajo del botón: "¿No tienes cuenta? Regístrate", con un `Link` a `/register`.

![Login con contraseña mal](img/05_react_fitval/08_login_error.png)

## 3.4 La cabecera según la sesión

En `Header.tsx`:

- [ ] Lee el usuario: `const { user } = useCurrentUser();`
- [ ] Si **no** hay usuario: el `NavLink` "Entrar" de antes.
- [ ] Si hay usuario: `<span className={styles.user}>` con el correo y "socio" o "staff", y un botón **Salir** (`className={styles.logout}`) que hace `dispatch(loggedOut())`, `dispatch(api.util.resetApiState())` (olvida el usuario de `/me`) y `navigate("/")`.
- [ ] Solo si el usuario es **staff**: un `NavLink` "Panel" a `/staff`, después de "Clases".

Como en el apunte 04, apartado 7, "La cabecera según la sesión".

## 3.5 Crear cuenta: `RegisterPage` con React Hook Form y Zod

Para el registro usamos una **mutation** de RTK Query (en vez de `fetch` a mano) y **React Hook Form + Zod** para validar.

En `api.ts`, añade el endpoint y exporta su hook `useRegisterMutation`:

```ts
// POST /users. json-server guarda lo que le mandes: el rol lo ponemos nosotros
register: builder.mutation<User, { email: string; password: string }>({
  query: (body) => ({ url: "/users", method: "POST", body: { ...body, role: "member" } }),
}),
```

(Importa también el tipo `User` arriba.)

Un error de la API llega a React como un objeto con muchas cosas dentro. Esta función saca el mensaje. Está pensada para **tu API de NestJS** (json-server casi nunca da mensajes de error): la dejamos ya hecha para la Parte 5. `src/app/apiError.ts`, **cópialo tal cual:**

```ts
// Saca un texto legible de un error de la API (se copia tal cual).
// NestJS responde { message: "texto" } o { message: ["texto 1", "texto 2"] }
export function getErrorMessage(error: unknown): string {
  if (typeof error === "object" && error !== null && "data" in error) {
    const data = (error as { data?: { message?: string | string[] } }).data;
    if (Array.isArray(data?.message)) return data.message.join(". ");
    if (data?.message) return data.message;
  }
  return "No se ha podido conectar con la API.";
}
```

`src/features/auth/RegisterPage.tsx`. **Complétalo** (apunte 02, apartado 9):

```tsx
import { useState } from "react";
import { useForm } from "react-hook-form";
import { zodResolver } from "@hookform/resolvers/zod";
import { z } from "zod";
import { Link } from "react-router";
import { useRegisterMutation } from "../../app/api";
import { getErrorMessage } from "../../app/apiError";

// Las reglas del formulario: las mismas que pondrá el RegisterDto de tu API
const registerSchema = z.object({
  email: z.email("Escribe un correo válido"),
  // TODO: password, un texto de al menos 8 caracteres.
  //       Mensaje: "La contraseña debe tener al menos 8 caracteres"
});

type RegisterForm = z.infer<typeof registerSchema>; // el tipo sale de las reglas

export function RegisterPage() {
  const {
    register,
    handleSubmit,
    formState: { errors },
  } = useForm<RegisterForm>({ resolver: zodResolver(registerSchema) });
  const [registerUser, { isLoading }] = useRegisterMutation();
  const [apiError, setApiError] = useState<string | null>(null);
  const [createdEmail, setCreatedEmail] = useState<string | null>(null);

  // Solo se llama si Zod da el visto bueno
  async function onSubmit(data: RegisterForm) {
    setApiError(null);
    try {
      const user = await registerUser(data).unwrap(); // unwrap: si la API falla, lanza el error
      // TODO: guarda user.email en createdEmail
    } catch (error) {
      // TODO: guarda en apiError el mensaje (getErrorMessage)
    }
  }

  // Cuenta creada: ya no se enseña el formulario
  if (createdEmail) {
    return (
      <p>
        Cuenta creada para {createdEmail}. Ya puedes <Link to="/login">entrar</Link>.
      </p>
    );
  }

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <h1>Crear cuenta</h1>
      {/* TODO: si apiError, <p className="error" role="alert"> */}
      <label htmlFor="email">Correo</label>
      <input id="email" type="email" {...register("email")} />
      {errors.email && <p className="field-error">{errors.email.message}</p>}
      {/* TODO: label, input y error de password (type="password") */}
      <button type="submit" disabled={isLoading}>
        {isLoading ? "Creando..." : "Crear cuenta"}
      </button>
    </form>
  );
}
```

Añade las rutas `login` y `register` al router.

Validación de Zod: el formulario no llega a llamar a la API.

![Contraseña corta](img/05_react_fitval/06_registro_validacion.png)

### Ejercicio 3.1

Entra como `lucia@gym.com`. Recarga la página. Después abre **Aplicación** (Application) › **Local storage** en las herramientas del navegador.

**Pregunta 5:** ¿sigues dentro después de recargar? ¿Qué hay guardado en `localStorage`? ¿Por qué ese token no le serviría a una API de verdad?

## Pruebas de la Parte 3

| Prueba | Resultado esperado |
|---|---|
| Login con los campos vacíos | "Escribe el correo y la contraseña." (en Red no sale ninguna petición) |
| Login con contraseña mal | "Correo o contraseña incorrectos." |
| Login como `lucia@gym.com` | Va a `/classes`. Arriba: el correo · socio y Salir. No sale "Panel" |
| Login como `staff@fitval.es` | Va a `/staff` (todavía 404). Arriba sale "Panel" |
| Recargar con sesión | Sigues dentro |
| Salir | Vuelve a `/`. Arriba sale "Entrar" |
| Registro con contraseña `123` | Error de Zod debajo del campo. No llega a la API |
| Registro con un correo nuevo | "Cuenta creada para...". Después puedes entrar. El usuario está en `db.json` |
| Registro con `lucia@gym.com` otra vez | ¡También lo crea! json-server no comprueba nada. Tu API sí: lo verás en la Parte 5 |

---

# PARTE 4 — El panel del personal

**Rama:** `feature/staff-panel`. **Apuntes:** 04 (mutations, tags y rutas protegidas) y 05 (tests).

![Panel del personal](img/05_react_fitval/09_panel.png)

## 4.1 Solo para el staff: `ProtectedRoute`

`src/features/auth/ProtectedRoute.tsx`. Parte del del apunte 04, apartado 8, con dos cambios:

- [ ] Los roles son `"member" | "staff"`.
- [ ] Si hay sesión pero no tiene el rol: en vez del mensaje, `<Navigate to="/" replace />`.

En el router, la ruta `staff` va **dentro** de la ruta protegida:

```tsx
{
  element: <ProtectedRoute role="staff" />,
  children: [{ path: "staff", element: <StaffPage /> }],
},
```

Prueba en una terminal, sin sesión ni token: `curl -X DELETE http://localhost:3001/classes/7`. Abre `/classes`. Después cópiale a `db.json` el `db.initial.json`.

**Pregunta 6:** ¿se ha borrado el Crossfit? Entonces, `ProtectedRoute` ¿basta para que un socio no pueda borrar clases? ¿Quién tiene que protegerlas de verdad?

## 4.2 El token en cada petición

En `api.ts`, dentro de `fetchBaseQuery`, añade `prepareHeaders`. Es **exactamente** el del apunte 04, "Enviar el token en cada petición". Necesitarás importar el tipo `RootState`.

json-server ignora el token. Tu API no: crear y borrar clases solo funcionará con un token de staff. Lo dejas hecho ya.

## 4.3 Crear y borrar: mutations

En `api.ts`, añade los dos endpoints y exporta sus hooks:

```ts
// POST /classes (solo staff)
createClass: builder.mutation<GymClass, NewGymClass>({
  query: (newClass) => ({ url: "/classes", method: "POST", body: newClass }),
  invalidatesTags: ["Class"], // el horario está viejo: RTK Query lo vuelve a pedir
}),
// TODO: deleteClass. DELETE /classes/:id. Recibe el id (string) y devuelve la GymClass borrada.
//       También invalida "Class"
```

## 4.4 La tabla y el botón Borrar

`src/features/staff/StaffPage.module.css`. **Cópialo tal cual:**

```css
.layout {
  display: flex;
  flex-wrap: wrap;
  align-items: flex-start;
  gap: 24px;
}

.form {
  flex: 1 1 320px;
}

.table {
  flex: 999 1 520px;
  min-width: 0;
  overflow-x: auto;
}

.delete {
  background: white;
  color: var(--danger);
  border: 1px solid #fca5a5;
  padding: 6px 12px;
}
```

`src/features/staff/StaffPage.tsx`. **Complétalo**:

```tsx
import { useState } from "react";
import { useDeleteClassMutation, useGetClassesQuery } from "../../app/api";
import { getErrorMessage } from "../../app/apiError";
import { formatDate } from "../../app/format";
import type { GymClass } from "../../app/types";
import { Loading } from "../../components/ui/Loading";
import { CreateClassForm } from "./CreateClassForm";
import styles from "./StaffPage.module.css";

export function StaffPage() {
  const { data: classes = [], isLoading } = useGetClassesQuery(""); // "" = todas
  // Una mutation devuelve [la función que la lanza, su estado]
  const [deleteClass] = useDeleteClassMutation();
  const [deleteError, setDeleteError] = useState<string | null>(null);

  async function remove(gymClass: GymClass) {
    setDeleteError(null);
    try {
      // TODO: await deleteClass(...).unwrap() con el id de la clase
    } catch (error) {
      // Con tu API: 409 si la clase tiene reservas
      setDeleteError(`«${gymClass.activity}»: ${getErrorMessage(error)}`);
    }
  }

  return (
    <>
      <h1>Panel del personal</h1>
      <div className={styles.layout}>
        <CreateClassForm />
        <div className={styles.table}>
          {/* TODO: si deleteError, <p className="error" role="alert"> */}
          {isLoading ? (
            <Loading text="Cargando clases..." />
          ) : (
            <table>
              <thead>
                <tr>
                  <th>Actividad</th>
                  <th>Día y hora</th>
                  <th>Sala</th>
                  <th>Plazas</th>
                  <th></th>
                </tr>
              </thead>
              <tbody>
                {/* TODO: una <tr> por clase con sus 4 datos y, en la última celda,
                    <button className={styles.delete} onClick={...}>Borrar</button> */}
              </tbody>
            </table>
          )}
        </div>
      </div>
    </>
  );
}
```

Mientras no tengas el formulario, comenta la línea `<CreateClassForm />` y su `import`.

## 4.5 El formulario: `CreateClassForm`

En el formulario, el monitor se elige en un desplegable. Los monitores no cambian, así que van en el código. `src/app/trainers.ts`, **cópialo tal cual:**

```ts
// Los monitores del gimnasio (se copia tal cual)
export const TRAINERS = [
  { id: 1, name: "Carla Navarro" },
  { id: 2, name: "Javier Ortega" },
  { id: 3, name: "Elena Romero" },
  { id: 4, name: "Rubén Castro" },
];
```

`src/features/staff/CreateClassForm.tsx`. **Complétalo**. Es como `RegisterPage`, con más campos.

```tsx
import { useState } from "react";
import { useForm } from "react-hook-form";
import { zodResolver } from "@hookform/resolvers/zod";
import { z } from "zod";
import { useCreateClassMutation } from "../../app/api";
import { getErrorMessage } from "../../app/apiError";
import { TRAINERS } from "../../app/trainers";
import styles from "./StaffPage.module.css";

const classSchema = z.object({
  activity: z.string().trim().min(1, "Escribe la actividad"),
  room: z.string().trim().min(1, "Escribe la sala"),
  startsAt: z.string().min(1, "Elige el día y la hora"),
  spots: z.number("Escribe un número").int().min(1, "Al menos 1 plaza"),
  // TODO: trainerId, número entero mayor o igual que 1. Mensaje (en los dos sitios): "Elige un monitor"
});

type ClassForm = z.infer<typeof classSchema>;

export function CreateClassForm() {
  const {
    register,
    handleSubmit,
    reset,
    formState: { errors },
  } = useForm<ClassForm>({ resolver: zodResolver(classSchema) });
  const [createClass, { isLoading }] = useCreateClassMutation();
  const [apiError, setApiError] = useState<string | null>(null);

  async function onSubmit(data: ClassForm) {
    setApiError(null);
    try {
      // json-server guarda lo que le mandes: si no enviamos el nombre del monitor,
      // la tarjeta del horario saldría sin él
      const trainer = TRAINERS.find((t) => t.id === data.trainerId);
      await createClass({
        ...data,
        // El input datetime-local da "2026-11-20T18:00" (hora de aquí, sin zona).
        // toISOString() lo pasa a UTC con zona: "2026-11-20T17:00:00.000Z"
        startsAt: new Date(data.startsAt).toISOString(),
        trainerName: trainer?.name ?? "",
      }).unwrap();
      reset(); // vacía el formulario
    } catch (error) {
      // TODO: guarda el mensaje de la API (getErrorMessage)
    }
  }

  return (
    <form className={styles.form} onSubmit={handleSubmit(onSubmit)}>
      <h2>Nueva clase</h2>
      {apiError && (
        <p className="error" role="alert">
          {apiError}
        </p>
      )}
      <label htmlFor="activity">Actividad</label>
      <input id="activity" {...register("activity")} />
      {errors.activity && <p className="field-error">{errors.activity.message}</p>}

      {/* TODO: Sala (room). Igual que actividad */}

      <label htmlFor="startsAt">Día y hora</label>
      <input id="startsAt" type="datetime-local" {...register("startsAt")} />
      {errors.startsAt && <p className="field-error">{errors.startsAt.message}</p>}

      <label htmlFor="spots">Plazas</label>
      {/* valueAsNumber: el input da texto; así llega a Zod como número */}
      <input id="spots" type="number" {...register("spots", { valueAsNumber: true })} />
      {errors.spots && <p className="field-error">{errors.spots.message}</p>}

      <label htmlFor="trainerId">Monitor</label>
      {/* TODO: un <select id="trainerId"> con register("trainerId", { valueAsNumber: true }).
                Primera opción: <option value="">Elige uno</option>.
                Después, una <option> por cada monitor de TRAINERS: value={trainer.id}, el texto es el nombre */}
      {errors.trainerId && <p className="field-error">{errors.trainerId.message}</p>}

      <button type="submit" disabled={isLoading}>
        {isLoading ? "Guardando..." : "Crear clase"}
      </button>
    </form>
  );
}
```

Validación de Zod con el formulario vacío:

![Errores de Zod en el panel](img/05_react_fitval/10_panel_validacion.png)

### Ejercicio 4.1

Crea una clase. Fíjate en la tabla: **no** has escrito nada para que se actualice.

**Pregunta 7:** ¿quién hace que la tabla (y el horario de `/classes`) muestre la clase nueva? Explícalo con las palabras `providesTags` e `invalidatesTags`.

### Ejercicio 4.2. Rómpelo a propósito

En `createClass`, comenta `invalidatesTags` y crea otra clase. Después déjalo como estaba.

**Resultado esperado:** json-server responde `201` y la clase está en `db.json`, pero la tabla no cambia hasta que recargas.

## 4.6 Tus primeros tests

Antes de cerrar la primera parte, dos tests **obligatorios**: uno de lógica (el slice de la sesión) y otro de un componente (`SpotsBadge`). Es lo mismo que el apartado 4 del apunte 05.

Instala las herramientas:

```bash
npm install -D vitest @testing-library/react @testing-library/jest-dom jsdom
```

Sustituye `vite.config.ts`. **Cópialo tal cual:**

```ts
/// <reference types="vitest/config" />
import react from "@vitejs/plugin-react";
import { defineConfig } from "vite";

export default defineConfig({
  plugins: [react()],
  test: {
    environment: "jsdom", // el navegador de mentira
    setupFiles: "./src/test/setup.ts", // se ejecuta antes de los tests
  },
});
```

Crea `src/test/setup.ts`. **Cópialo tal cual:**

```ts
import "@testing-library/jest-dom/vitest"; // añade comprobaciones como toBeInTheDocument()
import { afterEach } from "vitest";
import { cleanup } from "@testing-library/react";

// Después de cada test, borra lo que ha pintado
afterEach(() => {
  cleanup();
});
```

En `package.json`, añade el script `"test": "vitest"`. `npm test` se queda vigilando; `npx vitest run` los pasa una vez y termina.

### Test 1: el slice de la sesión

Un reducer es una función: recibe un estado y una acción y devuelve el estado nuevo. Es lo más fácil de probar.

Crea `src/features/auth/authSlice.test.ts`, al lado del slice. **Complétalo:**

```ts
import { describe, expect, it } from "vitest";
import authReducer, { loggedIn, loggedOut } from "./authSlice";

// Un usuario de prueba. "as const": para TypeScript, role es "member", no un texto cualquiera
const lucia = { id: "2", email: "lucia@gym.com", role: "member" as const };

describe("authSlice", () => {
  it("al entrar, guarda el token y el usuario", () => {
    // estado de partida (nadie dentro) + la acción loggedIn
    const state = authReducer({ token: null, user: null }, loggedIn({ token: "abc", user: lucia }));

    expect(state.token).toBe("abc");
    expect(state.user).toEqual(lucia); // toEqual compara el contenido del objeto
  });

  it("al salir, borra el token y el usuario", () => {
    // TODO: parte de un estado con sesión ({ token: "abc", user: lucia }) y aplica loggedOut()
    // TODO: comprueba que state.token y state.user son null (toBeNull())
  });
});
```

### Test 2: un componente

Crea `src/features/classes/SpotsBadge.test.tsx`. **Complétalo:**

```tsx
import { describe, expect, it } from "vitest";
import { render, screen } from "@testing-library/react";
import { SpotsBadge } from "./SpotsBadge";
import styles from "./SpotsBadge.module.css"; // para saber cómo se llama la clase few

describe("SpotsBadge", () => {
  it("muestra las plazas", () => {
    render(<SpotsBadge spots={12} />); // pinta el componente

    expect(screen.getByText("12 plazas")).toBeInTheDocument(); // lo busca por su texto
  });

  it("con 3 plazas, se pone naranja (clase few)", () => {
    // TODO: pinta <SpotsBadge spots={3} />
    // TODO: busca "3 plazas" y comprueba que tiene la clase: .toHaveClass(styles.few)
  });

  it("con 4 plazas, no", () => {
    // TODO: igual, con 4 plazas y .not.toHaveClass(styles.few)
  });
});
```

Ejecuta `npx vitest run`.

**Resultado esperado:** `Test Files 2 passed`, `Tests 5 passed`.

### Ejercicio 4.3. Rómpelo a propósito

En `SpotsBadge.tsx`, cambia `spots <= 3` por `spots <= 4` y pasa los tests. Después déjalo como estaba.

**Resultado esperado:** falla un test, `con 4 plazas, no`, y Vitest te enseña qué esperaba y qué ha encontrado. Para eso sirven los tests: avisan cuando un cambio rompe algo que funcionaba.

### Los tests en el CI

En `.github/workflows/ci.yml`, dentro del job `lint`, añade este paso **entre** `Lint` y `Build`:

```yaml
      - name: Tests
        run: npx vitest run
```

Así, una PR que rompa un test sale en rojo y no se puede fusionar.

## Pruebas de la Parte 4

| Prueba | Resultado esperado |
|---|---|
| `/staff` sin sesión | Va a `/login` |
| `/staff` como **member** | Va a `/` |
| `/staff` como **staff** | La tabla con las clases y el formulario |
| Crear con todos los campos vacíos | Errores de Zod debajo de cada campo |
| Crear una clase correcta | Aparece en la tabla y en `/classes` sin recargar, con el nombre del monitor |
| Borrar la clase que acabas de crear | Desaparece |
| Borrar el primer Spinning | También desaparece: json-server no sabe de reservas. Con tu API no te dejará (Parte 5) |
| `npx vitest run` | 2 ficheros y 5 tests en verde |
| La PR en GitHub | Los checks `htmlhint` y `lint` en verde, con el paso `Tests` dentro |

Al terminar, cópiale a `db.json` el `db.initial.json` para dejar los datos como al principio.

---

# Fin de la primera parte: cierra tu web con json-server

Antes de tocar tu API, **deja hecho y marcado lo que llevas**. Así tu trabajo de React queda a la vista, aunque luego la conexión se complique.

1. Las PR de las partes 1 a 4, **fusionadas** y con los checks en verde.
2. En el `README.md`: cómo arrancar la web (`npm install`, `npm run api` y `npm run dev`) y **una captura de cada página**: planes, horario con `yoga`, detalle, login, registro y panel.
3. El archivo `respuestas.md` en la raíz, con las preguntas 1 a 7.
4. Marca esta versión con una **etiqueta** (tag): es la foto de tu repositorio en este momento.

```bash
git switch main
git pull
git tag v1-json-server
git push origin v1-json-server
```

En GitHub la verás en **Tags**. Aunque sigas cambiando `main`, la etiqueta sigue apuntando a esta versión: `https://github.com/TU_USUARIO/fitval-web/tree/v1-json-server`. Es lo que se corrige de la primera parte.

Antes de crear la etiqueta, comprueba:

- [ ] `npx vitest run` pasa los 5 tests.
- [ ] Con `npm run api` y `npm run dev` pasan todas las pruebas de las partes 1 a 4.
- [ ] `npm run lint` y `npm run build` terminan sin errores.
- [ ] No queda ningún `// TODO` en el código.
- [ ] El `.env` **no** está en GitHub. El `.env.example` sí.

---

# PARTE 5 — Conecta con tu API

**Rama:** `feature/connect-api`. **Apuntes:** 03 (`fetch`), 04 (RTK Query y login), 05 (Netlify) y los de Render.

Tu web funciona con json-server. Ahora la vas a conectar con **tu API de NestJS**: primero en tu ordenador y después en internet (Netlify + Render).

json-server y tu API **no se comportan igual**: las rutas, los filtros, los `id`, el login y lo que aceptan en el body son distintos. Por eso **van a salir errores**. No es que lo hayas hecho mal: es lo que pasa siempre que se juntan un frontend y un backend hechos por separado. Esta parte va de **encontrarlos y arreglarlos**.

## 5.1 Prepara tu API

Necesitas tu API de FitVal funcionando en tu ordenador y en Render (Parte 13 de la entrega de NestJS).

- [ ] En tu ordenador: `docker start postgres-fitval` y `npm run start:dev` en `api-fitval`. `GET http://localhost:3000/api/plans` responde `200`.
- [ ] En Render: `GET https://api-fitval-tunombre.onrender.com/api/plans` responde `200`.
- [ ] Tienes un usuario **staff** y otro **member** creados con `POST /api/auth/register`. Al staff le cambiaste el rol con `UPDATE users SET role = 'staff' WHERE email = '...'`.

> Si hiciste la ampliación de la entrega de NestJS, tus rutas llevan versión: `/api/v1/plans`. Pon `/api/v1` donde aquí pone `/api`.

## 5.2 Cambia la dirección

1. Para json-server (`Ctrl + C`). No borres `db.json`: te puede servir para comparar.
2. En el `.env`: `VITE_API_URL=http://localhost:3000/api` (y lo mismo en `.env.example`).
3. Para y vuelve a arrancar `npm run dev`.

Ya está: la web habla con tu API. Ahora, a ver qué falla.

## 5.3 Cómo vas a trabajar: depurar con la IA

Para cada cosa que no funcione, sigue siempre estos pasos:

1. **Mira la pestaña Red** (Network): ¿qué petición sale? Método, dirección, body, código de respuesta y lo que responde la API.
2. **Mira la consola**: los errores de CORS y de JavaScript salen ahí.
3. **Compara con tu API**: ¿qué ruta tiene tu controlador? ¿Qué campos acepta tu DTO? ¿Qué devuelve? Postman o el Swagger de tu API te ayudan.
4. **Arregla** el código de React (o el de la API, si el fallo está allí).
5. **Apúntalo** en tu diario de errores (apartado 5.5).

### Usa la IA con cabeza

En esta parte **sí** puedes (y deberías) usar la IA: Claude, ChatGPT, Copilot... Es lo que harás en una empresa. Pero con normas:

- **Dale contexto**: el mensaje de error **exacto**, la petición (método, dirección, body), el fichero de React que la hace y el controlador o DTO de tu API.
- **Pídele que te explique la causa** antes que el código. Si no entiendes la causa, no aceptes el arreglo.
- **Nunca pegues secretos**: ni el `.env` de la API, ni la dirección de Supabase, ni `JWT_SECRET`, ni tokens.
- **Comprueba** cada arreglo en el navegador. La IA se equivoca, y con seguridad.
- **Cambios pequeños**: un error cada vez. Si te propone reescribir medio proyecto, no.
- Tienes que saber **explicar** cada línea que cambies. En la corrección te puede tocar hacerlo.

Un ejemplo de buena pregunta:

```text
Tengo una web React (RTK Query) y una API NestJS. Al buscar "yoga" en el
horario, la web pide GET http://localhost:3000/api/classes?activity:contains=yoga
y la API responde 200 con las 7 clases, sin filtrar.
Este es mi endpoint en React: (pegas getClasses de api.ts)
Este es mi controlador en NestJS: (pegas el @Get() de classes.controller.ts)
¿Por qué no filtra? Explícame la causa antes de darme el código.
```

## 5.4 Lo que tienes que comprobar

Repite las pruebas de las partes 1 a 4, ahora con tu API. Esta tabla te dice **qué** puede fallar y te da una **pista**. El arreglo lo buscas tú.

| Prueba | Lo que puede salir mal | Pista |
|---|---|---|
| Abrir `/` | Error en rojo. En la consola: `blocked by CORS policy` | ¿Tu API tiene `enableCors` (Parte 13 de la entrega de NestJS)? ¿Lo has arrancado de nuevo? |
| Abrir `/` | Error en rojo. En Red: `404` en todas las peticiones | Mira `VITE_API_URL`: ¿lleva el puerto de tu API y `/api`? |
| Buscar `yoga` en el horario | Salen las 7 clases | ¿Cómo se llama el filtro en tu controlador (`@Query`)? |
| Entrar con tu usuario | "Error 404" | Tu API no tiene `GET /users`. Su login es `POST /auth/login` (te da solo `{ accessToken }`) y quién eres lo dice `GET /auth/me` con la cabecera `Authorization: Bearer ...` (te da `{ id, email, role }`). Cambia el `authService` y el endpoint `getMe` de `api.ts` |
| Crear cuenta | `404`, o un `400` que dice `property role should not exist` | ¿Qué ruta registra usuarios en tu API? ¿Qué campos acepta tu `RegisterDto`? ¿Quién decide el rol? |
| Crear cuenta con un correo que ya existe | Ahora **sí** da error: `409` | No hay que arreglar nada: comprueba que se ve el mensaje de tu API |
| Crear una clase | `400`: `property trainerName should not exist` | Mira tu `CreateClassDto`. ¿Quién sabe el nombre del monitor, la web o la API? |
| Crear o borrar una clase | `401` | ¿Sigues con la sesión de json-server (`token-fake-1`)? Sal y vuelve a entrar con tu login nuevo |
| Borrar el primer Spinning | `409` | No hay que arreglar nada: tiene reservas y tu API no deja. Comprueba que se ve su mensaje |
| Nada falla, pero... | `types.ts` dice que los `id` son `string` | Mira en Red qué `id` devuelve tu API. Corrige los tipos para que digan la verdad |

**Pregunta 8:** en tu `main.ts` de la API, comenta la línea de `enableCors`, recarga la web y mira la consola y la pestaña Red. Copia la línea importante del error. ¿La API ha recibido la petición y ha respondido? ¿Quién la bloquea? ¿Por qué con Postman sí funciona? (Después, descomenta la línea.)

**Pregunta 9:** con el login nuevo, copia tu token de `localStorage` en [jwt.io](https://jwt.io). ¿Qué datos lleva dentro? ¿Lleva tu rol? ¿Para qué hace falta entonces `GET /auth/me`?

## 5.5 Tu diario de errores

En `respuestas.md`, añade una sección **Diario de errores** con una fila por cada error que hayas arreglado (como mínimo, 5):

```markdown
## Diario de errores

| Qué vi (síntoma y código) | Causa | Cómo lo arreglé (fichero) | Qué le pregunté a la IA |
|---|---|---|---|
| Buscar "yoga" daba las 7 clases | Mi API filtra con ?activity=, no con ?activity:contains= | Cambié getClasses en api.ts | "¿Por qué no filtra?..." / Sin IA |
```

En la columna de la IA, resume la pregunta y di si su respuesta fue buena a la primera, si tuviste que corregirla o si te llevó por mal camino.

## 5.6 Publícala: Netlify + Render

Cuando todo funcione en tu ordenador, conecta las dos partes publicadas:

```text
https://fitval-tunombre.netlify.app  ──pide datos a──▶  https://api-fitval-tunombre.onrender.com/api
            (la web, Netlify)                                  (la API, Render)
```

1. **Netlify**: **Project configuration** › **Environment variables** › **Add a variable**: `VITE_API_URL` = `https://api-fitval-tunombre.onrender.com/api`. Después, **Deploys** › **Trigger deploy** › **Deploy project**.
2. **Render**: en tu servicio `api-fitval`, **Environment** › `CORS_ORIGIN` = `https://fitval-tunombre.netlify.app` (sin `/` al final). Guarda: Render vuelve a desplegar.

> Las variables `VITE_` se meten **dentro** del JavaScript al hacer el build. Por eso, si las cambias, hay que volver a desplegar.

> Si arreglaste algo en la API (por ejemplo, CORS), haz su PR en `api-fitval` y fusiónala: Render la despliega sola.

| Prueba (en Netlify) | Resultado esperado |
|---|---|
| Abrir `/` con la API dormida | "Cargando planes..." y, a los 5 s, el aviso de que se está despertando |
| Recargar en `/classes?activity=yoga` | El horario filtrado, sin 404 |
| Login como staff, crear y borrar una clase | Funciona igual que en local |
| Recargar en `/staff` con sesión | Sigues en el panel |

### Ejercicio 5.1. Rómpelo a propósito

En Render, cambia `CORS_ORIGIN` a `https://otra-web.netlify.app`. Espera al despliegue y recarga tu web. Mira la consola. Después déjalo bien.

**Resultado esperado:** los mensajes de error en la web y el error de CORS en la consola, igual que en la pregunta 8.

## Pruebas de la Parte 5

Con tu API (primero en local y después en Netlify):

| Prueba | Resultado esperado |
|---|---|
| `/` | Los 4 planes |
| Buscar `yoga` | Solo las clases de yoga |
| `/classes/99` y `/classes/abc` | "Esa clase no existe." |
| Login con contraseña mal | "Correo o contraseña incorrectos." |
| Login como staff | Va a `/staff` y sale "Panel" |
| Registro con un correo que ya existe | El mensaje del `409` de tu API |
| Crear una clase | Aparece en la tabla y en `/classes`, con el nombre del monitor |
| Borrar la clase nueva | Desaparece |
| Borrar el primer Spinning | El mensaje del `409` de tu API |

En la PR: la tabla con tus resultados y el enlace al diario de errores. En el `README.md`, añade la dirección de Netlify.

---

# Entrega

Una sola entrega en Moodle, al terminar la Parte 5:

1. La dirección de tu repositorio `fitval-web`, con las PR de las partes 1 a 5 **fusionadas** y los checks en verde.
2. La dirección de la etiqueta de la primera parte: `https://github.com/TU_USUARIO/fitval-web/tree/v1-json-server`.
3. La dirección de Netlify: `https://fitval-tunombre.netlify.app`, funcionando con tu API de Render.
4. El archivo `respuestas.md` completo: preguntas 1 a 9 y el **diario de errores**.

Antes de entregar, comprueba:

- [ ] La web publicada funciona con tu API de Render.
- [ ] El `.env` **no** está en GitHub. Ningún secreto en el código, en el `README` ni en las PR.
- [ ] `npm run lint` y `npm run build` terminan sin errores.

---

## Si algo falla

| Síntoma | Causa probable |
|---|---|
| "No se han podido cargar los planes." y en Red `Failed to fetch` | json-server (o tu API) no está arrancado, o el puerto de `VITE_API_URL` no es el suyo |
| `npm run api` da `EADDRINUSE` | Ya tienes algo en el puerto 3001 (otro json-server abierto en otra terminal) |
| `db.json` está lleno de pruebas | Cópiale encima `db.initial.json` |
| `import.meta.env.VITE_API_URL` es `undefined` | El `.env` no está en la raíz, la variable no empieza por `VITE_` o no has reiniciado `npm run dev` |
| En Netlify, recargar da `404` | Falta `public/_redirects` |
| En Netlify, la web llama a `localhost` | Falta `VITE_API_URL` en Netlify, o no has vuelto a desplegar después de ponerla |
| En la consola: `blocked by CORS policy` | La API no tiene `enableCors`, o `CORS_ORIGIN` no es exactamente la dirección de tu web (¿`http` o `https`? ¿una `/` al final?) |
| `npm run build` avisa: `Some chunks are larger than 500 kB` | Es un aviso, no un error: el build ha terminado bien. Puedes ignorarlo |
| El login da `429` | Si hiciste la ampliación, tu API solo deja 5 intentos de login por minuto. Espera un minuto |
| La página se queda en blanco | Mira la consola: casi siempre es un error de JavaScript con el nombre del fichero y la línea |
| `Cannot find module './ClassesPage'` | El fichero no existe todavía o el nombre no coincide (mayúsculas) |

---

## Retos opcionales

1. **Monitores desde la API.** Añade `GET /trainers` a tu API y cambia `TRAINERS` por una query de RTK Query.
2. **Confirmar antes de borrar.** Usa `window.confirm("¿Seguro?")` antes de `deleteClass`.
3. **Buscar sin saturar la API.** Espera 300 ms desde la última tecla antes de cambiar la URL (reto 4 del apunte 03).
4. **Reservar plaza.** Si hiciste el reto de reservas de la entrega de NestJS: un botón "Reservar plaza" en el detalle, solo con sesión, que hace `POST /classes/:id/bookings`.
5. **Horario por páginas.** Si hiciste la ampliación: usa `GET /api/v2/classes?page=1&limit=3` y añade botones "Anterior" y "Siguiente".
6. **Más tests, escritos por ti** (apunte 05, apartado 4):
   - `getErrorMessage`: con `{ data: { message: ["a", "b"] } }` devuelve `"a. b"`, y con cualquier otra cosa, "No se ha podido conectar con la API.".
   - `PlanCard`: muestra el nombre y el precio (`"29,90"`; ojo, entre el número y el `€` va un espacio especial). Como lleva un `Link`, píntalo dentro de un `<MemoryRouter>`.
   - `RegisterPage`: con `@testing-library/user-event`, escribe una contraseña de 3 letras, pulsa "Crear cuenta" y comprueba que sale "La contraseña debe tener al menos 8 caracteres". Necesita el `Provider` de Redux con el `store`.

---

## Documentación para consultar

- React: <https://react.dev/learn>
- React Router: <https://reactrouter.com/>
- Redux Toolkit y RTK Query: <https://redux-toolkit.js.org/rtk-query/overview>
- React Hook Form: <https://react-hook-form.com/get-started>
- Zod: <https://zod.dev/>
- json-server: <https://github.com/typicode/json-server>
- CSS Modules en Vite: <https://vite.dev/guide/features#css-modules>
- Variables de entorno en Vite: <https://vite.dev/guide/env-and-mode>
- CORS (MDN): <https://developer.mozilla.org/es/docs/Web/HTTP/Guides/CORS>
- Netlify, redirecciones: <https://docs.netlify.com/manage/routing/redirects/overview/>
