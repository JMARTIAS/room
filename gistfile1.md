# ROOM
para iniciar un proyecto con roon hay que implementar unas dependencias 
```gradle
// Room
implementation "androidx.room:room-runtime:2.2.5"
kapt "androidx.room:room-compiler:2.2.5"

// Kotlin Extensions and Coroutines support for Room
implementation "androidx.room:room-ktx:2.2.5"

// Coroutines
implementation 'org.jetbrains.kotlinx-coroutines-core:1.3.7'
implementation 'org.jetbrains.kotlinx-coroutines-android:1.3.9'

// Coroutines Lifecycle Scopes
implementation "androidx.lifecycle:lifecycle-viewmodel-ktx:2.2.0"
implementation "androidx.lifecycle:lifecycle-runtime-ktx:2.2.0"
```

tambien se debe poner en plugins (un poco mas arriba)
```gradle
id 'kotlin-kapt'
```

despues se hace el sync now

creo un paquete entities para agregar nuestras tablas, que son data clases, hay que poner el Entity para que lo reconozca. en tableName le cambio el nombre a la tabla en room, sino se llamará igual que la clase
```kotlin
@Entity (tableName = "director_table")
data class Director{}
```
