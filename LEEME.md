# Hugo Click Guide — versión 1.0

Proyecto fuente de un mod para Geometry Dash 2.2081, Windows de 64 bits y Geode 5.11.2. Implementación propia inspirada en el comportamiento visual del video «Click Indicator Mod Redux». No es una distribución del mod de Bogdoner.

**Estado de entrega:** el núcleo de carga de macros, grabación y puntuación pasó 41 comprobaciones en C++ en Linux. La DLL de Windows y el paquete `.geode` todavía no se han compilado aquí ni probado dentro del juego. El ZIP contiene el proyecto para compilar; no se instala directamente en `geode/mods`.

## Qué hace

- Franjas verdes delante del icono: principio = pulsar, longitud = mantener, final = soltar.
- Carril lateral en perspectiva: pulsa cuando la nota llega a la línea amarilla; las notas largas representan mantener.
- Línea blanca y recuadro alrededor del icono para alinear los indicadores.
- Puntuación PERFECT, OK, MISS y EXTRA, con adelanto o retraso en milisegundos.
- Jugador 2 en rosa y un segundo carril mientras el nivel esté en dual.
- Grabación de las pulsaciones de un intento como referencia, con guardado por nivel.
- Importación de macros GDR v1: `.gdr` MessagePack y `.gdr.json`. También acepta `.xd` si contiene ese formato GDR v1.
- Preferencias para opacidad, anticipación, desfase y tolerancias.
- Carga automática de la última referencia seleccionada al volver al nivel.

Las franjas muestran **una referencia de clics**. No calculan las ventanas seguras del nivel. La calidad de la guía depende de la macro o del intento grabado. Un intento que falla sirve como referencia parcial, no como solución del nivel.

## Primer uso sin tener una macro

1. Desactiva el InputBuffer anterior para que la guía mida tus entradas sin que otro mod las retrase.
2. Entra en un nivel normal desde el inicio, fuera de práctica y sin StartPos.
3. Pulsa **F7 / REC**. Esto reinicia el nivel y empieza la grabación.
4. Juega. F7 detiene la grabación; una muerte también la detiene y conserva el intento parcial.
5. Pulsa **F8 / GUARDAR**. La grabación se guarda como archivo nuevo y se selecciona como referencia. Al completar el nivel se guarda automáticamente.
6. Reinicia el nivel: aparecerán los indicadores de esa grabación.

Antes de grabar otro intento, guarda el anterior. Salir del nivel sin guardar descarta la grabación pendiente. Los archivos guardados no se sobrescriben; cada intento tiene su propio nombre.

## Cargar una macro existente

1. Entra en el nivel de esa macro.
2. Pulsa **F10 / CARPETA**. Se abre la carpeta de macros del mod.
3. Copia la macro con el ID del nivel como nombre: por ejemplo, `123456.gdr` o `123456.gdr.json`.
4. Pulsa **F9 / CARGAR** y reinicia el nivel.

El mensaje superior muestra el nombre base requerido. En niveles locales sin ID, usa el nombre `local-...` que muestra ese mensaje. Si el nombre de un nivel local cambia, cambia también su clave. Dos niveles locales con el mismo nombre comparten clave: cámbiales el nombre para separarlos.

La carpeta está bajo el directorio de guardado de Geode: normalmente `geode/saved/hugo.click-guide/macros`. El botón CARPETA abre la ruta efectiva.

CARGAR da prioridad a los archivos nombrados exactamente como el ID. Al entrar de nuevo en el nivel, se recuerda la última referencia seleccionada. Si un archivo del ID está dañado, se muestra el error; no se oculta cargando otro silenciosamente.

No se incluyen macros de niveles reales. `examples/formato.json` solo documenta el formato propio. **GDR2, Silicate/SLC y otros formatos binarios no están implementados.** Si el bot que usas puede exportar GDR v1 o GDR JSON, elige esa opción.

## Controles

| Tecla / botón | Acción |
|---|---|
| F6 / GUIA | Activar o desactivar la guía |
| F7 / REC | Reiniciar y grabar, o detener la grabación actual |
| F8 / GUARDAR | Guardar el intento y utilizarlo como referencia |
| F9 / CARGAR | Cargar el archivo del ID del nivel |
| F10 / CARPETA | Abrir la carpeta de macros |
| AJUSTES | Pausar y abrir las preferencias del mod |

Los atajos se consultan durante el juego activo; en pausa usa las preferencias de Geode. Si otro mod usa las mismas teclas, desactiva ese atajo en el otro mod.

## Configuración para jugar a 480 FPS

La referencia se interpreta en segundos. **No cambies el framerate de una macro a 480 solo porque juegas a 480 FPS.** Si una macro cuenta 240 pasos por segundo, tiene que declarar 240. Una grabación propia guarda segundos directamente.

Valores iniciales: guía activada; franjas verdes al 20 % de opacidad; 2 segundos de anticipación sobre el nivel; 1.2 segundos en el carril; hasta 32 notas visibles; PERFECT ±8 ms y OK ±33 ms. Estas tolerancias califican la práctica; no alteran los clics ni las colisiones.

Desfase positivo retrasa las notas; negativo las adelanta. Por ejemplo, +20 ms muestra cada objetivo 20 ms después. El ajuste se aplica al reanudar el juego.

El núcleo no avanza el reloj por fotogramas ni supone que render y física tienen la misma frecuencia. La integración toma `m_gameState.m_levelTime` del juego y observa los eventos de `PlayerObject`. No es una medición directa del instante de hardware del ratón, ni una garantía de precisión entre pasos de física.

## Límites importantes para usarlo bien

- Las franjas del nivel se proyectan usando la velocidad horizontal actual. Un portal de velocidad futuro, una cámara rotada, un teleport o una inversión pueden cambiar su ubicación visual. El carril mantiene la referencia temporal; no es una simulación de la geometría futura.
- En platformer solo aparece el carril; no se dibujan franjas de posición. La grabación observa salto, no izquierda/derecha.
- La grabación se limita a intentos desde el inicio, sin práctica ni StartPos. La visualización puede usarse en práctica: se reinicia la puntuación al cargar checkpoint usando el reloj restaurado por el juego. Esa integración necesita prueba dentro de GD.
- Con StartPos o macros cuyo origen temporal sea distinto, puede hacer falta ajustar el desfase manualmente. No hay alineación automática por posición.
- La puntuación evalúa las pulsaciones. Las liberaciones determinan la duración visual de las notas, pero no reciben una puntuación aparte.
- Una macro con FPS declarados incorrectamente se verá desfasada. No se adivina ni cambia automáticamente su frecuencia.
- Los efectos de CBF, speedhack y otros mods sobre los eventos observados deben probarse en tu instalación.
- El código no manda pulsaciones automáticas, modifica la física ni bloquea tu entrada.

## Compilar en Windows

Instala Visual Studio 2022 con las herramientas de C++, CMake, Git y Geode CLI/SDK 5.11.2. Configura `GEODE_SDK` apuntando al SDK. Desde la raíz del proyecto:

```powershell
cmake -S . -B build -A x64
cmake --build build --config Release
```

Busca el archivo `.geode` resultante en `build`. Copia ese archivo a `Geometry Dash/geode/mods` y reinicia el juego. El SDK se encarga de generar el paquete; no renombres este ZIP a `.geode`.

También se incluye `.github/workflows/build.yml`: si subes el contenido de esta carpeta a la raíz de un repositorio propio en GitHub, puedes ejecutar **Actions → Build Click Guide for Windows → Run workflow**. El flujo primero prueba el núcleo y después intenta construir el paquete de Windows con Geode. El resultado aparece en Artifacts. Este flujo se entrega preparado, pero no ha sido ejecutado desde esta sesión.

Si tu GD o Geode tienen otra versión, no cambies solo `mod.json`: hay que comprobar los bindings y volver a compilar.

## Ejecutar las pruebas del núcleo

Sin Geode, en Linux con g++:

```sh
g++ -std=c++20 -O2 -Wall -Wextra -pedantic tests/core_tests.cpp -o core-tests
./core-tests
```

O mediante CMake:

```sh
cmake -S . -B build-tests -DGUIDE_CORE_TESTS=ON
cmake --build build-tests
ctest --test-dir build-tests --output-on-failure
```

Se comprueban conversión de tiempos de macros de 60 a 1200 pasos/s, pulsación/liberación, dual, eventos duplicados, entrada corrupta, límites de formato, guardado sin sobrescritura, puntuación única por nota, reinicios y grabación. No sustituyen las pruebas del render ni de los hooks dentro de GD.

## Dependencias y referencias

Código propio en `src/` y `tests/`, bajo la licencia MIT adjunta. La única biblioteca vendorizada es nlohmann/json 3.12.0, también MIT; su licencia está en `third_party/LICENSE-json.txt`.

- Geode SDK y bindings: https://github.com/geode-sdk/geode y https://github.com/geode-sdk/bindings
- Especificación de GDR v1: https://github.com/maxnut/GDReplayFormat
- Referencia funcional del video: https://github.com/Andrew3381/Click-Indicators-Redux

No se ha copiado código ni recursos gráficos de Click Indicators Redux. Los gráficos del carril se dibujan con primitivas de Cocos2d.
