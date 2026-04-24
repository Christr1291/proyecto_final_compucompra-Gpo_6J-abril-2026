Fase 1: Creación de la Habilidad Global .agents
Primero, vamos a crear la estructura de tu agente de automatización (diseño, código y scraping).

Abre tu terminal y ejecuta los siguientes comandos para crear la estructura de carpetas:

Bash
mkdir -p .agents/scripts
mkdir -p .agents/ejemplos
mkdir -p .agents/resources
touch .agents/SKILL.md
Contenido sugerido para .agents/SKILL.md:

Markdown
# Agent Skill: Autodev & Scraping
**Descripción:** Habilidad global para automatizar tareas de diseño UI, generación de código Flutter/Dart y web scraping de datos.
**IDE Soportados:** VS Code, Antigravity IDE.
**Herramientas:** Flutter, Firebase, Dart.
Fase 2: Prerrequisitos y Entorno de Trabajo
Vamos a verificar y preparar tu entorno. Abre tu terminal (en VS Code o Antigravity IDE) y sigue estos pasos:

1. Verificar Flutter:

Bash
flutter doctor
Asegúrate de que no haya errores críticos (marcas rojas) en la instalación de Flutter o Android Studio/SDK.

2. Instalar Firebase CLI (Flutterbase):
Si no tienes Firebase CLI instalado (requiere Node.js):

Bash
npm install -g firebase-tools
3. Conectar con Firebase:

Bash
firebase login
Esto abrirá tu navegador. Inicia sesión con tu cuenta de Google.

4. Instalar FlutterFire CLI:
Esta herramienta conecta tu proyecto Flutter con Firebase automáticamente.

Bash
dart pub global activate flutterfire_cli
Fase 3: Creación del Proyecto proyectocompucompra
1. Crear el proyecto en Flutter:

Bash
flutter create proyectocompucompra
cd proyectocompucompra
2. Configurar Firebase Firestore:
Ve a la Consola de Firebase, crea un proyecto llamado CompuCompra, y en el menú lateral ve a Compilación > Base de datos de Firestore. Haz clic en "Crear base de datos" (puedes iniciar en Modo de prueba para desarrollo).

3. Vincular Flutter con Firebase:
Desde la terminal en la carpeta de tu proyecto, ejecuta:

Bash
flutterfire configure
Selecciona el proyecto que acabas de crear en la consola y las plataformas (Android, iOS).

4. Agregar dependencias a pubspec.yaml:
Instala los paquetes necesarios ejecutando:

Bash
flutter pub add firebase_core cloud_firestore
Fase 4: Estructura de Carpetas y Código Dart
Dentro de tu carpeta lib/, vamos a crear la siguiente estructura para mantener un código limpio (MVC/Servicios):

Plaintext
lib/
├── models/
│   └── computadora.dart
├── screens/
│   ├── home_screen.dart
│   └── edit_screen.dart
├── services/
│   └── firestore_service.dart
├── main.dart
└── firebase_options.dart (Autogenerado por flutterfire)
Crea estas carpetas y archivos, y pega el código correspondiente:

1. lib/models/computadora.dart (Modelo de Datos)
Dart
class Computadora {
  String? id;
  String marca;
  String modelo;
  double precio;

  Computadora({this.id, required this.marca, required this.modelo, required this.precio});

  Map<String, dynamic> toMap() {
    return {
      'marca': marca,
      'modelo': modelo,
      'precio': precio,
    };
  }

  factory Computadora.fromMap(Map<String, dynamic> map, String documentId) {
    return Computadora(
      id: documentId,
      marca: map['marca'] ?? '',
      modelo: map['modelo'] ?? '',
      precio: (map['precio'] ?? 0).toDouble(),
    );
  }
}
2. lib/services/firestore_service.dart (Lógica del CRUD)
Dart
import 'package:cloud_firestore/cloud_firestore.dart';
import '../models/computadora.dart';

class FirestoreService {
  final CollectionReference _computadorasCollection =
      FirebaseFirestore.instance.collection('computadoras');

  // Create
  Future<void> addComputadora(Computadora computadora) {
    return _computadorasCollection.add(computadora.toMap());
  }

  // Read
  Stream<List<Computadora>> getComputadoras() {
    return _computadorasCollection.snapshots().map((snapshot) {
      return snapshot.docs.map((doc) {
        return Computadora.fromMap(doc.data() as Map<String, dynamic>, doc.id);
      }).toList();
    });
  }

  // Update
  Future<void> updateComputadora(Computadora computadora) {
    return _computadorasCollection.doc(computadora.id).update(computadora.toMap());
  }

  // Delete
  Future<void> deleteComputadora(String id) {
    return _computadorasCollection.doc(id).delete();
  }
}
3. lib/screens/home_screen.dart (Pantalla de Inicio / Leer y Borrar)
Dart
import 'package:flutter/material.dart';
import '../models/computadora.dart';
import '../services/firestore_service.dart';
import 'edit_screen.dart';

class HomeScreen extends StatelessWidget {
  final FirestoreService _firestoreService = FirestoreService();

  HomeScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('CompuCompra - Inventario'),
        backgroundColor: Colors.blueAccent,
      ),
      body: StreamBuilder<List<Computadora>>(
        stream: _firestoreService.getComputadoras(),
        builder: (context, snapshot) {
          if (snapshot.hasError) return const Center(child: Text('Error al cargar datos'));
          if (snapshot.connectionState == ConnectionState.waiting) {
            return const Center(child: CircularProgressIndicator());
          }

          final computadoras = snapshot.data ?? [];

          if (computadoras.isEmpty) {
            return const Center(child: Text('No hay computadoras registradas.'));
          }

          return ListView.builder(
            itemCount: computadoras.length,
            itemBuilder: (context, index) {
              final compu = computadoras[index];
              return ListTile(
                leading: const Icon(Icons.computer),
                title: Text('${compu.marca} ${compu.modelo}'),
                subtitle: Text('\$${compu.precio.toStringAsFixed(2)}'),
                trailing: Row(
                  mainAxisSize: MainAxisSize.min,
                  children: [
                    IconButton(
                      icon: const Icon(Icons.edit, color: Colors.orange),
                      onPressed: () => Navigator.push(
                        context,
                        MaterialPageRoute(builder: (context) => EditScreen(computadora: compu)),
                      ),
                    ),
                    IconButton(
                      icon: const Icon(Icons.delete, color: Colors.red),
                      onPressed: () => _firestoreService.deleteComputadora(compu.id!),
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
          MaterialPageRoute(builder: (context) => const EditScreen()),
        ),
        child: const Icon(Icons.add),
      ),
    );
  }
}
4. lib/screens/edit_screen.dart (Pantalla de Formulario / Crear y Actualizar)
Dart
import 'package:flutter/material.dart';
import '../models/computadora.dart';
import '../services/firestore_service.dart';

class EditScreen extends StatefulWidget {
  final Computadora? computadora;

  const EditScreen({super.key, this.computadora});

  @override
  State<EditScreen> createState() => _EditScreenState();
}

class _EditScreenState extends State<EditScreen> {
  final _formKey = GlobalKey<FormState>();
  final _marcaController = TextEditingController();
  final _modeloController = TextEditingController();
  final _precioController = TextEditingController();
  final FirestoreService _firestoreService = FirestoreService();

  @override
  void initState() {
    super.initState();
    if (widget.computadora != null) {
      _marcaController.text = widget.computadora!.marca;
      _modeloController.text = widget.computadora!.modelo;
      _precioController.text = widget.computadora!.precio.toString();
    }
  }

  void _saveData() {
    if (_formKey.currentState!.validate()) {
      final compu = Computadora(
        id: widget.computadora?.id,
        marca: _marcaController.text,
        modelo: _modeloController.text,
        precio: double.parse(_precioController.text),
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
        title: Text(widget.computadora == null ? 'Agregar Computadora' : 'Editar Computadora'),
      ),
      body: Padding(
        padding: const EdgeInsets.all(16.0),
        child: Form(
          key: _formKey,
          child: Column(
            children: [
              TextFormField(
                controller: _marcaController,
                decoration: const InputDecoration(labelText: 'Marca (Ej. Dell, HP)'),
                validator: (value) => value!.isEmpty ? 'Campo requerido' : null,
              ),
              TextFormField(
                controller: _modeloController,
                decoration: const InputDecoration(labelText: 'Modelo'),
                validator: (value) => value!.isEmpty ? 'Campo requerido' : null,
              ),
              TextFormField(
                controller: _precioController,
                decoration: const InputDecoration(labelText: 'Precio'),
                keyboardType: TextInputType.number,
                validator: (value) => value!.isEmpty ? 'Campo requerido' : null,
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
5. lib/main.dart (Punto de entrada)
Dart
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
      debugShowCheckedModeBanner: false,
      title: 'CompuCompra',
      theme: ThemeData(
        primarySwatch: Colors.blue,
      ),
      home: HomeScreen(),
    );
  }
}
Para probar tu aplicación en el emulador de Android o en la web, simplemente ejecuta en tu terminal:

Bash
flutter run
