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

## Relación 1:1

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

Luego hay que crear un objeto DAO (Data Access Object) que es una interface donde definimos las funciones para acceder a nuestra base de datos

```kotlin
@Dao
interface SchoolDao {
 @Insert(OnConflict = OnConflictStrategy.REPLACE)
 suspend fun insertSchool (school:School)
 
 @Insert(OnConflict = OnConflictStrategy.REPLACE)
 suspend fun insertDirectpr (director:Director)
 
 @Transaction
 @Query("SELECT * FROM school WHERE schoolName =:schoolName")
 suspend fun getSchoolAndDirectorWithSchoolName(schoolName:String): List<SchoolAndDirector>
 
}
```
se usan funciones suspend pq se realizan en el background, para no bloquear el hilo principal.
el on conflict lo que hace es que si quiero insertar una escuela que ya existe la voy a reemplazar

podría pasar que haya unos problemas con los multi threads por eso se le pone Transaction, ya que estamos mezclando consultas a 2 tablas

## Relación 1:N

hay que ver en que tabla ponemos la primary key de la otra en esta relación, porque hay multiples alumnos que van a un colegio.

```kotlin
@Entity
data class Student(
@PrimaryKey(autoGenerate = false)
 val studentName:String,
 val semester:Int,
 //esta es la primarykey de la otra entity
 val schoolName: String
)
```

en nuestro paquete de relations creamos una nueva clase de kotlin que por convención se llama tabla única With la tabla N

```kotlin

data class SchoolWithStudents(
 @Embedded val school:School,
 @Relation(
 //este corresponde al school name de la clase que contiene multiples instancias de la otra
  parentColumn = "schoolName",
  //este corresponde al de los students
  entityColumn = "schoolName"
 )
 val students: List<Student>
)
```
el list va pq ahora tenemos varias instancias de alumnos.

Luego lo que hay que hacer es actualizar nuestro DAO con las queries que necesitamos

```kotlin
 @Insert(OnConflict = OnConflictStrategy.REPLACE)
 suspend fun insertStudent (student:Student)
 
 @Transaction
 @Query("SELECT * FROM school WHERE schoolName =:schoolName)
 suspend fun getSchoolWithStudents(schoolName:String): List<SchoolWithStudents>

```

## Relación N:M 

cada estudiante puede tener muchos ramos y cada ramo puede tener muchos alumnos, 
así que creamos una nueva clase en nuestro paquete entity.

```kotlin
@Entity
data class Subject(
@PrimaryKey (autoGenerate = false)
val subjectName:String)

```

ahora la relación entre nuestras tablas necesita una tabla con ambas PK, la hacemos en nuestro paquete relations


```kotlin
@Entity(primaryKeys = ["studentName", "subjectName"])
data class StudentSubjectCrossRef(
 val studentName:String,
 val subjectName:String
)
```

acá ninguno de los 2 es la PK sino que su PK es una combinación de ambas, pero hayque decirle  a Room que esta entity establece una relación entre 2 tablas. hay que insertarle datos a esta tabla al agregar alumnos o ramos
decirle que las 2 tablas están relacionadas
queremos consultar 1 estudiante y que tenga todos sus ramos y viceversa, así que hacemos 2 clases mas para estas relaciones

```kotlin
data class StudentWithSubjects(
 @Embedded val student: Student,
 @Relation(
  parentColumn = "studentName"
  entityColumn = "subjectName"
  assoiciateBy = Junction(StudentSubjectCrossRef::class)
  )
  val subjects: List<Subject>
)
```

a esta clase hayque decirle que tabla hace esta relación, para que room lo sepa, eso es lo que hace el associateBy. Es una clase helper.

se hace lo mismo con una clase SubjectWithStudents

```kotlin
data class SubjectWithStudents(
 @Embedded val subject: Subject,
 @Relation(
  parentColumn = "subjectName"
  entityColumn = "studentName"
  assoiciateBy = Junction(StudentSubjectCrossRef::class)
  )
  val students: List<Student>
)
```

hay que agregar mas funciones al dao 

```kotlin
 @Insert(OnConflict = OnConflictStrategy.REPLACE)
 suspend fun insertSubject (subject:Subject)
 
  @Insert(OnConflict = OnConflictStrategy.REPLACE)
 suspend fun insertStudentSubjectCrossRef (crossRef: StudentSubjectCrossRef)
 
 @Transaction
 @Query("SELECT * FROM subject WHERE subjectName =:subjectName")
 suspend fun getStudentsOfSubject(subjectName:String): List<SubjectWithStudents>
 
  @Transaction
 @Query("SELECT * FROM student WHERE studentName =:studentName")
 suspend fun getSubjectsOfStudents(studentName:String): List<StudentWithSubjects>

```

queremos crear una base de datos, entonces en nuestro root package hacemos una nueva clase

```kotlin
 abstract class SchoolDataBase

```




