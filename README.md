<!--
  PORTADA DEL REPOSITORIO PÚBLICO DE DESCARGAS.

  No se edita allí: el trabajo `publicar` de .github/workflows/instaladores.yml copia este archivo
  como README.md del repositorio público en cada release, junto con la licencia, el aviso de
  terceros y la carpeta Documentación. Cualquier cambio hecho a mano en el otro repositorio se
  pierde en la siguiente publicación; se edita aquí.
-->

<div align="center">

<img src="Assets/Icons/judo-256.png" alt="JudoCombates" width="128">

# JudoCombates

**Los combates que vienen, en la pared del pabellón.**

[![Descargar](https://img.shields.io/badge/Descargar-última%20versión-brightgreen?logo=github)](../../releases/latest)
[![Licencia](https://img.shields.io/badge/Licencia-Uso%20libre%20·%20sin%20derivados-green.svg)](LICENSE)
[![Windows · macOS · Linux](https://img.shields.io/badge/Windows%20·%20macOS%20·%20Linux-multiplataforma-informational)](#instalación)

</div>

---

## Qué es

La aplicación de las **pantallas de visualización** de una competición de judo: el televisor de la
entrada, el de la zona de calentamiento, el de la mesa de control. Enseña los combates que quedan por
disputar, tatami por tatami y cuatro por tatami, y se refresca sola en cuanto un tatami anota un
resultado.

```
┌──────────────────────────────────────────────────────────────────────────────┐
│ Campeonato Autonómico Cadete · Alicante 2026     Tatamis 1 – 5  1/2    12:41 │
├──────────────────────────────────────────────────────────────────────────────┤
│ Tatami 1                                                       12 pendientes │
│ ▌42 │ 🏳 ALC  PÉREZ GARCÍA, Luis       │  M -66 kg  │  LÓPEZ SANZ, Ana  VCJ 🏳│
│  43 │ 🏳 CST  GÓMEZ ESTEVE, Marta      │    (12)    │  DÍAZ MOYA, Iván  SAG 🏳│
├──────────────────────────────────────────────────────────────────────────────┤
│ Tatami 2                                                        4 pendientes │
│ …                                                                            │
└──────────────────────────────────────────────────────────────────────────────┘
```

- **Un tatami por fila**, con sus cuatro próximos combates. Hasta cinco tatamis a la vez.
- **Con más de cinco tatamis**, dos páginas que se alternan cada veinte segundos —o al pulsar `Enter`—.
- **Solo los combates pendientes**: en cuanto un tatami anota un resultado, esa fila desaparece.
- **Con el escudo del club, la comunidad o el país** de cada competidor, según el ámbito del evento.

Es un cliente de [**JudoAdministración**](https://github.com/JuSaFe/JudoAdministracion-Releases),
hermano de [**JudoCrono**](https://github.com/JuSaFe/JudoCrono-Releases) —el marcador del tatami—. No
tiene base de datos propia ni funciona sola: se conecta por HTTPS al mismo servidor que los puestos de
administración, en la misma red local del pabellón. Y es de **solo lectura**: no puede modificar nada
de la competición.

## Instalación

Descarga el instalador de tu sistema desde la [**última versión**](../../releases/latest):

| Sistema | Archivo |
|---|---|
| Windows 10/11 (x64) | `.exe` |
| macOS (Apple Silicon) | `.dmg` |
| Linux (x64) | `.AppImage` |

Es **autocontenido**: no hace falta instalar .NET en el equipo. Sí hace falta un servidor de
[JudoAdministración](https://github.com/JuSaFe/JudoAdministracion-Releases) ya en marcha en la misma
red, y un usuario con rol `pantalla`.

Cómo dejar el equipo montado —dirección IP, certificado del servidor, número de pantalla— está en la
**[documentación](Documentación/)**.

## Documentación

| Documento | Para qué |
|---|---|
| [01 · Red de las pantallas](Documentación/01-Red-de-las-Pantallas.md) | Montar una pantalla de cero: dirección IP, certificado, servidor y problemas frecuentes |
| [02 · El tablón de combates](Documentación/02-El-Tablón-de-Combates.md) | Qué combates salen y por qué, cómo se reparten, cada cuánto se refresca y las teclas |

## Soporte

¿Un fallo, una duda o una propuesta? Abre una
[**incidencia**](../../issues/new) describiendo qué ocurre, en qué sistema y con qué versión.

El desarrollo se lleva en un repositorio privado; este de aquí es el punto de descarga y el canal de
incidencias.

## Licencia

Software **de código propietario y uso gratuito**. Texto completo en **[LICENSE](LICENSE)**.

**Se permite**, gratis y sin límite de equipos, usuarios ni tiempo:

- Descargar, instalar y ejecutar el programa, incluso con fines profesionales o comerciales.
- Redistribuirlo **íntegro y sin modificar**, a título gratuito y conservando todos los avisos.

**No se permite** modificarlo, crear obras derivadas, distribuir versiones modificadas, republicarlo
en otro repositorio ni reutilizar partes de él en otros proyectos.

La **propiedad intelectual del programa es de Juan Cotolí San Félix**.

> **Marcas y logotipos.** Los escudos federativos que incluye la aplicación son propiedad de sus
> respectivas entidades y no están cubiertos por esta licencia.

Las bibliotecas de terceros que incorpora se rigen por sus propias licencias, detalladas en
[TERCEROS.md](TERCEROS.md).

---

<div align="center">
<sub>Hecho para el tatami. 🥋</sub>
</div>
