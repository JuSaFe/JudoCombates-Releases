# Componentes de terceros

Bibliotecas que JudoCombates incorpora y bajo qué licencia. Todas son permisivas y compatibles con la
distribución de este proyecto en los términos de su [licencia](LICENSE) — ninguna de ellas obliga a
publicar el programa como código abierto ni a permitir obras derivadas.

**Ninguna impone condiciones de uso.** A diferencia de JudoAdministración, aquí no entra QuestPDF
(la única del otro proyecto con licencia dual): el tablón no genera PDF.

---

## 1. Paquetes NuGet

### Aplicación de escritorio (`JudoCombates.csproj`)

| Paquete | Versión | Licencia | Notas |
|---|---|---|---|
| Avalonia, Avalonia.Desktop, Avalonia.Themes.Fluent, Avalonia.Diagnostics | 11.3.6 | MIT | `Avalonia.Diagnostics` se excluye en Release |
| Avalonia.Fonts.Inter | 11.3.6 | MIT (paquete) · **SIL OFL 1.1** (tipografía Inter) | Redistribución libre; hay que conservar el aviso de copyright de la fuente |
| CommunityToolkit.Mvvm | 8.4.0 | MIT | |
| Microsoft.Extensions.DependencyInjection | 10.0.3 | MIT | |

### Cliente de la API (`JudoCombates.Client`) y contratos (`JudoCombates.Shared`)

Sin dependencias externas: solo la biblioteca base de .NET (`HttpClient`, `ClientWebSocket`,
`System.Text.Json`).

## 2. Plataforma y herramientas

| Componente | Licencia | Cómo se usa |
|---|---|---|
| .NET 9 | MIT | Runtime incluido en los paquetes autocontenidos |

## 3. Marcas y logotipos

Los archivos de `Assets/Images` (escudo de la Federación de Judo de la Comunidad Valenciana) son
propiedad de sus respectivas entidades. La licencia de este proyecto cubre el código, **no** esas
marcas: su uso requiere permiso de sus titulares.

Los iconos de `Assets/Icons` y los dibujos vectoriales de `Styles/Icons.axaml` son obra del titular
del proyecto y se rigen por la [licencia](LICENSE) general.

## 4. Relación con JudoAdministración

JudoCombates **no** enlaza con ningún ensamblado de JudoAdministración: los dos repositorios son
independientes. Lo que comparten son los contratos de la API —las clases de `JudoCombates.Shared`, que
son una copia de las del servidor— y eso es código del mismo titular, bajo la misma licencia. Ver
[README.md](README.md), sección «Relación con JudoAdministración».
