¡Entendido! Vamos a estructurar este proyecto paso a paso. Crearemos primero la definición de la habilidad de tu agente global y luego prepararemos el entorno de desarrollo para construir la aplicación `proyectocompucompra` con Flutter y Firebase Firestore. 

Como asistente de IA, te guiaré a través del código y la configuración con instrucciones precisas. *Nota: Asumo que al mencionar "flutterbase cli" te referías a **Firebase CLI** y **FlutterFire CLI**, que son las herramientas oficiales para esta integración.*

---

### 1. Estructura del Agente Global (`.agents`)

Primero, vamos a definir la estructura de carpetas para tus habilidades de automatización (diseño, código, scraping). En tu directorio de trabajo o repositorio, crea la siguiente estructura:

```text
.agents/
├── SKILL.md
├── scripts/
│   ├── design_agent.sh
│   ├── code_agent.sh
│   └── scraping_agent.sh
├── ejemplos/
└── resources/
```

**`SKILL.md`** (Plantilla base):
```markdown
# Habilidad: CompuCompra Dev Agent
Este agente automatiza la creación y mantenimiento de la aplicación móvil de venta de computadoras.

## Capacidades
- **Diseño**: Generación de interfaces UI/UX.
- **Código**: Generación de lógica CRUD en Dart/Flutter.
- **Scraping**: Recolección de datos de catálogos de hardware.

## Entorno
- **Lenguaje**: Dart
- **Framework**: Flutter
- **Base de datos**: Firebase Firestore
- **IDE Recomendado**: VS Code, Antigravity
```

---

### 2. Prerrequisitos y Configuración del Entorno

Abre tu terminal en VS Code (o Antigravity) y verifica que tienes todo lo necesario instalado.

**Paso 2.1: Verificar Flutter**
```bash
flutter --version
flutter doctor
```
*(Asegúrate de que no haya errores graves en el reporte de `flutter doctor` referentes a Android Studio/SDK).*

**Paso 2.2: Instalar Firebase CLI y Autenticación**
Si no tienes Firebase CLI instalado (requiere Node.js):
```bash
npm install -g firebase-tools
```
Inicia sesión en tu cuenta de Google donde creaste el proyecto en Firebase Console:
```bash
firebase login
```

**Paso 2.3: Instalar FlutterFire CLI**
Esta herramienta conecta tu proyecto Flutter con Firebase automáticamente:
```bash
dart pub global activate flutterfire_cli
```

---

### 3. Creación del Proyecto Flutter (`proyectocompucompra`)

En tu terminal, ejecuta:

```bash
flutter create proyectocompucompra
cd proyectocompucompra
```

**Configurar Firebase en el proyecto:**
Ejecuta el siguiente comando y selecciona tu proyecto creado en Firebase Console (asegúrate de marcar la plataforma Android/iOS):
```bash
flutterfire configure
```

**Instalar dependencias en `pubspec.yaml`:**
```bash
flutter pub add firebase_core cloud_firestore
```

---

### 4. Estructura de Archivos del CRUD de Computadoras

Dentro de la carpeta `lib/` de tu proyecto, crearemos la siguiente estructura:

```text
lib/
├── models/
│   └── computadora.dart
├── services/
│   └── firestore_service.dart
├── screens/
│   ├── home_screen.dart
│   └── add_edit_screen.dart
├── main.dart
└── firebase_options.dart (Generado automáticamente por FlutterFire)
```

---

### 5. Código Fuente del Proyecto

Copia y pega el siguiente código en los archivos correspondientes.

#### `lib/main.dart`
Inicializa Firebase y lanza la aplicación.

```dart
import 'package:flutter/material.dart';
import 'package:firebase_core/firebase_core.dart';
import 'firebase_options.dart';
import 'screens/home_screen.dart';

void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  await Firebase.initializeApp(
    options: DefaultFirebaseOptions.currentPlatform,
  );
  runApp(const CompuCompraApp());
}

class CompuCompraApp extends StatelessWidget {
  const CompuCompraApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'CompuCompra',
      theme: ThemeData(primarySwatch: Colors.blue),
      home: const HomeScreen(),
      debugShowCheckedModeBanner: false,
    );
  }
}
```

#### `lib/models/computadora.dart`
El modelo de datos.

```dart
class Computadora {
  String? id;
  String marca;
  String modelo;
  double precio;
  int stock;

  Computadora({this.id, required this.marca, required this.modelo, required this.precio, required this.stock});

  Map<String, dynamic> toMap() {
    return {
      'marca': marca,
      'modelo': modelo,
      'precio': precio,
      'stock': stock,
    };
  }

  factory Computadora.fromMap(Map<String, dynamic> map, String documentId) {
    return Computadora(
      id: documentId,
      marca: map['marca'] ?? '',
      modelo: map['modelo'] ?? '',
      precio: (map['precio'] ?? 0).toDouble(),
      stock: map['stock'] ?? 0,
    );
  }
}
```

#### `lib/services/firestore_service.dart`
La lógica de conexión a la base de datos (CRUD).

```dart
import 'package:cloud_firestore/cloud_firestore.dart';
import '../models/computadora.dart';

class FirestoreService {
  final CollectionReference _collection = FirebaseFirestore.instance.collection('computadoras');

  // CREAR
  Future<void> addComputadora(Computadora compu) {
    return _collection.add(compu.toMap());
  }

  // LEER
  Stream<List<Computadora>> getComputadoras() {
    return _collection.snapshots().map((snapshot) {
      return snapshot.docs.map((doc) {
        return Computadora.fromMap(doc.data() as Map<String, dynamic>, doc.id);
      }).toList();
    });
  }

  // ACTUALIZAR
  Future<void> updateComputadora(Computadora compu) {
    return _collection.doc(compu.id).update(compu.toMap());
  }

  // BORRAR
  Future<void> deleteComputadora(String id) {
    return _collection.doc(id).delete();
  }
}
```

#### `lib/screens/home_screen.dart`
Pantalla principal de navegación y listado (Read & Delete).

```dart
import 'package:flutter/material.dart';
import '../models/computadora.dart';
import '../services/firestore_service.dart';
import 'add_edit_screen.dart';

class HomeScreen extends StatelessWidget {
  const HomeScreen({super.key});

  @override
  Widget build(BuildContext context) {
    final FirestoreService firestoreService = FirestoreService();

    return Scaffold(
      appBar: AppBar(title: const Text('CompuCompra - Inventario')),
      body: StreamBuilder<List<Computadora>>(
        stream: firestoreService.getComputadoras(),
        builder: (context, snapshot) {
          if (snapshot.connectionState == ConnectionState.waiting) {
            return const Center(child: CircularProgressIndicator());
          }
          if (!snapshot.hasData || snapshot.data!.isEmpty) {
            return const Center(child: Text('No hay computadoras en el inventario.'));
          }

          final computadoras = snapshot.data!;
          return ListView.builder(
            itemCount: computadoras.length,
            itemBuilder: (context, index) {
              final compu = computadoras[index];
              return ListTile(
                title: Text('${compu.marca} ${compu.modelo}'),
                subtitle: Text('Precio: \$${compu.precio} | Stock: ${compu.stock}'),
                trailing: Row(
                  mainAxisSize: MainAxisSize.min,
                  children: [
                    IconButton(
                      icon: const Icon(Icons.edit, color: Colors.blue),
                      onPressed: () => Navigator.push(
                        context,
                        MaterialPageRoute(builder: (_) => AddEditScreen(computadora: compu)),
                      ),
                    ),
                    IconButton(
                      icon: const Icon(Icons.delete, color: Colors.red),
                      onPressed: () => firestoreService.deleteComputadora(compu.id!),
                    ),
                  ],
                ),
              );
            },
          );
        },
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: () => Navigator.push(
          context,
          MaterialPageRoute(builder: (_) => const AddEditScreen()),
        ),
        child: const Icon(Icons.add),
      ),
    );
  }
}
```

#### `lib/screens/add_edit_screen.dart`
Formulario para Crear y Actualizar.

```dart
import 'package:flutter/material.dart';
import '../models/computadora.dart';
import '../services/firestore_service.dart';

class AddEditScreen extends StatefulWidget {
  final Computadora? computadora;
  const AddEditScreen({super.key, this.computadora});

  @override
  State<AddEditScreen> createState() => _AddEditScreenState();
}

class _AddEditScreenState extends State<AddEditScreen> {
  final FirestoreService _firestoreService = FirestoreService();
  final _formKey = GlobalKey<FormState>();
  
  late TextEditingController _marcaController;
  late TextEditingController _modeloController;
  late TextEditingController _precioController;
  late TextEditingController _stockController;

  @override
  void initState() {
    super.initState();
    _marcaController = TextEditingController(text: widget.computadora?.marca ?? '');
    _modeloController = TextEditingController(text: widget.computadora?.modelo ?? '');
    _precioController = TextEditingController(text: widget.computadora?.precio.toString() ?? '');
    _stockController = TextEditingController(text: widget.computadora?.stock.toString() ?? '');
  }

  @override
  void dispose() {
    _marcaController.dispose();
    _modeloController.dispose();
    _precioController.dispose();
    _stockController.dispose();
    super.dispose();
  }

  void _saveData() {
    if (_formKey.currentState!.validate()) {
      final compu = Computadora(
        id: widget.computadora?.id,
        marca: _marcaController.text,
        modelo: _modeloController.text,
        precio: double.parse(_precioController.text),
        stock: int.parse(_stockController.text),
      );

      if (widget.computadora == null) {
        _firestoreService.addComputadora(compu);
      } else {
        _firestoreService.updateComputadora(compu);
      }
      Navigator.pop(context);
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text(widget.computadora == null ? 'Nueva Computadora' : 'Editar Computadora'),
      ),
      body: Padding(
        padding: const EdgeInsets.all(16.0),
        child: Form(
          key: _formKey,
          child: ListView(
            children: [
              TextFormField(
                controller: _marcaController,
                decoration: const InputDecoration(labelText: 'Marca'),
                validator: (val) => val!.isEmpty ? 'Requerido' : null,
              ),
              TextFormField(
                controller: _modeloController,
                decoration: const InputDecoration(labelText: 'Modelo'),
                validator: (val) => val!.isEmpty ? 'Requerido' : null,
              ),
              TextFormField(
                controller: _precioController,
                decoration: const InputDecoration(labelText: 'Precio'),
                keyboardType: TextInputType.number,
                validator: (val) => val!.isEmpty ? 'Requerido' : null,
              ),
              TextFormField(
                controller: _stockController,
                decoration: const InputDecoration(labelText: 'Stock'),
                keyboardType: TextInputType.number,
                validator: (val) => val!.isEmpty ? 'Requerido' : null,
              ),
              const SizedBox(height: 20),
              ElevatedButton(
                onPressed: _saveData,
                child: const Text('Guardar'),
              )
            ],
          ),
        ),
      ),
    );
  }
}
```

**Verificación Final:**
Para comprobar la funcionalidad desde tu IDE (VS Code / Antigravity), ejecuta:
```bash
flutter run
```
Asegúrate de tener un emulador Android corriendo o un dispositivo físico conectado. La base de datos en Firebase Firestore debe estar en modo prueba o tener las reglas de seguridad configuradas para permitir la lectura y escritura.
