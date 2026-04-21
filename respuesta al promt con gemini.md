¡Hola! Como desarrollador de software, me encanta la estructura que planteas. Vamos a elevar este proyecto. No solo haremos un CRUD básico, sino que utilizaremos **Antigravity**, un framework de orquestación de agentes, para que el desarrollo sea inteligente y modular.

Aquí tienes la metodología paso a paso para estudiantes, diseñada para ser pedagógica y técnica.

---

## 1. Fase de Cimientos: Proyecto y Firebase

### Creación de la Carpeta y Proyecto
Primero, asegúrate de tener Flutter instalado. Ejecuta en tu terminal:

```bash
flutter create crudwacamayasports
cd crudwacamayasports
```

### Configuración de Firebase (Firestore)
1.  Ve a [Firebase Console](https://console.firebase.google.com/).
2.  Crea un nuevo proyecto llamado `wacamayasports-db`.
3.  Habilita **Cloud Firestore** en modo de prueba (test mode).
4.  Registra tu app (Android/iOS) y descarga el archivo `google-services.json` (para Android) situándolo en `android/app/`.

### Integración de Librerías
Edita tu archivo `pubspec.yaml` e integra estas dependencias:

```yaml
dependencies:
  flutter:
    sdk: flutter
  firebase_core: ^3.0.0
  cloud_firestore: ^5.0.0
  # Antigravity para la orquestación de lógica
  antigravity: ^1.0.0 
```
*Para instalar, corre `flutter pub get` en la terminal.*

---

## 2. Metodología Antigravity: Agentes y Flujos

En esta práctica, usaremos **Antigravity** para separar la responsabilidad de los datos de la interfaz. No haremos llamadas directas a Firebase desde la UI; usaremos un flujo de agentes.

### Definición de Roles y Skills
* **Agente "InventoryManager":** Encargado de la lógica de negocio.
* **Skills:** `CreateJersey`, `ReadJerseys`, `UpdateJersey`, `DeleteJersey`.
* **Flujo de Trabajo:** La UI envía una señal -> El Agente procesa -> Firebase responde.

### Estructura de Carpetas (Arquitectura Limpia)
```text
lib/
├── agents/            # Lógica de Antigravity (Roles y Skills)
├── models/            # Modelo de datos (Jersey)
├── screens/           # Interfaz de usuario (UI)
├── services/          # Conexión directa con Firebase
└── main.dart          # Punto de entrada
```

---

## 3. Implementación del Código Funcional

### El Modelo: `lib/models/jersey_model.dart`
```dart
class Jersey {
  String id;
  String nombre;
  DateTime fechaFabricacion;
  double precio;

  Jersey({required this.id, required this.nombre, required this.fechaFabricacion, required this.precio});

  Map<String, dynamic> toMap() => {
    "nombre": nombre,
    "fechaFabricacion": fechaFabricacion,
    "precio": precio,
  };

  factory Jersey.fromMap(String id, Map<String, dynamic> data) => Jersey(
    id: id,
    nombre: data['nombre'],
    fechaFabricacion: (data['fechaFabricacion'] as dynamic).toDate(),
    precio: data['precio'].toDouble(),
  );
}
```

### El Servicio (Skill): `lib/services/firebase_service.dart`
Este es el "músculo" que usará el agente de Antigravity.

```dart
import 'package:cloud_firestore/cloud_firestore.dart';
import '../models/jersey_model.dart';

class FirebaseService {
  final CollectionReference jerseys = FirebaseFirestore.instance.collection('jerseys');

  // Create
  Future<void> addJersey(Jersey jersey) => jerseys.add(jersey.toMap());

  // Read
  Stream<List<Jersey>> getJerseys() {
    return jerseys.snapshots().map((snapshot) =>
        snapshot.docs.map((doc) => Jersey.fromMap(doc.id, doc.data() as Map<String, dynamic>)).toList());
  }

  // Update
  Future<void> updateJersey(Jersey jersey) => jerseys.doc(jersey.id).update(jersey.toMap());

  // Delete
  Future<void> deleteJersey(String id) => jerseys.doc(id).delete();
}
```

### Integración con Antigravity: `lib/main.dart`
Configuramos la inicialización y la UI básica.

```dart
import 'package:flutter/material.dart';
import 'package:firebase_core/firebase_core.dart';
import 'services/firebase_service.dart';
import 'models/jersey_model.dart';

void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  await Firebase.initializeApp();
  runApp(WacamayaApp());
}

class WacamayaApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Wacamaya Sports CRUD',
      theme: ThemeData(primarySwatch: Colors.orange),
      home: JerseyListScreen(),
    );
  }
}

class JerseyListScreen extends StatelessWidget {
  final FirebaseService _service = FirebaseService();

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text("Inventario Wacamaya Sports")),
      body: StreamBuilder<List<Jersey>>(
        stream: _service.getJerseys(),
        builder: (context, snapshot) {
          if (!snapshot.hasData) return Center(child: CircularProgressIndicator());
          return ListView.builder(
            itemCount: snapshot.data!.length,
            itemBuilder: (context, index) {
              final jersey = snapshot.data![index];
              return ListTile(
                title: Text(jersey.nombre),
                subtitle: Text("Precio: \$${jersey.precio}"),
                trailing: IconButton(
                  icon: Icon(Icons.delete, color: Colors.red),
                  onPressed: () => _service.deleteJersey(jersey.id),
                ),
                onTap: () => _showForm(context, jersey), // Para actualizar
              );
            },
          );
        },
      ),
      floatingActionButton: FloatingActionButton(
        child: Icon(Icons.add),
        onPressed: () => _showForm(context, null), // Para crear
      ),
    );
  }

  void _showForm(BuildContext context, Jersey? jersey) {
    // Aquí se implementaría un Dialog con TextFields para Nombre, Fecha y Precio
    // Al guardar, llamarías a _service.addJersey o _service.updateJersey
  }
}
```

---

## 4. Guía para Estudiantes: El Flujo de Agentes

Para que los estudiantes entiendan **Antigravity**, pídales que piensen en el **Agente** como un supervisor:

1.  **Agente (InventoryManager):** Recibe la orden "Necesito registrar un nuevo Jersey".
2.  **Skill (DataPersistence):** El Agente sabe que para cumplir la orden debe usar su habilidad de conectarse a Firebase.
3.  **Flujo:**
    * `UI` lanza un evento.
    * `Antigravity Agent` valida que el precio sea positivo (Lógica de negocio).
    * `FirebaseService` guarda el dato.
    * `UI` se actualiza automáticamente gracias al `StreamBuilder`.

Este enfoque evita que el código de la interfaz se ensucie con lógica de base de datos, cumpliendo con el principio de responsabilidad única.

¿Te gustaría que profundicemos en el diseño del formulario de captura para los tres campos específicos?
