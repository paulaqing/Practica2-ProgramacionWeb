# 🛍️ Portal de E-commerce con GraphQL

> Aplicación completa de comercio electrónico con autenticación JWT, gestión de productos, carrito de compra, sistema de pedidos y chat en tiempo real.

## 📋 Tabla de Contenidos

- [Características](#características)
- [Tecnologías Utilizadas](#tecnologías-utilizadas)
- [Estructura del Proyecto](#estructura-del-proyecto)
- [Instalación y Configuración](#instalación-y-configuración)
- [Modelos de Datos](#modelos-de-datos)
- [Esquema GraphQL](#esquema-graphql)
- [API REST](#api-rest)
- [Sistema de Chat](#sistema-de-chat)
- [Decisiones de Diseño](#decisiones-de-diseño)
- [Uso de la Aplicación](#uso-de-la-aplicación)

---

## ✨ Características

### Funcionalidades Principales

#### 👤 **Sistema de Usuarios**
- Registro y login con autenticación JWT
- Roles diferenciados: `user` y `admin`
- Los usuarios solo se registran como `user` (seguridad)
- Gestión completa de usuarios para administradores

#### 🛍️ **Gestión de Productos**
- CRUD completo para administradores
- Subida de imágenes de productos 
- Visualización pública de productos
- Búsqueda y filtrado

#### 🛒 **Carrito de Compra**
- Añadir/eliminar productos
- Ajustar cantidades
- Persistencia en base de datos
- Cálculo automático de totales

#### 📦 **Sistema de Pedidos**
- Creación de pedidos via GraphQL
- Estados: `pending` (en curso) y `completed` (completado)
- Historial de pedidos por usuario
- Panel de gestión para administradores

#### 💬 **Chat en Tiempo Real**
- Comunicación entre usuarios y administradores
- Historial persistente (últimos 50 mensajes)
- Salas privadas 1 a 1

#### 🎨 **Interfaz Moderna**
- Diseño con gradientes y animaciones
- Responsive design
- Efectos hover y transiciones
- Experiencia de usuario optimizada

---

## 🚀 Tecnologías Utilizadas

### Backend
- **Node.js** - Entorno de ejecución
- **Express 5** - Framework web
- **MongoDB** - Base de datos NoSQL
- **Mongoose** - ODM para MongoDB
- **GraphQL** - API de consultas (express-graphql)
- **Socket.IO** - Comunicación en tiempo real
- **JWT** - Autenticación mediante tokens
- **Multer** - Manejo de archivos/imágenes
- **bcrypt** - Hash de contraseñas

### Frontend
- **Vanilla JavaScript** - Sin frameworks adicionales
- **HTML5/CSS3** - Estructura y estilos modernos
- **Socket.IO Client** - Cliente WebSocket

---

## 📁 Estructura del Proyecto

```
portal-productos-chat/
├── src/
│   ├── config.js                 # Variables de entorno
│   ├── server.js                 # Punto de entrada principal
│   │
│   ├── models/                   # Modelos de Mongoose
│   │   ├── User.js              # Usuario (con carrito)
│   │   ├── Product.js           # Producto (con imagen)
│   │   ├── Order.js             # Pedido
│   │   └── Message.js           # Mensaje de chat
│   │
│   ├── routes/                   # Rutas REST
│   │   ├── authRoutes.js        # Login/Registro
│   │   ├── productRoutes.js     # CRUD Productos
│   │   ├── cartRoutes.js        # Gestión de carrito
│   │   ├── userRoutes.js        # Gestión de usuarios
│   │   └── orderRoutes.js       # Gestión de pedidos
│   │
│   ├── middleware/               # Middlewares
│   │   ├── authenticateJWT.js   # Autenticación JWT
│   │   └── upload.js            # Multer config
│   │
│   ├── graphql/                  # GraphQL
│   │   ├── schema.js            # Esquema GraphQL
│   │   └── resolvers.js         # Resolvers
│   │
│   └── public/                   # Frontend
│       ├── index.html           # Login/Inicio
│       ├── products.html        # Productos
│       ├── cart.html            # Carrito
│       ├── my_orders.html       # Mis pedidos
│       ├── admin.html           # Panel admin
│       ├── chat.html            # Chat
│       └── uploads/             # Imágenes subidas
│
├── .env                          # Variables de entorno
├── package.json
└── README.md
```

---

## 🗄️ Modelos de Datos

### User
```javascript
{
  username: String (único),
  password: String (hasheado),
  role: String (enum: 'user', 'admin'),
  cart: [{
    product: ObjectId (ref: Product),
    quantity: Number
  }],
  timestamps: true
}
```

### Product
```javascript
{
  name: String,
  description: String,
  price: Number,
  image: String (ruta a la imagen),
  timestamps: true
}
```

### Order
```javascript
{
  user: ObjectId (ref: User),
  products: [{
    product: ObjectId,
    name: String,      // snapshot del nombre
    price: Number,     // snapshot del precio
    quantity: Number
  }],
  total: Number,
  status: String (enum: 'pending', 'completed'),
  timestamps: true
}
```

### Message
```javascript
{
  room: String,        // Identificador de la sala
  sender: String,      // Username del emisor
  message: String,     // Contenido del mensaje
  timestamp: Date,
  timestamps: true
}
```

---

## 🔷 Esquema GraphQL

### Tipos Definidos

```graphql
type User {
  id: ID!
  username: String!
  role: String!
  createdAt: String
}

type Product {
  id: ID!
  name: String!
  description: String
  price: Float!
  createdAt: String
}

type Order {
  id: ID!
  user: User!
  products: [OrderProduct!]!
  total: Float!
  status: String!
  createdAt: String
}

type OrderProduct {
  product: ID
  name: String!
  price: Float!
  quantity: Int!
}
```

### Queries Disponibles

```graphql
# Productos
products: [Product!]!              # Todos los productos
product(id: ID!): Product          # Producto específico

# Pedidos
orders: [Order!]!                  # Todos (solo admin)
order(id: ID!): Order              # Pedido específico
myOrders: [Order!]!                # Mis pedidos

# Usuarios
users: [User!]!                    # Todos (solo admin)
user(id: ID!): User                # Usuario específico (solo admin)
```

### Mutations Disponibles

```graphql
# Crear pedido desde carrito
createOrder(products: [OrderProductInput!]!): Order!

# Actualizar estado de pedido (admin)
updateOrderStatus(id: ID!, status: String!): Order!

# Gestión de usuarios (admin)
deleteUser(id: ID!): DeleteResponse!
updateUserRole(id: ID!, role: String!): User!
```

### Inputs

```graphql
input OrderProductInput {
  productId: ID!
  quantity: Int!
}
```

---

## 🔌 API REST

### Autenticación

| Método | Endpoint | Descripción | Auth |
|--------|----------|-------------|------|
| POST | `/register` | Registrar nuevo usuario | No |
| POST | `/login` | Login y obtener JWT | No |

### Productos

| Método | Endpoint | Descripción | Auth |
|--------|----------|-------------|------|
| GET | `/api/products` | Listar productos | JWT |
| POST | `/api/products` | Crear producto (con imagen) | Admin |
| PUT | `/api/products/:id` | Actualizar producto | Admin |
| DELETE | `/api/products/:id` | Eliminar producto | Admin |

### Carrito

| Método | Endpoint | Descripción | Auth |
|--------|----------|-------------|------|
| GET | `/api/cart` | Ver carrito | JWT |
| POST | `/api/cart/add` | Añadir producto | JWT |
| PUT | `/api/cart/update` | Actualizar cantidad | JWT |
| DELETE | `/api/cart/remove/:id` | Eliminar producto | JWT |
| DELETE | `/api/cart/clear` | Vaciar carrito | JWT |

### Usuarios (Admin)

| Método | Endpoint | Descripción | Auth |
|--------|----------|-------------|------|
| GET | `/api/users` | Listar usuarios | Admin |
| GET | `/api/users/:id` | Obtener usuario | Admin |
| PUT | `/api/users/:id/role` | Cambiar rol | Admin |
| DELETE | `/api/users/:id` | Eliminar usuario | Admin |

### Pedidos

| Método | Endpoint | Descripción | Auth |
|--------|----------|-------------|------|
| GET | `/api/orders` | Listar pedidos | Admin |
| GET | `/api/orders/my-orders` | Mis pedidos | JWT |
| GET | `/api/orders/:id` | Pedido específico | JWT |
| PUT | `/api/orders/:id/status` | Actualizar estado | Admin |

---

## 💬 Sistema de Chat

### Arquitectura

El sistema de chat utiliza **Socket.IO** para comunicación en tiempo real con las siguientes características:

#### Salas Privadas
- Cada conversación 1 a 1 tiene una sala única
- Formato: `chat-{usuario1}-{usuario2}` (ordenado alfabéticamente)
- Solo los participantes pueden acceder

#### Persistencia
- Últimos 50 mensajes por sala guardados en MongoDB
- Se eliminan automáticamente los mensajes más antiguos
- Historial disponible al reconectar

#### Eventos Socket.IO

```javascript
// Cliente → Servidor
socket.emit('getAvailableUsers')           // Obtener lista de usuarios
socket.emit('joinRoom', room)               // Unirse a sala
socket.emit('chatmessage', { message, room }) // Enviar mensaje

// Servidor → Cliente
socket.on('availableUsers', users)         // Lista de usuarios
socket.on('messageHistory', history)       // Historial de mensajes
socket.on('chatmessage', data)             // Nuevo mensaje
```

---

## 🎯 Decisiones de Diseño

### 1. **Arquitectura Híbrida: REST + GraphQL**

**Decisión:** Mantener REST para operaciones CRUD tradicionales y usar GraphQL para consultas complejas y relacionadas.

**Razones:**
- REST es más intuitivo para operaciones simples (login, subida de archivos)
- GraphQL es ideal para queries con relaciones (pedidos con usuarios y productos)
- Aprovechar las fortalezas de cada tecnología
- Facilita la migración gradual

**Implementación:**
```javascript
// REST para autenticación y archivos
app.use('/', authRoutes);
app.use('/api/products', productRoutes);

// GraphQL para queries complejas
app.use('/graphql', graphqlHTTP({ schema, rootValue: resolvers }));
```

---

### 2. **Persistencia del Carrito en Base de Datos**

**Decisión:** Guardar el carrito en el modelo User en lugar de localStorage.

**Razones:**
- ✅ Persiste entre dispositivos
- ✅ No se pierde al limpiar caché
- ✅ Disponible en servidor para validaciones
- ✅ Mejor para futura expansión (recomendaciones)

**Alternativas consideradas:**
- ❌ LocalStorage: Se pierde entre dispositivos
- ❌ Cookies: Limitación de tamaño
- ❌ Sesión temporal: Se pierde al cerrar navegador

---

### 3. **Snapshot de Productos en Pedidos**

**Decisión:** Guardar nombre y precio del producto en el momento de la compra.

**Razones:**
- ✅ Histórico consistente (aunque cambien precios)
- ✅ No afecta informes financieros
- ✅ Auditoría de transacciones

**Implementación:**
```javascript
orderProducts.push({
  product: product._id,
  name: product.name,      // Snapshot
  price: product.price,    // Snapshot
  quantity: item.quantity
});
```

---

### 4. **Sistema de Roles Restrictivo**

**Decisión:** No permitir auto-registro como admin.

**Razones:**
- 🔒 Seguridad: Solo el código puede crear admins
- 🔒 Control de acceso centralizado
- 🔒 Prevención de escalada de privilegios

**Implementación:**
```javascript
router.post('/register', async (req, res) => {
  // Siempre registra como 'user'
  const userRole = 'user';
  // ...
});
```

---

### 5. **Chat con Salas Ordenadas Alfabéticamente**

**Decisión:** Ordenar nombres de usuarios al crear sala.

**Razones:**
- ✅ Consistencia: Misma sala sin importar quién inicia
- ✅ Evita duplicados: `chat-alice-bob` === `chat-bob-alice`
- ✅ Simplifica búsquedas en BD

**Implementación:**
```javascript
const users = [currentUser, targetUser].sort();
const room = `chat-${users[0]}-${users[1]}`;
```

---

### 6. **Límite de 50 Mensajes por Chat**

**Decisión:** Mantener solo últimos 50 mensajes por sala.

**Razones:**
- ⚡ Rendimiento: Carga rápida del historial
- 💾 Espacio: Control del crecimiento de BD
- 📱 UX: Scroll manejable

**Implementación:**
```javascript
const count = await Message.countDocuments({ room });
if (count > 50) {
  const toDelete = await Message.find({ room })
    .sort({ createdAt: 1 })
    .limit(count - 50);
  await Message.deleteMany({ _id: { $in: toDelete } });
}
```

---

### 7. **Subida de Imágenes con Multer**

**Decisión:** Almacenar imágenes en servidor (no CDN externo).

**Razones:**
- ✅ Simplicidad: No requiere servicios externos
- ✅ Control total sobre archivos
- ✅ Gratis para desarrollo

**Configuración:**
```javascript
const storage = multer.diskStorage({
  destination: 'src/public/uploads/',
  filename: (req, file, cb) => {
    const uniqueName = `product-${Date.now()}-${Math.random()}`;
    cb(null, uniqueName + path.extname(file.originalname));
  }
});
```

---

### 8. **GraphQL sin Caché en Cliente**

**Decisión:** Usar fetch nativo en lugar de Apollo Client.

**Razones:**
- ✅ Menos dependencias
- ✅ Más control sobre requests
- ✅ Suficiente para este proyecto

---

### 9. **Autenticación JWT sin Refresh Tokens**

**Decisión:** JWT de 2 horas sin refresh token.

**Razones:**
- ✅ Simplicidad de implementación
- ✅ Suficiente para sesiones normales

---

### 10. **Diseño Visual Moderno**

**Decisión:** Gradientes, animaciones y efectos hover.

**Razones:**
- 🎨 Diferenciación visual
- 🎨 Feedback inmediato al usuario
- 🎨 Experiencia premium
- ⚡ Todo con CSS puro (sin librerías)

**Técnicas usadas:**
```css
/* Gradientes */
background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);

/* Animaciones */
@keyframes fadeIn {
  from { opacity: 0; transform: translateY(20px); }
  to { opacity: 1; transform: translateY(0); }
}

/* Hover effects */
.card:hover {
  transform: translateY(-5px);
  box-shadow: 0 8px 25px rgba(0,0,0,0.15);
}
```

---

## 🚀 Instalación y Configuración

### Requisitos Previos
- Node.js v18 o superior
- MongoDB instalado y corriendo
- npm o yarn

### Pasos de Instalación

1. **Clonar el repositorio**
```bash
git clone <url-del-repositorio>
cd portal-productos-chat
```

2. **Instalar dependencias**
```bash
npm install
```

3. **Configurar variables de entorno**

Crear archivo `.env` en la raíz:
```env
PORT=3000
MONGO_URI=mongodb://localhost:27017/portalProductos
JWT_SECRET=tu_secreto_super_seguro_aqui
```

4. **Crear carpeta de uploads**
```bash
mkdir -p src/public/uploads
```

5. **Iniciar el servidor**
```bash
npm start
```

6. **Acceder a la aplicación**
- Frontend: `http://localhost:3000`
- GraphQL Playground: `http://localhost:3000/graphql`

---

## 📖 Uso de la Aplicación

### Como Usuario Normal

1. **Registro y Login**
   - Registrarse con username y contraseña
   - Iniciar sesión

2. **Explorar Productos**
   - Ver catálogo de productos
   - Ver imágenes y precios

3. **Usar el Carrito**
   - Añadir productos al carrito
   - Ajustar cantidades
   - Ver total

4. **Realizar Pedido**
   - Finalizar compra (usa GraphQL internamente)
   - Ver estado del pedido en "Mis Pedidos"

5. **Chat**
   - Hablar con administradores
   - Conversar con otros usuarios
   - Historial persistente

### Como Administrador

1. **Gestionar Productos**
   - Crear productos con imágenes
   - Editar productos existentes
   - Eliminar productos

2. **Panel de Administración**
   - Ver todos los usuarios
   - Cambiar roles (user ↔ admin)
   - Eliminar usuarios

3. **Gestión de Pedidos**
   - Ver todos los pedidos
   - Filtrar por estado
   - Cambiar estado de pedidos

4. **Chat**
   - Responder a usuarios
   - Coordinar con otros admins

---

## 🧪 Testing


### Probar GraphQL

Accede a `http://localhost:3000/graphql` y prueba:

**Query de ejemplo:**
```graphql
query {
  products {
    id
    name
    price
  }
}
```

**Mutation de ejemplo (requiere autenticación):**
```graphql
mutation {
  createOrder(products: [
    { productId: "ID_DEL_PRODUCTO", quantity: 2 }
  ]) {
    id
    total
    status
  }
}
```

**Headers necesarios:**
```json
{
  "Authorization": "Bearer TU_TOKEN_JWT"
}
```

---

## 🔐 Seguridad

- ✅ Contraseñas hasheadas con bcrypt
- ✅ Autenticación JWT
- ✅ Validación de roles en cada endpoint
- ✅ Sanitización de inputs
- ✅ Límite de tamaño de archivos (5MB)
- ✅ Solo imágenes permitidas en uploads

---

## 📦 Dependencias Principales

```json
{
  "express": "^5.1.0",
  "mongoose": "^8.19.2",
  "jsonwebtoken": "^9.0.2",
  "bcrypt": "^6.0.0",
  "socket.io": "^4.8.1",
  "express-graphql": "^0.12.0",
  "graphql": "^15.8.0",
  "multer": "^1.4.5",
  "cors": "^2.8.5",
  "dotenv": "^17.2.3"
}
```
