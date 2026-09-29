# AGENTS.md — Calito (Stellaris mod)

Documento de orientacion para agentes de IA que trabajen en este mod.
Verificado contra **Stellaris 4.5.1 (Cygnus v4.5.1, build 358e)**.

---

## 1. Que es este mod

Mod de escenario para Stellaris. Contiene dos cosas:

1. **El escenario "5 Arks"** (lo principal, nuevo).
   Convierte un imperio asentado en un **imperio nomada postapocaliptico**:
   la Tierra fue destruida, solo cinco arcas escaparon, y ahora el jugador
   navega con ellas. Crea las especies, los arcos, la poblacion y el origen.
2. **`calito_spawn`** (heredado, arreglado).
   El spawn original: 30 grupos de poblacion humana en Tesia (planeta del
   mod Asari) mas tres flotas de Gigastructural Engineering.

La localizacion va en ingles y espanol. **El usuario juega en espanol.**

---

## 2. Estructura de ficheros

```
calito/
├── calito.mod                      # descriptor que lee el launcher
├── descriptor.mod                  # mismo descriptor dentro de la carpeta
├── AGENTS.md                       # este fichero
├── MOD_REFERENCES.md               # rutas de referencia
├── events/
│   ├── 00_calito_test_events.txt      # eventos de CONSOLA (is_test_event)
│   ├── 01_calito_five_arks_events.txt # ventana jugable (calito.100/110/120)
│   ├── 02_calito_gatekeeper_events.txt# puente on_action -> evento
│   └── calito_events.txt              # calito.8005 heredado (huerfano)
├── common/
│   ├── on_actions/
│   │   └── 00_calito_on_actions.txt   # engancha el scenario al juego
│   ├── script_values/
│   │   └── 00_calito_values.txt       # constantes @calito_*
│   ├── scripted_triggers/
│   │   └── 00_calito_triggers.txt     # triggers calito_*
│   ├── scripted_effects/
│   │   ├── 01_calito_species_effects.txt      # creacion de especies
│   │   ├── 02_calito_ark_effects.txt          # creacion de arcos
│   │   ├── 03_calito_five_arks_effects.txt    # orquestador
│   │   ├── 04_calito_ark_upgrade_effects.txt   # arcos -> tier 3 + kitting
│   │   ├── 05_calito_government_effects.txt   # monarquia absolutista
│   │   ├── 06_calito_character_effects.txt    # reina + investigadora
│   │   └── calito_effects.txt                 # calito_spawn (heredado, arreglado)
│   ├── static_modifiers/
│   │   └── 00_calito_static_modifiers.txt
│   ├── governments/
│   │   ├── civics/01_calito_origins.txt       # origen de script
│   │   └── councilors/01_calito_councilors.txt# cargo de research
│   └── traits/
│       ├── 01_calito_leader_traits.txt        # rasgos de los dos leaders
│       └── sm_scientist_traits.txt            # rasgo huerfano (ver §7)
└── localisation/
    ├── english/calito_l_english.yml
    └── spanish/calito_l_spanish.yml
```

### La flota de 5 arcos

| Arco | Tipo | Poblacion | Especie | Kitting |
|---|---|---|---|---|
| **Genesis** (capital) | el que elige el jugador | 2500 | Asari | tier 3 + Mansion Noble |
| **Ferrum** | military | 2000 | Asari | tier 3 + **Devastator** (arma titanic) |
| **Argent** | science | 1250 | Asari | tier 3 + sensores |
| **Cinis** | civilian | 2000 | Asari | tier 3 + economia |
| **Vigil** | science | 1250 | Asari | tier 3 + sensores |

Todas se suben a **tier 3** y se kitean. Si el jugador elige militar en
`calito.100`, habrá dos arcos militares; es intencional, para que la
elección se note.

### Los dos personajes

| Personaje | Clase | Edad | Retrato | Cargo | Poder |
|---|---|---|---|---|---|
| **Sela'vhi T'Neyt** (reina) | official | **19** | `mec_asari_1` | `councilor_ruler_oligarchic` (el trono) | `calito_leader_trait_iron_vow` — mando |
| **Archivera Vethara Il'in** | scientist | **767** | `mec_asari_4` | `calito_councilor_ark_research` | `calito_leader_trait_ark_of_mnemosine` — research +100% |

`@calito_research_buff = 1.00` es **10x** el mejor rasgo nativo
(`leader_trait_maniacal_3` da 0.10). Bajarlo en `00_calito_values.txt`
afecta a la vez al rasgo, al cargo y al buff de la leader.

### Flujo del escenario en partida

```
on_game_start_country
   -> calito.gatekeeper        (oculto, programa 30 dias + cooldown de 20 anos)
      -> calito.100            (VENTANA: elegir tipo de arco, o cancelar)
         -> calito.five_arks_setup_effect
            -> calito.110         (confirmacion; marca asari_done)

on_yearly_pulse_country
      -> calito.reoffer_gate    (oculto, re-ofrece calito.100 si no se eligio)
```

Un "no" en `calito.100` pone `calito_five_arks_aborted` y no vuelve a aparecer.
La oferta de asari a 180 dias se ELIMINO: los asari son la UNICA especie nueva
y ya vienen a bordo en el setup, asi que no hay segunda tanda que ofrecer.

---

## 3. Como probarlo

### Desde la consola de depuracion del juego

Todos los eventos de `events/00_calito_test_events.txt` son
`is_test_event = yes` + `trigger = { always = no }`, asi que **nunca se
disparan solos**. Se lanzan a mano:

| Comando | Que hace |
|---|---|
| `event calitotest.1` | **ESCENARIO COMPLETO "5 Arks"** (el MVP) |
| `event calitotest.2` | Anade un unico arco cientifico |
| `event calitotest.3` | Crea solo la especie asari |
| `event calitotest.4` | Crea solo la especie humana superviviente |
| `event calitotest.5` | `calito_spawn` original (Tesia + Giga) |
| `event calitotest.9` | Revierte el estado nomada |

> **OJO: el namespace es `calitotest`, no `calito`.** Ver §7.1.

> **OJO: los eventos de test NO tienen `trigger`.** La consola de
> depuracion **si evalua el trigger del evento** antes de dispararlo.
> Vanilla usa `is_test_event = yes` + `trigger = { always = no }` en sus
> eventos de test, y copiar ese patron hace que `event <id>` falle
> siempre con un error de condicion. Para que el evento no se dispare
> solo basta `is_triggered_only = yes`, que es lo que se usa aqui.

Para lanzar el comando hay que tener el pais **seleccionado** en el mapa.

### En partida

`calito.100` se abre sola. Flujo:

1. Arranca una partida normal (cualquier imperio, cualquier origen).
2. A los ~30 dias aparece **"Los Ultimos Cinco"**. Elige arco civil,
   militar o cientifico.
3. El script convierte tu imperio en nomada, crea las dos especies,
   convierte tu planeta inicial en el arco principal, crea 4 satelites
   y siembra todo de poblacion.
4. A los ~6 meses aparece **"Voces en una Frecuencia Muerta"**: los asari.
5. En los anos siguientes el juego ya es un escenario nomada normal.

Si quieres saltar al escenario sin jugar la ventana:

| Comando | Que hace |
|---|---|
| `event calitotest.1` | ESCENARIO COMPLETO, sin ventanas intermedias |
| `event calitotest.9` | Revierte el estado nomada (no borra los arcos) |

> **Nota:** `calitotest.9` NO borra los arcos ya creados. Para una
> prueba de verdad, guarda y recarga la partida.

### Logs

`%USERPROFILE%\Documents\Paradox Interactive\Stellaris\logs\error.log`

---

## 4. Dependencias

| Mod | Workshop ID | Para que |
|---|---|---|
| **Mass Effect Civilizations - Asari** | `790903721` | **Imprescindible.** Retrato `mec_asari` y name list `mec_asari_names`. **No** se usa ningun rasgo suyo |
| **Gigastructural Engineering & More** | `1121692237` | Solo para `calito_spawn` heredado. El escenario "5 Arks" **no** lo necesita |
| **Nomads (DLC de pago)** | - | Imprescindible: arkships, `is_nomadic`, `pc_ark` |

El escenario "5 Arks" crea **una sola especie, la asari**, y por eso el mod
Asari paso de ser opcional a ser obligatorio: sin el, `portrait = mec_asari`
no existe y `create_species` falla. El imperio conserva su propia especie,
asi que el jugador sigue jugando con la que quiera.

**Por que no se usa `mec_biotics_trait_biotic`:** arrastra
`resources = { category = planet_mec_biotics  inline_script = "traits/mec_biotics_resources_augment" }`.
Si esa categoria o ese inline_script no resuelven, `create_species` **aborta
entero**: no crea la especie y no aparece ni un pop. El retrato viene del
`portrait_set`, no de un rasgo, asi que los rasgos van todos vanilla.

---

## 5. Rutas de referencia

```
JUEGO BASE      -> E:\SteamLibrary\steamapps\common\Stellaris
WORKSHOP        -> E:\SteamLibrary\steamapps\workshop\content\281990\1121692237   (Giga)
                   E:\SteamLibrary\steamapps\workshop\content\281990\790903721    (Asari)
LOGS            -> %USERPROFILE%\Documents\Paradox Interactive\Stellaris\logs\error.log
```

Documentacion oficial del juego, muy util y en la propia instalacion:

| Fichero | Contenido |
|---|---|
| `events/000_added_pre_triggers_to_planet_events.txt` | `pre_triggers` |
| `events/000_fleet_action_examples.txt` | `queue_actions` |
| `events/000_how_to_use_variables_in_script.txt` | Variables |
| `common/on_actions/99_README_ON_ACTIONS.txt` | On actions |
| `common/situations/99_README_SITUATIONS.txt` | Situaciones |
| `common/inline_scripts/00_README.txt` | Inline scripts |
| `common/scripted_effects/99_advanced_documentation.txt` | Parametros de scripted effects |
| `common/special_projects/documentation.txt` | Special projects |

Ademas hay una guia completa en
`Stellaris/mod/GUIA_STELLARIS_EVENTOS/` (7 ficheros markdown, en espanol).

---

## 6. Reglas del motor que hay que respetar

Estas son las trampas reales de Stellaris 4.5.1. Casi todos los bugs de este
mod venian de ignorarlas.

### 6.1 El orden technology -> designs -> create_ship

**Este es el fallo mas comun y el mas silencioso.**

```txt
give_technology = { tech = tech_civilian_arkship }   # 1
refresh_auto_generated_ship_designs = yes             # 2
create_ship = { random_existing_design = civilian_arkship_tier_1 }   # 3
```

Si te saltas el paso 2, `random_existing_design` no encuentra diseno y
**la nave no se crea, sin error ni aviso**.

### 6.2 `random_existing_design` vs `design`

| Forma | Que espera |
|---|---|
| `design = "Nombre Del Diseno"` | El id literal del `ship_design` (case sensitive) |
| `random_existing_design = <ship_size>` | Elige al azar entre disenos **que el pais ya posee** de ese tamano |

`random_existing_design` con un `ship_size` es una bomba: si el jugador no
tiene ninguno de ese tamano, no ocurre nada. Para contenido de mod, usa
`design = "..."` con el id exacto.

### 6.3 Los modificadores van en `common/static_modifiers/`

Un `add_modifier = { modifier = X }` solo resuelve si `X` esta declarado
como modificador. **No vale declararlo en `common/scripted_effects/`**
(eso es para efectos, no para modificadores).

Si el modificador no existe, el error va a `error.log` y el efecto no
aplica, pero el juego **sigue funcionando**.

### 6.4 `trait_survivor` no puede ir en `create_species`

```txt
trait_survivor = {
	species_potential_add = { always = no }   # <-- por esto
	...
}
```

`always = no` significa que el jugador no puede elegirlo, y
`create_species = { traits = { trait = trait_survivor } }` **tampoco**
funciona. Hay que anadirlo despues:

```txt
create_species = {
	traits = { trait = trait_organic ... }
	effect = {
		change_species_characteristics = { add_trait = trait_survivor }
	}
}
```

Es exactamente lo que hace vanilla para `origin_post_apocalyptic`
(`common/scripted_effects/01_start_of_game_effects.txt:2149`).

### 6.5 `is_nomadic` es un country code flag, no un origen ni un country type

```txt
set_country_as_nomad_effect = { IS_NOMAD = yes }   # efecto vanilla
```

Eso es todo. `set_country_as_nomad_effect` **solo pone el flag**: no da
tecnologias, no crea arcos, no lanza situaciones. Todo lo demas hay que
hacerlo a mano.

No existe un `country_type` nomada para cambiar. El `country_type = nomad`
existe pero es para el pais NPC de la historia Namarian.

### 6.6 Un arco sin colonia muere

De `common/ship_sizes/00_ship_sizes.txt`:

> *"Ships that are not carrying a colony when supposed to are killed on a
> daily basis."*

Si creas un arkship con `create_colony = no` y nunca llamas a
`transfer_carrier`, la nave desaparece. Por eso `calito_convert_planet_to_ark_effect`
crea con `create_colony = no` **y acto seguido** transfiere el planeta.

### 6.7 Los arcos son `shipclass_starbase` con `carries_colony = pc_ark`

No hay flag `is_arkship` ni `set_arkship`. Ser arco es una propiedad del
`ship_size`. Solo tres tamanos son "de arranque"
(`is_starting_arkship = yes`):

```
civilian_arkship_tier_1
science_arkship_tier_1
military_arkship_tier_1
```

Los tier 2 y 3 son solo mejoras.

### 6.8 `create_arkship_effect` de vanilla esta incompleto

`common/scripted_effects/nomads_effects.txt:6925` crea el arco pero **no**
llama a `initialize_arkship_starbase_effect`, asi que el arco nace sin
modulos ni edificios de estrella. Por eso este mod define su propia
version (`calito_create_ark_effect`).

### 6.9 La clase de especie del mod Asari

El mod Asari define la clase `MECASA`, pero **registra sus retratos contra
`HUM`**, no contra `MECASA`:

```txt
# common/portrait_sets/01_mec_asari_portrait_sets.txt
mec_asari = {
	species_class = HUM          # <-- HUM, no MECASA
	portraits = { "mec_asari" }
}
```

Por eso las especies de este mod usan `class = HUM` + `portrait = mec_asari`
(es lo que hace el propio mod Asari en sus paises prescriptados).
Usar `MECASA` haria que los retratos no cargaran y rompe el portrait
modding vanilla.

El trigger para comparar retrato es **`species_portrait`**, no `portrait`:

```txt
any_owned_species = { species_portrait = mec_asari }
```

### 6.10 No uses NINGUN rasgo del mod Asari

Ni `mec_asari_trait_core` (el "Amaranthine"), ni `mec_biotics_trait_biotic`.
La especie se crea con rasgos **vanilla** y nada mas.

`mec_asari_trait_core` esta descartado porque ademas de forzar todos los sexos
a hembra e imponer `mec_asari_leader_trait_cycle` a cada lider, se opone a
`trait_venerable` / `trait_enduring` y no se puede quitar nunca
(`species_possible_remove = { always = no }`).

`mec_biotics_trait_biotic` esta descartado por un motivo mas grave: arrastra

```txt
resources = {
	category = planet_mec_biotics
	inline_script = "traits/mec_biotics_resources_augment"
}
```

Si esa categoria o ese `inline_script` no resuelven, **`create_species` aborta
entero**: no crea la especie y no aparece ni un pop, sin error visible.

El retrato **no depende de ningun rasgo**: viene del `portrait_set`
(`portrait = mec_asari` con `class = HUM`). Los rasgos son solo
cosmeticos/gameplay, asi que perderlos no cuesta nada.

### 6.11 Sintaxis que NO existe en 4.5.1

Confirmado por grep sobre toda la instalacion:

```
space_event            -> ship_event
planet_event           -> carrier_event (planet_event es legacy)
diplomatic_event       -> country_event + diplomatic = yes
imminent_country_event -> country_event + mean_time_to_happen
create_pop             -> create_pop_group = { species = X }
add_society_value      -> add_monthly_resource_mult = { resource = unity }
add_obs_value          -> idem
add_mining_station     -> enable_special_project
add_building_construction -> add_building
min_days / max_days    -> days + random AL DISPARAR
auto_pause             -> no existe
hidden = yes (evento)  -> hide_window = yes
hide_effects           -> hidden_effect = { }
meta_effect            -> es de CK3, no existe aqui
on_action = { on_action = X } -> fire_on_action = { on_action = X }
set_criticality        -> set_crisis_stage_1..4
complete_crisis        -> end_crisis = yes
advance_event_chain    -> add_event_chain_counter
cancel_event_chain     -> end_event_chain
create_situation       -> start_situation
add_tech_minor         -> add_research_option + add_tech_progress
composite_trigger      -> AND / OR / NOR / NAND
custom_trigger         -> common/scripted_triggers/
```

### 6.12 Nombres reales de los efectos de etica, civics y gobierno

Verificado por grep. Usar estos, no los intuitivos:

| Querias | **Existe** | NO existe |
|---|---|---|
| Anadir etica | `country_add_ethic = ethic_x` | `add_ethic` |
| Quitar etica | `country_remove_ethic = ethic_x` | `remove_ethic` |
| Anadir civic | `force_add_civic = civic_x` | `add_civic` |
| Quitar civic | `force_remove_civic = civic_x` | `remove_civic` |
| Cambiar gobierno | `change_government = { authority = ... civics = ... }` | `set_government = gov_x` |

**El gobierno NO se puede fijar.** Solo cambias la **autoridad** y los
**civics**; el gobierno se recalcula solo tomando el primer bloque
`possible` que se cumpla. Por eso la monarquia se consigue con:

```
auth_oligarchic     -> la unica autoridad que cumple is_imperial_authority
                    -> la unica con herederos (dinastia)
+ ethic_militarist  -> selecciona gov_star_empire
                    -> titulos RT_EMPEROR / RT_EMPRESS
```

Los eticos **fanaticos no se pueden quitar**. Si el jugador es fanatico
espiritualista, pacifista, igualitario o xenofilo, `gov_star_empire` es
inalcanzable y el gobierno cae en `gov_theocratic_monarchy` o
`gov_irenic_monarchy`. El efecto lo detecta y escribe un `log_error`.

### 6.13 Arcos: ranuras, no capacidades

`starbase_module_capacity_add` y `starbase_building_capacity_add` son el
**tope**, pero los ranuras que existen salen de
`common/starbase_levels/06_arkship_levels.txt`:

| Tier | Ranuras de modulo | Ranuras de edificio |
|---|---|---|
| 1 | `1 2` | `1 2 3 4` |
| 2 | `1`..`4` | `1`..`8` |
| 3 | `1`..`6` + `spinal_mount_1` | `1`..`8` + `lower_1`..`lower_4` |

**El indice 7 de modulo NO es "el septimo modulo": es el ranur especial
`spinal_mount_1`.** Ahi va el Devastator (la unica arma titanic del
juego) o el Arsenal.

`set_starbase_size` **no comprueba nada**: puedes llamarlo con cualquier
tamanio en cualquier momento. Los tier 2 y 3 tienen
`potential_construction = { always = no }`, asi que no se pueden
CONSTRUIR: solo existen por script. Aun asi hay que dar las technologies
antes (`calito_give_ark_upgrade_techs`), porque cada modulo y edificio
comprueba `has_technology` en su bloque `potential`.

### 6.14 Armas: no se elige el componente

No existen `add_component`, `set_component` ni `set_hull`. Las armas de
un arco vienen por el `component_set` del modulo, y el motor lo resuelve
solo al nivel mas alto que hayas investigado. En la practica: para tener
laser t7, da `tech_lasers_*`; no puedes elegir el componente.

El unico componente que se anade a mano es
`add_starbase_component = { component = "CLAVE" }` (solo scope starbase,
y vanilla solo lo usa para auras).

### 6.15 `create_leader`: los parametros que existen

Verificados sobre los 385 `create_leader` del juego:

```
class sub_type tier name species gender set_age leader_age_min
leader_age_max immortal hide_age skill traits randomize_traits
event_leader can_assign_to_council can_manually_change_location
skip_background_generation background_ethic background_job
background_planet custom_description custom_catch_phrase
hide_leader effect
```

**Los que no existen** (y que se buscan por costumbre):

| No existe | Usar |
|---|---|
| `age = N` | `set_age = N` (directo, o dentro de `effect`) |
| `portrait = X` | `change_leader_portrait = X` **dentro de `effect = { }`** |
| `add_trait` como key directo | `add_trait = { trait = X }` dentro de `effect` |
| `is_ai` | no se puede forzar |
| `appearance` | no existe |

Muerte por edad (`common/defines/00_defines.txt`): 0% hasta los 80 anos,
+0.2%/ano hasta los 96, +2%/ano a partir de ahi. Por eso la reina (19)
es inmortal por edad sola, y la investigadora (767) necesita
`set_immortal = yes` o la mortandad se la come en unas cuantas decadas.

### 6.16 Dar el trono

No existe `set_ruler = <leader>`. El trono es un **cargo del consejo**:
`councilor_ruler_oligarchic`, declarado con
`possible = { always = no }` (el jugador no puede asignarlo, el script si).
El orden correcto es:

```
1. change_government = { authority = auth_oligarchic }   # el cargo debe existir
2. ruler = { kill_leader = { ruler = yes } }            # vaciar el trono
3. create_leader = { ... }
4. event_target:la_reina = { set_council_position = councilor_ruler_oligarchic }
```

Este es el punto mas fragil del mod. Si la reina no aparece como
gobernadora, el paso 2 es el sospechoso: mata al ruler y el juego genera
otro. Se comprueba en partida con `exists = ruler`.

---

## 7. Bugs historicos

### 7.0 Los 7 bugs de la PRIMERA ejecucion (arreglados)

Estos los encontre en el `logs/error.log` de la primera partida en la que se
intento lanzar el escenario. **Todos estan ya corregidos**, pero estan
aqui porque son las trampas que mas rompen cosas en Stellaris, y
porque volver a toparse con ellas es facil.

| # | Sintoma en error.log | Causa | Arreglo |
|---|---|---|---|
| 1 | `Event calito.test.1 ... has an invalid ID` | Los ids de evento con `is_test_event = yes` deben tener **exactamente 2 partes** (`namespace.NUMERO`). Los 25 eventos de test de vanilla lo cumplen; los 27 ids de 3 partes que existen en vanilla son eventos normales | Namespace propio `calitotest`, ids `calitotest.1` … `calitotest.9` |
| 2 | `Localization file ... should be in utf-8-bom encoding` / `Missing UTF8 BOM` | Los `.yml` de localizacion exigen **UTF-8 CON BOM**. PowerShell 7 escribe sin BOM | Reescritos con BOM en los dos idiomas |
| 3 | `Missing localization key [calito.100.desc]` (todas) | Consecuencia del 2: al descartar el fichero, desaparecian las 59 claves | Arreglado con el 2 |
| 4 | `Unknown inline_script "trait/icon_element/rarity_legendary"` | Los valores validos de `RARITY` son `common`, `event`, `free_or_veteran`, `veteran`, `paragon`. **`legendary` no existe** | `RARITY = veteran` (usado por 260 rasgos nativos) |
| 5 | `Malformed token: @calito_research_buff` | En `common/traits/` y `common/governments/councilors/` el parser **no resuelve `@script_values`** | Numeros en duro, con nota de que el ajuste se hace en esos ficheros, no en `script_values` |
| 6 | `Wrong scope for effect 'set_starbase_building'` | Estaba dentro de `colony = { }`. Necesita scope de estrella o flota | Movido a `fleet = { starbase? = { ... } }` |
| 7 | `Scripted Trigger military_arkship is invalid` | `limit = { $ARK_TYPE$ = military_arkship }` se expande a `limit = { military_arkship = military_arkship }`, que el parser lee como una **llamada a trigger**, no como comparacion de cadenas. **No se pueden comparar strings en un trigger** | Parametros booleanos (`IS_MILITARY = yes`) con bloques `[[X]] ... ]` anidados |
| 8 | `Invalid building 'arkship_grand_archive' for starbase` | Ese modulo exige `tech_galactic_archivism` en su `potential` | Tech anadida a `calito_give_ark_upgrade_techs` |
| 9 | `Missing modifier localization: calito_kaiser_moon_upkeep_reduction` | Todo modificador necesita una clave loc con su nombre exacto | Anadida en ambos idiomas |
| 10 | `Wrong scope for trigger 'is_ai'` (en `sm_scientist_traits.txt`) | `leader_potential_add` se evalua en scope de PAIS, no de lider | Sustituido por `always = no` |
| 11 | La consola dice "no cumple la condicion" al lanzar `event calitotest.1` | Los eventos de test tenian `trigger = { always = no }`. **La consola de depuracion SI evalua el trigger del evento**, asi que un trigger que nunca se cumple hace que `event <id>` falle siempre. El mensaje de error puede además mencionar flags de otro evento, lo que despista | Fuera el `trigger`. `is_triggered_only = yes` ya impide que se dispare solo |

**Regla general que sale de todo esto:** despues de tocar el mod,
mira `logs/error.log` y filtra por el nombre del mod. Es mucho mas rapido
que adivinar. Y para depurar eventos a mano, **no les pongas trigger**:
eso se comprueba al lanzarlos, no solo al Programar.

### 7.1 Los 4 tokens rotos del calito_spawn original

| # | Bug | Sintoma | Arreglo |
|---|---|---|---|
| 1 | `calito.8005` no lo disparaba nadie | El mod no hacia nada en una partida normal | Anadidos los eventos de consola `calitotest.*` y el evento jugable `calito.100` |
| 2 | `kaiser_moon_upkeep_reduction` no existia | Error en `error.log`, sin efecto | Declarado `calito_kaiser_moon_upkeep_reduction` en `common/static_modifiers/`. Giga lo referencia pero nunca lo define |
| 3 | `NAME_Progenitor` no existia | La flota titan nacia vacia | Sustituido por `NAME_Giga_Tiyanki_Small` + `design = "Ancient Asteroid Artillery"` |
| 4 | `apply_ast_art_tech_upgrades` fallaba en silencio | Usaba `ROOT.from` esperando el pais; aqui no hay ese scope | Comprobacion directa de `has_technology` + `add_modifier` |
| 5 | `random_existing_design = asteroid_artillery` | `asteroid_artillery` es un `ship_size`, no un diseno. Elige entre disenos del jugador | `design = "Ancient Asteroid Artillery"` |
| 6 | `if = { }` sin `limit` dentro de `create_ship` | Bloque vacio: la condicion nunca se evaluaba | Anadido `limit = { has_country_flag = ... }` real |
| 7 | `sm_leader_trait_inquisitive_3` huerfano | Clave nueva que no esta en ninguna lista de seleccion, asi que el motor nunca la elige | **Sigue sin arreglar** (ver abajo) |
| 8 | `descriptor.mod` apuntando a 3.14 | El launcher avisaba de incompatibilidad | Actualizado a `v4.5.*` |

### Pendiente: el rasgo `sm_scientist_traits.txt`

`sm_leader_trait_inquisitive_3` esta declarado pero **nunca saldra en juego**:

- Es una clave nueva (`sm_` prefija "Stellaris Mod"), asi que **no**
  sustituye al `leader_trait_inquisitive_3` de vanilla.
- No esta en ninguna lista de seleccion de rasgos de lider, asi que el
  motor no lo elige al generar científicos.
- Sus valores son absurdos: `all_technology_research_speed = 100`,
  `num_tech_alternatives_add = 20`, y multiplicadores de 10 en arqueologia.

Para arreglarlo hay dos caminos, ninguno trivial:
1. Renombrar la clave a `leader_trait_inquisitive_3` (sobreescribe el
   vanilla) y bajar los valores.
2. Buscar donde vanilla engancha los rasgos de cientifico de nivel 3 y
   anadir la clave ahi.

---

## 8. Convenciones de este mod

- **Prefijo `calito_` en todo**: eventos, efectos, triggers, flags, especies.
- **Nombres de script en `lowercase_snake_case`.** Los ids de eventos
   usan el namespace: `calito.100`, `calitotest.1`.
- **Flags de pais con prefijo `calito_`**: `calito_five_arks_initialised`,
  `calito_five_arks_pending`, `calito_five_arks_asari_offer`,
  `calito_five_arks_asari_done`, `calito_five_arks_aborted`.
   - **Flags de especie**: `calito_asari_arkborn`. Marca la especie que creo este mod.
  creo este mod.
- **Constantes de balance en `common/script_values/`** con prefijo
  `@calito_`. No pongas numeros magicos en los efectos.
- **Tabs para indentar**, como el juego base. Nunca espacios mezclados.
- **Comentarios explicando el *por que*, no el *que*.** Los bugs de este
  mod se evitaron por los comentarios; no los borres.
- **Localiza siempre en los dos idiomas.** Si anades una clave loc, anadela
  a `english/` y a `spanish/`.

---

## 9. Que falta / siguientes pasos

Lo de "colgarlo de un on_action" **ya esta hecho** (ver §2). Queda:

1. **Arreglar el rasgo `sm_leader_trait_inquisitive_3`** (§7). Ahora mismo
   no sale nunca en juego.
2. **Elegir clase de gobierno y etica** para el escenario. Ahora el
   escenario deja el gobierno que tuviera el jugador, lo cual puede dar
   combinaciones absurdas: un imperio mecanico sin marina con 5 arcos
   civilizadas, o una colonia de gestalt que no deberia haber sobrevivido.
   Habria que forzar algo tipo `auth_oligarchic` mas eticas coherentes con
   la supervivencia, o dejarlo como eleccion del jugador en la ventana.
3. **Imagenes propias.** `descriptor.mod` referencia `thumbnail.png` pero
   el fichero no existe. El origen reutiliza el icono nomada de vanilla
   (`origins_default_nomads.dds`).
4. **Repartir los satelites por mas de un sistema.** Ahora los 4 arcos
   satelite aparecen todos en el sistema de la capital, a 60 de distancia.
   Un setup mas interesante usaria `closest_system` con `min_steps` /
   `max_steps` para que queden dispersos por la vecindad.
5. **Los satelites no tienen lider de flota.** `home_arkship_initial_setup`
   de vanilla asigna un commander; `calito_create_ark_effect` no lo hace.
   Si quieres que cada arco tenga su propio comandante, anadir un
   `create_leader` + `set_leader` en el `create_ship` effect.
6. **Decidir que pasa con `calito.8005`.** Esta huerfano a proposito
   (compatibilidad). Si no lo referencia nadie, se puede borrar.

---

## 10. Guia rapida de las piezas del escenario

```
calito_five_arks_setup_effect = { ARK_TYPE = <tipo> }
    Orquestador. Hace, en este orden:
      1. guarda targets (sistema, pais)
      2. set_country_as_nomad_effect + set_origin calito_origin_five_arks
      3. lanza nomads.20 (situacion) + arkship.1 (eventos aleatorios)
      4. technologies de arkship + refresh_auto_generated_ship_designs
      5. crea las dos especies
      6. convierte el planeta inicial en arco (transfer_carrier)
      7. siembra el arco principal
      8. deja la Tierra muerta (pc_nuked, sin gente)
      9. crea 4 arcos satelite
     10. recursos de arranque

calito_create_ark_effect = { ARK_TYPE OWNER LOCATION DISTANCE POPS SPECIES ARK_NAME }
    Un arco nuevo con colonia, modulos, distritos y poblacion.

calito_populate_ark_effect = { POPS SPECIES }
    Distritos + edificios + grupos de poblacion dentro de una colonia pc_ark.

calito_convert_planet_to_ark_effect = { PLANET ARK_TYPE }
    Convierte un planeta en arco in situ (create_colony = no + transfer_carrier).

calito_kill_homeworld_effect = { PLANET }
    pc_nuked + devastation 100 + se lleva a la gente + borra distritos.

   calito_create_asari_species = yes
       UNICA especie que crea el mod: asari (HUM, retrato mec_asari,
       mec_asari_names, trait_survivor). Reutiliza la del jugador si ya
       tiene una especie asari. SIN rasgos del DLC Asari: si el
       inline_script de planet_mec_biotics no resuelve, create_species
       aborta entero y no aparece ni un pop.
       El imperio conserva SU propia especie: el scenario no impone la
       biologia del jugador, solo la de las arcas.
```

Scopes de evento (`events/01_calito_five_arks_events.txt`):
`root` y `owner` son el pais. `capital_scope` es el arco principal una vez
convertido, asi que **guarda el planeta original en un `event_target` ANTES
de convertirlo** (es lo que hace el paso 6 del orquestador).
