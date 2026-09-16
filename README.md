# Argentina nuevos eventos — arreglos para 1.13.11

Arreglos para el mod
[La Argentina: New Events (Add-on)](https://steamcommunity.com/sharedfiles/filedetails/?id=3750562635)
de Victoria 3, verificados contra el juego **1.13.11 (Matcha)** con todos los DLC.

Los arreglos del mod principal están en un repo aparte:
[la-argentina-fixes](https://github.com/Juanpintoselso33/la-argentina-fixes).

## Estado del mod

**Con vanilla: anda.** En una partida nueva de prueba, la journal entry de la era de Rosas queda
activa y sus eventos disparan. Lo único que estaba mal eran los bugs de script de más abajo.

**Con Better Politics Mod: no anda sin compatch.** Todos sus eventos piden `law_autocracy`, y con
BPM Argentina arranca con **Military Junta**. Resultado: no dispara nada, ni Rosas, ni la Mazorca,
ni los bloqueos, ni los salones, ni los saladeros, ni los eventos de Piratini. El compatch de este
repo lo resuelve, y está verificado en partida: con BPM y compatch, en julio de 1838 la journal
entry de Rosas está activa y el compatch no agrega ningún error al log.

**Lo que está estructuralmente bien**, revisado y sin hallazgos: no reemplaza ningún archivo del
juego, no tiene objetos definidos dos veces, no le falta ningún `=`, no llama eventos inexistentes
y no tiene referencias rotas a leyes, grupos de interés, journal entries ni modificadores.

## Los bugs y sus arreglos

### 1. `owner = this` dentro de `every_state`

En `events/customs_laws.txt`, líneas 84 y 93. Dentro de `every_state`, `this` es el estado, no el
país, así que compara un país con un estado. **Genera 1.942 errores en tres meses de juego**, que
era el 25 % de todo el `error.log`. Va `owner = root`.

### 2. `c:PNI` sin comprobar que exista

En `events/piratini.txt`, línea 91: `c:PNI = { exists = yes ... }`. Si Piratini no existe, esa
referencia ya falla antes de poder comprobar nada. Va `exists = c:PNI` y después `c:PNI ?= { ... }`.

### 3. Modificador mal escrito

En `common/laws/historical_economic_system.txt`: `ountry_production_tech_research_speed_mult`, sin
la `c` inicial. No existe, así que **Agrarianismo y Economía Extractiva no aplican ese efecto**.

### 4. Modificador inexistente

En `common/laws/historical_slavery.txt`: `building_group_bg_light_industry_throughput_mult`. Para
grupos de edificios el nombre válido termina en `_add`. **Trata de Esclavos y Esclavitud Heredada
pierden ese efecto.**

## Compatch con Better Politics Mod

En `bpm-compatch/`. Hace dos cosas.

**Una:** amplía las nueve condiciones de `law_autocracy` a `law_autocracy`,
`law_military_junta` y `law_oligarchy`, en estos archivos: `events/supreme_power.txt`,
`events/mazorca.txt`, `events/salons.txt`, `common/journal_entries/rosas.txt`,
`common/decisions/new_laws.txt` y `common/scripted_buttons/mazorca.txt`.

**Dos:** destraba la **Era de los Caudillos** (`events/brazil/caudillo.txt`). El evento del juego
que abre esa journal entry exige que las Fuerzas Armadas tengan `ideology_caudillismo`, pero con
BPM nunca la reciben: su efecto `bpm_ig_make_caudillismo` la traduce a `ideology_dop_oligarch` y
`ideology_gov_liberal_republican`. Con BPM, entonces, la Era de los Caudillos **no arranca nunca**,
y eso además deja inalcanzable el contenido que el propio BPM escribió para los caudillos, porque
se dispara desde esa misma journal entry. El compatch acepta las dos combinaciones.

Va cargado después del mod.

### Orden de carga con BPM

Conviene cargar **este mod antes que BPM**. Redefine nueve leyes que BPM también redefine
(agrarianismo, latifundios, latifundios expandidos, arrendatarios, propiedad campesina,
agricultura comercial, trata de esclavos y esclavitud heredada). Si carga después, le pisa las
versiones de BPM y rompe su sistema político.

## Cómo usarlo

- **Mod ya corregido:** `mod/`. Es el mod completo **sin los 14 videos de eventos** (`.bk2`,
  155 MB), que no hacía falta duplicar acá: copiá `mod/` sobre tu instalación y los videos quedan
  como están.
- **Cambio por cambio:** `parches/`, en formato diff unificado contra el original.
- **Compatch:** `bpm-compatch/`, como mod local aparte.
