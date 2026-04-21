¡Hola! Es un gusto saludarte. Como creador de software, me encanta la estructura que planteas. Vamos a construir este proyecto **CRUDLibreria** utilizando Flutter y Firebase, pero con el giro metodológico de **Antigravity**.

Un pequeño detalle de "colega": mencionaste que trabajaremos con "empleados" pero usaremos campos de libros (**Título, Editorial y Costo**). Para que el ejercicio sea coherente, trataremos a estos registros como **Ejemplares** del inventario.

Aquí tienes la guía maestra para tus estudiantes.

---

## 1. Configuración Inicial y Firebase

Antes de tocar el código, necesitamos el "cerebro" de datos.

1.  **Crear carpeta:** En tu terminal, ejecuta `mkdir crudlibreria` y entra en ella.
2.  **Crear proyecto Flutter:** `flutter create .`
3.  **Consola Firebase:**
    * Ve a [Firebase Console](https://console.firebase.google.com/).
    * Crea un proyecto llamado "CRUD Libreria".
    * Añade una aplicación Android/iOS.
    * **Habilita Firestore Database** en modo de prueba y elige una ubicación de servidor.
    * Crea una colección llamada `libros`.

---

## 2. Metodología Antigravity: Agentes y Roles

Para que los estudiantes entiendan cómo se orquesta un desarrollo moderno, definiremos el flujo de trabajo basado en **Agentes de IA/Desarrollo**.

### Estructura de Agentes
| Agente | Rol | Skill (Habilidad) |
| :--- | :--- | :--- |
| **Architect** | Definir estructura de datos y carpetas. | Diseño de Arquitectura Clean. |
| **Cloud Engineer** | Conexión con Firebase. | Configuración de `firebase_core`. |
| **Dev Frontend** | Crear interfaces de usuario. | Manejo de Widgets y Formularios. |
| **Dev Backend** | Lógica de negocio (CRUD). | Dominio de `cloud_firestore`. |

### Flujo de Trabajo (Workflow)
1.  **Definición:** El Architect crea la estructura de carpetas.
2.  **Integración:** El Cloud Engineer vincula el archivo `google-services.json` y dependencias.
3.  **Modelado:** Se crea la clase `Libro`.
4.  **Implementación:** Se escriben las funciones Create, Read, Update, Delete.
5.  **Refactor:** Limpieza de código.

---

## 3. Configuración de Dependencias

En tu archivo `pubspec.yaml`, añade las siguientes líneas bajo `dependencies`:

```yaml
dependencies:
  flutter:
    sdk: flutter
  firebase_core: ^3.0.0
  cloud_firestore: ^5.0.0
```

Luego ejecuta en terminal: `flutter pub get`.

---

## 4. Estructura de Carpetas del Proyecto

Siguiendo la metodología de orden profesional (Architect Role):

```text
crudlibreria/
├── lib/
│   ├── models/
│   │   └── libro_model.dart
│   ├── services/
│   │   └── firebase_service.dart
│   ├── screens/
│   │   ├── home_screen.dart
│   │   └── add_edit_screen.dart
│   └── main.dart
└── pubspec.yaml
```

---

## 5. Implementación del Código Funcional

### A. El Modelo (models/libro_model.dart)
Define qué es un libro en nuestro sistema.

```dart
class Libro {
  String id;
  String titulo;
  String editorial;
  double costo;

  Libro({required this.id, required this.titulo, required this.editorial, required this.costo});

  // Convertir de Firestore a Objeto
  factory Libro.fromFirestore(Map<String, dynamic> data, String id) {
    return Libro(
      id: id,
      titulo: data['titulo'] ?? '',
      editorial: data['editorial'] ?? '',
      costo: (data['costo'] ?? 0).toDouble(),
    );
  }

  // Convertir de Objeto a JSON para Firestore
  Map<String, dynamic> toMap() {
    return {
      'titulo': titulo,
      'editorial': editorial,
      'costo': costo,
    };
  }
}
```

### B. El Servicio CRUD (services/firebase_service.dart)
Aquí reside la lógica del Backend Dev.

```dart
import 'cloud_firestore.dart';
import '../models/libro_model.dart';

final FirebaseFirestore _db = FirebaseFirestore.instance;

// CREATE
Future<void> addLibro(String titulo, String editorial, double costo) async {
  await _db.collection('libros').add({
    'titulo': titulo,
    'editorial': editorial,
    'costo': costo,
  });
}

// READ (Stream para actualizar en tiempo real)
Stream<List<Libro>> getLibros() {
  return _db.collection('libros').snapshots().map((snapshot) =>
      snapshot.docs.map((doc) => Libro.fromFirestore(doc.data(), doc.id)).toList());
}

// UPDATE
Future<void> updateLibro(String id, String titulo, String editorial, double costo) async {
  await _db.collection('libros').doc(id).update({
    'titulo': titulo,
    'editorial': editorial,
    'costo': costo,
  });
}

// DELETE
Future<void> deleteLibro(String id) async {
  await _db.collection('libros').doc(id).delete();
}
```

### C. Pantalla Principal (screens/home_screen.dart)
Visualización de los datos.

```dart
import 'package:flutter/material.dart';
import '../services/firebase_service.dart';
import '../models/libro_model.dart';
import 'add_edit_screen.dart';

class HomeScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text("Librería Antigravity")),
      body: StreamBuilder<List<Libro>>(
        stream: getLibros(),
        builder: (context, snapshot) {
          if (!snapshot.hasData) return Center(child: CircularProgressIndicator());
          final libros = snapshot.data!;
          return ListView.builder(
            itemCount: libros.length,
            itemBuilder: (context, index) {
              return ListTile(
                title: Text(libros[index].titulo),
                subtitle: Text("${libros[index].editorial} - \$${libros[index].costo}"),
                trailing: Row(
                  mainAxisSize: MainAxisSize.min,
                  children: [
                    IconButton(
                      icon: Icon(Icons.edit, color: Colors.blue),
                      onPressed: () => Navigator.push(context, MaterialPageRoute(
                        builder: (context) => AddEditScreen(libro: libros[index]))),
                    ),
                    IconButton(
                      icon: Icon(Icons.delete, color: Colors.red),
                      onPressed: () => deleteLibro(libros[index].id),
                    ),
                  ],
                ),
              );
            },
          );
        },
      ),
      floatingActionButton: FloatingActionButton(
        child: Icon(Icons.add),
        onPressed: () => Navigator.push(context, MaterialPageRoute(builder: (context) => AddEditScreen())),
      ),
    );
  }
}
```

### D. Pantalla de Formulario (screens/add_edit_screen.dart)

```dart
import 'package:flutter/material.dart';
import '../services/firebase_service.dart';
import '../models/libro_model.dart';

class AddEditScreen extends StatefulWidget {
  final Libro? libro;
  AddEditScreen({this.libro});

  @override
  _AddEditScreenState createState() => _AddEditScreenState();
}

class _AddEditScreenState extends State<AddEditScreen> {
  final _tituloCtrl = TextEditingController();
  final _editorialCtrl = TextEditingController();
  final _costoCtrl = TextEditingController();

  @override
  void initState() {
    if (widget.libro != null) {
      _tituloCtrl.text = widget.libro!.titulo;
      _editorialCtrl.text = widget.libro!.editorial;
      _costoCtrl.text = widget.libro!.costo.toString();
    }
    super.initState();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text(widget.libro == null ? "Agregar" : "Editar")),
      body: Padding(
        padding: EdgeInsets.all(16.0),
        child: Column(
          children: [
            TextField(controller: _tituloCtrl, decoration: InputDecoration(labelText: "Título")),
            TextField(controller: _editorialCtrl, decoration: InputDecoration(labelText: "Editorial")),
            TextField(controller: _costoCtrl, decoration: InputDecoration(labelText: "Costo"), keyboardType: TextInputType.number),
            SizedBox(height: 20),
            ElevatedButton(
              onPressed: () async {
                if (widget.libro == null) {
                  await addLibro(_tituloCtrl.text, _editorialCtrl.text, double.parse(_costoCtrl.text));
                } else {
                  await updateLibro(widget.libro!.id, _tituloCtrl.text, _editorialCtrl.text, double.parse(_costoCtrl.text));
                }
                Navigator.pop(context);
              },
              child: Text("Guardar"),
            )
          ],
        ),
      ),
    );
  }
}
```

### E. Archivo Principal (main.dart)

```dart
import 'package:flutter/material.dart';
import 'package:firebase_core/firebase_core.dart';
import 'screens/home_screen.dart';

void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  await Firebase.initializeApp(); // Inicialización clave
  runApp(MaterialApp(
    home: HomeScreen(),
    debugShowCheckedModeBanner: false,
  ));
}
```

---

### Reflexión de la Práctica
Este flujo enseña a los estudiantes que el software no es solo escribir código, sino separar responsabilidades:
1.  **Firebase** maneja la persistencia.
2.  **Antigravity** organiza quién hace qué (Roles).
3.  **Flutter** se encarga de que todo se vea bien.

¿Hay alguna parte específica del flujo de agentes de Antigravity que te gustaría profundizar para la clase?
