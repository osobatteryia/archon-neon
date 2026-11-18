# Archon Neón

Versión propia del clásico **Archon: The Light and the Dark** (Atari, 1983), hecha como una sola página web en estilo *pixel neón*. Dos ejércitos, la **Luz** y la **Oscuridad**, se enfrentan en un tablero estratégico de 9×9. Cuando una pieza entra en la casilla de un rival, ambas pelean en una **arena de combate en tiempo real**.

**Jugar:** https://osobatteryia.github.io/archon-neon/

Este documento es la referencia del proyecto: de qué se trata, cómo se juega, qué reglas y valores usa, cómo está construido por dentro y cómo actualizarlo.

---

## Índice

1. [Resumen del proyecto](#1-resumen-del-proyecto)
2. [Modos de juego](#2-modos-de-juego)
3. [Reglas del tablero](#3-reglas-del-tablero)
4. [Las piezas](#4-las-piezas)
5. [Hechizos y elementales](#5-hechizos-y-elementales)
6. [La arena de combate](#6-la-arena-de-combate)
7. [Controles](#7-controles)
8. [Modo online](#8-modo-online)
9. [Estructura técnica](#9-estructura-técnica)
10. [Equilibrio y simulaciones](#10-equilibrio-y-simulaciones)
11. [Cómo actualizar y publicar](#11-cómo-actualizar-y-publicar)
12. [Normas del proyecto](#12-normas-del-proyecto)
13. [Limitaciones conocidas y pendientes](#13-limitaciones-conocidas-y-pendientes)
14. [Historial de versiones](#14-historial-de-versiones)
15. [Créditos y aviso legal](#15-créditos-y-aviso-legal)

---

## 1. Resumen del proyecto

| | |
|---|---|
| **Qué es** | Juego de estrategia por turnos + combate de acción, inspirado en Archon (1983). |
| **Dueño** | Oso (`osobatteryia`). |
| **Plataforma** | Navegador web: celular (táctil, mejor en horizontal) y PC (teclado). |
| **Tecnología** | Un único archivo `index.html` con HTML, CSS y JavaScript sin frameworks. Dibujo con `<canvas>`. Multijugador online con PeerJS (WebRTC). |
| **Publicación** | GitHub Pages desde la rama `main`, carpeta raíz. |
| **Estilo visual** | Pixel neón: fondo azul noche, **cian** para la Luz, **magenta** para la Oscuridad, **amarillo** para los puntos de poder. Tipografías *Press Start 2P* e *Inter* (Google Fonts). |
| **Idioma** | Español. |

El objetivo fue recrear la sensación del original (estrategia + habilidad con el joystick) con arte, nombres de diseño y valores propios.

---

## 2. Modos de juego

Se eligen en la pantalla de inicio (botón **Reglas** para volver a ella).

| Modo | Descripción |
|---|---|
| **2 jugadores** | Dos personas en el mismo dispositivo. En celular, cada uno tiene su joystick y su botón. |
| **Contra la computadora** | Tú juegas con la Luz; la computadora con la Oscuridad (tablero y arena). |
| **Ver simulación** | La computadora juega por ambos bandos. Velocidad x1, x2 o x4. Sirve para ver una partida y probar el equilibrio. |
| **Jugar online** | Dos personas en dispositivos distintos, conectadas con un código de sala (ver [§8](#8-modo-online)). |

Además hay dos **modos de combate** (también en la pantalla de inicio; se recuerda la elección en el navegador):

| Modo de combate | Movimiento en la arena | ¿Moverse mientras ataca? |
|---|---|---|
| **Clásico** (por defecto, fiel al original) | Todas las criaturas a velocidad normal (165 px/s); **Gólem, Trol y elemental de Tierra lentos** (105 px/s). | **No.** Mientras se mantiene el ataque presionado la pieza queda quieta y la dirección solo apunta: *dispara y escapa*. |
| **Dinámico** | Cada criatura tiene su propia velocidad: `60 + vel_arena × 22` px/s. | Sí. |

---

## 3. Reglas del tablero

### Disposición inicial
- Tablero de **9×9**. La Luz empieza en las columnas izquierdas (0 y 1); la Oscuridad en las derechas (7 y 8).
- **Columna 0 (Luz), de arriba abajo:** Valquiria, Gólem, Unicornio, Genio, **Mago**, Fénix, Unicornio, Gólem, Valquiria.
- **Columna 1 (Luz):** Arquera, 7 Caballeros, Arquera.
- **Columna 8 (Oscuridad):** Banshee, Trol, Basilisco, Cambiaformas, **Hechicera**, Dragón, Basilisco, Trol, Banshee.
- **Columna 7 (Oscuridad):** Mantícora, 7 Goblins, Mantícora.
- Cada ejército tiene **18 unidades** de **8 tipos**. Los líderes empiezan sobre un punto de poder.
- **Empieza la Luz.**

### Turnos y movimiento
- En cada turno el jugador **mueve una pieza** o **lanza un hechizo** (solo el líder).
- **Tierra:** avanza en línea recta (sin diagonales) hasta su alcance, sin pasar por encima de otras piezas.
- **Vuelo:** puede llegar a cualquier casilla dentro de su alcance (distancia en cualquier dirección, incluida la diagonal), pasando por encima de otras piezas.
- **Teletransporte (Mago y Hechicera):** igual que el vuelo.
- Si el destino tiene una pieza enemiga, **se combate en la arena**. Si gana quien ataca, ocupa la casilla; si gana quien defiende, se queda y el atacante muere.

### Ciclo de luz y color de las casillas
- Hay **6 tonos**, de 0 (más oscuro) a 5 (más claro).
- **Casillas fijas:** en la mitad izquierda son claras (tono 5); en la derecha, oscuras (tono 0).
- **Casillas variables** (marcadas con un cuadradito en la esquina): la columna central, una de cada dos casillas en forma de damero y tres puntos de poder. Su tono avanza **un paso por turno** y rebota al llegar a 0 o 5.
- El punto de poder de la izquierda es siempre claro y el de la derecha siempre oscuro.
- **Efecto en combate:** el color de la casilla donde se pelea da vida extra o quita vida:
  `bonus = redondear((tono − 2,5) / 2,5 × 3)` → de **+3** a **−3**. Es positivo para la Luz en casillas claras y para la Oscuridad en casillas oscuras.

### Puntos de poder
- **5 casillas** marcadas con un rombo amarillo: centro (4,4), centro de cada borde (4,0), (4,8), y los de cada lado (0,4) y (8,4).
- Curan **por completo** a la pieza que está encima al inicio de su turno.
- Las piezas sobre un punto de poder son **inmunes a los hechizos**.

### Curación
- Al inicio de su turno, cada pieza herida recupera **1 de vida** (o toda si está en un punto de poder).
- La vida perdida en un combate se conserva en el tablero (barra bajo la pieza).

### Victoria
Gana el bando que:
1. **Ocupa los 5 puntos de poder**, o
2. **Elimina todas las piezas rivales**, o
3. Deja al rival **sin movimientos posibles** (todas sus piezas encarceladas o bloqueadas y sin hechizos).

Si no queda ninguna pieza en el tablero, es **empate**.

---

## 4. Las piezas

Todas las piezas son sprites **originales** de 12×12 píxeles ampliados a 24×24 con suavizado (*Scale2x*) y sombreado en 4 tonos (sombra, base, brillo, reflejo). El Dragón tiene un sprite propio de 24×24 dibujado a mano.

### Valores actuales (versión 1.4)

Columnas: **Vida** (en el tablero; en la arena se multiplica ×3), **Daño** por golpe, **Recarga** en segundos, **Proy.** = velocidad del proyectil (1–10; en px/s = 150 + valor × 45), **Arena** = velocidad de desplazamiento en modo Dinámico (1–10), **Mov.** = tipo y alcance en el tablero.

#### Luz (cian)

| Pieza | Unid. | Vida | Daño | Recarga | Proy. | Arena | Mov. | Ataque |
|---|---|---|---|---|---|---|---|---|
| Mago | 1 | 10 | 10 | 1,6 | 5 | 5 | Teletransporte 3 | Bola de fuego. Líder: 7 hechizos. |
| Valquiria | 2 | 8 | 7 | 1,4 | 4 | 7 | Vuelo 3 | Lanza mágica |
| Unicornio | 2 | 9 | 7 | 0,9 | 9 | 8 | Tierra 4 | Rayo de energía |
| Genio | 1 | 15 | 6 | 1,6 | 5 | 6 | Vuelo 4 | Torbellino |
| Arquera | 2 | 6 | 6 | 1,2 | 6 | 6 | Tierra 3 | Flecha |
| Fénix | 1 | 12 | 10 | 3,0 | — | 6 | Vuelo 5 | Explosión en área (radio 120). Invulnerable 0,7 s al explotar. |
| Gólem | 2 | 15 | 10 | 2,4 | 3 | 3 | Tierra 3 | Roca. Lento. |
| Caballero | 7 | 5 | 5 | 0,6 | — | 7 | Tierra 3 | Espada, cuerpo a cuerpo |

#### Oscuridad (magenta)

| Pieza | Unid. | Vida | Daño | Recarga | Proy. | Arena | Mov. | Ataque |
|---|---|---|---|---|---|---|---|---|
| Hechicera | 1 | 10 | 7 | 1,1 | 9 | 5 | Teletransporte 3 | Rayo eléctrico. Líder: 7 hechizos. |
| Banshee | 2 | 8 | 9 | 2,5 | — | 7 | Vuelo 3 | Grito en área (radio 105) |
| Basilisco | 2 | 6 | 9 | 1,0 | 9 | 6 | Tierra 3 | Rayo ocular |
| Dragón | 1 | 17 | 9 | 2,6 | 6 (9 en Clásico) | 5 | Vuelo 4 | Llamarada |
| Mantícora | 2 | 8 | 6 | 1,4 | 6 | 6 | Tierra 3 | Púas de la cola |
| Cambiaformas | 1 | 10 | — | — | — | — | Vuelo 5 | Se transforma en el rival |
| Trol | 2 | 15 | 10 | 2,4 | 3 | 3 | Tierra 3 | Roca. Lento. |
| Goblin | 7 | 5 | 5 | 0,6 | — | 7 | Tierra 3 | Maza, cuerpo a cuerpo |

#### Parejas equivalentes
Mago ↔ Hechicera · Unicornio ↔ Basilisco · Gólem ↔ Trol · Fénix ↔ Cambiaformas · Genio ↔ Dragón · Valquiria ↔ Banshee · Arquera ↔ Mantícora · Caballero ↔ Goblin.

### El Cambiaformas
- En el tablero se ve con su propio sprite.
- Al entrar a la arena hace una animación de transformación y **toma la forma del rival**: su sprite (pintado en magenta), su ataque, su velocidad y su vida inicial, escalada según la vida que le quede al Cambiaformas (sobre 10).
- Al terminar el combate, si sobrevive, vuelve a su forma.

---

## 5. Hechizos y elementales

Solo el **Mago** y la **Hechicera** pueden lanzarlos. Cada hechizo se usa **una vez por partida**, gasta el turno y **no afecta a piezas sobre puntos de poder**. No se pueden lanzar si el líder está muerto o encarcelado.

| Hechizo | Efecto |
|---|---|
| **Teletransporte** | Lleva una pieza aliada a cualquier casilla. Si cae sobre un enemigo, combaten. |
| **Curar** | Devuelve toda la vida a una pieza aliada herida. |
| **Alterar el tiempo** | Invierte el sentido del ciclo de luz. |
| **Intercambiar** | Intercambia la posición de dos piezas cualesquiera (propias o enemigas). |
| **Invocar elemental** | Aparece un elemental al azar que combate contra una pieza enemiga; lo controla quien lo invocó. Desaparece al terminar el combate. |
| **Revivir** | Devuelve al tablero una pieza aliada eliminada, en una casilla junto al lanzador. |
| **Encarcelar** | Una pieza enemiga no puede moverse hasta que el ciclo llegue al extremo de su color (tono 5 para la Luz, 0 para la Oscuridad). |

### Elementales

| Elemental | Vida | Daño | Recarga | Proy. | Arena | Nota |
|---|---|---|---|---|---|---|
| Aire | 12 | 5 | 1,0 | 8 | 8 | Rápido |
| Tierra | 17 | 9 | 2,4 | 3 | 3 | Lento |
| Fuego | 10 | 9 | 1,4 | 6 | 6 | |
| Agua | 14 | 6 | 1,6 | 5 | 6 | |

Los elementales no reciben bonus por el color de la casilla.

---

## 6. La arena de combate

- Campo de 800×480 unidades lógicas, escalado a la pantalla. El fondo toma el **color de la casilla** donde se pelea.
- **La Luz aparece a la izquierda y la Oscuridad a la derecha**, sin importar quién atacó.
- Se generan al azar hasta **5 pilares** en la zona central. Bloquean el paso y los proyectiles.
- Cuenta regresiva de 1,2 s ("¡LISTOS!" / "¡YA!") antes de empezar.
- **Vida en la arena** = (vida en el tablero + bonus de la casilla) × 3.
- **Tipos de ataque:**
  - *Proyectil:* sale en la dirección en que mira la pieza; se detiene al chocar con un pilar, el borde o el rival.
  - *Cuerpo a cuerpo* (Caballero, Goblin): golpe inmediato a corta distancia frente a la pieza.
  - *Área* (Fénix, Banshee): onda circular que crece hasta su radio y daña una vez al rival si lo alcanza.
- Termina cuando una pieza llega a 0. Si ambas caen, mueren las dos.
- En "Ver simulación" un combate se corta a los 75 s; en ese caso nadie muere.

---

## 7. Controles

### PC (teclado)
| | Moverse / apuntar | Atacar |
|---|---|---|
| Luz | W A S D | Espacio (o F) |
| Oscuridad | Flechas | Enter |

En modo online cada jugador puede usar cualquiera de los dos juegos de teclas.

### Celular (táctil)
- **Recomendado: teléfono horizontal.**
- **Contra la computadora u online:** joystick con el **pulgar izquierdo** (esquina inferior izquierda) y botón **FUEGO** con el **pulgar derecho** (esquina inferior derecha). La arena se achica para dejar libres los costados.
- **2 jugadores en el mismo teléfono:** cada uno tiene su joystick y su botón; la Luz a la izquierda y la Oscuridad a la derecha.
- En el tablero se toca la pieza y luego el destino.
- El juego **bloquea el zoom** por doble toque y pellizco, el desplazamiento de la página y el menú de mantener presionado (necesario en Safari de iPhone, que ignora `user-scalable=no`).

---

## 8. Modo online

### Cómo se usa
1. Un jugador toca **Jugar online → Crear sala**. Recibe un **código de 5 caracteres** y un **enlace** (`…/archon-neon/?sala=CÓDIGO`). Ese jugador es la **Luz**.
2. El otro abre el enlace o escribe el código y toca **Unirme**. Juega con la **Oscuridad**.
3. Solo quien creó la sala puede empezar una partida nueva.

### Cómo funciona por dentro
- **Conexión directa entre navegadores (WebRTC)** usando **PeerJS 1.5.4**, incluido dentro de `index.html` (no se carga desde una CDN).
- El servidor público gratuito de PeerJS (`0.peerjs.com`) solo pone en contacto a los dos navegadores; el juego no pasa por ningún servidor propio. El identificador de la sala es `archonneon-CÓDIGO`.
- **El navegador del anfitrión (Luz) es el árbitro:**
  - Ejecuta toda la lógica del tablero y simula la arena.
  - Después de cada cambio en el tablero envía el estado completo (`state`).
  - Durante el combate envía el estado de la arena cada 2 cuadros, unas 30 veces por segundo (`arena`), y avisa al terminar (`arenaEnd`).
- **El invitado (Oscuridad)** no calcula nada:
  - Envía sus acciones del tablero (`act`: tocar casilla, hechizo, cancelar).
  - En la arena envía sus controles cada 33 ms (`in`).
  - Dibuja lo que recibe.
- Si se corta la conexión aparece un aviso y el juego vuelve al modo local.

### Requisitos y limitaciones
- **No funciona dentro de Claude** (la vista previa bloquea WebRTC): hay que usar la versión de GitHub Pages.
- No hay servidor TURN. En algunas redes (ciertas compañías móviles o redes de empresa) la conexión directa puede fallar. La solución sería agregar un servidor TURN a la configuración de PeerJS.
- Si el anfitrión cambia de aplicación durante un combate, el navegador pausa la arena hasta que vuelva.

---

## 9. Estructura técnica

### Archivos del repositorio
| Archivo | Contenido |
|---|---|
| `index.html` | **Todo el juego**: estilos, la librería PeerJS (minificada, primer `<script>`), los datos de los sprites y el código del juego (segundo `<script>`). |
| `README.md` | Este documento. |
| `apple-touch-icon.png` | Ícono de 180×180 para la pantalla de inicio del iPhone. |
| `icon-192.png`, `icon-512.png`, `favicon.png` | Ícono en otros tamaños (Android y pestaña del navegador). |
| `manifest.webmanifest` | Datos de la app web (nombre, colores, íconos) para instalarla en el inicio. |

No hay proceso de compilación ni dependencias que instalar: se edita `index.html` y se sube.

### Ícono y app en el inicio
- El ícono es la opción **"Tablero"**: un mini tablero de 3×3 con un Caballero (Luz), un Goblin (Oscuridad) y el punto de poder amarillo al centro, sobre fondo azul noche. iOS redondea las esquinas solo.
- Para anclarlo en iPhone: en Safari, **Compartir → Agregar a inicio**. Se abre a pantalla completa, sin las barras de Safari, con el nombre "Archon Neón".
- Si se cambia el ícono, hay que regenerar los cuatro PNG con el mismo diseño. iOS guarda el ícono al anclarlo, así que para ver uno nuevo hay que quitar el acceso y volver a agregarlo.

### Organización del código (segundo `<script>` de `index.html`)
Está envuelto en un bloque `{ … }` y dividido en secciones con comentarios `/* ---------- … ---------- */`:

| Sección | Qué hace |
|---|---|
| Constantes | `DATA` (mapas de píxeles), `HP_MULT = 3`, `BONUS = 3`, `MODE`, `U` (estadísticas de las 16 piezas), `ELEM` (elementales), `PAL` (paletas de color), `SPELLS`, `PP` (puntos de poder). |
| `sprites` | `grid12`, `scale2x`, `shade` y `sprite(nombre, paleta)`: generan y guardan en caché cada sprite como un canvas de 24×24. |
| `game state` | `newGame`, `toneAt`, `bonus`, `movesFor` (movimientos válidos). |
| `spells` | `spellTargets`, `spellOk`, `canCast`. |
| `turn flow` | `checkWin`, `endTurn` (ciclo de luz, liberar presos, curación), `battle`, `clickCell`, `castStep`, `startSpell`, `cancel`. |
| `online` | `act`/`apply` (punto de entrada de toda acción del jugador), `snap`/`loadSnap` (estado del tablero), `arenaSnap`/`guestArena`, `wire`, `lobby`, `host`, `join`, `placeTouch`. |
| `computer player` | `power` (estimación de fuerza), `aiTurn` (elige la jugada por puntaje: ataques con buenas probabilidades, puntos de poder, acercarse, evitar amenazas, algunos hechizos), `aiInput` (combate: esquiva proyectiles, mantiene distancia, busca línea de tiro), `simulate(n)` (partida completa instantánea, para pruebas). |
| `board rendering` | `layout`, `drawBoard`, `render` (también el panel lateral). |
| `overlays` | Ventanas: hechizos, reglas/inicio, victoria, nueva partida; bloqueo de zoom táctil. |
| `arena` | Teclado, joystick táctil, `combat`, `rawInput`/`input`, `attack`, `step` (física y colisiones), `draw`, `loop`/`tick`, `finish`. |

### Cómo cambiar cosas habituales
- **Equilibrio:** editar los valores en `const U = {…}` (piezas) y `const ELEM = {…}` (elementales). `ar` es el radio de los ataques en área, `pr` el tamaño del proyectil, `psC` la velocidad de proyectil solo en modo Clásico, `slow:1` marca a una pieza como lenta en Clásico.
- **Bonus por color de casilla:** constante `BONUS`.
- **Multiplicador de vida en la arena:** `HP_MULT`.
- **Velocidades del modo Clásico:** en `step()` (`105` lenta, `165` normal).
- **Colores:** `PAL` (piezas) y variables CSS `--L`, `--K`, `--Y`.
- **Sprites:** `DATA.S` contiene un mapa de 12×12 por pieza (`#` = cuerpo, `+` = detalle, `.` = vacío). `DATA.DRAGON` es el mapa de 24×24 del Dragón (`D`, `M`, `B`, `H` = tonos; `Y`, `O` = fuego).
- **Versión visible:** texto "Versión X.Y" en la función `help()`. Conviene subirla en cada cambio para comprobar que el navegador cargó la versión nueva.

### Pruebas
Las pruebas se hicieron con Playwright y Chrome sin pantalla: movimientos, combates, hechizos, controles táctiles en pantalla de teléfono vertical y horizontal, y el modo online con dos navegadores conectados por una conexión simulada. Dentro de la página existe `window.__archon` con funciones de apoyo para pruebas (por ejemplo `__archon.simulate(400)` juega una partida completa instantánea y devuelve el resultado).

---

## 10. Equilibrio y simulaciones

Los valores se basan en las descripciones del manual original (qué tan rápido es cada ataque, cuánto daño hace, cuánta resistencia tiene), traducidas a números propios y luego ajustadas con **partidas simuladas** (computadora contra computadora, modo Clásico, 40 partidas por ronda).

| Ronda | Cambios | Victorias | Observaciones |
|---|---|---|---|
| 1 | Valores iniciales | Oscuridad 24 · Luz 16 | Dragón (72 %) y Hechicera (73 %) muy fuertes; Mantícora (18 %), Fénix (28 %), Arquera (30 %) y Banshee (35 %) débiles. |
| 2 | Dragón daño 11→9 · Fénix daño 9→10, radio 95→120 · Banshee daño 8→9, radio 80→105 · Mantícora daño 4→6, proyectil 4→6 | Luz 23 · Oscuridad 16 · 1 empate | Fénix 51 %, Banshee 52 %, Mantícora 34 %, Dragón 62 %. |
| 3 | Arquera vida 5→6, daño 5→6 · Hechicera daño 8→7 | Oscuridad 21 · Luz 18 · 1 empate | Hechicera 68 %, Arquera 28 %, Cambiaformas 29 %. |

**Datos de una partida típica:** unos 93 turnos y 35 combates.

**Cuidado al leer estos números:** la computadora juega de forma básica y 40 partidas tienen bastante azar (el Gólem y el Trol, con valores idénticos, intercambiaron resultados entre rondas). Sirven para detectar desequilibrios grandes; el ajuste fino debe venir de partidas entre personas.

La planilla de equilibrio original ("Tabla de equilibrio – Archon") se armó en Claude como referencia y puede quedar desactualizada: **los valores que manda son los de `index.html`** y la tabla de la [§4](#4-las-piezas).

---

## 11. Cómo actualizar y publicar

1. Editar `index.html` (en GitHub, en un editor o pidiéndoselo a Claude).
2. Subir el cambio a la rama `main`.
3. **GitHub Pages se actualiza solo** en 1 a 5 minutos.
4. Para ver la versión nueva en el celular, cerrar la pestaña y volver a abrir el enlace (Safari guarda copias), y comprobar el número de versión en la pantalla de inicio.

**Configuración de Pages** (ya hecha): *Settings → Pages → Deploy from a branch → `main` / `(root)`*. El repositorio debe ser **público** para que Pages funcione con una cuenta gratuita.

---

## 12. Normas del proyecto

- **Arte y nombres propios.** No se copian sprites, imágenes ni textos del juego original ni de Electronic Arts. Las mecánicas y estadísticas inspiradas en el original sí se usan, porque no tienen derechos de autor.
- **Un solo archivo.** El juego debe seguir funcionando abriendo solo `index.html`, sin compilación ni servidores propios. Las librerías van incluidas dentro del archivo, no desde CDN (salvo las fuentes de Google).
- **Celular primero.** Todo cambio debe funcionar con el teléfono en horizontal, con dos pulgares, sin zoom accidental. Los botones deben tener al menos 44 px.
- **Modo Clásico como referencia.** Los cambios de equilibrio se evalúan primero en modo Clásico.
- **Cambios de equilibrio documentados.** Al modificar valores en `U` o `ELEM`, actualizar la tabla de la [§4](#4-las-piezas) y agregar una fila en la [§10](#10-equilibrio-y-simulaciones).
- **Subir la versión.** Cada cambio publicado incrementa el número de "Versión X.Y" de la pantalla de inicio y suma una línea al [historial](#14-historial-de-versiones).
- **El anfitrión manda.** En el modo online, toda la lógica corre en el navegador de la Luz. Cualquier regla nueva debe ejecutarse solo ahí y llegar al invitado mediante el estado sincronizado.
- **Commits en español**, con una descripción breve de lo que cambió.

---

## 13. Limitaciones conocidas y pendientes

- **Sin sonido** ni música.
- **Computadora básica:** su estrategia es simple; sirve para practicar y simular.
- **Online sin servidor TURN:** puede no conectar en algunas redes.
- **No se puede elegir bando en online:** quien crea la sala siempre es la Luz.
- **Arena con poca latencia tolerada:** el invitado ve el combate con el retraso de su conexión.
- **Equilibrio pendiente:** Arquera y Cambiaformas siguen bajos en las simulaciones; la Hechicera, alta.
- **Ideas a futuro:** sonidos y música, animaciones de movimiento en el tablero, elegir bando y revancha en online, mejor inteligencia de la computadora, servidor TURN, guardar la partida.

---

## 14. Historial de versiones

| Versión | Cambios |
|---|---|
| 1.0 | Juego completo en un dispositivo: tablero, ciclo de luz, puntos de poder, arena, 7 hechizos, Cambiaformas, elementales, controles de teclado y táctiles. |
| 1.1 | Modos de combate Clásico y Dinámico. Computadora (contra la computadora y simulación). Ajustes de equilibrio en 2 rondas. |
| 1.2 | Modo online con PeerJS (salas con código y enlace). Publicación en GitHub Pages. Controles táctiles a dos manos en horizontal (joystick izquierda, FUEGO derecha). |
| 1.3 | Bloqueo del zoom por doble toque y pellizco en iPhone. Número de versión visible en la pantalla de inicio. |
| 1.4 | Ícono "Tablero" para la pantalla de inicio del iPhone y manifiesto de app web (se abre a pantalla completa al anclarlo). |

---

## 15. Créditos y aviso legal

- Inspirado en **Archon: The Light and the Dark** (1983), creado por Free Fall Associates (Jon Freeman, Anne Westfall y Paul Reiche III) y publicado por Electronic Arts. Este proyecto es un homenaje no oficial y sin fines comerciales; no está afiliado a ellos.
- Sprites, interfaz y código: originales de este proyecto, creados por Oso con ayuda de Claude (Anthropic).
- [PeerJS](https://peerjs.com/) (licencia MIT), incluido dentro de `index.html`.
- Fuentes [Press Start 2P](https://fonts.google.com/specimen/Press+Start+2P) e [Inter](https://fonts.google.com/specimen/Inter) (SIL Open Font License), vía Google Fonts.
