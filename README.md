# mu-cliente-mac

Receta para compilar el cliente [MuMain](https://github.com/sven-n/MuMain) (versión 1.2.5, la publicada en el fork [yesid-bocanegra/MuMain](https://github.com/yesid-bocanegra/MuMain/releases/tag/v1.2.5)) para macOS Apple Silicon y para Windows x64 usando las máquinas de GitHub Actions, con ocho arreglos pequeños aplicados antes de compilar (`cliente-mac-parches.patch`):

1. Shaders: se buscan junto al ejecutable (`Contents/MacOS/shaders`), que es donde el paquete de Mac los trae. Sin esto el cliente de Mac abre con «SDL_gpu Renderer Error».
2. Música: una pista que no se puede abrir no se reintenta en cada cuadro. Sin esto, con la carpeta `Data/Music` ausente, el cliente se puede quedar pegado al cambiar de aplicación (bloqueo en miniaudio).
3. MU Helper: los nombres de ítems de 15 o más caracteres se recortan a 14 al guardar, en vez de perderse en silencio.
4. Intercambio: la caja de intercambio acepta el ítem también al soltar el botón del mouse (como la bóveda). Sin esto no se podía poner ningún ítem en el intercambio, porque la ventana del inventario consume el clic de «apretar» mientras hay un ítem tomado.
5. Máquina del caos: lo mismo que el punto 4 para la caja de la máquina del caos (arrastrar un ítem a la caja; el clic derecho ya funcionaba).
6. Opción de nivel 380 (Guardian): el cliente ya recibía la opción del servidor pero la descartaba al leer el ítem, así que no salía en la descripción ni surtía efecto visible. Faltaba una línea en `ParseItemData` que copiara el bit `HasGuardian`.
7. Textos del MU Helper: diez etiquetas de la ventana del MU Helper se dejan en inglés, porque la traducción al español era más larga que el hueco y los textos se solapaban. No toca `Automatic Attack`, que es una cadena global usada en otras ventanas.
8. Traducciones arregladas (10 textos). La más importante: `Level: %u | Resets: %u` pasa a «Nivel: %u | Resets: %u», del mismo largo que el inglés, porque en español («Restablecimientos») el texto invadía la zona del número naranja de puntos por repartir en la ventana de personaje. **Hasta ahora esto se aplicaba como parche binario sobre el ejecutable y se perdía en cada actualización del cliente.** Las otras nueve son errores de sentido del traductor automático: `Set` significa «conjunto» y no el verbo «colocar», así que la casilla de recoger ancients del MU Helper aparecía como «Establecer elemento» y era imposible de encontrar, y la etiqueta del inventario decía «[Establecer opción]». También se unifica «mobs», que estaba traducido de dos formas distintas en la misma ventana.

El workflow (`.github/workflows/build-clientes.yml`) se lanza a mano desde la pestaña Actions (Run workflow). Usa los mismos pasos que el CI del proyecto (`build-macos` y `build-windows` en `ci.yml`) y publica un Release con:

- `MuMain-macos-native-arm64-release-editor-off-no-data-parchado.tar.gz`: runtime de Mac, para `actualizar-cliente-mac.sh`.
- `Mu-cliente-windows-parche8.zip`: para los jugadores de Windows; `Main.exe`, las DLL y los shaders. No trae `Data`, `fonts` ni `config.ini`, así que se descomprime encima de la carpeta del juego y cada uno conserva su configuración.
- `MuMain-windows-native-x64-release-editor-off-no-data-parchado.tar.gz`: runtime completo de Windows con el `config.ini` de plantilla, por si hay que armar un cliente de cero.
- `sha256.txt` con las sumas de todo.

Ninguno de los paquetes incluye la data del juego (`Data`, `fonts`), igual que los oficiales.

Uso personal, sin garantía. Solo cambia el cliente; no toca nada del protocolo ni del servidor.
