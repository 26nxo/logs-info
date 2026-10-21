# Logs de Minecraft y Lunar Client

Referencia práctica de ubicaciones, nombres, formato y diagnóstico de logs.

Convenciones:

- `%APPDATA%` = `C:\Users\<usuario>\AppData\Roaming`
- `%LOCALAPPDATA%` = `C:\Users\<usuario>\AppData\Local`
- `%USERPROFILE%` = `C:\Users\<usuario>`
- `<perfil>` = nombre del perfil de Lunar Client (por ejemplo `1.8`, `1.21`, `vanilla-1.8`)
- `<PID>` = ID del proceso de Java

## 1. Resumen rápido

| Qué buscas | Minecraft Java / vanilla | Lunar Client |
|---|---|---|
| Sesión actual | `%APPDATA%\.minecraft\logs\latest.log` | `%USERPROFILE%\.lunarclient\profiles\<perfil>\logs\latest.log` |
| Sesiones anteriores | `%APPDATA%\.minecraft\logs\YYYY-MM-DD-N.log.gz` | `%USERPROFILE%\.lunarclient\profiles\<perfil>\logs\YYYY-MM-DD-N.log.gz` |
| Crash del juego | `%APPDATA%\.minecraft\crash-reports\crash-...-client.txt` | `%USERPROFILE%\.lunarclient\profiles\<perfil>\crash-reports\` o `%APPDATA%\.minecraft\crash-reports\` |
| Crash de la JVM | `%APPDATA%\.minecraft\hs_err_pid<PID>.log` | `%USERPROFILE%\.lunarclient\profiles\<perfil>\hs_err_pid<PID>.log` o `%APPDATA%\.minecraft\hs_err_pid<PID>.log` |
| Log del launcher | `%APPDATA%\.minecraft\launcher_log.txt` | `%USERPROFILE%\.lunarclient\logs\launcher\main.log` |
| Servidor | `<carpeta-del-servidor>\logs\latest.log` | No aplica |

Equivalentes en macOS y Linux:

| Cosa | macOS | Linux |
|---|---|---|
| Carpeta de Minecraft | `~/Library/Application Support/minecraft` | `~/.minecraft` |
| Carpeta de Lunar | `~/.lunarclient` | `~/.lunarclient` |
| Logs de Minecraft | `.../minecraft/logs/latest.log` | `~/.minecraft/logs/latest.log` |
| Logs de Lunar | `~/.lunarclient/profiles/<perfil>/logs/latest.log` | `~/.lunarclient/profiles/<perfil>/logs/latest.log` |

Si usas un directorio de juego personalizado en una instalación de Minecraft, todos los paths de esa instalación se mueven a ese directorio.

## 2. Minecraft Java Edition

### 2.1 Carpeta de juego

Por defecto:

- Windows: `%APPDATA%\.minecraft`
- macOS: `~/Library/Application Support/minecraft`
- Linux: `~/.minecraft`

Dentro de esa carpeta están `logs`, `crash-reports`, `saves`, `mods`, `config`, `resourcepacks`, etc.

En un servidor, la "carpeta de juego" es la carpeta donde está el `server.jar` y desde donde se ejecuta el servidor.

### 2.2 Archivos que te vas a encontrar

| Archivo | Ubicación | Qué es | Se sobrescribe |
|---|---|---|---|
| `latest.log` | `logs\` | Sesión actual o última sesión. Texto plano. | Sí, en el siguiente arranque |
| `YYYY-MM-DD-N.log.gz` | `logs\` | Sesiones anteriores comprimidas con gzip. `N` empieza en 1 y sube por cada arranque del día. | No |
| `crash-YYYY-MM-DD_HH.MM.SS-client.txt` | `crash-reports\` | Crash report del cliente generado por Minecraft. | No |
| `crash-YYYY-MM-DD_HH.MM.SS-server.txt` | `crash-reports\` | Crash report del servidor. | No |
| `hs_err_pid<PID>.log` | Carpeta de juego o de trabajo | Crash fatal de la JVM o de código nativo. No es un crash report de Minecraft. | No |
| `hs_err_pid<PID>.jfr` | Junto al `hs_err` o en `jfr\` | Grabación de Java Flight Recorder del crash, si está activada. | No |
| `launcher_log.txt` | Carpeta de juego | Log del launcher oficial. Incluye la salida del juego y errores de descarga/arranque. | Se regenera |
| `debug\` | Carpeta del servidor | Informes de profiling creados con `/debug start` y `/debug stop`. No es un log de errores. | No |

Notas:

- **Copia `latest.log` antes de volver a abrir el juego**: se sobrescribe.
- Cierra el juego antes de copiar o abrir el archivo; mientras corre puede estar bloqueado.
- Los `.gz` no se abren con Notepad. Usa 7-Zip/WinRAR o `gzip -dc` en macOS/Linux.
- `latest.log` empieza después de que el juego arranca; si el fallo ocurre antes de la pantalla de título, el archivo que lo cubre es `launcher_log.txt` o el log de la JVM.

### 2.3 Formato de una línea y niveles

Formato típico:

```text
[18:39:12] [Client thread/INFO]: Setting user: <usuario>
[18:41:56] [Client thread/WARN]: [LC] Took longer than 100ms to shut down background scheduler!
[18:41:56] [Client thread/INFO]: Stopping!
```

En versiones modernas cambian los nombres de los hilos:

```text
[22:42:30] [main/WARN]: Force-disabling mixin 'features.render.world.sky.LevelRendererMixin'
[22:43:26] [Render thread/WARN]: [LC] Couldn't find dev cosmetics file: lunar:dev_cosmetics.json
```

Estructura:

```text
[HH:mm:ss] [Hilo/NIVEL]: mensaje
```

| Nivel | Significado | ¿Es un crash? |
|---|---|---|
| `TRACE` / `DEBUG` | Detalle interno, normalmente solo para desarrollo. | No |
| `INFO` | Funcionamiento normal. | No |
| `WARN` | Algo inesperado o recuperable. Puede ser ruido o el aviso previo a un fallo. | No necesariamente |
| `ERROR` | Una operación falló. Puede ser inofensivo o el síntoma principal. | No necesariamente |
| `FATAL` | Error no recuperable. | Sí, o está a punto de serlo |

`[STDOUT]` no es un nivel: es salida cruda de un mod, de Lunar u otro componente. `[LC]`, `[IchorPipeline]` y `[OptiFine]` son prefijos de subsistemas.

Qué buscar en un log:

- `ERROR`, `FATAL`, `Exception`, `Caused by`, `Suppressed`
- `Mixin`, `OutOfMemory`, `GLFW`, `Pixel format`, `UnsupportedClassVersion`
- `NoClassDefFound`, `NoSuchMethod`, `DuplicateMods`, `Missing or unsupported`
- `Ticking`, `Watching Server`, `crash`

Cómo leer una excepción:

1. La primera línea es el fallo inmediato: `java.lang.NullPointerException: ...`.
2. Las líneas con `at` son el stack trace.
3. `Caused by:` encadena la causa real. La última causa de la cadena suele ser la raíz.
4. `Suppressed:` son errores secundarios.
5. `... 13 more` significa que se omitieron frames repetidos, no que falte información importante.
6. El primer frame que no sea `java.*`, `net.minecraft.*` o `sun.*` suele apuntar al mod, plugin o subsistema responsable.

### 2.4 Anatomía de un crash report

Cabecera típica:

```text
---- Minecraft Crash Report ----
// <comentario aleatorio>

Time: 2026-10-21 18:32
Description: Ticking entity

java.lang.NoClassDefFoundError: dev/basic/bridge/PacketBridge
    at Genesis//net.minecraft.network.NetworkManager.sendPacket(SourceFile)
    at Genesis//net.minecraft.client.network.NetHandlerPlayClient.addToSendQueue(SourceFile:498)
    ...

Caused by: java.lang.ClassNotFoundException: Failed to find class dev.basic.bridge.PacketBridge
    at ...
    ... 13 more

A detailed walkthrough of the error, its code path and all known details is as follows:
---------------------------------------------------------------------------------------

-- Head --
Stacktrace:
    ...

-- Entity being ticked --
Details:
    Entity Type: ...
    Entity ID: ...
    Entity's Exact location: ...

-- Affected level --
Details:
    Level name: ...
    Level seed: ...

-- System Details --
Details:
    Minecraft Version: ...
    Operating System: ...
    Java Version: ...
    Memory: ...
    Launcher: ...
    Is Modded: ...
    ...
```

Secciones:

| Sección | Qué aporta |
|---|---|
| `Time` | Fecha y hora local del crash. |
| `Description` | Qué estaba haciendo el juego: arrancar, cargar mundo, tickear una entidad, renderizar, etc. |
| Primera excepción | Fallo inmediato. |
| `Caused by` | Cadena de causas; la última suele ser la raíz. |
| `-- Head --` | Stack trace del punto exacto del fallo. |
| `-- Affected level --` / `-- Entity being ticked --` | Mundo, entidad, coordenadas y estado. |
| `-- System Details --` | Versión de Minecraft, Java, RAM, GPU/OpenGL, launcher, mods, resource packs. |
| `Is Modded` | Indica si el cliente está modificado y qué marca usa. En Lunar aparece `lunarclient:v...`. |

`Description` más habituales:

| Description | Significado |
|---|---|
| `Initializing game` | Fallo al arrancar: mods, configuración, Java, recursos. |
| `Loading world` / `Generating world` | Fallo al cargar o generar un mundo. |
| `Ticking entity` | Una entidad lanzó una excepción durante un tick. |
| `Ticking block entity` | Un bloque con entidad (cofre, horno, spawner, etc.) falló. |
| `Rendering overlay` / `Rendering screen` | Fallo de renderizado o de una interfaz/mod de GUI. |
| `Exception in server tick loop` | Fallo del tick del servidor: plugin, mod o mundo. |
| `Watching Server` | El watchdog del servidor lo mató por no responder. |
| `Manually triggered debug crash` | Crash provocado con `F3 + C`. |

El texto que muestra el launcher, tipo `The game crashed whilst ticking entity`, es un resumen; no es la causa. La causa está en el crash report.

### 2.5 Errores comunes

| Firma en el log o crash report | Causa probable | Qué revisar |
|---|---|---|
| `java.lang.OutOfMemoryError: Java heap space` | RAM insuficiente o fuga de memoria. | Subir `-Xmx`, cerrar programas, revisar mods. No asignes más del 50-60 % de la RAM del sistema. |
| `OutOfMemoryError: Metaspace` | Demasiadas clases/mods o Java incorrecto. | Bajar mods, revisar versión de Java. |
| `OutOfMemoryError: Direct buffer memory` | Uso excesivo de memoria nativa. | Mods de renderizado, drivers, `-Xmx`. |
| `Pixel format not accelerated` | El driver de GPU no ofrece OpenGL adecuado. | Actualizar driver, forzar GPU dedicada, probar en ventana. |
| `GLFW error 65542` | WGL/OpenGL no disponible. | Driver de GPU, GPU integrada vs dedicada. |
| `Failed to set display mode` | Fullscreen o resolución incompatible. | Arrancar en ventana, revisar `options.txt`. |
| `UnsupportedClassVersionError` | Java demasiado viejo o demasiado nuevo para esa versión/mods. | Usar el Java que pide la versión (17, 21 o 25 según el caso). |
| `NoClassDefFoundError` / `ClassNotFoundException` | Falta una librería, un mod o un archivo; instalación incompleta. | Revisar `mods`, dependencias, reinstalar el perfil. |
| `NoSuchMethodError` / `NoSuchFieldError` | Mod o API incompatible con esa versión. | Alinear versiones de mods, loader y Minecraft. |
| `Mixin apply failed` / `MixinTransformerError` / `InvalidMixinException` | Conflicto de mixins entre mods. | Quitar el mod más reciente, revisar el crash report. |
| `DuplicateModsFoundException` | Dos copias del mismo mod. | Limpiar `mods`. |
| `Missing or unsupported mandatory dependencies` | Falta una dependencia obligatoria o tiene versión incorrecta. | Instalar la dependencia que indica el log. |
| `java.lang.StackOverflowError` | Recursión infinita. | Mod, renderizado o mundo; revisar el stack trace. |
| `Ticking entity` / `Ticking block entity` | Entidad, bloque o mod falló durante un tick. | Coordenadas del crash report, mods recientes, backup del mundo. |
| `Exception in server tick loop` | Plugin, mod o mundo del servidor. | Logs del servidor, plugins, backups. |
| `Watching Server` | El servidor se colgó y el watchdog lo terminó. | TPS, lag, chunks, plugins, RAM. |
| `java.lang.VerifyError` / `ClassFormatError` | Jar corrupto o incompatible. | Reinstalar el mod o el perfil. |
| `EXCEPTION_ACCESS_VIOLATION` con `nvoglv64.dll`, `atio6axx.dll` o `ig9icd64.dll` | Crash nativo del driver de GPU. | Actualizar/reinstalar driver, probar otra versión. |

Ruido que no debería confundirse con un crash:

- `Couldn't initialize twitch stream`: normalmente inofensivo.
- `Unable to parse metadata from ...png`: resource pack con metadatos raros; suele ser recuperable.
- `Took longer than 100ms to shut down background scheduler`: aviso de apagado.
- `[IchorPipeline] MappingSet ... has 0 field mappings`: aviso interno de Lunar; no es un crash por sí solo.
- `Force-disabling mixin ...`: un mod desactiva un mixin de otro; puede ser normal.

### 2.6 Códigos de salida

Windows usa códigos NTSTATUS de 32 bits. Algunos launchers los muestran con signo (`-1073741819`) y otros sin signo (`3221225477`). Son el mismo valor:

```text
3221225477 = 0xC0000005 = -1073741819
```

| Código con signo | Sin signo / hex | Significado | Causa típica |
|---|---|---|---|
| `0` | `0x0` | Salida normal | Cierre correcto |
| `1` | `0x1` | Fallo genérico | Crash de Minecraft, mod, configuración |
| `-1` | `4294967295` / `0xFFFFFFFF` | Fallo genérico | Cierre anómalo sin más detalle |
| `-1073741819` | `3221225477` / `0xC0000005` | Access violation | Driver de GPU, código nativo, mod con librería nativa |
| `-1073741571` | `3221225725` / `0xC00000FD` | Stack overflow | Recursión infinita o stack agotado |
| `-1073741510` | `3221225786` / `0xC000013A` | Proceso terminado | Ctrl+C, consola cerrada o terminación externa |
| `-1073741701` | `3221225595` / `0xC000007B` | Invalid image format | DLL de 32/64 bits incorrecta |
| `-1073741676` | `3221225620` / `0xC0000094` | Integer divide by zero | Fallo nativo/matemático |
| `-1073740940` | `3221226356` / `0xC0000374` | Heap corruption | Memoria nativa corrupta, driver o mod |
| `-805306369` | `3489660927` / `0xCFFFFFFF` | Sin memoria | RAM insuficiente o fuga |
| `139` | `0x8B` | SIGSEGV | Crash nativo en Linux/macOS |
| `134` | `0x86` | SIGABRT | Abort de la JVM o de una librería nativa |
| `255` | `0xFF` | Fallback | En macOS/Linux cuando el estado supera 255 |

El código de salida orienta, pero no identifica la causa. Correlaciónalo con `latest.log`, el crash report y, si existe, `hs_err_pid`.

Para convertir a hex en PowerShell:

```powershell
'{0:X8}' -f [uint32]3221225477
```

### 2.7 Si no hay crash report

Un crash report solo se genera cuando Minecraft atrapa una excepción de Java. Si no hay `crash-reports`, busca:

- `hs_err_pid<PID>.log` en la carpeta de juego. Cabecera: `# A fatal error has been detected by the Java Runtime Environment:`.
- `hs_err_pid<PID>.jfr` o `jfr\`, si Java Flight Recorder estaba activo.
- `launcher_log.txt`, sobre todo si el juego ni siquiera llegó a arrancar.
- El Visor de eventos de Windows (`eventvwr.msc` > Registros de Windows > Aplicación), buscando `Application Error`, `Java` o `Minecraft`.
- `%LOCALAPPDATA%\CrashDumps`, si Windows está configurado para guardar volcados.
- `%LOCALAPPDATA%\Microsoft\Windows\WER\ReportArchive`, donde Windows guarda informes de errores.

En un `hs_err` mira:

- `EXCEPTION_ACCESS_VIOLATION`, `EXCEPTION_STACK_OVERFLOW` u otra excepción.
- `Problematic frame`: módulo o función nativa donde murió.
- `Current thread` y el stack.
- `Command Line`: qué se estaba ejecutando y con qué argumentos.
- `Host`, `Time`, memoria y versión de Java.

## 3. Servidores de Minecraft

En la carpeta del servidor:

| Archivo | Qué es |
|---|---|
| `logs\latest.log` | Sesión actual o última sesión. |
| `logs\YYYY-MM-DD-N.log.gz` | Sesiones anteriores comprimidas. |
| `crash-reports\crash-...-server.txt` | Crash report del servidor. |
| `hs_err_pid<PID>.log` | Crash fatal de la JVM del servidor. |
| `debug\` | Informes de profiling del servidor, si se usó `/debug`. |

Formato:

```text
[12:34:56] [Server thread/INFO]: Starting minecraft server version ...
[12:34:58] [Server thread/WARN]: ...
[12:35:01] [Server thread/ERROR]: ...
```

Cosas típicas de servidor:

- `Exception in server tick loop`: un plugin, mod o mundo lanzó una excepción.
- `Watching Server`: el watchdog mató el servidor por un cuelgue.
- `Ticking entity` / `Ticking block entity`: misma lógica que en cliente, pero en el mundo del servidor.
- `Could not load 'plugins/...jar'`: plugin incompatible o corrupto.
- `FAILED TO BIND TO PORT`: el puerto ya está en uso.
- `java.lang.OutOfMemoryError`: RAM insuficiente; revisar `-Xmx` y plugins.

Copia `latest.log` antes de reiniciar el servidor, porque se sobrescribe.

## 4. Lunar Client

### 4.1 Estructura de directorios

Windows:

```text
%USERPROFILE%\.lunarclient\
├─ logs\
│  └─ launcher\
│     ├─ main.log
│     ├─ main.old.log
│     ├─ auth.log
│     ├─ gameipc.log
│     ├─ updater.log
│     ├─ updater.old.log
│     ├─ window.log
│     ├─ assetserver.log
│     ├─ checkout.log
│     ├─ promotions.log
│     ├─ styngr.log
│     └─ overwolf.log
├─ profiles\
│  └─ <perfil>\
│     ├─ logs\
│     │  ├─ latest.log
│     │  ├─ YYYY-MM-DD-N.log.gz
│     │  ├─ ichor-boot.log
│     │  ├─ config.xml
│     │  └─ telemetry\
│     ├─ crash-reports\
│     │  └─ crash-...-client.txt
│     ├─ hs_err_pid<PID>.log
│     ├─ hs_err_pid<PID>.jfr
│     └─ mods\, config\, saves\, ...
├─ offline\
│  └─ multiver\          <- natives, classpath y runtime interno
├─ jre\                  <- Java incluido por Lunar
├─ game-cache\
├─ launcher-cache\
└─ shared\
```

No todos los perfiles tienen la misma estructura. Lunar puede usar dos layouts:

- **Perfil aislado**: todo el juego vive en `profiles\<perfil>`; ahí están `logs`, `crash-reports`, `hs_err_pid`, `options.txt`, `mods`, etc.
- **Perfil que comparte `.minecraft`**: los logs siguen en `profiles\<perfil>\logs`, pero los crash reports y los `hs_err_pid` pueden caer en `%APPDATA%\.minecraft`.

Por eso, si no encuentras el crash report en el perfil, revisa también `%APPDATA%\.minecraft\crash-reports` y `%APPDATA%\.minecraft\hs_err_pid*.log`.

macOS/Linux:

```text
~/.lunarclient/logs/launcher/main.log
~/.lunarclient/profiles/<perfil>/logs/latest.log
~/.lunarclient/profiles/<perfil>/crash-reports/
```

`%APPDATA%\lunarclient` (Windows) es la carpeta de datos de Electron: `Cache`, `Code Cache`, `Crashpad`, `sentry`, `Local Storage`, `Preferences`, etc. No es donde están los logs del juego. `Crashpad` y `sentry` pueden contener información de crashes del launcher.

### 4.2 Logs del launcher

| Archivo | Contenido |
|---|---|
| `main.log` | Ciclo de vida del launcher, actualizaciones, sesiones de launch y código de salida de la JVM. Es el log principal. |
| `main.old.log` | Rotación del `main.log` anterior. |
| `auth.log` | Autenticación y cuenta. **Puede contener datos sensibles; no lo compartas sin redactar.** |
| `gameipc.log` | Comunicación entre launcher y juego: handshake, servicios, cierre del WebSocket. |
| `updater.log` / `updater.old.log` | Comprobación y descarga de actualizaciones. |
| `window.log` | Eventos de ventana/UI del launcher. |
| `assetserver.log` | Servidor local de assets. |
| `checkout.log`, `promotions.log`, `styngr.log` | Módulos concretos (tienda, promociones, música). |
| `overwolf.log` | Solo si Overwolf está presente. |

Formato del launcher:

```text
[2026-10-20 22:57:47.027] [info]  [GameIPC] Client connection has closed. Launch ID: <uuid>, Code: 1006, Reason:
[2026-10-20 22:57:48.241] [info]  [Launch] JVM for launch session <uuid> has exited...
[2026-10-20 22:57:48.242] [info]           * Code: 3221225477
```

Estructura:

```text
[YYYY-MM-DD HH:mm:ss.SSS] [nivel] [Módulo] mensaje
```

Niveles: `debug`, `info`, `warn`, `error`. Son minúsculas y no son los mismos que los niveles de Minecraft.

### 4.3 Logs del juego

| Archivo | Contenido |
|---|---|
| `latest.log` | Sesión actual o última sesión del perfil. |
| `YYYY-MM-DD-N.log.gz` | Sesiones anteriores comprimidas. |
| `ichor-boot.log` | Arranque de Ichor/Genesis, transformación de clases y mappings. Útil si el juego falla antes de `latest.log`. |
| `config.xml` | Configuración de log4j usada por el perfil. No es un log. |
| `telemetry\` | Datos de telemetría, no diagnóstico. |
| `blclient\` | Si aparece, son logs de Badlion Client (BLC), no de Lunar. No los uses para diagnosticar Lunar. |

Formato del juego:

```text
[18:39:12] [Client thread/INFO]: Setting user: <usuario>
[18:39:15] [Client thread/INFO]: [STDOUT]: [18:39:15] [IchorPipeline/Client thread/WARN] MappingSet ... has 0 field mappings.
[18:39:19] [Client thread/ERROR]: Couldn't initialize twitch stream
[18:41:56] [Client thread/INFO]: Stopping!
```

En versiones modernas:

```text
[22:42:30] [main/WARN]: Force-disabling mixin 'features.render.world.sky.LevelRendererMixin'
[22:43:26] [Render thread/WARN]: [LC] Couldn't find dev cosmetics file: lunar:dev_cosmetics.json
```

Prefijos propios de Lunar:

| Prefijo / marca | Significado |
|---|---|
| `[LC]` | Mensaje de Lunar Client. |
| `[IchorPipeline]` | Pipeline de transformación/mappings de Lunar. |
| `[LC IPC]` / `[LC Assets]` | Comunicación interna y assets de Lunar. |
| `Genesis//` | Prefijo de stack traces del bootstrap Genesis de Lunar. |
| `com.moonsworth.lunar.*` | Clases internas de Lunar. |
| `lunarclient:v...` | Client brand que aparece en el crash report. |
| `LUNARCLIENT_STATUS_INIT` | Marca de inicialización de Lunar. |

### 4.4 Crash reports y crashes de JVM

Los crash reports de Lunar usan el mismo formato que Minecraft:

```text
---- Minecraft Crash Report ----
// <comentario>

Time: 2026-10-21 18:32
Description: Ticking entity

java.lang.NoClassDefFoundError: dev/basic/bridge/PacketBridge
    at Genesis//net.minecraft.network.NetworkManager.sendPacket(SourceFile)
    ...
Caused by: java.lang.ClassNotFoundException: Failed to find class dev.basic.bridge.PacketBridge
    ...

-- System Details --
Details:
    Minecraft Version: 1.8.9
    Java Version: 17.0.18, Azul Systems, Inc.
    Launched Version: 1.8.9
    Is Modded: Definitely; Client brand changed to 'lunarclient:v<version>'
    OptiFine Version: ...
```

Cómo reconocer que es de Lunar:

- `Is Modded: Definitely; Client brand changed to 'lunarclient:v...'`.
- Stack traces con `Genesis//`.
- Paquetes `com.moonsworth.lunar.*`.
- En 1.8 suele aparecer OptiFine; en versiones modernas pueden aparecer Sodium, Iris y mixins de Fabric.
- El crash report puede estar en `profiles\<perfil>\crash-reports` o en `%APPDATA%\.minecraft\crash-reports`.

No confíes solo en el nombre de la carpeta del perfil. Un `profiles\1.21` puede contener un crash report de 1.8.9 si el perfil se reutilizó. Verifica `Minecraft Version`, `Launched Version`, `Is Modded` y la fecha.

Si no hay crash report, busca `hs_err_pid<PID>.log`. Cabecera:

```text
#
# A fatal error has been detected by the Java Runtime Environment:
#
#  EXCEPTION_ACCESS_VIOLATION (0xc0000005) at pc=...
# Problematic frame:
# C  ...
```

El `hs_err` de Lunar puede estar en `profiles\<perfil>` o en `%APPDATA%\.minecraft`, según el layout del perfil. El `Command Line` del `hs_err` suele incluir `-Dichor.logsFile=...\profiles\<perfil>\logs\ichor-boot.log` y `--gameDir`, lo que ayuda a saber qué perfil estaba activo.

### 4.5 Cómo abrir y compartir logs

Desde el launcher:

1. Abre Lunar Client.
2. Ve a `Settings` (abajo a la izquierda).
3. Pulsa `Open Logs`. En Windows, los logs del launcher están en `%USERPROFILE%\.lunarclient\logs\launcher`.
4. Para los logs del juego, abre `%USERPROFILE%\.lunarclient\profiles\<perfil>\logs`.
5. Para crash reports, abre `...\profiles\<perfil>\crash-reports` y revisa también `%APPDATA%\.minecraft\crash-reports`.

Para subir logs a soporte:

1. Inicia sesión en el launcher.
2. Abre el menú `Lunar Client` (arriba a la izquierda).
3. Selecciona `Upload Logs`.
4. El launcher mostrará un `Log ID`.
5. Comparte ese ID solo con el staff que lo haya pedido.

`Upload Logs` sube los logs al servicio de Lunar; no sustituye el crash report local. Úsalo solo cuando soporte lo solicite.

### 4.6 Flujo de diagnóstico

| Síntoma | Primer archivo | Después |
|---|---|---|
| El launcher no abre o no actualiza | `logs\launcher\main.log`, `updater.log` | `main.old.log`, `Crashpad`, `sentry`, Visor de eventos |
| El launcher abre, pero el juego no inicia | `logs\launcher\main.log` (código de salida), `gameipc.log` | `profiles\<perfil>\logs\latest.log`, `ichor-boot.log` |
| El juego crashea con pantalla de error | `profiles\<perfil>\crash-reports\...` | `latest.log`, `%APPDATA%\.minecraft\crash-reports` |
| El juego se cierra sin crash report | `hs_err_pid<PID>.log` en el perfil o en `.minecraft` | `.jfr`, Visor de eventos, `CrashDumps` |
| El juego va lento o tira warnings | `profiles\<perfil>\logs\latest.log` | Buscar `WARN`/`ERROR`, drivers, mods |
| Problema de login/cuenta | `logs\launcher\auth.log` | Redactar antes de compartir |
| Chat | `profiles\<perfil>\logs\blclient\chat\latest.log` si existe | No es un log estándar de Lunar; puede no existir |
| Servidor | `logs\latest.log` | `crash-reports\...-server.txt`, `hs_err_pid` |

Códigos en el launcher:

- `[Launch] JVM ... has exited ... * Code: 0`: cierre normal.
- `* Code: 1` o `* Code: 4294967295`: fallo genérico.
- `* Code: 3221225477`: access violation (0xC0000005).
- `[GameIPC] ... Code: 1000`: cierre normal del WebSocket entre launcher y juego.
- `[GameIPC] ... Code: 1006`: cierre anómalo del WebSocket; suele coincidir con un crash o cierre brusco del juego.
- Los códigos `1000`/`1006` de `gameipc.log` **no** son el código de salida de la JVM.

## 5. Identificación rápida por firma

| Archivo / tipo | Firma que lo delata | Qué es |
|---|---|---|
| Log de juego Java | `[HH:mm:ss] [Hilo/NIVEL]:` | `latest.log` de Minecraft o de un perfil de Lunar |
| Crash report | `---- Minecraft Crash Report ----` | Crash atrapado por Minecraft |
| Crash report de servidor | `---- Minecraft Crash Report ----` y nombre `...-server.txt` | Crash del servidor |
| Crash de JVM | `# A fatal error has been detected by the Java Runtime Environment:` | `hs_err_pid<PID>.log` |
| Launcher oficial | `launcher_log.txt` | Log del launcher de Mojang |
| Launcher Lunar | `[YYYY-MM-DD HH:mm:ss.SSS] [info] [Módulo]` | `main.log` de Lunar |
| Bootstrap Lunar | `[HH:mm:ss] [Genesis/main/INFO]` | `ichor-boot.log` |
| Crash report Lunar | `Client brand changed to 'lunarclient:v...'` | Crash report generado por Lunar |
| Servidor | `[Server thread/INFO]` | `logs/latest.log` del servidor |

Pistas de sesión:

- Un log empieza con `Setting user: ...` y termina con `Stopping!` si el cierre fue limpio.
- Si termina abruptamente, sin `Stopping!`, probablemente hubo crash o kill.
- `latest.log` no lleva fecha en cada línea; usa la fecha de modificación del archivo y los timestamps.
- Los `.log.gz` conservan el orden por fecha en el nombre.
- El crash report sí lleva `Time:` y `Description:`.

## 6. Privacidad: qué redactar antes de compartir

Los logs pueden contener datos personales. Revisa y redacta:

- `--accessToken ...` en el `Command Line` de `hs_err_pid*.log`. **Puede ser un token de sesión válido.**
- `--username`, `--uuid`, `--xuid` y `--launchId`.
- Correos, tokens o datos de cuenta en `auth.log`.
- IPs/dominios de servidores y mensajes de chat.
- Rutas con el nombre de usuario de Windows.
- Coordenadas, seed del mundo y lista de mods en crash reports.

Recomendación: para pedir ayuda, comparte `latest.log` + el crash report más reciente + `main.log`. Si el problema es un crash nativo, comparte `hs_err_pid` **redactado**. No compartas `auth.log` salvo que sea imprescindible y siempre con los tokens eliminados.

## 7. Comandos útiles

Windows PowerShell:

```powershell
# Abrir carpetas
explorer "$env:APPDATA\.minecraft\logs"
explorer "$env:APPDATA\.minecraft\crash-reports"
explorer "$env:USERPROFILE\.lunarclient\logs\launcher"
explorer "$env:USERPROFILE\.lunarclient\profiles"

# Últimas líneas del log
Get-Content "$env:APPDATA\.minecraft\logs\latest.log" -Tail 100

# Buscar errores
Select-String -Path "$env:APPDATA\.minecraft\logs\latest.log" -Pattern "ERROR|FATAL|Exception|Caused by|Mixin|OutOfMemory"

# Ver la fecha y el tamaño de los logs
Get-ChildItem "$env:APPDATA\.minecraft\logs" | Sort-Object LastWriteTime -Descending | Select-Object -First 10
```

macOS/Linux:

```bash
# Últimas líneas
tail -n 100 ~/.minecraft/logs/latest.log

# Buscar errores
grep -E "ERROR|FATAL|Exception|Caused by|Mixin|OutOfMemory" ~/.minecraft/logs/latest.log

# Leer un log comprimido
gzip -dc ~/.minecraft/logs/2026-10-21-1.log.gz | less

# Logs de Lunar
tail -n 100 ~/.lunarclient/profiles/1.8/logs/latest.log
```

## 8. Checklist rápido

1. Cierra el juego antes de copiar logs.
2. Copia `latest.log` antes de volver a abrirlo.
3. Busca el crash report más reciente por fecha y hora.
4. Si no hay crash report, busca `hs_err_pid*.log`.
5. Lee `Description`, la primera excepción y el último `Caused by`.
6. Identifica el primer frame que no sea Java/Minecraft.
7. Revisa `System Details`: Java, RAM, GPU, mods, client brand.
8. En Lunar, confirma el perfil activo y revisa `main.log` para el código de salida.
9. Redacta tokens, usuario, UUID, IPs y chat antes de compartir.
10. Si el problema es del launcher, revisa `main.log`/`launcher_log.txt` antes que el log del juego.

## 9. Fuentes

- Minecraft Wiki — Crash: <https://minecraft.wiki/w/Crash>
- Minecraft Wiki — Tutorial: Obtaining a crash report: <https://minecraft.wiki/w/Tutorial:Obtaining_a_crash_report>
- MinecraftHopper — Launcher log: <https://minecrafthopper.net/help/guides/getting-minecraft-launcher-log/>
- MinecraftHopper — Latest log: <https://minecrafthopper.net/help/guides/getting-minecraft-latest-log/>
- MinecraftHopper — Crash report: <https://minecrafthopper.net/help/guides/getting-minecraft-crash-report/>
- MinecraftHopper — JVM crash report: <https://minecrafthopper.net/help/guides/getting-minecraft-jvm-crash-report/>
- MinecraftHopper — Exit statuses: <https://minecrafthopper.net/help/exit-code/>
- Lunar Client — How to get Lunar Client Support: <https://www.lunarclient.com/news/how-to-get-lunar-client-support>
- Lunar Client Support — Uploading Launcher Logs: <https://support.lunarclient.com/solutions/uploading-launcher-logs>
- Lunar Client Support — Pixel Format Launch Fail: <https://support.lunarclient.com/solutions/pixel-format-launch-fail>
- Lunar Client Support — Failed to Extract Natives: <https://support.lunarclient.com/solutions/failed-to-extract-natives>
- Lunar Client Support — Failed to Launch in Fullscreen: <https://support.lunarclient.com/solutions/failed-to-launch-in-fullscreen>

Rutas y firmas verificadas en una instalación Windows con Lunar Client 3.7.25 y perfiles 1.8/1.21. Los nombres exactos pueden variar entre versiones.
