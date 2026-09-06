# mu-cliente-mac

Receta para compilar el cliente [MuMain](https://github.com/sven-n/MuMain) (versión 1.2.5, la publicada en el fork [yesid-bocanegra/MuMain](https://github.com/yesid-bocanegra/MuMain/releases/tag/v1.2.5)) para macOS Apple Silicon usando los Mac de GitHub Actions, con cuatro arreglos pequeños aplicados antes de compilar (`cliente-mac-parches.patch`):

1. Shaders: se buscan junto al ejecutable (`Contents/MacOS/shaders`), que es donde el paquete de Mac los trae. Sin esto el cliente abre con «SDL_gpu Renderer Error».
2. Música: una pista que no se puede abrir no se reintenta en cada cuadro. Sin esto, con la carpeta `Data/Music` ausente, el cliente se puede quedar pegado al cambiar de aplicación (bloqueo en miniaudio).
3. MU Helper: los nombres de ítems de 15 o más caracteres se recortan a 14 al guardar, en vez de perderse en silencio.
4. Intercambio: la caja de intercambio acepta el ítem también al soltar el botón del mouse (como la bóveda). Sin esto no se podía poner ningún ítem en el intercambio, porque la ventana del inventario consume el clic de «apretar» mientras hay un ítem tomado.

El workflow (`.github/workflows/build-macos.yml`) usa los mismos pasos que el CI del proyecto (`build-macos` en `ci.yml`) y publica un Release con `MuMain-macos-native-arm64-release-editor-off-no-data-parchado.tar.gz` y su sha256. El paquete no incluye la data del juego (`Data`, `fonts`), igual que el oficial.

Uso personal, sin garantía. Solo cambia el cliente; no toca nada del protocolo ni del servidor.
