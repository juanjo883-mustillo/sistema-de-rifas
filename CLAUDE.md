# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
npm run dev       # Start dev server (Vite HMR)
npm run build     # Production build
npm run lint      # ESLint
npm run preview   # Preview production build
```

## Architecture

React 19 + Vite SPA with Firebase (Firestore + Auth) backend. No server — all logic runs client-side with real-time Firestore listeners.

### Routing (src/App.jsx)

| Route | Page | Access |
|---|---|---|
| `/` | Home | Public |
| `/pago` | Pago | Public (needs router state) |
| `/confirmacion` | Confirmacion | Public |
| `/mi-reserva` | MiReserva | Public |
| `/login` | Login | Public |
| `/vendedor` | Vendedor | Firebase Auth required |

### Firebase layer (src/firebase/)

- **config.js** — initializes Firebase app, exports `db` (Firestore) and `auth`
- **rifaService.js** — all Firestore operations: one active rifa document at `rifas/activa`, participants in `participantes` collection

### Data model

**`rifas/activa`** — single document for the active raffle:
- `titulo`, `descripcion`, `precio`, `whatsapp`, `linkMercadoPago`
- `numeros` — map of `"1"..="100"` → `{ estado: 'libre' | 'ocupado' }`
- `reservas_temp` — map of number → `{ reservaId, expiraEn }` (10-min temp locks)
- `premios[]` — array of `{ id, nombre, descripcion, foto }`
- `fechaFin` — Timestamp or null

**`participantes`** collection — one doc per reservation:
- `numeros[]`, `nombre`, `apellido`, `telefono`, `nombrePago`, `metodoPago`, `pagado`, `_eliminado`

### Key flows

**Reservation (user side):**
1. `Home` → user selects numbers → `crearReservaTemporal()` locks them 10 min → navigate to `/pago` with router state
2. `Pago` → user fills personal data → `reservarNumeros()` marks numbers `ocupado` and creates participant doc → navigate to `/confirmacion`

**Vendor panel (`/vendedor`):**
- Protected via `onAuthStateChanged`; redirects to `/login` if unauthenticated
- Real-time participant list via `suscribirParticipantes()`
- `RifaEditor` component (modal) for editing rifa config and prizes
- Reset: `resetearNumeros()` sets all numbers back to `libre` and soft-deletes all participants (`_eliminado: true`)

### Custom hook

`useRifa()` (`src/hooks/useRifa.js`) — initializes rifa if missing, subscribes to real-time updates, returns `{ rifa, cargando, errorFirebase }`.

### Styling

Tailwind CSS + inline styles. Dark theme (`#1a1a1a` bg, `#c9a035`/`#e8bc4a` gold accent). Font `font-rustica` used for headings (loaded separately, not via npm). No UI component library.
