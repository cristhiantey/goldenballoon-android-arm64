# Validación del port Android ARM64

> HISTÓRICO DEL ZIP RECIBIDO, NO DE LA REVISIÓN 2. Lee LEEME_REVISION_2.md para la validación del código nuevo. Los binarios/símbolos antiguos no se incluyen en la nueva entrega de fuentes.

Fecha: 12 de septiembre de 2026. Plataforma anfitriona: Linux x86_64.

## Resultados observados

| Comprobación | Resultado |
|---|---|
| CMake + NDK r28c; fuentes completas del juego y SDL2 | Compilación y enlace correctos |
| Biblioteca `libmain.so` | ELF64, little endian, AArch64 |
| Enlace `--no-undefined` | Correcto |
| Java + recursos Android mediante Gradle 8.9 / AGP 8.7.3 | Correcto |
| `:app:assembleDebug` | BUILD SUCCESSFUL |
| Ruta alternativa SDK directa sin Gradle/Maven | APK generado y firmado |
| D8, aapt2, zipalign y apksigner | Correcto |
| Firma del APK SDK directo | Verificada, esquemas v2 y v3 |
| `libmain.so`, `libSDL2.so`, `libc++_shared.so` en ambos APK | ARM64 y segmentos LOAD alineados a 16 KiB |
| `zipalign -c -P 16 4` del APK entregado | Correcto |
| `tests/check_address_domains.py` del proyecto original | PASS: conversiones controladas en límites de direcciones |
| Test C de adaptación de interpolación afín | PASS, incluye errores de capacidad y correspondencia de variables |
| 12 pares de shaders de combinadores + filtro de salida + mapa de sombras | Compilaron y enlazaron en OpenGL ES 3.2 Mesa 25.2.8 |
| Cuaderno de Colab | JSON válido y sintaxis de celdas Python comprobada |

El APK entregado usa la ruta SDK directa y contiene el mismo código final verificado. SHA-256:

`8e29158f2b2ec4f46d6f1bcedbef1c84b4133e2ea20381244a695653bed74db2`

Los registros abreviados están en `validation/`. Los símbolos de las bibliotecas del APK entregado están en `validation/GoldenBalloon-arm64-symbols.zip`.

## Límites de esta validación

No hubo ROM ni teléfono conectado. No se verificaron carreras, gráficos finales, tiempo de carga, FPS, audio, consumo, temperatura, entrada táctil física, mando, suspensión real, restauración de contexto EGL ni guardados mediante una partida real. Mesa sirve para detectar errores de sintaxis/enlace de GLSL, no certifica un controlador Mali o Adreno. La matriz de 14 pares es representativa, no exhaustiva.

El procedimiento de Colab ejecuta la ruta SDK directa que sí terminó aquí, pero no se lanzó dentro del servicio Colab. No se ejecutaron Docker, GitHub Actions, Windows, macOS ni iOS. No se declara compatibilidad con todas las GPUs o con cualquier entorno sin requisitos.

## Correcciones específicas

- Objetivo Android como biblioteca compartida `libmain.so`, ABI `arm64-v8a`, PIC y enlace de librerías Android.
- SDL2 se compila desde fuentes fijadas; sus clases Java pertenecen a la misma versión.
- Inicio `SDL_main`, proceso separado para el juego, importación de documentos y validador de ROM original por JNI.
- Eliminación de la dependencia del loader OpenGL de escritorio en Android; contexto ES 3.2 y GLSL ES 3.20.
- Interpolación afín mediante variar `atributo*w` y recuperar con `gl_FragCoord.w`; variables de vertex/fragment coinciden y no requieren `noperspective`.
- Precisión explícita alta; conversión explícita `float(...)` de las constantes del mezclado de texturas.
- Depth clamp sólo cuando la extensión ES está disponible; buffer sin color explícito para framebuffer de sombras.
- Android API 26 usa `posix_memalign` para la arena. Lanzar procesos auxiliares de escritorio devuelve `ENOSYS` en Android.
- Android conserva su mecanismo de informes de cierre nativo en lugar de depender de `execinfo.h`.

## Reproducir las pruebas

Desde la raíz del proyecto, en Linux con compilador C:

```bash
cc -std=c11 -Wall -Wextra -Werror -Iplatform/fast3d \
  tests/test_android_gles_shader.c platform/fast3d/gfx_gles_shader.c \
  -o /tmp/test_android_gles
/tmp/test_android_gles
python3 tests/check_address_domains.py
python3 tools/android/verify_apk.py out/GoldenBalloon-arm64-debug.apk
```

Para los shaders del generador real, se necesita EGL/Mesa ES 3.2, compilador C y SDL2 preparado con `--prepare`:

```bash
export ANDROID_NDK_HOME=/ruta/Android/Sdk/ndk/28.2.13676358
python3 tests/check_android_gles_driver.py
```

Las referencias oficiales de las interfaces usadas aparecen al final de `LEEME_ANDROID_ARM64.md`.
