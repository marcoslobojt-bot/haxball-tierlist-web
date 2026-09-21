# Prime Hax — primehaxball.com.ar

Sitio informativo del equipo: fixture en calendario, las 3 ligas (Renacimiento, AFAHAX, LNEH), planteles, firmas, guía del SS y panel de carga.

## Archivos
- `index.html`: todo el sitio (HTML, CSS y JS).
- `assets/`: escudo de Prime y logos de las ligas.
- `database.rules.json`: reglas de la Realtime Database.
- `CNAME`: dejar el que ya está en el repo.

## Datos (Realtime Database, nodo /prime)
- `jugadores/{id}`: nombre y usuario de Discord.
- `ligas/{id}`: links, cierre de firma, cupo, Anti-DF, DF y `plantel/{jugador}` con estado, rol y nick.
- `partidos/{id}`: liga, fecha, hora, sala, rival, local/visitante, goles, tipo (DF/DN), rec, firmas y SS por jugador.
- `tablas/{liga}`: posiciones en texto, una línea por equipo.
- `reglamentos` y `servidor`: textos de reglas.

Los admins salen de `config/admins/{uid}: true`, igual que antes.
