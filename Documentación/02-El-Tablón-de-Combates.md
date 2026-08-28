# El tablón de combates

Qué se ve en la pared del pabellón, cómo se reparte, cada cuánto se refresca y con qué teclas se
maneja. Es la única pantalla de trabajo de la aplicación, y prácticamente toda ella.

---

## 1. El recorrido, entero

```
  inicio de sesión ──▶ TABLÓN
        │
        ├──▶ configuración de este equipo   (rueda dentada)
        └──▶ actualización                  (rueda dentada)
```

Y ya está. No hay secciones que navegar, ni menús, ni una segunda pantalla de trabajo: se entra, sale
el tablón y ahí se queda el día entero.

Las dos pantallas de la rueda dentada cuelgan del **inicio de sesión** y no del tablón, y las dos por
el mismo motivo de fondo: son las que hacen falta cuando la aplicación **no** consigue entrar. Una
pantalla de configuración a la que solo se llega con sesión iniciada no sirve para nada el día que
hace falta. Y la de actualización, además, cierra la aplicación al aplicar: ofrecerla desde el tablón
sería poner un botón que apaga la pared del pabellón en mitad de la competición.

---

## 2. Qué sale y qué no

Un combate está en el tablón si **no tiene ganador** y **no está marcado como sin disputa**.

Lo segundo no es un detalle. Una casilla que se quedó vacía —porque se dio de baja o se descalificó al
único competidor que la ocupaba— no tiene ganador, pero tampoco se va a disputar nunca (es
`sin_disputa`, ver `fn_combates_sin_disputa_evento` en JudoAdministración). Sin esa condición, ese
hueco muerto se quedaría eternamente en la primera fila de su tatami mandando a la sala a esperar a
alguien que no va a subir.

**Sí salen** los combates cuyo rival todavía depende de un resultado anterior: la final que espera sus
semifinales, el bronce que espera su repesca. Están en el orden del tatami, tienen su número, y la
sala tiene que poder verlos venir. Se pintan **apagados**, con «— por determinar —» en el hueco que
falta.

Y no salen, porque el servidor no los devuelve:

- los combates que **libran** (un solo contendiente, ganador ya marcado);
- los **bloques diferidos** —finales y bronces de los cuadros grandes cuando el evento los tiene
  desactivados— mientras no se hayan generado desde «Combates → Orden combates»;
- los combates **sin tatami asignado**: si el orden no se ha establecido todavía, no hay fila en la
  que ponerlos, y el tablón lo dice con un cartel en lugar de enseñar una pantalla vacía.

---

## 3. Cómo está montada la pantalla

### 3.1 Los tatamis van en filas, no en columnas

Cada tatami ocupa una **banda horizontal** con sus cuatro próximos combates uno debajo de otro. En
columnas caben más tatamis a la vez, pero entonces el nombre de un competidor se queda en tres letras
y unos puntos suspensivos, y la pantalla deja de servir para lo único que se puso: que alguien
reconozca su nombre desde el otro lado de la sala.

### 3.2 Cuatro filas por tatami, siempre cuatro

Un tatami con dos combates pendientes deja **dos huecos vacíos** en vez de encogerse, y los tatamis de
debajo no se mueven de sitio. En un tablón que se lee de lejos, que las cosas no cambien de posición
vale más que aprovechar el espacio: la gente aprende dónde está su tatami y mira ahí.

Por lo mismo, un tatami que ha terminado **no desaparece**: se queda con su rótulo y un «Sin combates
pendientes». Que un tatami haya acabado es información, y quitarlo movería a todos los de debajo justo
cuando ya se ha aprendido dónde mirar.

### 3.3 Cuánto le queda a cada tatami

En la cabecera de cada tatami, a la derecha, van **los pendientes y al lado el tiempo estimado**:

```
 Tatami 1                                          12 pendientes  ·  ≈ 48 min
```

Son la misma respuesta partida en dos, y por eso van pegados. «Doce pendientes» no dice nada por sí
solo: doce combates de alevín a 2:00 son media hora, y doce de sénior a 4:00 con medio minuto de
cambio entre uno y otro son casi una hora. Es la diferencia entre irse a comer o quedarse, y es lo que
la gente pregunta en la mesa.

**Es exactamente la misma cuenta que hace JudoAdministración** al repartir los pesos por tatami
(`PesoDistribucion.TiempoTotal`) y al proponer la hora de las medallas del cartel de horarios
(`CompositorAgendaTatami`). Tiene que serlo: ese cartel está colgado en la pared del pabellón al lado
de esta pantalla, y si uno dijera una hora y la otra otra, la que parecería rota sería ésta. Lo único
que cambia es **sobre qué** se hace: allí, sobre todos los combates del peso, para repartir los
tatamis antes de empezar; aquí, sobre los que **quedan**, que es lo que importa a media mañana.

Tres cosas entran en la suma, y las tres hacen falta:

| | |
|---|---|
| **El tiempo de combate de cada peso** | Lo pone su **categoría**, no el evento: en el mismo tatami se mezclan alevines de 2:00 con cadetes de 4:00. Sale de `pesos.tiempo`, que el servidor da en `GET /sorteo/distribucion` |
| **Las repescas a Golden Score** | Con `eventos.repescas_gs` puesto, una repesca se disputa entera a Golden Score y dura menos. Se estima con el tiempo de Golden Score **del evento** si lo tiene, si no con el **del peso**, y si el peso lo tiene sin fin, con **la mitad** del tiempo normal |
| **El tiempo extra entre combates** | `eventos.tiempo_extra_segundos`, **una vez por combate**: que suban los judokas, el saludo, la anotación del árbitro. No sale de ningún cronómetro, pero a treinta segundos por combate son veinte minutos en un tatami con cuarenta pendientes |

> **Ojo con la notación de los tiempos.** En la base de datos son `DECIMAL(4,2)` en **M,SS**: `2,30`
> son dos minutos y **medio**, no 2,3. Multiplicado tal cual, un peso de 2,30 × 10 combates da 23
> minutos donde son 25, y con cincuenta pendientes el tablón se equivoca en un cuarto de hora. La
> conversión vive en un solo sitio,
> [`TiempoCombate.cs`](../JudoCombates.Shared/Models/TiempoCombate.cs), que es una copia deliberada
> de su gemela en JudoAdministración: **si se toca una, se toca la otra.**

El rótulo lleva siempre delante un **«≈»** y nunca una hora concreta. Un combate puede acabar en
cuatro segundos por ippon o llegar al Golden Score; lo que esto calcula es el tope de lo que ocupan
los que quedan, igual que el cartel de horarios, y en la práctica sale largo. Decir «termina a las
13:42» sería prometer algo que ningún tatami puede cumplir.

Y **desaparece** cuando no se sabe —el servidor no llegó a dar los tiempos de los pesos— en vez de
enseñar un cero. Cero es «este tatami ha terminado» y no saberlo es otra cosa; enseñar «≈ 0 min» sobre
un tatami con treinta pendientes sería mentir en la pared. Los combates salen igual: la estimación es
un dato de apoyo y no puede dejar sin ellos a la sala.

### 3.4 Una fila

```
 ▌ 42 │ 🏳 ALC  PÉREZ GARCÍA, Luis        │  M -66 kg (12)  │  LÓPEZ SANZ, Juan  VCJ 🏳 │
 ↑  ↑      ↑           ↑                       ↑       ↑              ↑          ↑   ↑
 │  │      │           │                       │       │              │          │   │
 │  │      │           └ nombre y apellidos    │       │              │          │   └ bandera
 │  │      └ código del club/comunidad/país    │       └ nº del combate dentro de su peso
 │  └ nº del combate en la cola del tatami     └ peso del combate
 └ el que toca ahora
```

- **A la izquierda, el número del combate en la cola de su tatami**: el de la hoja de papel de la
  mesa. Se cuenta sobre la cola **completa**, disputados incluidos, porque es el número por el que se
  llama al combate; si el tatami va por el 42, el primer pendiente pone 42 y no 1.
- **Competidor 1, con fondo blanco**, a la izquierda: bandera, código y nombre, de fuera hacia dentro.
- **En medio, el peso** y a su derecha, entre paréntesis, **el número del combate dentro de ese
  peso**: es con lo que se localiza en el cuadro colgado en la pared. Los dos van en la **misma
  línea** porque son un solo dato —«el combate 12 de los -66»—; partido en dos renglones se leía como
  dos cosas distintas, y de paso se iba el alto que ahora aprovechan los nombres.
- **Competidor 2, con fondo azul**, a la derecha: en espejo —nombre, código y bandera—, con la bandera
  pegada al borde derecho.
- **La barra verde** de la izquierda marca el primer pendiente del tatami: el que se está disputando o
  el que sube en seguida. Es la referencia desde la que se cuenta todo lo demás.

**Al poner el ratón encima sale un emergente** con la ronda del combate, su número dentro del peso y
los dos contendientes con su código delante. Es el mismo que la línea de combate de «Combates > Orden
combates» en JudoAdministración, y hace falta por lo mismo que allí: la fila dice el peso y los
nombres, pero no la **ronda**, y «-66 kg (12)» no distingue unos cuartos de una repesca. No es
información para la sala —desde diez metros no se pone nadie el ratón encima— sino para quien se
acerca: el entrenador, la mesa, el que sube a comprobar la pantalla.

```
M -44 kg: Cuartos (3)
(GAL) TORMO PEIRÓ, Natalia
(MAD) SANCHÍS QUILES, Eva
```

Y **nada más**: ni el tatami ni el número de la cola, que llevaba al principio. Los dos ya están
delante de quien lo está leyendo —el tatami es la banda en la que está la fila, con su rótulo arriba,
y el número de la cola es la cifra grande del principio de la propia fila—, así que repetirlos solo
alargaba el cartelito.

Los dos fondos son **los mismos degradados del marcador de JudoCrono**, y eso no es un guiño: quien
mira el tablón y quien mira el marcador del tatami tienen que ver el mismo blanco y el mismo azul,
porque es lo que dice qué cinturón le toca a cada uno.

El código —«ALC», «CVA», «ESP»— es el del **club, la comunidad o el país** según el ámbito del evento,
exactamente el mismo que enseña el marcador del tatami. Lo resuelve el servidor
(`fn_agrupaciones_evento`), no la pantalla.

### 3.5 La cabecera

Delgada a propósito: cada píxel que se lleve es un píxel menos de alto para las cuarenta filas de
debajo. Lleva el nombre de la competición, qué tatamis se están viendo («Tatamis 1 – 5»), el indicador
de página cuando hay dos, la hora y dos botones pequeños de mantenimiento: **cerrar sesión** y
**cerrar la aplicación**.

**La hora no es un adorno.** Es lo que dice de un vistazo que la pantalla está viva: una imagen
congelada por un cuelgue y un tablón sin cambios porque no ha terminado ningún combate se ven
exactamente igual, y el reloj los distingue.

Eran tres botones. **El de recargar se ha ido**: el tablón se relee solo en cuanto el servidor avisa
y, por si acaso, cada treinta segundos, así que era un botón que hacía lo que la pantalla ya hacía
sola. Quien de verdad quiera forzar la lectura tiene F5. Y **el que abría la configuración ahora es de
cerrar sesión**, con el pictograma de la puerta y no con la rueda dentada, igual que en el marcador
del crono: es lo que hace de verdad —volver al inicio de sesión—, y desde ahí se llega a la
configuración, que es un paso más y no el sitio al que se va.

En la esquina de arriba a la derecha, por encima de todo, va **el punto de la conexión**: verde
conectado, rojo sin conexión. Es la única forma de saber si lo que hay en la pared sigue al día,
porque un orden de hace diez minutos se ve igual que uno recién leído. La cabecera del tablón le
**reserva su hueco** con un relleno derecho más ancho que el izquierdo: el punto lo pinta la ventana y
no la pantalla, y sin ese hueco caía justo encima del botón de cerrar la aplicación.

### 3.6 El tamaño de la letra no se escribe: se calcula

Con diez tatamis van cinco por página y hay que apretar; con tres, van los tres solos y el mismo
tamaño dejaría media pantalla vacía. Así que todos los tamaños de la fila salen de **un solo número**
—el alto de letra del nombre—, y ese número depende de cuántos tatamis comparten la pantalla
([`MedidasTablon.cs`](../ViewModels/Tablon/MedidasTablon.cs)):

| Tatamis en la página | Letra del nombre |
|---|---|
| 1 | 44 |
| 2 | 36 |
| 3 | 29 |
| 4 | 24 |
| 5 | 21 |

Los demás tamaños —el código, el peso, el número, la bandera, los anchos de las columnas fijas— salen
de ése en proporción. Hay un solo sitio que ajustar si algún día un pabellón pide más grande, y las
proporciones no se descuadran.

Los valores están calculados para **1080 px de alto**, que es lo que tiene el televisor de un pabellón
normal.

#### Y encima, los botones − y +

El tamaño automático es el que llena la pantalla **a lo alto**, y ésa no es la única dimensión que
importa. Con tres tatamis por página la letra sube a 29 px y entonces lo que se recorta son los
nombres **largos**: «CAÑIZARES CARBONELL, Alejandro» no cabe a lo ancho por mucho alto que sobre. El
alto y el ancho se pelean, y cuál de los dos manda depende de la competición: un nacional trae
apellidos compuestos donde un autonómico no.

De ahí los dos botones de la cabecera, que aplican una **reducción** sobre el tamaño automático:

| | |
|---|---|
| **Máximo** | El automático de esa página. No se sube por encima, y no es capricho: los tamaños de la tabla están calculados para que las cuatro filas de cada tatami quepan en su banda, y una letra más grande **se sale por abajo** en vez de recortarse |
| **Mínimo** | Los 21 px de cinco tatamis, que es lo más pequeño que se lee desde el fondo del pabellón |
| **Paso** | 1 px por pulsación. Ocho pasos llevan una página de tres tatamis de 29 a 21, y eso permite pararse justo donde el nombre más largo de esa competición deja de recortarse |

La reducción es **una para todo el tablón**, no una por página, y cada página la aplica sobre su
propio tamaño y la recorta contra su propio suelo. Con siete tatamis —cuatro y tres— la página de tres
baja más que la de cuatro antes de tocar el suelo, que es justo lo que se quiere: la que tiene la
letra más grande es la que tiene margen para bajar.

**Está en la cabecera del tablón y no en la pantalla de configuración** a propósito: es un ajuste que
hay que hacer *mirándolo* —se baja hasta que el apellido más largo deja de recortarse, y eso no se
sabe de antemano—. Meterlo en la configuración obligaría a salir del tablón, cambiar un número, volver
a entrar y mirar, diez veces seguidas.

Lo que se elija **se guarda solo**, un par de segundos después de la última pulsación
(`ConfiguracionApp.ReduccionDeLetra`). Una pantalla de sala se apaga por la noche y no puede amanecer
con los nombres recortados otra vez. También responden las teclas **−** y **+**, las del teclado
numérico y las de la fila de arriba.

---

## 4. Las dos páginas y la rotación

Diez tatamis de cuatro filas no caben en una pantalla con un tamaño de letra que se lea desde el fondo
del pabellón. Así que se parten y se alternan.

| Tatamis del evento | Páginas | Reparto |
|---|---|---|
| 1 – 5 | **1** | Todos a la vez, sin rotación |
| 6 | 2 | 1–3 · 4–6 |
| 7 | 2 | 1–4 · 5–7 |
| 8 | 2 | 1–4 · 5–8 |
| 9 | 2 | 1–5 · 6–9 |
| 10 | 2 | 1–5 · 6–10 |

Dos cosas de este reparto:

- **Con cinco o menos, una sola página y la rotación se para.** No hay nada entre lo que alternar, y
  hacer rotar una pantalla contra otra vacía sería enseñar la mitad del tiempo un hueco.
- **Con más de cinco se parte por la mitad, no en «cinco y el resto».** Con seis tatamis, tres y tres
  deja las filas mucho más altas que cinco y uno, y una pantalla que se lee de lejos vive de eso. Con
  diez sale el reparto natural, cinco y cinco.

El cambio va con un **fundido cruzado** de medio segundo. No es adorno: en una pantalla grande,
cambiar de golpe cinco tatamis por otros cinco se ve como un fogonazo y quien estaba leyendo una fila
pierde el sitio.

**Cada cuánto** lo dice la configuración del equipo —veinte segundos por defecto, ajustable entre 5 y
120 (guía [01](01-Red-de-las-Pantallas.md), §3, paso 2 bis)—. Se ajusta porque la distancia de lectura
no es la misma en todos los pabellones, y porque con dos tatamis por página se lee en la mitad de
tiempo que con cinco.

Y se puede forzar en cualquier momento con **Enter**, que además **reinicia la cuenta**: quien acaba de
cambiar de página a mano no quiere que se le cambie debajo al segundo siguiente.

> **Truco de montaje.** Si hay dos televisores juntos, ponerles rotaciones distintas —20 y 23
> segundos, por ejemplo— hace que casi nunca cambien de página a la vez, y así siempre hay una de las
> dos mitades de la sala a la vista.

### 4.1 Señalar el tatami que acaba de cambiar

Cuando cambia lo que un tatami tiene pintado, el tablón **salta a la página donde está ese tatami** y
**pone su rótulo —«Tatami 4»— en amarillo diez segundos**. Es lo que convierte una pared que la gente
mira de vez en cuando en una que **avisa**: el movimiento se ve desde el otro lado del pabellón aunque
no se esté leyendo.

**Diez segundos, y no menos.** El aviso tiene que aguantar el recorrido entero de quien lo recibe:
alguien que estaba de espaldas se gira, busca en qué banda está el amarillo, y solo entonces empieza a
leer las cuatro filas de ese tatami. Con cinco, la mitad de las veces se apagaba justo cuando esa
persona llegaba a mirar, y un aviso que se pierde a mitad es peor que no avisar.

Se señala el **título** y no el marco del tatami, y no es lo mismo: el título es exactamente lo que se
busca cuando alguien mira el tablón —«¿dónde está el 4?»—, así que pintarlo de amarillo lleva el ojo
al sitio en el que ya estaba mirando. Un borde alrededor de una banda que ocupa un quinto de la
pantalla señala «por aquí», que es mucho menos.

Cuatro detalles, y ninguno es decorativo:

- **Solo cuenta lo que se ve.** Si el resultado que llega es de un combate que no estaba pintado —el
  número quince de la cola, cuando la mesa se salta uno porque falta un competidor—, el tablón **no
  hace nada**: no salta de página ni enciende el amarillo. Mandar a la sala a mirar un tatami que se
  ve exactamente igual que hace un segundo es peor que no avisar, porque la próxima vez que parpadee
  de verdad ya nadie mira.
- **De uno en uno, con cola.** Si mientras hay uno señalado llega otro, el segundo **espera a que se
  cumplan los diez segundos del primero** antes de encenderse y de llevarse la pantalla a su página.
  Encadenar dos saltos de página no lo sigue nadie, y dos rótulos amarillos a la vez no señalan nada.
  Un tatami que ya está en la cola **no se apunta dos veces**, así que la cola no puede ser más larga
  que el número de tatamis del evento.
- **La rotación se para mientras hay algo señalado.** Si no, el reloj de los veinte segundos podría
  llevarse de la pantalla justo el tatami que se acaba de señalar. Se rearranca al vaciarse la cola.
- **La primera lectura no cuenta.** Montar el tablón no es que un tatami acabe de anotar un resultado;
  sin esa distinción, arrancar la aplicación resaltaría los diez tatamis en cadena y la pantalla se
  pasaría el primer minuto saltando de página sola.

Qué se considera «un cambio»: que las **cuatro filas pintadas** de ese tatami no sean las mismas que
en la lectura anterior. Se mira tanto **qué combates hay** como **si cada uno tiene ya a sus dos
competidores**: un resultado anotado quita un combate y sube a los de abajo, y la propagación del
ganador enciende la final que estaba «por determinar». Las dos cosas merecen que la sala mire.

> **Cuidado al tocar esto.** El amarillo va como propiedad de color enlazada
> (`TatamiEnTablon.ColorTitulo`) y **no** como una clase de estilo. Se intentó lo segundo y no llegaba
> a verse nunca: el color en reposo está puesto como valor local en el XAML, y en Avalonia un valor
> local gana a cualquier `Setter` de un `Style`.

> **Se puede apagar**, y en competiciones con muchos tatamis hay que hacerlo: a media mañana entra un
> resultado cada pocos segundos, y a diez segundos por aviso —con la rotación parada mientras dura— la
> cola no se vacía nunca y la pantalla deja de alternar entre las dos mitades de la sala. El ajuste está en la configuración
> del equipo, junto al del cambio de página (guía [01](01-Red-de-las-Pantallas.md), §3).

---

## 5. Cómo se entera de que algo ha cambiado

Por las dos vías, y hacen falta las dos.

### 5.1 El aviso del servidor, que es la principal

Un WebSocket a `wss://judo-server:8443/ws/eventos/{id}`. En cuanto un tatami anota un resultado, el
servidor lo cuenta y la fila desaparece del tablón en menos de un segundo. **Nunca llega nada por ahí
que la pantalla mande**: solo recibe.

El aviso lleva **solo la tabla y el evento, nunca los datos** (`AvisoCambio`). La pantalla recibe «ha
cambiado algo de los combates del evento 31» y relee por la API lo que necesita. Dos motivos: la carga
útil de `pg_notify` no puede pasar de 8000 bytes, y un aviso con datos dentro puede llegar ya caducado
si entretanto hubo otro cambio.

Es también la razón de que esto **no** sea un sondeo: con diez tatamis en marcha y varias pantallas en
la sala, preguntar en bucle serían miles de consultas al día para enterarse de lo mismo.

**Los avisos llegan en ráfaga, y hay antirrebote.** Anotar un resultado dispara el trigger que propaga
al ganador al combate siguiente del cuadro, así que un solo combate terminado produce varios `UPDATE`
y varios avisos en el mismo instante. El tablón espera **400 ms** a que la ráfaga pare antes de releer;
sin eso serían tres o cuatro lecturas completas del evento por combate.

### 5.2 Y un repaso periódico

No es redundancia inútil. Un WebSocket se puede quedar **mudo sin dar error** —un cable que se mueve,
un conmutador que reinicia, el equipo que vuelve de suspensión— y entonces el aviso simplemente no
llega. En un marcador de tatami eso lo nota alguien en seguida; en una pantalla colgada en la pared
que nadie mira de cerca, puede pasar la mañana entera con el orden de las nueve.

Media hora de tablón viejo es un problema mucho mayor que una lectura de más cada medio minuto. Son
unos cientos de filas en una sola petición, en una red local.

**Cada cuánto lo dice la configuración del equipo** (`SegundosRefresco`): treinta segundos por
defecto, ajustable entre 5 y 600 (guía [01](01-Red-de-las-Pantallas.md), §3, paso 2 bis). Se ajusta
porque el equilibrio no es el mismo en todas partes, y los dos extremos son reales: una red del
pabellón que corta a menudo pide bajarlo, y **diez pantallas releyendo el evento entero** contra un
servidor justo de fuerza piden subirlo —cada repaso es una lectura completa **por pantalla**—.

Además se relee **en cuanto la escucha se recupera**: mientras estaba caída han podido pasar cosas de
las que nadie avisó.

### 5.3 Lo que pasa cuando el servidor no está

**El tablón se queda con lo que tenía.** Un orden de hace un minuto sigue sirviéndole a la sala, y
borrarlo para poner un mensaje de error solo conseguiría que la gente se quede sin saber por dónde va
la competición. Que puede estar quedándose viejo lo dice el punto rojo de la esquina.

El mensaje solo sustituye al tablón cuando **no hay nada que enseñar todavía**: el fallo ha caído en la
primera lectura y la pantalla está en blanco de todos modos.

### 5.4 Y por qué el tablón no parpadea nunca

Los tatamis, sus cuatro filas y las páginas son objetos **estables**: se crean una vez y las lecturas
solo cambian lo que llevan dentro ([`FilaTablon.cs`](../ViewModels/Tablon/FilaTablon.cs)). Con
cientos de avisos al día, rehacer la pantalla en cada uno sería un parpadeo constante en una pared que
está mirando el pabellón entero.

Lo único que se rehace es el reparto en páginas, y solo si **cambia el conjunto de tatamis** del
evento: al cargar la primera vez, y si administración reparte los combates en más o menos tatamis a
media jornada.

---

## 6. Las teclas

La aplicación va a pantalla completa y sin marco. Lo que hay en un equipo de sala es un teclado suelto
encima de una mesa, y con él tiene que bastar.

| Tecla | Qué hace |
|---|---|
| **Enter** | Cambia de página. Reinicia la cuenta de la rotación |
| **→ ↓** | Lo mismo que Enter |
| **← ↑** | La página anterior |
| **F5** · **R** | Volver a leer los combates ahora, sin esperar el repaso |
| **−** · **+** | Letra más pequeña o más grande, lo mismo que los botones de la cabecera (§3.6) |
| **Esc** | Volver al inicio de sesión (y desde ahí, a la configuración o a salir) |

Los botones de la cabecera hacen lo mismo que −, +, Esc y salir. Son pequeños y apagados a propósito:
no son para el público, son para quien sube a la escalera. Recargar **solo** está en el teclado: un
botón para lo que la pantalla ya hace sola solo sirve para que alguien lo pulse pensando que hacía
falta.

> **Las teclas se reparten en la fase de túnel**, antes de que las vea ningún control de la pantalla
> ([`MainWindow.axaml.cs`](../MainWindow.axaml.cs)). Es la diferencia entre un tablón que responde a
> Enter y uno que no: un botón que se haya quedado con el foco —el de cerrar sesión de la cabecera,
> sin ir más lejos— se llevaría Enter, y pulsarlo para cambiar de página volvería a pulsar ese botón.

---

## 7. Lo que esta aplicación no hace

**No escribe nada.** Ni una sola escritura contra el servidor: el cliente de esta aplicación no tiene
POST ni PUT ([`ClienteApi.cs`](../JudoCombates.Client/ClienteApi.cs)). De ahí se siguen unas cuantas
cosas que en el crono existen y aquí no hacen falta:

| En el crono | Aquí |
|---|---|
| Cola de resultados sin enviar, con reintento cada minuto | No hay nada que enviar |
| Detección del 409 de edición concurrente | No se puede chocar con nadie |
| Pregunta antes de cerrar («hay resultados sin enviar») | Cerrar no le quita nada a nadie |
| Solo entra el rol `marcador`, para que el acta la firme quien la anotó | Entra cualquier usuario; el rol `pantalla` es una recomendación, no una barrera (guía [01](01-Red-de-las-Pantallas.md), §3, paso 6) |
| Modo sin conexión, para llevar la competición a mano | Sin servidor no hay orden que enseñar |

Tampoco genera PDF, ni suena, ni guarda imágenes: la portada del campeonato es cosa del marcador del
tatami, que la enseña entre combates. Aquí no hay hueco donde ponerla, porque no hay ni un momento en
el que la pantalla no esté enseñando combates.

---

## 8. Las banderas

**No viajan dentro del programa: las sirve el servidor.** Es lo que evita que dibujar el escudo de un
club nuevo obligue a actualizar la aplicación de todas las pantallas del pabellón, o a copiar archivos
a mano equipo por equipo la mañana de la competición. El PNG se añade en el servidor —que es de donde
lo sacan también los informes en PDF— y las pantallas lo enseñan en cuanto vuelven a entrar.

Son dos llamadas y hacen falta las dos
([`ServicioBanderas.cs`](../Services/Competicion/ServicioBanderas.cs)):

1. **La lista**: qué archivo le toca a cada código de agrupación. El orden de combates solo trae el
   código («ALC») y de ahí no sale el nombre del archivo, que lleva la jerarquía entera dentro
   («ESP_CVA_ALC») y depende de cuál de los candidatos esté dibujado de verdad. Eso lo resuelve el
   servidor, que es donde están los PNG.
2. **Las imágenes**, una a una y en segundo plano. **No se espera a que estén**: el tablón se pinta con
   el código y cada bandera aparece sola en su hueco cuando llega, que en una red local es un
   instante.

Nada de esto puede dejar el tablón sin combates. Si el servidor no contesta o una bandera no está, se
pinta el código y no se avisa de nada: una bandera que falta no es un problema de competición.

Los PNG se guardan en memoria mientras la aplicación esté abierta y **sobreviven al cambio de evento**:
el escudo de un club es el mismo en cualquier competición. Se decodifican a un techo de 256 px de alto,
que es mucho más de lo que el tablón necesita —una bandera ocupa el alto de media fila— y deja margen
para un televisor 4K.

---

## 9. Notas para tocar esta pantalla

### El número de la cola se cuenta antes de filtrar

Es el error fácil de cometer al tocar
[`TablonViewModel.Montar`](../ViewModels/Tablon/TablonViewModel.cs): si el número se calcula sobre los
pendientes, el primero de la cola pone siempre 1 y el tablón deja de coincidir con la hoja de papel de
la mesa. Se numera sobre la cola **completa** y se filtra después.

El orden lo da el servidor (`ORDER BY tatami, orden_tatami` en `fn_combates_orden_evento`), así que la
posición en la lista **es** el número; no hay que ordenar nada aquí.

### Los anchos fijos van en los hijos, no en las columnas

El `Width` de una `ColumnDefinition` es un `GridLength`, y ahí no llega un enlace a un número. Los
anchos que salen de `MedidasTablon` se ponen en el propio `TextBlock` o `Panel`, con las columnas en
`Auto`. Es lo que hace que el número, el escudo, el código y el peso caigan en la misma vertical en las
cuarenta filas de la pantalla; con ancho automático, cada fila los colocaría según el largo de su
contenido.

### Las medidas bajan por el objeto, no por el árbol visual

Un enlace que sube por el árbol —`$parent[ItemsControl].DataContext.Algo`— no se puede compilar y se
rompe en silencio en cuanto alguien cambia un contenedor de sitio. Así que las medidas se guardan en
cada `TatamiEnTablon` y en cada `FilaTablon`, y el tablón las reparte al montar las páginas.

### La fase se enseña por su rótulo, nunca por su nombre

Los nombres de fase los pone el sorteo y se guardan como identificadores, con guion bajo:
`mejor_bronce`, `grupo_a`. Para comparar valen tal cual; para enseñarlos, no. `FaseRotulo` los escribe
para leerlos («Mejor bronce», «Grupo A»). Hoy el tablón no enseña la fase en la fila —el sitio se lo
lleva el nombre del competidor, que es lo que se busca— pero el rótulo está resuelto por si algún día
cabe.

### El femenino se guarda como W y se rotula F

En la base de datos es **W**, la convención internacional; en el tatami y en la sala se escribe **F**.
La traducción se hace una vez, en `CombateEnTablon.SexoCodigo`, para que no haya dos criterios.

---

## Ver también

- [`01-Red-de-las-Pantallas.md`](01-Red-de-las-Pantallas.md) — plan de direcciones, montaje del equipo,
  certificado y dónde se guarda la configuración.
- [`README.md`](../README.md) — qué es JudoCombates y cómo compilarlo.
- [`TablonViewModel.cs`](../ViewModels/Tablon/TablonViewModel.cs) — el filtro de pendientes, el reparto
  en páginas y las dos vías de refresco, en código.
- [`TablonView.axaml`](../Views/Tablon/TablonView.axaml) — la maquetación de la fila y los degradados.
- JudoAdministración, `Documentación/02-Red-y-Direccionamiento-IP.md`, §2.4 — las pantallas de
  visualización dentro del plan de red.
