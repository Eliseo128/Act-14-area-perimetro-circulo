# 🚀 Guía Actualizada: Agentes, Roles, Habilidades y Subagentes en Google Antigravity para Flutter + Firebase

Basado en la documentación oficial de [Google Antigravity](https://docs.flutter.dev/tools/antigravity) [[2]] y el codelab de configuración [[10]], aquí tienes la configuración **nativa y optimizada** para tu proyecto Flutter con Firebase (Auth Email/Pass + Firestore) en Android/Web/Windows.

---

## 📐 Arquitectura de Agentes en Antigravity

Antigravity opera con dos superficies principales [[12]]:
| Superficie | Propósito | Uso en Flutter + Firebase |
|------------|-----------|---------------------------|
| **Editor View** | Codificación hands-on con autocomplete, comandos inline | Desarrollo de widgets, servicios Firebase, pruebas unitarias |
| **Agent Manager** | Orquestación asíncrona de múltiples agentes | Delegar tareas complejas: "Implementa auth + Firestore offline" |

```
Proyecto Flutter-Firebase
└── Agent Manager (Mission Control)
    ├── 🔹 Agente Principal: `flutter_firebase_lead`
    │   ├── 🎯 Rol: Coordinar arquitectura y contratos de datos
    │   └── 📦 Subagentes delegados:
    │       ├── `auth_specialist` → Auth Email/Password + recuperación
    │       ├── `firestore_engineer` → CRUD, streams, reglas de seguridad
    │       ├── `multiplatform_builder` → Config Android/Web/Windows
    │       └── `qa_security_agent` → Tests, reglas, emuladores
    └── 🧰 Skills globales y por workspace
```

---

## ⚙️ Configuración Paso a Paso en Antigravity

### 1️⃣ Inicialización del Proyecto
```bash
# En terminal de Antigravity (Ctrl+` para alternar)
flutter create flutter_firebase_app --platforms android,web,windows
cd flutter_firebase_app
```

### 2️⃣ Configuración de Firebase con FlutterFire CLI
```bash
# Instalar CLI globalmente
dart pub global activate flutterfire_cli

# Configurar proyecto (Antigravity pedirá aprobación por política de terminal)
flutterfire configure \
  --project=tu-project-id \
  --platforms=android,web,windows \
  --out=lib/firebase_options.dart \
  --ios-bundle-id=com.example.app \
  --android-package-name=com.example.app \
  --web-app-id=tu-web-app-id
```
> ✅ **Política recomendada**: `Review-driven development` para aprobar comandos sensibles [[10]].

### 3️⃣ Crear Skills Especializadas (Workspace Scope)

Las **Skills** en Antigravity son paquetes de conocimiento que se activan según el contexto [[10]]. Crea esta estructura:

```
flutter_firebase_app/
└── .agents/
    └── skills/
        ├── firebase-auth-email/
        │   ├── SKILL.md          # Metadata + instrucciones
        │   └── templates/
        │       ├── auth_service.dart
        │       └── login_widget.dart
        ├── firestore-offline/
        │   ├── SKILL.md
        │   └── templates/
        │       ├── firestore_repository.dart
        │       └── security_rules.fir
        └── multiplatform-config/
            ├── SKILL.md
            └── templates/
                ├── android_proguard.pro
                ├── web_index.html
                └── windows_cmake.txt
```

#### 📄 Ejemplo: `firebase-auth-email/SKILL.md`
```markdown
---
name: firebase-auth-email
description: Implements Firebase Authentication with email/password for Flutter apps. Handles registration, login, password reset, and error mapping. Use when auth flows are requested.
---

# Firebase Auth Email/Password Skill

## Output Requirements
- Generate `lib/src/features/auth/data/auth_service.dart`
- Generate `lib/src/features/auth/presentation/login_screen.dart`
- Generate `lib/src/features/auth/presentation/register_screen.dart`

## Implementation Rules
1. Use `firebase_auth: ^5.0.0` (compatible with FlutterFire v13+)
2. Map `FirebaseAuthException` codes to user-friendly messages:
   - `weak-password` → "La contraseña debe tener al menos 6 caracteres"
   - `email-already-in-use` → "Este correo ya está registrado"
3. Include loading states and retry logic in UI
4. Persist session automatically (Firebase lo hace nativamente)

## Testing Checklist
- [ ] Unit test: `auth_service_test.dart` con mock de FirebaseAuth
- [ ] Widget test: Login form validation
- [ ] Integration test: Flujo completo con Firebase Emulators
```

#### 📄 Ejemplo: `firestore-offline/SKILL.md`
```markdown
---
name: firestore-offline
description: Implements Cloud Firestore with offline persistence, real-time streams, and security rules for Flutter apps.
---

# Firestore Offline-First Skill

## Output Requirements
- Generate `lib/src/core/services/firestore_service.dart`
- Generate `firestore.rules` con reglas MVP
- Generate `lib/src/features/data/models/user_model.dart`

## Implementation Rules
1. Habilitar persistencia offline: `FirebaseFirestore.instance.settings = Settings(persistenceEnabled: true)`
2. Usar `snapshots()` para streams reactivos en UI
3. Estructura de colecciones: `users/{uid}/profile`, `public/posts/{postId}`
4. Reglas de seguridad mínimas (ver snippet abajo)

## Security Rules Template
```text
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /users/{userId} {
      allow read, write: if request.auth != null && request.auth.uid == userId;
    }
    match /public/{docId} {
      allow read: if true;
      allow write: if request.auth != null && resource.data.authorId == request.auth.uid;
    }
  }
}
```
```

---

## 🎯 Crear Workflows para Tareas Recurrentes

Los **Workflows** son prompts guardados que se activan con `/` [[10]]. Crea estos en `.agents/workflows/`:

### 🔹 Workflow: `/setup-firebase-project`
```markdown
---
name: setup-firebase-project
description: Configura Firebase Console, habilita Auth/Firestore y genera reglas iniciales.
---

# Firebase Project Setup Workflow

## Steps
1. Verificar que `firebase_options.dart` esté generado
2. Habilitar Email/Password en Firebase Console (instrucciones para usuario)
3. Desplegar reglas de seguridad con: `firebase deploy --only firestore:rules`
4. Iniciar emuladores: `firebase emulators:start --only auth,firestore`
5. Generar archivo `.env.example` con variables necesarias

## Output Artifacts
- ✅ `firestore.rules` actualizado
- ✅ `firebase.json` configurado para emuladores
- ✅ `README.md` con pasos de configuración
```

### 🔹 Workflow: `/test-auth-flow`
```markdown
---
name: test-auth-flow
description: Ejecuta tests de integración para auth con Firebase Emulators.
---

# Auth Flow Testing Workflow

## Steps
1. Iniciar emuladores en background: `firebase emulators:start --only auth --import=./test-data`
2. Ejecutar integration tests: `flutter test integration_test/auth_flow_test.dart`
3. Generar reporte de cobertura: `genhtml coverage/lcov.info -o coverage/html`
4. Capturar screenshot del flujo exitoso (usar browser agent)

## Validation Criteria
- [ ] Login exitoso con usuario de prueba
- [ ] Manejo de error: contraseña incorrecta
- [ ] Recuperación de contraseña funcional
```

---

## 🤖 Orquestación en Agent Manager: Ejemplo Práctico

### Prompt inicial en Agent Manager:
```
@flutter_firebase_lead Crea una app Flutter con:
- Auth email/password (registro, login, reset)
- Firestore: colección users/{uid}/profile con offline sync
- Pantalla principal que muestre datos del usuario
- Soporte para Android, Web y Windows
- Tests unitarios y de integración con emuladores

Usa las skills: firebase-auth-email, firestore-offline, multiplatform-config
Activa el workflow: setup-firebase-project
```

### Flujo esperado del agente:
1. **Planning Mode**: Genera `Implementation Plan` y `Task List` como artifacts [[10]]
2. **Delegación a subagentes**:
   - `auth_specialist` → Implementa `AuthService` + pantallas de auth
   - `firestore_engineer` → Crea repositorio + reglas de seguridad
   - `multiplatform_builder` → Ajusta configs por plataforma
3. **Verificación con Artifacts**:
   - 📋 Task List aprobada por usuario
   - 💻 Code diffs revisables estilo Google Docs
   - 🎥 Browser recording del flujo de login funcionando
4. **QA automático**: `qa_security_agent` ejecuta tests con emuladores

---

## 🔐 Configuración de Seguridad y Permisos

En `Settings > Agent` de Antigravity [[10]]:

| Política | Configuración Recomendada | Razón |
|----------|----------------------------|-------|
| **Artifact Review** | `Asks for Review` | Validar planes antes de codificar |
| **Terminal Execution** | `Request Review` + Allowlist | Permitir `flutter`, `firebase`, `dart` sin aprobación; bloquear `rm`, `sudo` |
| **File Access** | Workspace-only por defecto | Evitar acceso a archivos sensibles del sistema |
| **Browser Policy** | Allowlist: `firebase.google.com`, `flutter.dev` | Prevenir prompt injection en navegación |

#### Allowlist de comandos seguros (`Settings > Permissions > Always Allow`):
```
command(flutter)
command(dart)
command(firebase)
command(git)
command(npm)      # Para dependencias web
command(choco)    # Para Windows (si usas Chocolatey)
```

---

## 🧪 Testing con Firebase Emulators en Antigravity

### Script de validación (`.agents/scripts/test_emulators.sh`):
```bash
#!/bin/bash
# Skill: qa_security_agent

echo "🚀 Iniciando emuladores de Firebase..."
firebase emulators:start \
  --only auth,firestore \
  --import=./test-data \
  --export-on-exit=./test-data &

EMULATOR_PID=$!
sleep 10  # Esperar que los emuladores estén listos

echo "🧪 Ejecutando tests de integración..."
flutter test integration_test/ \
  --dart-define=USE_EMULATORS=true \
  --coverage

echo "📊 Generando reporte de cobertura..."
genhtml coverage/lcov.info -o coverage/html

# Detener emuladores
kill $EMULATOR_PID
```

### Skill para activar este script (`.agents/skills/run-emulator-tests/SKILL.md`):
```markdown
---
name: run-emulator-tests
description: Ejecuta tests de integración con Firebase Emulators y genera reporte de cobertura.
---

# Emulator Testing Skill

## Usage
Trigger with: `/run-emulator-tests`

## Requirements
- Firebase CLI instalado globalmente
- Java 11+ para emuladores
- Chrome para browser agent (si se requiere UI testing)

## Output
- ✅ Reporte de cobertura en `coverage/html/index.html`
- ✅ Browser recording del flujo testeado
- ✅ Artifact: `test_summary.md` con resultados
```

---

## 📦 Estructura Final Recomendada del Proyecto

```
flutter_firebase_app/
├── lib/
│   ├── src/
│   │   ├── core/
│   │   │   ├── services/          # firebase_service.dart, auth_service.dart
│   │   │   └── utils/             # error_mapper.dart, constants.dart
│   │   ├── features/
│   │   │   ├── auth/
│   │   │   │   ├── data/          # auth_repository.dart
│   │   │   │   ├── domain/        # user_entity.dart
│   │   │   │   └── presentation/  # login_screen.dart, register_screen.dart
│   │   │   └── profile/
│   │   │       ├── data/          # profile_repository.dart
│   │   │       └── presentation/  # profile_screen.dart
│   │   └── routing/
│   │       └── app_router.dart    # go_router con auth guard
│   ├── firebase_options.dart      # Generado por flutterfire configure
│   └── main.dart
├── test/                          # Unit tests
├── integration_test/              # Integration tests con emuladores
├── .agents/                       # Configuración de Antigravity
│   ├── skills/                    # Skills especializadas
│   ├── workflows/                 # Workflows reutilizables
│   └── scripts/                   # Scripts de automatización
├── firebase.json                  # Config de emuladores y hosting
├── firestore.rules                # Reglas de seguridad
├── pubspec.yaml                   # Dependencias versionadas
└── README.md                      # Instrucciones de setup
```

---

## ✅ Checklist de Validación Final en Antigravity

- [ ] Skill `firebase-auth-email` genera código con manejo de errores mapeados
- [ ] Skill `firestore-offline` incluye persistencia offline habilitada
- [ ] Workflow `setup-firebase-project` despliega reglas correctamente
- [ ] Agent Manager muestra artifacts: `Implementation Plan`, `Walkthrough`, `Browser Recording`
- [ ] Tests de integración pasan con emuladores (`firebase emulators:start`)
- [ ] Builds exitosos: `flutter build apk`, `flutter build web`, `flutter build windows`
- [ ] Políticas de seguridad configuradas: terminal review, file access restringido

---

## 💡 Tips Pro para Antigravity + Flutter

1. **Usa `@` para contexto**: `@lib/src/features/auth Implementa recuperación de contraseña` para enfocar al agente.
2. **Activa Planning Mode** para tareas complejas: el agente generará artifacts revisables antes de codificar.
3. **Comenta en artifacts estilo Google Docs**: selecciona código o pasos y añade feedback; el agente iterará sin reiniciar.
4. **Undo granular**: si un cambio no te convence, usa `↩️ Undo changes up to this point` en el chat del agente.
5. **Skills globales para estándares de equipo**: coloca `code-style-guide` en `~/.gemini/antigravity/skills/` para aplicar en todos los proyectos.

---

> 📚 **Recursos oficiales**:
> - Documentación Antigravity: https://docs.flutter.dev/tools/antigravity [[2]]
> - Codelab de configuración: https://codelabs.developers.google.com/getting-started-google-antigravity [[10]]
> - Blog oficial: https://developers.googleblog.com/build-with-google-antigravity [[12]]

¿Quieres que te genere:
1. 📄 El archivo `SKILL.md` completo para `multiplatform-config` con configs específicas de Android/Web/Windows?
2. 🔄 Un workflow `/deploy-firebase` para automatizar despliegue a hosting + reglas?
3. 🧪 El código de `integration_test/auth_flow_test.dart` listo para emuladores?

Indícame cuál prefieres y te lo entrego con sintaxis exacta para Antigravity. 🚀
