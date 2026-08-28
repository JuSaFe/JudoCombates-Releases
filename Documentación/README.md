# Documentación

Dos documentos. Cuál leer depende de qué haya que hacer.

| Voy a… | Documento |
|---|---|
| **Montar una pantalla**: dirección IP, certificado, servidor, número de pantalla | [01 · Red de las pantallas](01-Red-de-las-Pantallas.md) |
| **Entender qué se ve y por qué**: qué combates salen, cómo se reparten, cada cuánto se refresca, las teclas | [02 · El tablón de combates](02-El-Tablón-de-Combates.md) |

Y en la raíz del repositorio, el [`README.md`](../README.md): qué es JudoCombates, cómo encaja en la
competición y cómo compilarlo.

---

## Montar una pantalla, en corto

Se hace todo desde la propia aplicación, en la **rueda dentada de la pantalla de inicio de sesión** →
*Red y servidor*. No hace falta terminal, ni tener JudoAdministración instalada, ni haber podido
iniciar sesión —que es justo lo que no se puede hacer cuando el problema es la red—.

1. **Servidor**: `https://judo-server:8443`, y el nombre y la IP (`192.168.2.3`) con los que se
   escribe la línea del archivo `hosts`.
2. **Número de pantalla**: del 1 al 10. De él sale la dirección IP del equipo (pantalla N →
   `192.168.2.(N+19)`).
3. **Cambio de página**: cada cuántos segundos alterna entre las dos mitades de la sala. Veinte por
   defecto.
4. **Red**: interfaz, dirección, puerta de enlace, máscara y DNS → **Aplicar la red**. Un solo
   diálogo de permisos para todo.
5. **Certificado**: el `judo-server.crt` público → **Elegir e instalar…**
6. **Guardar**, y a iniciar sesión con el usuario de rol `pantalla`.

Al acabar la competición, **Deshacer y dejar la red como estaba** devuelve el equipo a como estaba
—incluida la configuración de red que tuviera antes, si traía la suya—.

El detalle de cada paso, y qué hacer cuando algo falla, está en la
[guía 01](01-Red-de-las-Pantallas.md).

---

## Lo que hay que saber antes de montar la sala

| | |
|---|---|
| **Una pantalla enseña todos los tatamis** | El número de pantalla no filtra nada: es solo la etiqueta del equipo en la red. Diez pantallas y diez tatamis son dos límites que coinciden por casualidad |
| **Aquí el wifi vale** | Una pantalla que se reconecta solo parpadea. En un marcador de tatami, un corte de red es un incidente arbitral; aquí no, porque esta aplicación no escribe nada |
| **La contraseña no se guarda** | Se teclea una vez al empezar la jornada y el token dura dieciséis horas. Estos equipos se quedan solos y a la vista |
| **Con dos televisores juntos**, rotaciones distintas | 20 y 23 segundos, por ejemplo: así casi nunca cambian de página a la vez y siempre hay una de las dos mitades a la vista |
| **Sin servidor, el tablón se queda** | Con lo último que leyó. Lo que avisa de que puede estar viejo es el punto rojo de la esquina |

---

## Publicar una versión

Se empuja un tag `v*` sobre `master` y lo hace todo
[`.github/workflows/instaladores.yml`](../.github/workflows/instaladores.yml):

```bash
# La versión manda desde Directory.Build.props: si no coincide con el tag, el workflow se para.
git tag v1.0.0.1 && git push origin v1.0.0.1
```

Compila los tres instaladores —cada uno en su propio *runner*, porque `iscc`, `hdiutil` y
`appimagetool` solo existen en su sistema—, copia esta documentación y la portada al repositorio
público `JudoCombates-Releases` y publica allí la release.

Para generar un instalador a mano, en el sistema que toque:

```bash
bash Empaquetado/generar-instaladores.sh          # .dmg en macOS, .AppImage en Linux
```
```powershell
.\Empaquetado\generar-instaladores.ps1            # .exe con Inno Setup, en Windows
```

Y si se cambia el diseño del icono (`Empaquetado/iconos/icono.swift`), en macOS:

```bash
./Empaquetado/iconos/generar-iconos.sh            # rehace los cuatro archivos de Assets/Icons
```
