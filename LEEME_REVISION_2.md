# Golden Balloon Android ARM64 — revisión 2 experimental
13 de septiembre de 2026

## Estado real

Este ZIP contiene código fuente modificado, no un APK nuevo verificado.
La compilación completa se intentó y se detuvo por falta del SDK/NDK Android.
Las pruebas sin ROM enumeradas abajo sí pasaron. No se ha probado esta revisión
en teléfono, GPU Mali/MediaTek ni mando físico. Los resultados históricos de
VALIDACION_ANDROID.md pertenecen al ZIP original y no validan estos cambios.

## Cambios

- Inicio renovado, configuración básica y opción de usar el teléfono como mando remoto.
- Engranaje dentro del juego; Atrás abre ajustes en vez de salir.
- Menú en español por secciones, explicaciones y «✓ Aplicar y volver al juego».
- Pausa mediante los mecanismos del motor, con liberación de entradas al cerrar.
- Desenfoque de una captura reducida del juego al abrir el menú (API 26+).
  Se procesa una vez y no se guarda. Si PixelCopy falla se conserva el fondo
  oscurecido. Android 12+ añade desenfoque del sistema cuando está disponible.
- FPS medidos desde SDL_GL_SwapWindow, no desde callbacks de interfaz.
- FPS, interpolación, panorámico, resolución, filtros, sombras, FOV, audio,
  cámara, jugabilidad y asignación de botones del mando.
- Ocultar HUD de carrera y minimapa, independientemente de los controles táctiles.
  Sus funciones siguen ejecutándose: solo se descartan sus comandos de dibujo.
- Ocultar, mover, redimensionar y cambiar opacidad de botones táctiles.
- Vibración del teléfono conectada a Rumble Pak de P1 cuando no hay un mando
  físico en esa plaza; SDL conserva la vibración de mandos compatibles.
- Respuesta táctil opcional al pulsar botones.
- Limitador opcional de picos de audio, enlazado entre canales.
- Mando remoto experimental y servidor Python TLS configurable.
- Corrección de extracción de SDL: sus enlaces internos se permiten mediante
  el filtro seguro de Python; no se admiten enlaces que escapen al destino.
- Versión 1.7.0-arm64.2-experimental, versionCode 2 en Gradle y SDK directo.

No se ofrecen opciones de escritorio, herramientas ImGui ni voz sin integración
Android como si funcionasen. Los idiomas originales del juego siguen siendo
los incluidos en la ROM: la interfaz nueva en español no traduce esa ROM.
No se añade descarga/instalación de paquetes de contenido.

## Cómo usarlo

1. Importa tu ROM, inicia el juego y toca ⚙. Si aún está cargando, espera.
2. Expande una sección, cambia valores y pulsa Aplicar. Los ajustes nativos se
   validan y guardan juntos en el siguiente límite seguro de fotograma.
3. Cada control indica «En vivo», «Próxima carrera» o «Reiniciar juego».
4. «✓ Aplicar y volver al juego» confirma y cierra. Un error de validación o
   guardado se muestra en el menú.
5. Los ajustes del teléfono se aplican al cambiarlos y se guardan al cerrar.
   Los del motor requieren Aplicar; Atrás descarta los cambios aún no aplicados.
6. «Mover botones y palanca» permite arrastrarlos. Pulsa ✓ para guardar.
   «Restablecer posiciones táctiles» recupera el diseño.
7. El engranaje sigue accesible al ocultar controles. El botón MODE/guía de
   un mando también abre ajustes si Android entrega ese evento.
8. Salir desde el menú termina el proceso :game y conserva el iniciador.

mobile-settings.json usa escritura atómica. video.ini es propiedad del motor:
la interfaz no lo sobrescribe ni llama a sus setters desde el hilo Android.

## Mali, pantalla ancha y fluidez

Se conserva OpenGL ES 3.2 y la adaptación afín de shaders del ZIP recibido.
No se degrada a GLES 3.0 ni se desactiva interpolación por el nombre Mali.
La primera ejecución usa los valores predeterminados del motor; no recibe
`--pure` ni `--video-launch-set`, porque esos argumentos bloquearían el editor
interno. Después se cargan los ajustes del jugador sin sustituirlos cada vez.

«Preparar perfil gráfico ligero» propone escala 1×, MSAA desactivado,
anisotropía 1, mipmaps desactivados, efectos/sombras desactivados y límite 60.
Conserva la elección de interpolación; pulsa Aplicar para confirmar.
Algunos cambios requieren reiniciar.

Más FPS exigen GPU, batería y una pantalla adecuada; no son una garantía de
rendimiento. Cadencia de simulación no es lo mismo que FPS de presentación:
para suavidad visual utiliza primero límite de presentación e interpolación.

## Audio y vibración: limitaciones

No se reprodujo el aumento de audio con nitro; no se afirma haber corregido
su causa. Se añadieron controles de música/efectos y una mitigación opcional:
un limitador tras la mezcla y el volumen maestro, de ataque inmediato y
recuperación gradual. Solo atenúa, no amplifica pasajes silenciosos, no cambia
el tono y no recupera señal ya saturada. Desactívalo para comparar: la ganancia
retorna gradualmente a la unidad.

La vibración necesita hardware y permiso VIBRATE. Se adapta la amplitud cuando
el teléfono lo admite; en caso contrario se usa su intensidad predeterminada.
Se detiene al abrir ajustes, pausar o salir. Los pulsos del teléfono se consultan
cada 40 ms y tienen duración finita de seguridad: no se promete reproducción
idéntica de pulsos más cortos. SDL/Android determinan qué mandos USB/Bluetooth
y motores de vibración se reconocen.

## Internet experimental

Lee services/android-controller/README_ES.md. Solo un teléfono ejecuta el juego.
Los otros usan el modo mando del mismo APK, sin ROM, y ocupan P2–P4. El anfitrión
selecciona un modo local multijugador. No hay rollback, lockstep, vídeo ni audio
remoto: los demás necesitan ver la pantalla anfitriona por otro medio.

No se ha desplegado un servidor ni utilizado un endpoint de terceros. No se
envían ROMs o guardados. La clave de sala no se guarda en preferencias o logs.
Un mando físico tiene prioridad en su plaza. Si no llegan estados durante más
de 500 ms se neutralizan los botones; desconectarse libera la plaza. Pasar la
aplicación a segundo plano cierra las conexiones. No hay reconexión automática.

## Compilar

Python 3.12+ con filtro tar seguro, JDK 17, NDK 28.2.13676358, Android API 35,
Build Tools 35.0.0 y CMake 3.22.1. Las dependencias incluidas SDL/HarfBuzz/SheenBidi
se verifican por SHA-256. SDK/NDK y Gradle no están incluidos.

Linux x86_64 / Colab, desde la raíz extraída:

```sh
python3 tools/android/build.py --prepare
python3 tools/android/build.py --sdk-only --install-sdk --jobs 2
```

--install-sdk solicita instalar el SDK y aceptar sus licencias.
Si ya está instalado, usa --sdk /ruta/al/sdk en su lugar.
Para Gradle, omite --sdk-only. En Android Studio abre android/.
GoldenBalloon_Colab.ipynb encuentra el proyecto por su estructura: admite el
nombre del ZIP nuevo. Sus celdas no se han ejecutado en Colab en esta revisión.

Salida esperada al completar: out/GoldenBalloon-arm64-debug.apk.
--release genera una versión sin firmar. Conserva tu clave de firma anterior
para actualizar sin desinstalar; si no la tienes, exporta partidas antes de
reinstalar. No se incluyen claves ni APK antiguos como si fueran nuevos.
No existe compatibilidad de compilación universal: cada anfitrión necesita
las herramientas adecuadas. La guía base describe Windows/macOS y Docker.

## Validación ejecutada

```sh
python3 tools/android/check_revision.py
```

Pasaron:

- Cinco binarios C con -Wall -Wextra -Werror: video_config, audio_volume,
  pad_router, android_output y adaptación afín GLES.
- Catálogo Java compilado y contrastado con el validador C: 402 comprobaciones
  de nombres/valores. Las opciones dinámicas de mandos también se incluyen.
- Análisis sintáctico de las clases Java. NO equivale a comprobar tipos con android.jar.
- Siete pruebas del relay: autorización, plazas, desconexión, secuencias
  repetidas, paquetes grandes y TLS local. TLS rechazó un nombre incorrecto y
  un certificado no confiado. El certificado temporal de pruebas no se distribuye.
- Preparación y hashes de las dependencias incluidas.

La compilación completa se detuvo por SDK/NDK ausentes. No se validaron en esta
revisión: JNI, enlace AArch64 completo, D8, recursos, instalación, interfaz
en dispositivo, carrera con ROM, vibración real o sesión Android por internet.

Antes de distribuir: compila e instala; abre/cierra ajustes repetidamente;
comprueba Aplicar y persistencia tras reiniciar, nitro con/sin limitador,
todos los botones, HUD/minimapa, segundo plano, orientación, interpolación en
tu Mali y desconexión/pérdida de red de P2–P4.

Referencias: [VibrationEffect](https://developer.android.com/reference/android/os/VibrationEffect),
[WindowManager.LayoutParams](https://developer.android.com/reference/android/view/WindowManager.LayoutParams).
La configuración, pausa y entrada reutilizan el código fuente recibido.
