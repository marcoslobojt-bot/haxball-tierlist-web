# Herramientas de tierlist — PRIME HAX

Dos páginas estáticas independientes, ambas sincronizadas en vivo entre
todos los que las tengan abiertas al mismo tiempo, vía Firebase Realtime
Database (el mismo sistema que usaste para la página del torneo):

- **`revelado-tiers.html`** — revela uno por uno el tier de cada jugador
  (sin armar equipos), tipo lotería.
- **`draft-equipos.html`** — arma equipos sorteando por ruleta a qué equipo
  va cada jugador, respetando que cada equipo complete 1 Tier A + 1 Tier B +
  el resto de su plantel (C/D/E mezclados) antes de pasar a la siguiente
  fase. Es la que arma el torneo de ejemplo con 8 equipos.

Ambas comparten la misma config de Firebase — configurala una sola vez.

## 1. Firebase

**Si ya tenés el proyecto de Firebase del torneo (`haxball-torneo`):** podés
reusarlo, no hace falta crear uno nuevo. Anda a
https://console.firebase.google.com → tu proyecto → ⚙️ **Configuración del
proyecto** → bajá hasta "Tus apps" → copiá el objeto `firebaseConfig`.

**Si preferís uno nuevo:** creá un proyecto en
https://console.firebase.google.com → **Firebase Database → Realtime
Database → Crear base de datos** → modo de prueba está bien para esto.
Después andá a **Configuración del proyecto** y copiá el `firebaseConfig`
igual que arriba.

Pegá ese objeto en **ambos archivos** (`revelado-tiers.html` y
`draft-equipos.html`), reemplazando este bloque cerca del final de cada uno:

```js
const firebaseConfig = {
  apiKey: "TU_API_KEY",
  authDomain: "TU_PROYECTO.firebaseapp.com",
  databaseURL: "https://TU_PROYECTO-default-rtdb.firebaseio.com",
  projectId: "TU_PROYECTO",
  storageBucket: "TU_PROYECTO.appspot.com",
  messagingSenderId: "TU_SENDER_ID",
  appId: "TU_APP_ID"
};
```

### Reglas de la base de datos

Como esta página escribe a `/ruleta` desde el navegador sin login, en
**Realtime Database → Reglas** dejá algo simple tipo:

```json
{
  "rules": {
    "ruleta": {
      ".read": true,
      ".write": true
    }
  }
}
```

Esto es lo mismo de siempre: cualquiera con el link de admin (PIN aparte)
puede escribir. Para algo interno de un server de Discord está bien; no uses
esta config para algo con datos sensibles.

## 2. Cambiar el PIN de admin

Buscá esta línea en `index.html` y poné el PIN que quieras:

```js
const ADMIN_PIN = "3156";
```

## 3. Publicarlo (GitHub Pages)

Mismo flujo que ya conocés:

```bash
git init
git add .
git commit -m "Ruleta de revelado tierlist"
git branch -M main
git remote add origin https://github.com/TU_USUARIO/NOMBRE_REPO.git
git push -u origin main
```

Después, en GitHub: **Settings → Pages** → Source: rama `main`, carpeta
`/root` → Save. En un minuto queda publicada en
`https://TU_USUARIO.github.io/NOMBRE_REPO/`.

(También podés simplemente agregar este `index.html` a tu repo existente
`haxball-torneo` en una subcarpeta, ej: `/ruleta/index.html`, si preferís no
crear un repo nuevo.)

## 4. Cómo usarla

- **Link público** (para postear en Discord): `https://tu-link/`
  Todos los que lo abran ven exactamente la misma animación al mismo tiempo.

- **Link de admin** (solo para vos): `https://tu-link/?admin=1`
  Te pide el PIN y te muestra el panel para cargar los resultados y disparar
  cada revelado.

### Pasos para revelar una tierlist

1. Cerrá la votación en Discord con `!haxcerrar` — copiá el resumen final
   que postea el bot (nombre y tier de cada jugador).
2. Abrí el link de admin, metés el PIN.
3. Pegá esa lista en el textarea, **una línea por jugador**, formato:
   ```
   Duko: B
   Paredes: B
   AmadeoNF: A
   ```
4. **Cargar resultados y barajar orden** — arma un orden aleatorio de
   revelado y lo guarda en Firebase (nadie lo ve todavía).
5. Compartí el link público en el canal de Discord.
6. Cuando estén todos mirando, clic en **🎬 Revelar siguiente** — la ruleta
   gira ~4 segundos y se detiene en el tier real de ese jugador, para todos
   al mismo tiempo. Repetís el botón para cada jugador.
7. Al terminar, la página muestra el resumen completo agrupado por tier.

**Reiniciar todo** borra el progreso — usalo solo si te equivocaste al
cargar los datos, no en medio de un revelado real.

---

## 5. Draft de equipos (`draft-equipos.html`)

Pensada para tu ejemplo de 8 equipos, 4 jugadores cada uno (1 Tier A + 1
Tier B + 2 más de C/D/E). El tamaño de plantel y la cantidad de equipos son
editables en el panel de admin, no están fijos en el código.

### Cómo funciona el sorteo

1. **Fase Tier A**: toma los jugadores Tier A en orden aleatorio y a cada
   uno lo sortea entre los equipos que todavía no tienen su Tier A.
2. **Fase Tier B**: igual, pero solo entre equipos sin Tier B todavía.
3. **Completando equipos**: los Tier C, D y E se juntan en un solo pool
   mezclado, y cada uno se sortea entre los equipos que todavía tienen
   lugares libres, hasta completar el tamaño de plantel definido.

Si en algún momento sobran jugadores de Tier A o B porque ya todos los
equipos tienen el suyo, esos jugadores pasan automáticamente al pool de
"resto" en vez de quedar sin asignar.

### Pasos para usarla

1. Abrí el link de admin (`?admin=1`) y metés el PIN.
2. En **"Nombres de los equipos"**, dejá o editá la lista (una línea por
   equipo — no tiene que ser justo 8, se ajusta sola).
3. En **"Jugadores por equipo"**, confirmá el tamaño de plantel (4 por
   defecto).
4. Pegá la lista completa de jugadores con su tier, formato `Nombre: Tier`
   — la podés armar copiando el resumen de `!haxcerrar` del bot de Discord.
5. **Cargar torneo** — arma los pools y arranca en cero (nadie ve nada
   todavía).
6. Compartí el **link público** (sin `?admin=1`) en el canal.
7. Con todos mirando, andá clickeando **🎬 Sortear siguiente** — cada click
   gira la ruleta ~4 segundos y muestra a qué equipo le tocó ese jugador.
   El botón avanza solo de fase (A → B → resto) a medida que se vacía cada
   pool.
8. Cuando diga "Draft completo", los 8 equipos ya están armados y visibles
   en la grilla de abajo, para todos los que tengan el link abierto.

**Ojo:** no hagas doble click ni dispares otro sorteo mientras la ruleta
sigue girando — el botón te avisa si intentás superponer sorteos.
