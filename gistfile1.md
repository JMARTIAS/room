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

creo un paquete entities para agregar nuestras tablas, que son data clases, hay que poner el Entity para que lo reconozca. en tableName le cambio el nombre a la tabla en room, sino se llamará igual que la clase.

Se pone el primary key con la anotación y para que no lo autogenere como los ids se setea en false
```kotlin
@Entity (tableName = "director_table")
data class Director(
@PrimaryKey (autoGenerate = false)
val directorName: String )
```
genero una segunda entity

```kotlin
@Entity 
data class School(
@PrimaryKey (autoGenerate = false)
val schoolName: String )
```

y luego tengo que generar una relación entre ambas tablas, así que creo un nuevo paquete llamado relations, en el que creo mi clase para la relación, la convención de nombre es primera tabla con un and y segunda tabla (es una relación 1:1). 
Room automaticamente obtiene los parametros de nuestra clase si le ponemos el Embedded.
en el caso de la segunda tabla hay qye poner que es la relation

```kotlin

data class SchoolAndDirector(
 @Embedded val school:School,
 @Relation(
 parentColumn = "schoolName",
 entityColumn = "schoolName"
 )
 val director:Director
```

estas 2 entidades tienen que tener una propiedad en común para que Room pueda ver cuales valores de estas entradas van juntos.
parentColumn es nuestro schooolName. el entityColumn es el mismo schoolName, pero que esta en la clase de director y no es pk

```kotlin
@Entity (tableName = "director_table")
data class Director(
@PrimaryKey (autoGenerate = false)
val directorName: String,
val schoolName: String)
```


