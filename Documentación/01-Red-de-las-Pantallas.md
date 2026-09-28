# Red de las pantallas de visualización

Lo que hay que dejar puesto en un equipo para que JudoCombates llegue al servidor de la competición,
y —sobre todo— **qué parte de eso hace esta aplicación y qué parte no**.

> **El plan de direcciones no se decide aquí.** La red de la competición es una sola y la manda
> JudoAdministración, en su documento `Documentación/02-Red-y-Direccionamiento-IP.md`. Este
> documento es la vista de la pantalla de sala: el mismo plan, contado desde la pared del pabellón,
> más lo que solo afecta a esta aplicación. Si los dos se contradicen, manda el de
> JudoAdministración.

---

## 1. Dónde encaja una pantalla de visualización

Red `192.168.2.0/24` · máscara `255.255.255.0` · puerta de enlace `192.168.2.1`.

| Rango | Nº | Para qué |
|---|---|---|
| `192.168.2.1` | 1 | Router (puerta de enlace) |
| `192.168.2.2` | 1 | Libre a propósito (algunos routers se cogen una segunda dirección) |
| `192.168.2.3` | 1 | **Servidor** — PostgreSQL y la API |
| `192.168.2.4` | 1 | Servidor de respaldo (reservada, §8) |
| `192.168.2.5` – `.9` | 5 | Puestos de administración |
| `192.168.2.11` – `.20` | 10 | Marcadores de tatami (JudoCrono) |
| **`192.168.2.21` – `.30`** | **10** | **Pantallas de visualización — esta aplicación** |
| `192.168.2.31` – `.99` | 69 | Libre (ampliación) |
| `192.168.2.100` – `.199` | 100 | Pool DHCP |
| `192.168.2.200` – `.254` | 55 | Libre (pruebas y diagnóstico) |

### 1.1 La convención: pantalla N → `192.168.2.(N + 20)`

La dirección termina en el número de pantalla más veinte: la pantalla 1 es la `.21`, la 5 la `.25`.
No es una obligación técnica —el servidor no mira de qué IP viene cada cliente— pero se respeta
siempre, porque es lo que permite saber a qué equipo estás mirando cuando algo va mal sin tener que
recorrer el pabellón.

> **Antes el rango empezaba en la `.20`** (pantalla 1 → `.20`). Se movió a la `.21` para que la
> primera pantalla acabe en 1 y no en 0, que era lo que confundía. Un equipo configurado con el plan
> viejo se queda con la dirección vieja hasta que se le vuelva a elegir la suya y se pulse
> **Aplicar la red**.

| Pantalla | Dirección | | Pantalla | Dirección |
|---|---|---|---|---|
| 1 | `192.168.2.21` | | 6 | `192.168.2.26` |
| 2 | `192.168.2.22` | | 7 | `192.168.2.27` |
| 3 | `192.168.2.23` | | 8 | `192.168.2.28` |
| 4 | `192.168.2.24` | | 9 | `192.168.2.29` |
| 5 | `192.168.2.25` | | 10 | `192.168.2.30` |

### 1.2 Por qué el máximo son diez pantallas

Porque el rango reservado son diez direcciones, `.21` a `.30`. De ahí sale el límite del campo
**Número de esta pantalla**, que acepta de 1 a 10 y no más
([`PantallaView.axaml`](../Views/Configuracion/PantallaView.axaml),
[`RangoPantallas`](../Services/Red/ModelosRed.cs)).

**Diez pantallas no tiene nada que ver con diez tatamis.** Son dos límites que coinciden por
casualidad: el de los tatamis lo pone el evento en JudoAdministración, y el de las pantallas, el
plan de direcciones. Una sola pantalla enseña los combates de **todos** los tatamis del evento, así
que lo normal es tener bastantes menos de diez: una en la entrada, una junto a la zona de
calentamiento, una en la mesa de control.

Si algún día hicieran falta más, lo que hay que cambiar es el **plan de direcciones** —hay setenta
libres a partir de la `.31`— y después el límite de `RangoPantallas`. Subir el límite del campo sin
ampliar el rango deja pantallas sin dirección propia.

### 1.3 Aquí el wifi sí vale

En los marcadores de tatami se insiste en el cable: un marcador que pierde el resultado de un
combate por un corte de wifi es un incidente arbitral. **Una pantalla de visualización no escribe
nada**, así que una reconexión no cuesta más que unos segundos de tablón sin refrescar —y ni eso se
nota, porque al recuperar la conexión relee sola—. Si llegar con cable hasta donde va colgada la
pantalla es un problema, el wifi es una solución legítima.

Lo único que hay que respetar es que la dirección siga siendo **fija y del rango**: el pool DHCP del
router va de `.100` a `.199` justamente para no pisar las estáticas.

---

## 2. Una sola pantalla de configuración, dos capas

Todo se configura en **la rueda dentada de la pantalla de inicio de sesión**, en un solo sitio:

| Capa | Qué incluye | Cómo se aplica |
|---|---|---|
| **Aplicación** | A qué dirección habla (`ApiBaseUrl`), qué número de pantalla es (`Pantalla`), cada cuánto cambia de página (`SegundosRotacion`) y relee los combates (`SegundosRefresco`), el tamaño de letra (`ReduccionDeLetra`) y si señala el tatami que cambia (`AvisarCambiosDeTatami`) | Botón **Guardar**. Se escribe en un archivo del usuario (§6) y como mucho cierra la sesión. Aquí está también el desvío temporal a un servidor de pruebas (§7) |
| **Sistema operativo** | Dirección IP fija, máscara, puerta de enlace, DNS, la línea del archivo `hosts` que traduce `judo-server`, y el certificado del servidor en el almacén de confianza | Botón **Aplicar la red**. Pide permisos de administrador y corta la conexión un momento |

Son dos capas con consecuencias muy distintas, y de ahí que tengan botones distintos: la segunda,
mal hecha, deja dos equipos con la misma dirección —el fallo más difícil de encontrar de todos— y hay
que poder deshacerla al acabar la competición. Pero **están en la misma pantalla**, y eso no es un
detalle de presentación.

### 2.1 El número de pantalla gobierna las dos capas

Es la razón de que la pantalla de configuración sea una. El número es **un solo dato** que sirve para
dos cosas:

- identifica al equipo dentro de la sala: es lo que se escribe en la pegatina del televisor;
- decide su dirección IP, que el plan deriva de él según la convención de §1.1.

Así que se pide **una vez**, y de él cuelga lo demás: la pantalla no pregunta qué dirección poner, la
deduce —pantalla 3 → `192.168.2.23`—.

**Y va en los dos sentidos: mover el número mueve la dirección, y elegir una dirección mueve el
número.** No son dos campos que puedan discrepar; son el mismo dato dicho de dos maneras. Siguen
siendo dos controles porque cada uno se entiende en su contexto —el número, dentro de la sala; la
dirección, dentro de la red— y porque el desplegable de direcciones es además donde se ve cuáles
están ocupadas.

Consecuencia de tenerlos atados: **el escaneo no salta a otra dirección por su cuenta.** Si la que le
toca a la pantalla está ocupada, se queda a la vista en rojo, con «Aplicar la red» apagado, y hay que
resolverlo —apagar el otro equipo, o cambiar de número a sabiendas—. Saltar solo a la primera libre
cambiaría el número del equipo, y eso no es algo que pueda decidir un escaneo.

### 2.2 Para qué **no** sirve el número de pantalla

Ésta es la diferencia de fondo con el crono, y conviene tenerla clara antes de montar la sala:

**el número de pantalla no decide qué combates se ven.** El tablón enseña todos los tatamis del
evento, sea la pantalla 1 o la 7. En el crono, el número de tatami es el filtro de la cola de
combates y sin él no se puede trabajar; aquí es solo la etiqueta del equipo en la red.

De eso se siguen dos cosas prácticas:

- **Se puede trabajar sin número.** El campo puede quedarse en blanco, y entonces lo único que falta
  es la dirección IP propuesta del plan. Es lo normal en un equipo que va por DHCP en una red que ya
  está montada. La aplicación no avisa de nada, porque no falta nada.
- **Dos pantallas iguales enseñan lo mismo.** Si hacen falta dos televisores en el mismo sitio, con
  poner uno en la pantalla 1 y otro en la 2 se ven idénticos, sin más configuración. Lo que sí puede
  interesar es darles rotaciones distintas (§3, paso 2 bis) para que no cambien de página a la vez y
  siempre haya una de las dos mitades a la vista.

### 2.3 Compatible con las herramientas de JudoAdministración

Hay tres vías para configurar la red de un equipo de la competición: los guiones
`Empaquetado/red/configurar-red.sh` (y `.ps1`) de JudoAdministración, su pantalla de red, y esta. Las
tres son **intercambiables**: se puede configurar por una y deshacer por otra, porque comparten
literalmente las dos cosas que dejan escritas en el equipo:

- el archivo de estado anterior, en la misma ruta y con el mismo formato
  —`/etc/judo-red-anterior.conf` en macOS y Linux, `C:\ProgramData\JudoAdministracion\red-anterior.json`
  en Windows; la pantalla enseña la ruta—;
- la marca `# JudoAdministracion` de las líneas del archivo `hosts`.

Por eso la marca dice `JudoAdministracion` aunque la escriba JudoCombates. Si cada aplicación pusiera
la suya, un equipo configurado aquí y deshecho con el guion se quedaría con su línea en el `hosts`
para siempre.

**Lo que esta aplicación no hace es emitir certificados.** Los instala, que es lo que necesita un
cliente; emitirlos es cosa del servidor y se hace una vez, desde JudoAdministración.

---

## 3. Montar una pantalla de cero

Todo desde la propia aplicación, sin terminal y sin tener JudoAdministración instalada. Se empieza en
la **rueda dentada de la pantalla de inicio de sesión**: no hace falta haber entrado, que es justo lo
que se necesita cuando el problema es que no conecta.

Lo único que hay que llevar al pabellón es el archivo **`judo-server.crt`** (en una memoria USB, por
correo, como sea). El `.crt` público, **nunca** el `.pfx`: ese lleva la clave privada del servidor y
no sale de él. El selector de archivos de la aplicación ni lo ofrece.

La pantalla se abre **diagnosticando sola**: quien llega aquí lo hace porque algo no va, y lo primero
que quiere saber es qué. Los apartados van en el orden en que hay que rellenarlos.

### Paso 1 · Servidor de la competición

| Campo | Valor en una pantalla de la red |
|---|---|
| Dirección del servidor | `https://judo-server:8443` |
| Nombre del servidor en la red | `judo-server` |
| Su dirección IP | `192.168.2.3` |

**El nombre, no la IP**, en la dirección: `https://judo-server:8443` y no
`https://192.168.2.3:8443`, para que el plan de contingencia de §8 funcione sin tocar ninguna
pantalla. Los otros dos campos son los que construyen la línea del archivo `hosts` que hace que ese
nombre signifique algo.

El botón **Probar conexión** pregunta por `/api/estado`, el único punto de la API que no pide sesión.
De momento va a fallar: todavía no hay ni dirección propia ni certificado. Son los pasos siguientes.

### Paso 2 · Número de esta pantalla

El número, de 1 a 10. Un solo campo, del que cuelga la dirección IP del paso siguiente (§2.1). Debajo
se ve la dirección que le toca en cuanto se pone. Se puede dejar en blanco (§2.2).

### Paso 2 bis · Ritmo del tablón

En el mismo sitio, apartado **«3. Ritmo del tablón»**, tres ajustes que contestan a lo mismo: cada
cuánto se mueve la pantalla.

- **Cambio de página.** Cada cuántos segundos el tablón pasa a la otra mitad de la sala cuando el
  evento tiene más de cinco tatamis. Veinte por defecto, entre 5 y 120.
- **Repaso de los combates.** Cada cuántos segundos vuelve a leer el evento entero por si algún aviso
  del servidor no llegó. Treinta por defecto, entre 5 y 600. No es la forma normal de enterarse —eso
  es el aviso por WebSocket, que llega en menos de un segundo—, es la red de abajo.
- **Señalar el tatami que acaba de cambiar.** Cuando cambia lo que un tatami tiene pintado, el tablón
  salta a la página donde está ese tatami y pone su rótulo en amarillo diez segundos. Encendido de
  fábrica.

Ninguno de los tres es de red ni hace falta tocarlo para trabajar, pero se ponen aquí porque son parte
de montar el equipo: dependen de la distancia a la que se lea esa pantalla concreta y de cómo vaya la
red del pabellón, y eso no se sabe hasta que está colgada.

> **El aviso conviene apagarlo en competiciones con muchos tatamis.** A media mañana, con diez en
> marcha, entra un resultado cada pocos segundos; a diez segundos por aviso, y con la rotación parada
> mientras dura, la cola no se vacía nunca y la pantalla deja de alternar entre las dos mitades de la
> sala. Con tres o cuatro tatamis es justo al revés: es lo que hace que la sala mire.

> **El repaso se multiplica por el número de pantallas.** Cada uno es una lectura del orden de
> combates completo, así que diez televisores a treinta segundos son veinte lecturas por minuto contra
> el servidor. Bajarlo a cinco «por si acaso» en una sala grande es la forma más directa de cargar el
> servidor sin que nadie note ninguna mejora.

**El tamaño de la letra no está aquí.** Es el único ajuste del tablón que se queda fuera de esta
pantalla, y a propósito: se ajusta *mirándolo*, hasta que el apellido más largo de esa competición
deja de recortarse. Sus botones —**−** y **+**— están en la cabecera del propio tablón, y lo que se
elija se guarda solo en este mismo archivo.

Los cuatro los cuenta enteros [`02-El-Tablón-de-Combates.md`](02-El-Tablón-de-Combates.md), §3.6 y §4.

### Paso 3 · Red de este equipo

Aquí se pide:

1. **Interfaz de red.** La que está conectada. En un equipo con adaptador USB de red hay varias y
   solo una lleva cable; en una pantalla de sala puede ser el wifi, que aquí sí vale (§1.3). La lista
   pone las conectadas primero y enseña la dirección que tiene cada una.
2. **Dirección IP.** Ya viene propuesta la que le toca al número del paso 2 (§1.1), con el color de
   cada una de las diez del rango:

   | Color | Significa |
   |---|---|
   | 🟢 Verde | Libre |
   | 🔴 Rojo | Ya está ocupada por otro equipo |
   | 🔵 Azul | La tiene este mismo equipo |
   | ⚪️ Gris | No se ha podido comprobar |

   **No deja aplicar una dirección ocupada.** Dos equipos con la misma IP funcionan a ratos, y es el
   fallo más difícil de encontrar de todos: mejor descubrirlo aquí que el día de la competición. La
   dirección ocupada se queda seleccionada y en rojo, con «Aplicar la red» apagado: hay que
   resolverlo, y no se resuelve solo (§2.1).

3. **Puerta de enlace, máscara y DNS alternativo.** Ya vienen con los del plan.

Y entonces **Aplicar la red**. Pedirá permisos de administrador **una sola vez** para toda la
operación, que hace cuatro cosas juntas:

- guarda la dirección del servidor, el número de pantalla, la rotación, el repaso, la letra y el aviso, para que el número no pueda
  quedar desparejado de la IP (§2.1);
- guarda la configuración de red que tenía el equipo, para poder devolverla («Al acabar la
  competición», al final de esta sección);
- pone la dirección fija, la máscara, la puerta y los DNS;
- añade la línea del archivo `hosts` que traduce el nombre:

  ```
  192.168.2.3    judo-server    # JudoAdministracion
  ```

> **Si el equipo todavía no está en esa red**, el escaneo no puede saber qué direcciones están
> ocupadas y las deja en gris: dice «no lo sé» en lugar de «libre». Para eso está el botón
> **Comprobar el rango de todos modos**, que se pone una dirección temporal en la interfaz, pregunta
> por todo el rango —mirando la tabla ARP y no solo el ping, porque el ping lo puede filtrar el
> cortafuegos del equipo de enfrente pero al ARP contesta la tarjeta de red— y la quita al terminar.

### Paso 4 · Certificado del servidor

El servidor sirve HTTPS con un certificado **autofirmado**, emitido para `judo-server`. Hasta que
esté en el almacén de confianza del equipo, JudoCombates no puede conectarse: el `HttpClient` de .NET
rechaza la conexión, y el diagnóstico de arriba lo dice con esas palabras en vez de dejar un «no
conecta».

En el mismo apartado: **Elegir e instalar…** → el `judo-server.crt`. Pide permisos y lo instala en el
almacén del **sistema**, no en el del usuario —que es lo que hace que valga para la aplicación
independientemente de quién la ejecute—. Por dentro es exactamente esto, si alguna vez hay que
hacerlo a mano:

```bash
# macOS — en el llavero del SISTEMA, no en el del usuario
sudo security add-trusted-cert -d -r trustRoot -k /Library/Keychains/System.keychain judo-server.crt

# Linux (Debian/Ubuntu)
sudo cp judo-server.crt /usr/local/share/ca-certificates/judo-server.crt
sudo update-ca-certificates
```
```powershell
# Windows, como administrador
Import-Certificate -FilePath judo-server.crt -CertStoreLocation 'Cert:\LocalMachine\Root'
```

### Paso 5 · Comprobar y guardar

El bloque de arriba de la pantalla vuelve a diagnosticar solo después de cada operación, y su lista
es la misma cadena de §5 comprobada desde dentro de la aplicación: dirección propia, ping al
servidor, resolución del nombre, puerto abierto y certificado de confianza.

La última es la que justifica tenerlo aquí y no solo en un guion: se prueba con un `HttpClient` de
.NET, **el mismo que va a usar la aplicación**. Lo que se ve ahí es exactamente lo que la aplicación
se va a encontrar, y no una aproximación con otra herramienta que tiene sus propios almacenes de
certificados.

Cuando estén todas en verde, **Guardar** y ya se puede iniciar sesión.

### Paso 6 · Con qué usuario se entra

**Con el usuario de rol `pantalla`**, que es de solo lectura. Es una recomendación de montaje, no una
barrera: cualquier usuario de la competición puede entrar en el tablón, y si el rol no es el
recomendado la aplicación lo dice al entrar y deja seguir
([`RolesUsuario`](../JudoCombates.Shared/Contratos/Sesion.cs)).

Que sea una recomendación y no una barrera es deliberado, y va en los dos sentidos:

- **No se impone**, porque rechazar a un usuario válido la mañana de la competición cuesta más de lo
  que arregla, y aquí no hay ningún riesgo de trazabilidad: esta aplicación no escribe **nada** en el
  servidor, así que no hay resultado que pueda quedar firmado por quien no lo anotó. Ésa es la razón
  por la que el crono sí lo impone y esto no.
- **Pero se recomienda en serio**, porque estos equipos se quedan solos, a la vista de todo el mundo
  y a veces prestados, y el token que se emite al entrar vale dieciséis horas. Un token de
  administrador en un ordenador que está detrás de un televisor colgado es un riesgo que no hace
  falta correr.

Por lo mismo, **la contraseña no se guarda en el equipo**: se teclea una vez al empezar la jornada y
el token dura el día entero.

### Al acabar la competición

Botón **Deshacer y dejar la red como estaba**, en la misma pantalla. Devuelve la interfaz a como
estaba —si venía en automático, a automático; si traía su propia IP fija y sus propios DNS, se le
reponen los suyos—, quita la línea del `hosts` y desinstala el certificado. También vale el guion de
JudoAdministración, que lee el mismo archivo de estado:

```bash
sudo Empaquetado/red/configurar-red.sh --deshacer
```

---

## 4. Puertos: lo que la pantalla necesita y lo que no debe alcanzar

| Puerto | Protocolo | Desde la pantalla | |
|---|---|---|---|
| `8443` | TCP · HTTPS y WebSocket | **Sí**, hacia `192.168.2.3` | La API. Es lo único que necesita |
| `5432` | TCP · PostgreSQL | **No** | La base de datos escucha solo en `127.0.0.1` del servidor |

El tablón **nunca** habla con PostgreSQL. No lleva cadena de conexión ni cliente de base de datos:
todo pasa por la API, igual que en los puestos de administración y en los marcadores.

Tráfico de una pantalla, completo: peticiones **GET** al 8443 y **un** WebSocket a
`wss://judo-server:8443/ws/eventos/{id}`, por donde llegan los avisos de que algo ha cambiado. **No
hay una sola escritura**: ni POST, ni PUT, ni DELETE (el cliente de esta aplicación no tiene esos
verbos, ver [`ClienteApi.cs`](../JudoCombates.Client/ClienteApi.cs)). Es la aplicación de la familia
con la superficie más pequeña contra el servidor.

---

## 5. Verificación, en orden

Se hacen en este orden porque cada una descarta la anterior. Sustituye la dirección por la que le
toque a esta pantalla.

| # | Qué se comprueba | macOS y Linux | Windows |
|---|---|---|---|
| 1 | Este equipo tiene su dirección | `ifconfig \| grep "inet "` | `ipconfig` |
| 2 | Llega al router | `ping 192.168.2.1` | `ping 192.168.2.1` |
| 3 | Llega al servidor | `ping 192.168.2.3` | `ping 192.168.2.3` |
| 4 | Traduce el nombre | `ping judo-server` | `ping judo-server` |
| 5 | El puerto está abierto | `nc -vz judo-server 8443` | `Test-NetConnection judo-server -Port 8443` |
| 6 | La API responde y el certificado vale | `curl https://judo-server:8443/api/estado` | `curl.exe https://judo-server:8443/api/estado` |

Cómo leerlo:

- **3 va y 4 no** → falta la línea del archivo `hosts` (paso 1).
- **4 va y 5 no** → cortafuegos del servidor, o la API no está arrancada.
- **5 va y 6 no** → el certificado no está en el almacén de confianza (paso 4). El `curl` sin
  `--cacert` es la prueba de fuego: si pasa ahí, pasa en la aplicación.
- **6 va y la aplicación no** → el problema está en la configuración de la aplicación (paso 1), no
  en la red.

El paso 6 es el mismo que hace **Probar conexión** desde la pantalla de configuración, y con el mismo
cliente HTTP que usa la aplicación después. Si el botón dice que va, va.

---

## 6. Dónde se guarda la configuración de la aplicación

En el archivo del usuario, **no** junto al ejecutable:

| Sistema | Ruta |
|---|---|
| Windows | `%APPDATA%\JudoCombates\configuracion.json` |
| macOS | `~/Library/Application Support/JudoCombates/configuracion.json` |
| Linux | `~/.config/JudoCombates/configuracion.json` |

Así una actualización de la aplicación no se lleva por delante la configuración del equipo, y en
macOS no hay que escribir dentro del paquete `.app` —lo que rompería su firma—.

> **La de macOS es Application Support, no `~/.config`.** Es lo que devuelve .NET para
> `SpecialFolder.ApplicationData` en ese sistema, y es fácil darlo por hecho al revés viniendo de
> Linux. Buscar la configuración en la ruta equivocada lleva a pensar que un archivo «no se aplica»
> cuando sí lo estaba haciendo. **La pantalla de configuración enseña la ruta de verdad**: es la que
> hay que mirar.

Orden de precedencia al leer, de más fuerte a menos
([`ConfiguracionApp.cs`](../Services/Configuracion/ConfiguracionApp.cs)):

| # | Origen | Para qué |
|---|---|---|
| 1 | El archivo del usuario de la tabla de arriba | Lo que se toca desde la aplicación. El único que escribe |
| 2 | `appsettings.Local.json` junto al ejecutable | Desarrollo (§7) y dejar una pantalla preconfigurada desde el instalador |
| 3 | `appsettings.json` junto al ejecutable | La configuración de fábrica que viaja con el programa |
| 4 | `JUDO_API_URL`, `JUDO_PANTALLA`, `JUDO_ROTACION`, `JUDO_REFRESCO` y `JUDO_AVISO_TATAMI` | Variables de entorno |
| 5 | Los valores por defecto del código | `https://judo-server:8443`, sin número, 20 y 30 segundos, letra automática, con aviso |

Cada archivo pisa **solo los valores que trae**, así que un `appsettings.Local.json` con la dirección
pero sin número deja el número que venga de más abajo.

El archivo guarda una clave más, `ServidorAnterior`: la dirección de la competición mientras el
equipo está desviado a un servidor de pruebas (§7). Vacía o ausente significa que no lo está.

> **Arista a tener presente.** Un `Pantalla` a `null` no borra el valor anterior: al leer solo se
> aplican números. Si el número viene de una capa inferior —de la variable `JUDO_PANTALLA` o de un
> `appsettings.Local.json`— y se borra el campo desde la pantalla, al reiniciar vuelve a aparecer.
> Con la configuración de fábrica del repositorio no se manifiesta.

---

## 7. Desarrollo: probar contra la API local

Para desarrollar no hace falta nada de §3: ni IP fija, ni línea de `hosts`, ni certificado. Se apunta
a la API que se arranca en el propio equipo.

**En desarrollo la API va por HTTP en el 8080 y sin certificado.** Es lo que fija `Servidor:Url` de
`JudoAdministracion.Api/appsettings.Local.json`, y por eso `http://` y no `https://`: el certificado
de la competición se emite para `judo-server`, y `dotnet dev-certs` no sirve como sustituto porque
solo emite para `localhost`.

### Cómo queda montado aquí

Con un [`appsettings.Local.json`](../appsettings.Local.json) junto al ejecutable:

```json
{
    "ApiBaseUrl": "http://localhost:8080",
    "Pantalla": 1,
    "SegundosRotacion": 8
}
```

Una rotación corta en desarrollo se agradece: ocho segundos es lo bastante poco para ver el cambio de
página sin esperar, y lo bastante para leer lo que hay.

Y las dos garantías de que eso no se escapa a un instalador, que son las mismas que en
JudoAdministración:

| Garantía | Dónde |
|---|---|
| No va a git | [`.gitignore`](../.gitignore) — `appsettings.Local.json` |
| No viaja en los publicados | [`JudoCombates.csproj`](../JudoCombates.csproj) — `CopyToPublishDirectory=Never` |

`CopyToPublishDirectory` y no una condición sobre `Debug`: atarlo a la configuración de compilación
dejaría sin configuración local cualquier compilación en `Release` hecha para probar en el propio
equipo, que es justo lo que se hace antes de empaquetar.

### Orden de arranque

1. **La API, desde el repositorio de JudoAdministración.** Compilar con `⇧⌘B` y después `F5` con la
   configuración *API*. El tablón es un cliente: sin servidor al otro lado no hay nada que enseñar.
2. **Comprobar que responde**, que es más rápido que averiguarlo desde la interfaz:

   ```bash
   curl http://localhost:8080/api/estado     # {"estado":"ok"}
   ```
3. **El tablón, desde aquí.** Compilar con `⇧⌘B` y después `F5`. En los tres repositorios el orden es
   ése y ninguna configuración de arranque compila, a propósito: dos compilaciones simultáneas de la
   misma solución se pelean por los archivos de `obj/`.

Hace falta además un evento **activo** con el orden de combates ya establecido; si no, el tablón
enseña su cartel de espera y no hay nada que mirar.

### Las tres configuraciones, juntas

| Escenario | `ApiBaseUrl` | Por qué |
|---|---|---|
| Desarrollo en este equipo | `http://localhost:8080` | La API de desarrollo, sin certificado |
| El propio equipo servidor, para pruebas | `https://localhost:8443` | Por loopback, con el certificado real |
| Pantalla de la red | `https://judo-server:8443` | El nombre, no la IP (§8) |

> **La trampa a recordar.** El archivo del usuario manda sobre `appsettings.Local.json` (§6). En
> cuanto guardes una vez desde la rueda dentada, es `~/.config/JudoCombates/configuracion.json` el
> que decide, y cambiar `appsettings.Local.json` deja de tener efecto. Si la aplicación no apunta
> donde crees, ése es el primer archivo que mirar —la pantalla de configuración enseña su ruta— y
> borrarlo devuelve el control a `appsettings.Local.json`.

### El desvío, sin tocar archivos

La segunda dirección de la tabla —`https://localhost:8443`, el servidor de competición montado en
este mismo equipo— está puesta como **un botón en la propia pantalla de configuración**, en el
apartado 1, bajo «Probar contra el servidor de este mismo equipo». Es lo que hace falta cuando hay
que comprobar un servidor recién montado sin salir a buscar otro equipo, y ahorra editar un JSON a
mano —que es donde se cuela el error que después cuesta media hora—.

La de desarrollo (`http://localhost:8080`) **no** tiene botón, a propósito: eso se pone en
`appsettings.Local.json`, y no merece ocupar sitio en una pantalla que se maneja el día de la
competición.

Lo que lo hace utilizable es que **el desvío es reversible y se ve**:

- la dirección a la que apuntaba el equipo se guarda **antes** de pisarla, en `ServidorAnterior` del
  archivo del usuario ([`ConfiguracionApp.cs`](../Services/Configuracion/ConfiguracionApp.cs));
- se guarda en el ARCHIVO y no en memoria, y eso es el punto: probando se cierra y se abre la
  aplicación muchas veces, y un desvío que se olvidara al cerrar obligaría a apuntar la dirección
  buena en un papel —el papel que no aparece la mañana de la competición—;
- mientras dura, la pantalla lo dice **en rojo**, con la dirección de la competición escrita y un
  botón para volver a ella;
- volver a escribir a mano la dirección de la competición y darle a **Guardar** cierra el desvío
  igual que el botón.

El desvío es de la capa de aplicación, no de la del sistema: no toca la IP del equipo, ni el archivo
`hosts`, ni el certificado. Se aplica **en caliente** —el cliente rehace su `HttpClient` y cierra la
sesión que hubiera, porque el token lo emitió el servidor anterior— y acto seguido prueba la conexión
para que se vea enseguida si el servidor local está escuchando.

---

## 8. Contingencia: por qué el nombre y no la IP

El servidor es punto único de fallo. El plan de JudoAdministración es tener un equipo de respaldo
preparado en la `192.168.2.4` y, si el `.3` cae, restaurar la última copia, **cambiarle la dirección
de `.4` a `.3` con el original apagado** y arrancar la API.

Lo que esto significa para las pantallas: **nada**. Ninguna hay que reconfigurarla, porque todas
apuntan a `judo-server` y ese nombre pasa a resolver al equipo nuevo. Es toda la razón por la que la
dirección del servidor se escribe con nombre y no con IP.

Lo que sí conviene saber: al conmutar, las sesiones abiertas se caen y hay que volver a entrar en
cada pantalla. El WebSocket reconecta solo, pero el token lo emitió el servidor anterior.

Y una ventaja de que la pantalla sea de solo lectura: mientras el servidor está caído, **el tablón se
queda con lo último que leyó** en vez de vaciarse. Un orden de hace cinco minutos sigue sirviéndole a
la sala; lo que avisa de que puede estar viejo es el punto rojo de la esquina.

---

## 9. Problemas frecuentes

| Síntoma | Causa probable | Qué comprobar |
|---|---|---|
| «No se ha podido conectar con el servidor» | La API no está arrancada, o la dirección apunta a otro sitio | §5, comprobaciones 5 y 6. Y la ruta del archivo de configuración que enseña la pantalla |
| Conecta por IP pero no por `judo-server` | Falta la línea del archivo `hosts` | §5, comprobación 4. Paso 1 del montaje |
| Error de certificado al probar la conexión | El `.crt` no está en el almacén de confianza del **sistema** | §5, comprobación 6. Paso 4 del montaje |
| «Todavía no hay ninguna competición activa» | No hay evento activo en JudoAdministración | Que alguien lo active desde un puesto de administración |
| «La competición todavía no tiene el orden de combates establecido» | El sorteo está hecho pero los combates no se han repartido por tatamis | JudoAdministración → Sorteo → Establecer orden |
| El punto de la esquina está en rojo | La escucha de cambios está caída | Se recupera sola; mientras tanto el tablón se refresca cada 30 s de todos modos |
| El tablón no cambia de página | El evento tiene cinco tatamis o menos: hay una sola página | El rótulo de la cabecera: sin «1 / 2» no hay otra página |
| Funciona a ratos, y en otro equipo también falla | Dos equipos con la misma IP | `arp -a`, o `nmap -sn 192.168.2.0/24` |
| Cambios en `appsettings.Local.json` que no hacen efecto | El archivo del usuario manda sobre él | §6 y la advertencia de §7 |
| Se cae la sesión sola tras muchas horas | El token ha caducado (16 h por defecto) | Volver a entrar |

---

## Ver también

- [`02-El-Tablón-de-Combates.md`](02-El-Tablón-de-Combates.md) — qué se ve en la pared, cómo se
  reparte, cada cuánto se refresca y con qué teclas se maneja.
- [`README.md`](../README.md) — qué es JudoCombates y cómo compilarlo.
- [`ConfiguracionApp.cs`](../Services/Configuracion/ConfiguracionApp.cs) — la precedencia de §6, en
  código.
- JudoAdministración, `Documentación/02-Red-y-Direccionamiento-IP.md` — el plan completo: pool DHCP,
  cortafuegos, emisión del certificado, hardware y contingencia.
- JudoAdministración, `Documentación/01-Guía-de-Instalación.md` — montaje del servidor y de los
  puestos.
