# 🦷 Guía Completa: "Clinica Dental" en Flutter + Firebase (Antigravity IDE)

Esta guía está optimizada para **Google Antigravity IDE** (2026), con arquitectura escalable, gestión de estado moderna, CRUD completo, facturación y diseño profesional para Android, Web y Windows.

---

## 📦 1. Configuración Inicial & Firebase

```bash
flutter create clinica_dental --platforms android,web,windows
cd clinica_dental
dart pub global activate flutterfire_cli
flutterfire configure --project=proyectoclinica --platforms=android,web,windows
```
> ✅ **Firebase Console**: Habilita `Authentication > Email/Password` y crea `Firestore Database` (modo prueba inicialmente, luego aplica reglas de producción).

---

## 📦 2. `pubspec.yaml` (Librerías Esenciales)

```yaml
name: clinica_dental
description: Aplicación de gestión para clínica dental.
publish_to: 'none'
version: 1.0.0+1

environment:
  sdk: '>=3.4.0 <4.0.0'
  flutter: '>=3.24.0'

dependencies:
  flutter:
    sdk: flutter
  # Firebase
  firebase_core: ^3.8.0
  firebase_auth: ^5.3.0
  cloud_firestore: ^5.4.0
  # Estado & Navegación
  flutter_riverpod: ^2.5.0
  riverpod_annotation: ^2.3.0
  go_router: ^14.2.0
  # UI & Formularios
  google_fonts: ^6.1.0
  flutter_form_builder: ^9.2.0
  form_builder_validators: ^11.0.0
  flutter_slidable: ^3.1.0
  shimmer: ^3.0.0
  intl: ^0.19.0
  fluttertoast: ^8.2.0
  # Facturación & PDF
  pdf: ^3.10.0
  printing: ^5.11.0
  uuid: ^4.4.0
  # Utilidades
  cached_network_image: ^3.3.0
  collection: ^1.18.0

dev_dependencies:
  flutter_test:
    sdk: flutter
  flutter_lints: ^4.0.0
  build_runner: ^2.4.0
  riverpod_generator: ^2.4.0
  custom_lint: ^0.6.0
```

### 📖 Explicación de dependencias:
| Paquete | Propósito |
|---------|-----------|
| `firebase_*` | Core, Auth (email/pass), Firestore (CRUD + offline) |
| `flutter_riverpod` + `riverpod_annotation` | Gestión de estado moderna, type-safe, compilación automática |
| `go_router` | Enrutamiento declarativo con guards de autenticación |
| `flutter_form_builder` + `validators` | Formularios CRUD dinámicos y validación nativa |
| `google_fonts` | Tipografía profesional (`Inter`, `Poppins`) |
| `pdf` + `printing` | Generación y visualización de facturas |
| `flutter_slidable` | Swipe-to-delete en listas |
| `shimmer` | Loading states elegantes |
| `uuid` | IDs únicos para documentos en Firestore |

---

## 📁 3. Estructura de Carpetas (`lib/`)

```
lib/
├── main.dart                          # Entry point, inicializa Firebase & Riverpod
├── firebase_options.dart              # Generado automáticamente por flutterfire configure
└── src/
    ├── core/
    │   ├── constants/
    │   │   ├── app_colors.dart        # Paleta moderna
    │   │   └── app_strings.dart       # Textos reutilizables
    │   ├── theme/
    │   │   └── app_theme.dart         # ThemeData completo
    │   ├── utils/
    │   │   ├── firestore_crud.dart    # Repositorio genérico
    │   │   └── invoice_generator.dart # Lógica de facturación PDF
    │   └── widgets/
    │       ├── custom_button.dart     # Botones consistentes
    │       ├── custom_text_field.dart # Campos estilizados
    │       └── loading_shimmer.dart   # Placeholders
    ├── features/
    │   ├── auth/                      # Login/Registro
    │   ├── dashboard/                 # Menú principal
    │   ├── patients/                  # CRUD Pacientes
    │   ├── doctors/                   # CRUD Doctores
    │   ├── services/                  # CRUD Servicios
    │   ├── appointments/              # CRUD Citas
    │   ├── users/                     # CRUD Usuarios (staff)
    │   └── billing/                   # Facturación & Historial
    └── routing/
        └── app_router.dart            # go_router + AuthGuard
```

### 📖 Explicación por carpeta:
| Carpeta | Responsabilidad |
|---------|-----------------|
| `core/` | Configuración global, temas, utilidades reutilizables, widgets base |
| `features/` | Arquitectura por feature: cada módulo contiene `data/`, `domain/`, `presentation/` |
| `routing/` | Definición de rutas, protección por autenticación, navegación adaptativa |

---

## 🎨 4. Tema Visual Moderno (`app_theme.dart`)

```dart
import 'package:flutter/material.dart';
import 'package:google_fonts/google_fonts.dart';
import '../constants/app_colors.dart';

class AppTheme {
  static ThemeData get lightTheme => ThemeData(
    useMaterial3: true,
    fontFamily: GoogleFonts.inter().fontFamily,
    colorScheme: ColorScheme.fromSeed(
      seedColor: AppColors.primary,
      brightness: Brightness.light,
    ),
    appBarTheme: AppBarTheme(
      backgroundColor: AppColors.primary,
      foregroundColor: AppColors.surface,
      elevation: 0,
    ),
    cardTheme: CardTheme(
      elevation: 2,
      shape: RoundedRectangleBorder(borderRadius: BorderRadius.circular(12)),
    ),
    inputDecorationTheme: InputDecorationTheme(
      filled: true,
      fillColor: AppColors.surface,
      border: OutlineInputBorder(borderRadius: BorderRadius.circular(8)),
      focusedBorder: OutlineInputBorder(
        borderSide: BorderSide(color: AppColors.secondary, width: 2),
        borderRadius: BorderRadius.circular(8),
      ),
    ),
    elevatedButtonTheme: ElevatedButtonThemeData(
      style: ElevatedButton.styleFrom(
        backgroundColor: AppColors.primary,
        foregroundColor: AppColors.surface,
        padding: EdgeInsets.symmetric(vertical: 14),
        shape: RoundedRectangleBorder(borderRadius: BorderRadius.circular(8)),
      ),
    ),
  );
}
```

**Paleta (`app_colors.dart`)**:
```dart
class AppColors {
  static const primary = Color(0xFF0077B6);   // Azul clínico
  static const secondary = Color(0xFF00B4D8); // Cian fresco
  static const accent = Color(0xFF48CAE4);    // Azul claro
  static const background = Color(0xFFF8F9FA);
  static const surface = Color(0xFFFFFFFF);
  static const error = Color(0xFFE63946);
  static const textPrimary = Color(0xFF1D3557);
  static const textSecondary = Color(0xFF6C757D);
}
```

---

## 🤖 5. Estructura de Agentes en Antigravity (`.agents/`)

```
.agents/
├── skills/
│   ├── firebase-auth/
│   │   └── SKILL.md
│   ├── firestore-crud/
│   │   └── SKILL.md
│   ├── pdf-billing/
│   │   └── SKILL.md
│   └── ui-dashboard/
│       └── SKILL.md
├── workflows/
│   ├── /setup-project.md
│   ├── /generate-crud.md
│   └── /deploy-firebase.md
└── scripts/
    └── test-emulators.sh
```

### 📄 Ejemplo: `skills/firestore-crud/SKILL.md`
```markdown
---
name: firestore-crud
description: Genera repositorios genéricos para entidades de Clínica Dental con soporte offline y manejo de errores.
---

# Firestore CRUD Skill

## Reglas de Generación
1. Usa `FirestoreRepository<T>` genérico
2. Incluye `Stream<List<T>> watchAll()` para UI reactiva
3. Maneja `FirebaseFirestoreException` con mensajes en español
4. Estructura: `data/`, `domain/`, `presentation/` por feature
5. Habilita offline: `FirebaseFirestore.instance.settings = const Settings(persistenceEnabled: true)`

## Entidades soportadas
- Patient, Doctor, Service, Appointment, Invoice, User
```

---

## ⚙️ 6. Código Base Esencial

### 🔹 `main.dart`
```dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import 'package:firebase_core/firebase_core.dart';
import 'firebase_options.dart';
import 'src/core/theme/app_theme.dart';
import 'src/routing/app_router.dart';

void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  await Firebase.initializeApp(options: DefaultFirebaseOptions.currentPlatform);
  runApp(ProviderScope(child: ClinicaDentalApp()));
}

class ClinicaDentalApp extends ConsumerWidget {
  const ClinicaDentalApp({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    return MaterialApp.router(
      title: 'Clínica Dental',
      theme: AppTheme.lightTheme,
      routerConfig: appRouter,
      debugShowCheckedModeBanner: false,
    );
  }
}
```

### 🔹 `core/utils/firestore_crud.dart` (Repositorio Genérico)
```dart
import 'package:cloud_firestore/cloud_firestore.dart';

class FirestoreRepository<T> {
  final CollectionReference<T> collection;

  FirestoreRepository(this.collection);

  Future<String> create(T data, String id) async {
    await collection.doc(id).set(data);
    return id;
  }

  Future<T?> read(String id) async {
    final doc = await collection.doc(id).get();
    return doc.data();
  }

  Future<void> update(String id, T data) => collection.doc(id).update(data);

  Future<void> delete(String id) => collection.doc(id).delete();

  Stream<List<T>> watchAll() => collection.snapshots().map((s) =>
      s.docs.map((d) => d.data()).toList());
}
```

### 🔹 `routing/app_router.dart` (go_router + AuthGuard)
```dart
import 'package:flutter/material.dart';
import 'package:go_router/go_router.dart';
import 'package:firebase_auth/firebase_auth.dart';
import '../features/auth/presentation/login_screen.dart';
import '../features/dashboard/presentation/dashboard_screen.dart';

final _auth = FirebaseAuth.instance;

final appRouter = GoRouter(
  initialLocation: '/login',
  redirect: (context, state) {
    final user = _auth.currentUser;
    final isLoggingIn = state.uri.path == '/login';
    return user == null ? (isLoggingIn ? null : '/login') : (isLoggingIn ? '/dashboard' : null);
  },
  routes: [
    GoRoute(path: '/login', builder: (_, __) => const LoginScreen()),
    GoRoute(path: '/dashboard', builder: (_, __) => const DashboardScreen()),
    // Rutas CRUD se añaden dinámicamente o por feature
  ],
);
```

### 🔹 `features/auth/presentation/login_screen.dart` (Snippet)
```dart
import 'package:flutter/material.dart';
import 'package:firebase_auth/firebase_auth.dart';
import '../../../core/utils/custom_toast.dart';

class LoginScreen extends StatefulWidget {
  const LoginScreen({super.key});
  @override
  State<LoginScreen> createState() => _LoginScreenState();
}

class _LoginScreenState extends State<LoginScreen> {
  final _emailCtrl = TextEditingController();
  final _passCtrl = TextEditingController();
  final _auth = FirebaseAuth.instance;

  Future<void> _login() async {
    try {
      await _auth.signInWithEmailAndPassword(
        email: _emailCtrl.text.trim(),
        password: _passCtrl.text.trim(),
      );
      if (mounted) Navigator.pushReplacementNamed(context, '/dashboard');
    } on FirebaseAuthException catch (e) {
      if (mounted) CustomToast.error(e.message ?? 'Error desconocido');
    }
  }

  @override
  Widget build(BuildContext context) => Scaffold(
    body: Padding(
      padding: const EdgeInsets.all(24),
      child: Column(
        mainAxisAlignment: MainAxisAlignment.center,
        children: [
          Text('Clínica Dental', style: Theme.of(context).textTheme.headlineLarge),
          const SizedBox(height: 32),
          TextField(controller: _emailCtrl, decoration: InputDecoration(labelText: 'Correo')),
          const SizedBox(height: 16),
          TextField(controller: _passCtrl, obscureText: true, decoration: InputDecoration(labelText: 'Contraseña')),
          const SizedBox(height: 24),
          ElevatedButton(onPressed: _login, child: Text('Iniciar Sesión')),
          TextButton(onPressed: () => _register(), child: Text('¿No tienes cuenta? Regístrate')),
        ],
      ),
    ),
  );
}
```

### 🔹 `core/utils/invoice_generator.dart` (Facturación PDF)
```dart
import 'package:pdf/pdf.dart';
import 'package:pdf/widgets.dart' as pw;
import 'package:printing/printing.dart';

Future<void> generateInvoice({
  required String patientName,
  required String serviceName,
  required double amount,
  required String doctorName,
  required DateTime date,
}) async {
  final pdf = pw.Document();
  pdf.addPage(pw.Page(
    build: (context) => pw.Column(
      crossAxisAlignment: pw.CrossAxisAlignment.start,
      children: [
        pw.Text('FACTURA - Clínica Dental', style: pw.TextStyle(fontSize: 24, fontWeight: pw.FontWeight.bold)),
        pw.SizedBox(height: 20),
        pw.Text('Paciente: $patientName'),
        pw.Text('Servicio: $serviceName'),
        pw.Text('Monto: \$${amount.toStringAsFixed(2)}'),
        pw.Text('Doctor: $doctorName'),
        pw.Text('Fecha: ${date.toLocal()}'),
        pw.SizedBox(height: 40),
        pw.Text('Gracias por su confianza.', style: pw.TextStyle(fontStyle: pw.FontStyle.italic)),
      ],
    ),
  ));
  await Printing.layoutPdf(onLayout: (format) async => pdf.save());
}
```

---

## 🔒 7. Reglas de Seguridad Firestore (`firestore.rules`)

```text
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    function isSignedIn() { return request.auth != null; }
    
    match /users/{userId} {
      allow read, write: if isSignedIn() && request.auth.uid == userId;
    }
    match /patients/{patientId}, /doctors/{doctorId}, /services/{serviceId}, 
          /appointments/{apptId}, /invoices/{invId} {
      allow read: if isSignedIn();
      allow create, update, delete: if isSignedIn() && request.resource.data.createdBy == request.auth.uid;
    }
  }
}
```
> 📌 Despliega con: `firebase deploy --only firestore:rules`

---

## 🚀 8. Flujo de Trabajo en Antigravity IDE

1. **Abre el proyecto** en Antigravity.
2. **Activa Agent Manager** (`Ctrl+Shift+A` o ícono lateral).
3. **Escribe el prompt maestro**:
   ```
   @clinica_lead Configura el proyecto "clinica_dental" con:
   - Auth email/password (login + registro)
   - Firestore CRUD para: pacientes, doctores, servicios, citas, usuarios, facturación
   - Dashboard con navegación lateral y botones (Crear, Editar, Listar, Borrar)
   - Tema moderno (azul clínico)
   - Genera PDF de facturas
   Usa las skills: firebase-auth, firestore-crud, pdf-billing, ui-dashboard
   Activa workflow: /setup-project
   ```
4. **Revisa los artifacts**: Antigravity generará `Implementation Plan`, `Code Diffs`, `Walkthrough`.
5. **Aprueba cambios** → El IDE aplica código, corre `flutter pub get`, y muestra preview en 3 plataformas.
6. **Prueba en emuladores**: Ejecuta `firebase emulators:start --only auth,firestore` desde terminal integrada.
7. **Build final**: `flutter build apk`, `flutter build web`, `flutter build windows`.

---

## ✅ Checklist de Producción

- [ ] `flutterfire configure` ejecutado sin errores
- [ ] Auth email/pass funciona en Android, Web, Windows
- [ ] Firestore CRUD genérico implementado en cada feature
- [ ] Dashboard con navegación y botones CRUD funcionales
- [ ] Facturación genera PDF y almacena metadatos en Firestore
- [ ] Reglas de seguridad desplegadas y validadas
- [ ] Offline persistence activo (`Settings(persistenceEnabled: true)`)
- [ ] Builds multiplataforma exitosos
- [ ] Agentes Antigravity configurados con skills reutilizables

---

¿Quieres que te genere:
1. 📄 El `SKILL.md` completo para `/generate-crud` con templates de formularios por entidad?
2. 🔄 El workflow `/deploy-firebase` con CI/CD para GitHub Actions + Firebase Hosting?
3. 📱 El código completo de `DashboardScreen` con `go_router` y navegación lateral?

Indícalo y te entrego el siguiente bloque listo para copiar/pegar en Antigravity. 🦷✨
