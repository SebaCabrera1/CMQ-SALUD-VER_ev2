# CMQ Salud

Aplicación móvil Android para la **gestión documental de funcionarios del Área Salud de la Corporación Municipal de Quilpué (CMQ)**. Proyecto semestral del ramo **DSY1105 – Desarrollo de Aplicaciones Móviles**.

> **Alcance:** es un MVP académico. Usa **solo datos ficticios** (no hay datos reales de funcionarios) y no se conecta a ningún servidor.

## Problema y objetivo

Los funcionarios entregan certificados, contratos y comprobantes de capacitación en papel o por correo, lo que dificulta saber qué se recibió, quién lo validó y cuándo. CMQ Salud permite:

- Que el funcionario **suba sus documentos** (con foto tomada con la cámara del teléfono) y vea su estado.
- Que el personal administrativo **valide o rechace** esos documentos, y el funcionario reciba una notificación.
- Que cada acción quede registrada para **auditoría**, con accesos distintos según el perfil de cada usuario.

## Tecnologías

| Área | Tecnología |
|---|---|
| Lenguaje | Kotlin |
| Interfaz | Jetpack Compose + Material 3 |
| Arquitectura | MVVM (Compose → ViewModel → Repository → Room) |
| Navegación | Navigation Compose |
| Persistencia local | Room (SQLite) con Flow/StateFlow |
| Recurso nativo | Cámara (`ActivityResultContracts.TakePicturePreview`) |
| Pruebas | JUnit 4 + `kotlinx-coroutines-test` (pruebas locales, sin emulador) |

## Requerimientos funcionales

| RF | Descripción | Estado |
|---|---|---|
| RF01 | Login diferenciado por perfil, con cierre de sesión | Implementado (usuarios ficticios) |
| RF02 | Subir documentos con cámara | Implementado, con Room y foto |
| RF03 | Validar documentos (aprobar/rechazar) | Implementado, con Room |
| RF04 | Buscar y filtrar documentos | Implementado (por tipo y estado) |
| RF05 | Aprobar/rechazar solicitudes (Jefatura) | Implementado, en memoria |
| RF06 | Registro de auditoría (quién, qué, cuándo) | Implementado, guardado en Room |
| RF07 | Antecedentes laborales (RR.HH.) | Implementado con datos de ejemplo |
| RF08 | Notificaciones al funcionario | Implementado, por usuario, en memoria |
| RF09 | Gestión de usuarios y roles | Implementado, en memoria |
| RF10 | Capacitaciones | Implementado, en memoria |
| RF11 | Historial de accesos | Reutiliza la pantalla de auditoría (el login queda registrado) |

"En memoria" significa que los datos se pierden al cerrar la app. Solo los **documentos** (RF02/RF03/RF04) se guardan en Room.

## Perfiles y permisos

El menú y la navegación se rigen por una sola tabla (`MenuPrincipalScreen.kt`). Lo que no aparece en el menú de un rol tampoco se puede abrir: el `NavHost` muestra "No tienes permiso".

| Rol | Puede acceder a |
|---|---|
| Funcionario | Subir documento, Mis documentos (solo los suyos), Capacitaciones, Notificaciones (solo las suyas) |
| Administrativo | Subir documento, Mis documentos (solo los suyos), Validar documentos (no los propios), Documentos de funcionarios (ve todos), Notificaciones |
| Jefatura | Subir documento, Mis documentos (solo los suyos), Solicitudes, Notificaciones |
| TI | Historial de auditoría |
| RRHH | Subir documento, Mis documentos (solo los suyos), Antecedentes laborales, Capacitaciones, Notificaciones |
| Administrador | Gestión de usuarios (no puede cambiar su propio rol) |

Pueden subir documentos: Funcionario, Jefatura, Administrativo y RRHH. En "Mis documentos" todos ven solo los suyos; solo el Administrativo tiene además "Documentos de funcionarios" con los de todos. Nadie puede validar sus propios documentos.

### Usuarios de prueba

Cualquier contraseña de **4 o más caracteres**. Un correo que no esté en la lista no entra.

| Correo | Rol |
|---|---|
| javier.munoz@cmq.cl | Funcionario |
| constanza.diaz@cmq.cl | Administrativo |
| marcela.soto@cmq.cl | Jefatura |
| rodrigo.pizarro@cmq.cl | TI |
| fernanda.castro@cmq.cl | RRHH |
| andrea.rojas@cmq.cl | Administrador |

## Estructura del código

El contenido de `cl/` va dentro de `app/src/main/java/` de tu proyecto Android Studio.

```
cl/cmqsalud/app/
├── model/        Clases de dominio (Documento, Usuario, Solicitud...) y reglas puras
├── data/
│   ├── local/        Room: DocumentoEntity, DocumentoDao, CmqSaludDatabase
│   └── repository/   DocumentosRepository (intermediario entre ViewModel y Room)
├── viewmodel/    Estado y lógica de cada pantalla (un ViewModel por tema)
├── ui/           Pantallas Compose (solo dibujan y emiten eventos)
└── navigation/   CmqSaludNavHost: rutas, ViewModels compartidos y guardia por rol
```

Más detalle de cada capa, el flujo de datos y las decisiones de diseño en [`docs/DOCUMENTACION_TECNICA.md`](docs/DOCUMENTACION_TECNICA.md).

## Cómo ejecutar el proyecto

Este zip contiene solo el código Kotlin. **No es un proyecto Android Studio completo** (falta Gradle, el manifest, `res/` y el tema que genera Android Studio al crear el proyecto).

1. Crea un proyecto **Empty Activity (Compose)** con el paquete `cl.cmqsalud.app`.
2. Copia `cl/cmqsalud/app/*` dentro de `app/src/main/java/cl/cmqsalud/app/`, respetando las carpetas. Reemplaza los archivos que ya existan.
3. Configura Gradle (ver abajo) y haz **Sync**.
4. Haz **Build → Run** en un emulador o teléfono con API 26 o superior.

### Configuración de Gradle

Este zip ya trae todo configurado (no hay que editar nada): `gradle/libs.versions.toml` declara las versiones
(KSP, Room, Navigation, Lifecycle, íconos extendidos, coroutines-test), `build.gradle.kts` (raíz) registra el plugin KSP
y `app/build.gradle.kts` lo aplica y agrega las dependencias. `minSdk = 26` porque `java.time` lo requiere.
`gradle.properties` incluye `android.builtInKotlin=false` y el plugin `kotlin-android` está aplicado, porque KSP no es compatible con el Kotlin integrado de AGP 9.

Si el Sync falla por una versión (por ejemplo `ksp` o `room` en `libs.versions.toml`), Android Studio indica la
versión disponible: ajústala ahí y vuelve a sincronizar. Después de cada cambio en Gradle: **Sync → Build → Run**.

### Cámara y permiso

El manifest declara `CAMERA` y la pantalla "Subir documento" lo **pide en tiempo de ejecución**: al tocar "Tomar foto" se revisa el permiso con `ContextCompat.checkSelfPermission`; si no está concedido se lanza `RequestPermission` (aparece el diálogo del sistema) y, solo si se acepta, se abre la cámara con `TakePicturePreview`. Si se deniega, se muestra un aviso. No hace falta `FileProvider`.

**Importante:** si declaras `CAMERA` en el manifest, siempre debes pedirlo en ejecución antes de abrir la cámara; si no, la app se cae con `SecurityException`.

## Animaciones

En "Subir documento" (`SubirDocumentoScreen.kt`) hay tres animaciones simples con `AnimatedVisibility` de Compose:

1. Vista previa de la foto: aparece con fundido y expansión vertical al tomarla, y se pliega al quitarla.
2. Aviso de cámara: mensaje animado si se cancela la captura o el dispositivo no tiene cámara.
3. Indicador de carga (`CircularProgressIndicator`): aparece mientras Room guarda (`guardando` en `DocumentosUiState`); el botón queda deshabilitado para evitar doble envío.

## Pruebas

Las pruebas viajan en un zip aparte (`CMQSalud_pruebas_unitarias.zip`). Se copian a `app/src/test/java/` (**no** a `androidTest`) y corren sin emulador.

- Desde Android Studio: clic derecho en la carpeta `test` → *Run Tests*.
- Desde terminal: `./gradlew test`.

Son **45 pruebas** en 10 clases: login, validación y repositorio de documentos, visibilidad por rol y validación, gestión de usuarios, solicitudes, repositorio de auditoría, notificaciones, capacitaciones y antecedentes.

`DocumentosViewModel` no tiene prueba directa porque es un `AndroidViewModel` que abre Room y necesita un `Context` real. Su lógica se prueba aparte: la validación como función pura (`validarCamposDocumento`), el guardado en `DocumentosRepository` con un DAO falso en memoria, y la visibilidad en `documentosVisiblesPara`.

## Limitaciones conocidas

- Solo `Documento` persiste en Room; el resto de las entidades vive en memoria.
- No hay contraseñas reales: el login valida solo el formato de la contraseña (MVP).
- La foto se guarda en la base de datos, pero las pantallas solo indican "con foto"; no la muestran en las listas.
- Las notificaciones y el directorio de usuarios se reinician al cerrar la app.
- El código se revisó leyéndolo y con pruebas unitarias; compílalo en Android Studio antes de entregar.

## Flujo de trabajo con Git

Un commit por cambio coherente, con mensaje que diga qué aporta (por ejemplo `Integra Room para documentos`, `Agrega captura de foto en subir documento`). Trabaja en ramas `feature/...` y evita un único commit final.
#   C M Q - S A L U D - V E R _ e v 2  
 