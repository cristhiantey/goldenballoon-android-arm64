# Golden Balloon para Android ARM64

> Guía histórica del port base. Para la revisión 2 lee [LEEME_REVISION_2.md](LEEME_REVISION_2.md). Los resultados de compilación de esta guía NO validan el código nuevo. Usa Python 3.12+ para extraer SDL de forma segura.

Este paquete adapta el proyecto original del ZIP a Android. Compila el código C/C++ del juego a AArch64 con el NDK y lo integra en una aplicación Android con SDL2 y OpenGL ES 3.2. No ejecuta código MIPS de la ROM: conserva el juego nativo existente y obtiene de tu ROM los recursos gráficos, sonidos y niveles durante la ejecución.

**Estado:** port experimental con compilación y empaquetado verificados. Consulta `VALIDACION_ANDROID.md`. No se ha ejecutado una carrera en un teléfono: la compilación correcta no garantiza ausencia de pantallas negras, fallos de audio o cierres en un controlador concreto.

## Lo que incluye

- Código original del juego, licencias y documentación, con cambios acotados en la plataforma y el renderer.
- App `org.goldenballoon.mobile`, nombre Golden Balloon e icono del proyecto.
- Arquitectura `arm64-v8a`, Android 8.0/API 26 o superior, OpenGL ES 3.2.
- Importación mediante el selector de documentos de Android, sin permisos generales de almacenamiento.
- Validación estricta de tamaño, orden de bytes, revisión, SHA-256 e índices de recursos; reutiliza el validador original.
- Controles multitáctiles: stick, A, B, Z, L, R, Start y cuatro botones C. Los mandos siguen pasando por SDL2.
- Juego horizontal con giro entre ambas orientaciones, pausa/audio de SDL al pasar a segundo plano y reinicio de proceso al volver a jugar.
- Partidas privadas persistentes y exportación ZIP. No hay interfaz de restauración de partidas en esta versión.
- Pantalla inicial con límites de presentación de 30, 60 o 120. Esto es un límite, no una garantía de FPS. Mantiene la simulación original.
- Registro actual y anterior exportables para diagnosticar el arranque.
- Dos rutas de compilación: Gradle/Android Studio y SDK directo sin Gradle/Maven.
- Cuaderno `GoldenBalloon_Colab.ipynb`, Dockerfile y flujo manual de GitHub Actions.

No incluye ROM, recursos extraídos de ROM, conexión multijugador nativa, port iOS, emulador ni backend Vulkan/WebGPU para Android. ARM64 es una arquitectura de CPU: un APK Android no se instala en iOS aunque ambos dispositivos usen ARM64. El soporte de GPU depende además de OpenGL ES y su controlador.

## Compilar en Google Colab

1. Abre Google Colab y selecciona **Archivo → Subir cuaderno**.
2. Sube `GoldenBalloon_Colab.ipynb`, incluido también dentro del ZIP.
3. Usa un entorno CPU. Ejecuta la celda de carga y sube `GoldenBalloon-Android-ARM64.zip`.
4. Ejecuta la celda de compilación. Instala Java 17 y el SDK/NDK; `--install-sdk` acepta las licencias de Android SDK.
5. Ejecuta la celda de descarga para obtener `GoldenBalloon-arm64-debug.apk`.
6. Instala el APK en el teléfono y selecciona tu ROM desde la pantalla inicial.

No subas la ROM a Colab: no se necesita para compilar. La primera instalación de las herramientas necesita Internet y varios GB de espacio. Recomendación práctica: 8 GB de RAM y 10 GB libres. El tiempo depende de la máquina y la conexión; `--jobs 2` limita la concurrencia para equipos modestos.

## Linux, WSL o servidor Google Cloud

Desde la carpeta que contiene este documento:

```bash
sudo apt-get update
sudo apt-get install -y openjdk-17-jdk-headless python3 ca-certificates
export JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64
python3 tools/android/build.py --sdk-only --install-sdk --jobs 2
```

Salida: `out/GoldenBalloon-arm64-debug.apk` y `out/GoldenBalloon-arm64-symbols.zip`.

`--sdk-only` ejecuta CMake, Clang, compilador Java, D8, aapt2, zipalign y apksigner. Comprueba bibliotecas AArch64, alineación ELF de 16 KiB, alineación del APK y firma. La misma ruta se verificó en este entorno Linux, aunque el servicio Colab no se ejecutó directamente.

Para usar un SDK ya instalado:

```bash
python3 tools/android/build.py --sdk-only --sdk /ruta/Android/Sdk --jobs 2
```

Para usar Gradle:

```bash
python3 tools/android/build.py --sdk /ruta/Android/Sdk --jobs 2
```

El script descarga Gradle 8.9 y verifica su SHA-256. No depende de una versión arbitraria de Gradle instalada globalmente. La ruta Gradle necesita acceso a Google Maven/Maven Central durante la primera ejecución. Los proxies de una red corporativa deben configurarse en Gradle según esa red.

## Android Studio, Windows y macOS

Requieren Python 3.10+, JDK 17 y un SDK/NDK que soporte el sistema anfitrión. El instalador automático `--install-sdk` está limitado a Linux x86_64. En Windows/macOS instala estos paquetes con Android Studio:

| Componente | Versión fijada |
|---|---|
| Android SDK Platform | 35 |
| Android Build Tools | 35.0.0 |
| Android NDK | 28.2.13676358 (r28c) |
| CMake | 3.22.1 |
| Gradle | 8.9 |
| Android Gradle Plugin | 8.7.3 |
| Java | 17 |
| SDL2 | 2.32.8 |

Prepara las fuentes de dependencias:

```bash
python3 tools/android/build.py --prepare
```

Abre la carpeta `android` como proyecto de Android Studio. Selecciona JDK 17 y Gradle 8.9. El script Python es la entrada de compilación distribuida; no se incluye un `gradlew` descargado de forma implícita.

Windows PowerShell, con SDK instalado:

```powershell
python tools/android/build.py --sdk-only --sdk "$env:LOCALAPPDATA\Android\Sdk"
```

macOS, con SDK instalado:

```bash
python3 tools/android/build.py --sdk-only --sdk "$HOME/Library/Android/sdk"
```

Estas entradas se prepararon para los anfitriones oficiales del NDK. La compilación se verificó en Linux x86_64, no en Windows/macOS. No se promete compilación directa en Termux/Android ARM64: el NDK distribuido por Google requiere un anfitrión compatible. Para compilar desde el navegador del teléfono, usa Colab.

## Docker y GitHub Actions

Linux Docker:

```bash
docker build --platform linux/amd64 -t goldenballoon-arm64 -f tools/android/Dockerfile .
docker run --rm --platform linux/amd64 -v "$PWD:/project" goldenballoon-arm64
```

En GitHub está preparado `.github/workflows/android-arm64.yml`. Tras subir tú el proyecto a un repositorio, ejecútalo desde Actions → Android ARM64 → Run workflow. Se genera un artefacto descargable; no publica una release. Este flujo y Docker no se ejecutaron en sus servicios durante esta entrega.

## ROM admitida y guardados

Usa una copia propia sin comprimir, de **12 MiB exactos (12 582 912 bytes)**. Acepta `.z64`, `.v64` y `.n64`: el motor normaliza el orden de bytes en memoria.

| Revisión | SHA-256 de la imagen normalizada |
|---|---|
| US 1.1 / `us.v80` | `7de1a8fb2a9558cfc3d9ad4497df698c1e89cf7095ac1531557df2af40ba8bcf` |
| EU 1.1 / `pal.v80` | `584d59412b3a8c675f5569516a0406128028929e31544490a4dbc3ab16a038b9` |

La validación rechaza ROM modificada, dañada o de otra revisión. No se desactivaron estas comprobaciones para ocultar un fallo de arranque. Importar una ROM inválida conserva la ROM válida anterior.

Partidas: almacenamiento interno de la app, subcarpeta `saves`. Usa **Exportar partidas guardadas** antes de desinstalar o borrar datos. La compilación SDK directa mantiene su clave de depuración en `.build-tools/goldenballoon-debug.keystore`; no se entrega esa clave. Conserva tu propia clave para poder instalar tus futuras compilaciones como actualizaciones. La ruta Gradle usa su propia clave de depuración: APK con claves distintas no se pueden actualizar entre sí. Una firma de depuración sirve para pruebas, no es una publicación de producción.

## Diagnóstico si sigue apareciendo pantalla negra

1. Vuelve a la pantalla inicial y pulsa **Compartir registros**. Si se cerró el proceso, el lanzador permanece independiente.
2. Indica modelo exacto del teléfono, versión de Android, GPU, revisión de ROM y momento del fallo.
3. Con ADB, guarda también el informe del cierre nativo:

```bash
adb logcat -b crash -d > android-crash.txt
adb shell dumpsys package org.goldenballoon.mobile > package-info.txt
```

`session.log` contiene validación de ROM, versión del renderer, compilación de shaders y errores del motor. Android debuggerd captura las fallas nativas; no se instala el manejador de señales de escritorio. `out/GoldenBalloon-arm64-symbols.zip` permite simbolizar las direcciones usando `ndk-stack` o `llvm-addr2line` del NDK. Un fallo anterior a `SDL_main` puede aparecer únicamente en Logcat.

No se fuerza un bucle infinito de reintentos gráficos ni se promete solucionar todas las pantallas negras sin medir el dispositivo.

## Cambios y comprobaciones

Consulta `CAMBIOS_ANDROID.patch` para el diff contra el ZIP de entrada y `VALIDACION_ANDROID.md` para los resultados. Las dependencias de fuentes están en `third_party/archives`, con hashes fijados en `tools/android/build.py`. SDK/NDK, cachés de Gradle y binarios temporales no se distribuyen dentro del ZIP de código.

Referencias técnicas: [SDL2 para Android](https://wiki.libsdl.org/SDL2/README-android), [soporte de páginas de 16 KB](https://developer.android.com/guide/practices/page-sizes), [NDK con CMake](https://developer.android.com/ndk/guides/cmake), [OpenGL ES 3.2 / GLSL ES 3.20](https://registry.khronos.org/OpenGL/specs/es/3.2/GLSL_ES_Specification_3.20.pdf).
