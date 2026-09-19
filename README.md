# mu-cliente-mac

Receta para compilar el cliente [MuMain](https://github.com/sven-n/MuMain) (versión 1.2.5, la publicada en el fork [yesid-bocanegra/MuMain](https://github.com/yesid-bocanegra/MuMain/releases/tag/v1.2.5)) para macOS Apple Silicon y para Windows x64 usando las máquinas de GitHub Actions, con diez arreglos pequeños aplicados antes de compilar (`cliente-mac-parches.patch`), más los parches del contenido Season 8 (ver abajo):

1. Shaders: se buscan junto al ejecutable (`Contents/MacOS/shaders`), que es donde el paquete de Mac los trae. Sin esto el cliente de Mac abre con «SDL_gpu Renderer Error».
2. Música: una pista que no se puede abrir no se reintenta en cada cuadro. Sin esto, con la carpeta `Data/Music` ausente, el cliente se puede quedar pegado al cambiar de aplicación (bloqueo en miniaudio).
3. MU Helper: los nombres de ítems de 15 o más caracteres se recortan a 14 al guardar, en vez de perderse en silencio.
4. Intercambio: la caja de intercambio acepta el ítem también al soltar el botón del mouse (como la bóveda). Sin esto no se podía poner ningún ítem en el intercambio, porque la ventana del inventario consume el clic de «apretar» mientras hay un ítem tomado.
5. Máquina del caos: lo mismo que el punto 4 para la caja de la máquina del caos (arrastrar un ítem a la caja; el clic derecho ya funcionaba).
6. Opción de nivel 380 (Guardian): el cliente ya recibía la opción del servidor pero la descartaba al leer el ítem, así que no salía en la descripción ni surtía efecto visible. Faltaba una línea en `ParseItemData` que copiara el bit `HasGuardian`.
7. Textos del MU Helper: diez etiquetas de la ventana del MU Helper se dejan en inglés, porque la traducción al español era más larga que el hueco y los textos se solapaban. No toca `Automatic Attack`, que es una cadena global usada en otras ventanas.
8. Traducciones arregladas (10 textos). La más importante: `Level: %u | Resets: %u` pasa a «Nivel: %u | Resets: %u», del mismo largo que el inglés, porque en español («Restablecimientos») el texto invadía la zona del número naranja de puntos por repartir en la ventana de personaje. **Hasta ahora esto se aplicaba como parche binario sobre el ejecutable y se perdía en cada actualización del cliente.** Las otras nueve son errores de sentido del traductor automático: `Set` significa «conjunto» y no el verbo «colocar», así que la casilla de recoger ancients del MU Helper aparecía como «Establecer elemento» y era imposible de encontrar, y la etiqueta del inventario decía «[Establecer opción]». También se unifica «mobs», que estaba traducido de dos formas distintas en la misma ventana.
9. Tooltip de la Phoenix Soul Star: `GetSpecialOptionText` no tenía caso para el skill Phoenix Shot (270); la línea quedaba vacía y `RenderTipTextList` corta la lista en la primera línea vacía, así que con skill desaparecían todas las opciones del ítem (skill, luck, option, excelentes) en la mochila y en el intercambio. Se añade el caso con el texto que ya existía en el resx.
10. Semilla de Tierra: el código buscaba la opción en la ficha `34 + nivel` de `SocketItem_*.bmd`, pero los ficheros (y OpenMU) la traen en la 36; la Seed (Earth) salía sin texto y la Seed Sphere (Earth) con «+0». Los cuatro sitios pasan a `36 + nivel`.

El workflow (`.github/workflows/build-clientes.yml`) se lanza a mano desde la pestaña Actions (Run workflow). Usa los mismos pasos que el CI del proyecto (`build-macos` y `build-windows` en `ci.yml`) y publica un Release con:

- `MuMain-macos-native-arm64-release-editor-off-no-data-parchado.tar.gz`: runtime de Mac, para `actualizar-cliente-mac.sh`.
- `Mu-cliente-windows-parche9.zip`: para los jugadores de Windows; `Main.exe`, las DLL y los shaders. No trae `Data`, `fonts` ni `config.ini`, así que se descomprime encima de la carpeta del juego y cada uno conserva su configuración.
- `MuMain-windows-native-x64-release-editor-off-no-data-parchado.tar.gz`: runtime completo de Windows con el `config.ini` de plantilla, por si hay que armar un cliente de cero.
- `sha256.txt` con las sumas de todo.

Ninguno de los paquetes incluye la data del juego (`Data`, `fonts`), igual que los oficiales.

Uso personal, sin garantía. Solo cambia el cliente; no toca nada del protocolo ni del servidor.

## Línea Season 8 (rama `s8-pruebas`, la rama por defecto desde el 18-09-2026)

Además de los diez de arriba, la receta de `s8-pruebas` aplica doce parches más, cada uno con
`git apply --check` delante y con controles de contenido después, **en los dos trabajos (Mac y Windows)**:

11. Contenido extra: MuMain lee `Data/Local/ExtraItems.txt` y `ExtraMonsters.txt` para dibujar ítems y bichos que el código no conoce.
12. Mapas nuevos (Acheron, Debenter, Illusion Temple Final y gemelos): enum de mundos, nombres, placas, skills 609-617.
13. Terreno: segunda capa de textura.
14. NPC y texturas de Acheron.
15. Diagnóstico (mapas, personajes, objetos).
16. Bichos del S8 antes del `default:` que los convertía en toro.
17. Diagnóstico de mallas.
18. Animación y NPC (Jin).
19. Minería de la Jewel of Pandora (protocolo, zona, cartel, textos).
20. Jin en el mapa 92.
21. Los puntos por repartir de la ventana de personaje se pisaban con el nivel y los resets.
22. Los ítems de misión se pueden tirar.
23. Los consumibles Season 8 (talismanes y sellos de Ascension, Blessing of Light, Scroll Master) se usan con clic derecho; antes el cliente no los conocía y el clic derecho los tiraba al piso sin mandar nunca «usar».
24. Los bufos Season 8 (190-204) se dibujan con el icono S6 equivalente; no tienen celda en ninguna lámina y salían como un recuadro negro.
25. Contador de tiempo bajo cada bufo y reinicio del temporizador al re-lanzar (de Mundy, rama `mu-chile-v1.2.5`, commit `fc40705ebb`; solo sus 6 ficheros nuevos, los otros 10 eran nuestros parches 1-10).

- `build-clientes.yml` (Mac + Windows, publica release): se lanza a mano. **La rama que se elige en «Run workflow» es la que se compila**: `main` sigue siendo la línea S6 (parches 1 a 10).
- `build-mac-s8.yml` (solo Mac, sin release): **dispara con cada `push` a `s8-pruebas`** (macOS factura ×10). Si se sube algo que no hace falta compilar, poner `[skip ci]` en el mensaje del commit.
- Las releases de esta línea llevan el tag `v1.2.5-s8-parche25-r<N>` y apuntan al commit compilado. La `v1.2.5-parche9-r5` del 17-09 es un binario S8 (1 a 20) con tag en `main`; su texto se corrigió el 18-09.
