<div align="center">

# 🎬 DAMFlix · Parte 4

**Aplicación Android para almacenar y consultar películas**

![Kotlin](https://img.shields.io/badge/Kotlin-7F52FF?style=for-the-badge&logo=kotlin&logoColor=white)
![Jetpack Compose](https://img.shields.io/badge/Jetpack%20Compose-4285F4?style=for-the-badge&logo=jetpackcompose&logoColor=white)
![Material 3](https://img.shields.io/badge/Material%203-757575?style=for-the-badge&logo=materialdesign&logoColor=white)
![Android Studio](https://img.shields.io/badge/Android%20Studio-3DDC84?style=for-the-badge&logo=androidstudio&logoColor=white)

**Módulo:** Programación Multimedia y Dispositivos Móviles  
**Unidad de Trabajo Nº2:** Desarrollo de aplicaciones Android  
**Parte 4 · Actividad 4:** Navegación  

2º DAM · CIFP Villa de Agüimes

</div>

---

## 🎯 Objetivo

Conectar las pantallas desarrolladas en las fases previas mediante el componente oficial de navegación de **Jetpack Compose**.

## ✅ Requisitos funcionales

| Requisito | Cómo se cumple |
|---|---|
| **1. Configuración del grafo de navegación** | Se implementan `NavHost` y `NavController`, y las rutas se centralizan en una `sealed class Ruta` |
| **2. Navegación entre las pantallas realizadas** | Se navega entre el formulario, el catálogo y la página de información de la aplicación |
| **3. Utilización de un BottomBar** | Una `NavigationBar` en el `bottomBar` del `Scaffold` permite cambiar de pantalla |

## ⚙️ Funcionamiento

### Dependencia de navegación

En `app/build.gradle.kts`, dentro de `dependencies`:

```kotlin
implementation("androidx.navigation:navigation-compose:2.10.2")
```

### Rutas con una sealed class

Cada destino de la barra inferior tiene una ruta (`path`), una etiqueta (`label`) y un icono (`icon`):

```kotlin
sealed class Ruta(val path: String, val label: String, val icon: ImageVector) {
    object Inicio : Ruta(path = "inicio", label = "Inicio", icon = Icons.Filled.Home)
    object Listado : Ruta(path = "listado", label = "Listado", icon = Icons.Filled.List)
    object Formulario : Ruta(path = "alta", label = "Añadir", icon = Icons.Filled.Add)
}
```

### Scaffold con TopAppBar y BottomBar

La pantalla principal, `PantallaCatalogoDAMFlix()`, crea el `NavController` y observa la ruta activa:

```kotlin
val navController = rememberNavController()
val navBackStackEntry by navController.currentBackStackEntryAsState()
val rutaActual = navBackStackEntry?.destination?.route
```

La barra inferior marca la pestaña seleccionada y navega al pulsar:

```kotlin
NavigationBar {
    listOf(Ruta.Inicio, Ruta.Listado, Ruta.Formulario).forEach { ruta ->
        NavigationBarItem(
            selected = rutaActual == ruta.path,
            onClick = {
                if (rutaActual != ruta.path) {
                    navController.navigate(ruta.path) {
                        popUpTo(navController.graph.startDestinationId) {
                            saveState = true
                        }
                        launchSingleTop = true
                        restoreState = true
                    }
                }
            },
            icon = { Icon(ruta.icon, contentDescription = ruta.label) },
            label = { Text(ruta.label) }
        )
    }
}
```

- `popUpTo(...) { saveState = true }`: limpia la pila hasta el destino raíz y conserva el estado de la pantalla anterior.
- `launchSingleTop = true`: evita apilar varias copias de la misma pantalla.
- `restoreState = true`: restaura el estado al volver a una pestaña ya visitada.

### Título que cambia según la pantalla

La barra superior muestra un título distinto en cada pantalla:

```kotlin
val titulo = when (rutaActual) {
    Ruta.Listado.path -> "Catálogo"
    Ruta.Formulario.path -> "Formulario"
    else -> "Información de la Aplicación"
}
```

### NavHost

El `NavHost` recibe el `innerPadding` del `Scaffold` para que las pantallas no queden tapadas por las barras:

```kotlin
NavHost(
    navController = navController,
    startDestination = Ruta.Inicio.path,
    modifier = Modifier.padding(innerPadding)
) {
    composable(Ruta.Inicio.path) { PantallaInicio() }
    composable(Ruta.Listado.path) {
        ListaPeliculas(
            peliculas = peliculas,
            onToggleFavorito = { id ->
                val pelicula = peliculas.first { it.id == id }
                pelicula.esFavorita = !pelicula.esFavorita
            }
        )
    }
    composable(Ruta.Formulario.path) { FormularioPelicula() }
}
```

### Pantallas

| Pantalla | Ruta | Contenido |
|---|---|---|
| **Inicio** | `inicio` | Página de información de la aplicación: el perfil de usuario (Actividad 1) |
| **Listado** | `listado` | Catálogo de 15 películas con `LazyColumn` y corazón de favorita (Actividad 3) |
| **Añadir** | `alta` | Formulario de alta de película con validación (Actividad 2) |

La lista de películas se crea con `remember` fuera del `NavHost`, por eso los favoritos se conservan al cambiar de pantalla.

## 🗂️ Estructura

```
app/src/main
├── java/com/example/damflix
│   ├── MainActivity.kt        # PantallaCatalogoDAMFlix: Scaffold, BottomBar y NavHost
│   ├── Ruta.kt                # sealed class Ruta con las rutas de la BottomBar
│   ├── Pantallas.kt           # PantallaInicio, FormularioPelicula y PerfilUsuario
│   └── Peliculas.kt           # clase Pelicula y ListaPeliculas
└── res
    └── values
        └── strings.xml        # Textos del perfil y del formulario
```

## 🧪 Cómo probar la navegación

1. Al abrir la app se ve la pantalla "Inicio" con el perfil de usuario, y el título de la barra superior es "Información de la Aplicación".
2. Pulsa "Listado" en la barra inferior: aparece el catálogo y el título cambia a "Catálogo".
3. Pulsa el corazón de una película: se pone rojo. Púlsalo otra vez y vuelve a gris.
4. Pulsa "Añadir": aparece el formulario y el título cambia a "Formulario".
5. Escribe un título y un director: el botón "Guardar Película" se activa.
6. Vuelve a "Listado": el corazón que marcaste sigue rojo.
7. Pulsa varias veces el mismo botón de la barra: no se apilan pantallas repetidas.

## 🚀 Cómo ejecutarlo

1. Clona el repositorio:
```bash
git clone https://github.com/ivanhernandezbetancor/DAMflix_PARTE_4.git
```
2. Ábrelo con **Android Studio**.
3. Espera a que termine la sincronización de Gradle.
4. Ejecuta la app en un emulador o en un dispositivo físico.

## 🛠️ Tecnologías

- Kotlin
- Jetpack Compose
- Material 3
- Navigation Compose

### Dependencias necesarias

En `dependencies` de `app/build.gradle.kts`:

```kotlin
implementation("androidx.navigation:navigation-compose:2.10.2")
implementation("androidx.compose.material:material-icons-core")
```

La segunda línea es necesaria para los iconos `Favorite` y `FavoriteBorder`.

```kotlin
implementation("androidx.compose.material:material-icons-core")
```
