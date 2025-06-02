## 🎯 Configuración del Ecosistema RGS Completo - Arquitectura Multi-Juego

## 🏗️ Arquitectura del Sistema (Diagrama Visual, Centralizado)

```
game/cliente ──► engine ──┐
game/cliente ──► engine ──┤
game/cliente ──► engine ──┘
                           │
                        [RGS]──►[DB]
                           │
                        [Backoffice]
```

**Leyenda:**
- Cada **game/cliente** se comunica solo con su propio **engine**.
- **Todos los engines** se integran con el **mismo RGS**.
- El **RGS** es el hub central y único punto de integración con la base de datos y el Backoffice.
- **Solo hay un Backoffice** para todo el ecosistema.
- No hay comunicación directa entre cliente y BO, ni entre engines distintos.



## 🚀 Pasos para Levantar el Ecosistema

### 1. **Iniciar MongoDB**
```bash
mongod --dbpath C:\data\db
```

### 2. **Iniciar RGS Engine (Puerto 8000)**
```bash
cd RGS
.\venv\Scripts\Activate.ps1
python main.py
```

### 3. **Iniciar cada Game Engine (ejemplo Batalla Naval en 8001, Juego2 en 8002, ...)**
```bash
cd Batalla-Naval/engine
.\venv\Scripts\Activate.ps1
python main.py

cd ../../Juego2/engine
.\venv\Scripts\Activate.ps1
python main.py
```

### 4. **Iniciar Backoffice (Puerto 3000)**
```bash
cd backoffice
npm install
npm run dev
```

## 🔗 URLs del Ecosistema

| Componente        | URL                        | Descripción                |
|-------------------|----------------------------|----------------------------|
| **Backoffice**    | http://localhost:3000      | Panel de administración    |
| **RGS Engine**    | http://127.0.0.1:8000      | API Backend genérica       |
| **Batalla Naval** | http://127.0.0.1:8001      | Motor del juego específico |
| **Juego2**        | http://127.0.0.1:8002      | Motor del juego 2          |
| **RGS API Docs**  | http://127.0.0.1:8000/docs | Documentación RGS          |
| **Game API Docs** | http://127.0.0.1:8001/docs | Documentación BatallaNaval |
| **MongoDB**       | mongodb://localhost:27017  | Base de datos              |

## 🧩 Resumen de Responsabilidades

| Componente        | Dueño de...                | Integración                |
|-------------------|---------------------------|----------------------------|
| Game Engine(s)    | Lógica de cada juego       | HTTP con RGS y Frontend    |
| RGS Engine        | Usuarios, sesiones, dinero | HTTP con todos los juegos  |
| Backoffice        | Administración             | HTTP/MongoDB               |
| Frontend          | Experiencia de usuario     | HTTP con Game Engines      |
| MongoDB           | Persistencia               | Usada por RGS y juegos     |

## ➕ Cómo agregar un nuevo juego
1. Crea un nuevo directorio `JuegoX/engine/` siguiendo la estructura de Batalla-Naval.
2. Expón endpoints HTTP para el frontend y para integración con RGS.
3. Integra con RGS vía HTTP (no imports directos).
4. Agrega endpoints y vistas en el Backoffice si es necesario.
5. Actualiza este documento con la nueva ruta y puerto.

## 📝 Notas
- Toda integración entre apps es vía HTTP (no imports directos).
- No hay colas de eventos ni RabbitMQ.
- MongoDB es la única base de datos.
- Cada app es dueña de su dominio y lógica.
- La documentación y estructura están alineadas con el código real y son escalables a múltiples juegos.

## 🔑 Credenciales por Defecto

### Backoffice Admin
- **Usuario:** admin
- **Contraseña:** admin123

### Base de Datos
- **MongoDB:** Sin autenticación (desarrollo)
- **Database:** rgs_database

## 📡 APIs Principales

### RGS Engine (Puerto 8000)
```
POST /api/v1/backoffice/token          # Login backoffice
GET  /api/v1/backoffice/users          # Gestión usuarios
POST /api/v1/backoffice/operators      # Gestión operadores
GET  /api/v1/rgs/sessions/active       # Sesiones activas
POST /api/v1/rgs/transaction/bet       # Procesar apuestas
POST /api/v1/rgs/transaction/win       # Procesar ganancias
```

### Batalla Naval Engine (Puerto 8001)
```
POST /game/tablero?dificultad=medio    # Crear tablero
POST /game/disparo                     # Procesar disparo
GET  /game/health                      # Health check
```

## 🔄 Flujo de Comunicación

### 1. **Backoffice ↔ RGS**
- Gestión de usuarios, operadores, reportes
- Autenticación y autorización
- Estadísticas y métricas

### 2. **Batalla Naval ↔ RGS**
- Inicio/fin de sesiones de juego
- Procesamiento de apuestas y ganancias
- Registro de transacciones

### 3. **Cliente ↔ Batalla Naval**
- Lógica específica del juego
- Generación de tableros
- Procesamiento de disparos

## 🎮 Flujo de Juego Completo

1. **Admin gestiona** operadores en Backoffice
2. **Usuario inicia** partida en Batalla Naval
3. **Batalla Naval** comunica con RGS para iniciar sesión
4. **RGS** registra apuesta y sesión en MongoDB
5. **Usuario juega** → Batalla Naval procesa lógica
6. **Si hay ganancia** → Batalla Naval notifica a RGS
7. **RGS** actualiza balance y estadísticas
8. **Backoffice** muestra reportes en tiempo real

## ✅ Ventajas de esta Arquitectura

- **Sin duplicación** de código entre componentes
- **Separación clara** de responsabilidades
- **Escalabilidad** - fácil agregar nuevos juegos
- **Mantenibilidad** - cambios aislados por componente
- **Reutilización** - RGS sirve para cualquier juego

## 🧪 Pruebas de Integración

### Test 1: Crear Operador
```bash
# En Backoffice → RGS → MongoDB
1. Login en http://localhost:3000
2. Crear operador
3. Verificar en MongoDB: db.operators.find()
```

### Test 2: Jugar Batalla Naval
```bash
# Cliente → Batalla Naval → RGS → MongoDB
1. POST http://127.0.0.1:8001/game/tablero
2. POST http://127.0.0.1:8001/game/disparo
3. Verificar transacciones en Backoffice
```

## 🐛 Troubleshooting

### Error: CORS
```python
# En engine/main.py - Ya configurado
app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)
```

### Error: MongoDB Connection
```bash
# Verificar que MongoDB esté corriendo
mongosh --eval "db.adminCommand('ismaster')"
```

### Error: Port Already in Use
```bash
# Matar proceso en puerto
lsof -ti:8000 | xargs kill -9  # Engine
lsof -ti:3000 | xargs kill -9  # Backoffice
lsof -ti:3001 | xargs kill -9  # Game
```

## 📊 Monitoreo

### Logs del Engine
```bash
# Ver logs en tiempo real
tail -f engine.log
```

### Estado de la Base de Datos
```javascript
// En MongoDB
db.sessions.find().sort({created_at: -1}).limit(10)
db.transactions.find().sort({created_at: -1}).limit(10)
```

## 🚀 Próximos Pasos

1. **Implementar WebSockets** para actualizaciones en tiempo real
2. **Agregar más juegos** al ecosistema
3. **Implementar sistema de pagos** real
4. **Configurar SSL/HTTPS** para producción
5. **Agregar monitoreo** con Grafana/Prometheus

## 🌐 URLs de Desarrollo y Pruebas

- **Frontend de juegos (desarrollo local):**
  - Puedes servir el frontend de cada juego con un servidor estático (ej: VSCode Live Server, http-server, live-server, etc.)
  - Ejemplo:
    - Batalla Naval: `http://127.0.0.1:5500/Batalla-Naval/` o `http://localhost:5500/Batalla-Naval/`
    - Juego2: `http://127.0.0.1:5500/Juego2/`
- **Backoffice:**
  - `http://localhost:3000` (login admin, reportes, gestión)
  - Ejemplo de reporte: `http://localhost:3000/reports`
- **APIs:**
  - RGS: `http://localhost:8000/docs`
  - Batalla Naval Engine: `http://localhost:8001/docs`
  - Juego2 Engine: `http://localhost:8002/docs`

## 🔌 Integración Frontend <-> Engine (casino-integration.js)

- El archivo `casino-integration.js` es el puente entre el frontend del juego y el backend (engine).
- Ejemplo de inicialización y llamada:

```js
const casinoIntegration = new CasinoIntegration({
  apiUrl: 'http://localhost:8001', // URL del engine del juego
  onGameStart: () => { /* ... */ },
  onBet: (data) => { /* ... */ },
  // ... otros callbacks
});

// Ejemplo de llamada a la API del engine
fetch('http://localhost:8001/game/tablero', { method: 'POST' })
  .then(res => res.json())
  .then(data => console.log(data));
```
- Asegúrate de que el engine tenga habilitado CORS para permitir llamadas desde el frontend local.

## 🧪 Guía de Pruebas Manuales (End-to-End)

1. **Levanta todos los servicios:**
   - MongoDB, RGS, cada Engine, Backoffice.
2. **Abre el frontend del juego en el navegador:**
   - Ejemplo: `http://localhost:5500/Batalla-Naval/`
3. **Realiza una jugada:**
   - Verifica que el engine responde y actualiza el estado del juego.
4. **Verifica en el Backoffice:**
   - Ingresa a `http://localhost:3000` con usuario admin.
   - Consulta reportes y verifica que la jugada se registró.
5. **Verifica en MongoDB:**
   - Usa `mongosh` para consultar las colecciones relevantes (`games`, `transactions`, etc.).

## 🛠️ Recomendaciones para Desarrollo Local

- Usa extensiones como Live Server o http-server para servir el frontend en desarrollo.
- Si tienes problemas de CORS, asegúrate de que el engine y el RGS permitan orígenes cruzados (`allow_origins=["*"]` en FastAPI).
- Usa datos mock o entornos de staging para pruebas tempranas.
- Documenta y versiona los endpoints de cada engine y del RGS.

## ✅ Checklist de Integración QA

- [ ] El engine responde correctamente a las llamadas del frontend.
- [ ] El RGS registra las transacciones y sesiones.
- [ ] El Backoffice muestra los reportes esperados.
- [ ] MongoDB persiste los datos de juego y usuario.
- [ ] El frontend puede comunicarse con el engine y, si es necesario, con el RGS.
- [ ] No hay errores de CORS ni de autenticación.
- [ ] Los endpoints están documentados y versionados.

---

**🎉 ¡Ecosistema optimizado !** 
