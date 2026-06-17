# venta-garage-baradero
const express = require('express');
const fs = require('fs');
const path = require('path');

const app = express();

// Aumentar límite de tamaño JSON para permitir imágenes Base64
app.use(express.json({ limit: '50mb' }));
app.use(express.static('public'));

const DB_PATH = './data/db.json';

// Leer base de datos
function readDB() {
  return JSON.parse(fs.readFileSync(DB_PATH, 'utf8'));
}

// Guardar base de datos
function writeDB(data) {
  fs.writeFileSync(DB_PATH, JSON.stringify(data, null, 2));
}

// Registro de usuario
app.post('/register', (req, res) => {
  const db = readDB();
  const { email, password, nombre } = req.body;

  // Verificar si el usuario ya existe
  if (db.users.find(u => u.email === email)) {
    return res.json({ ok: false, message: 'El usuario ya existe' });
  }

  const newUser = {
    id: Date.now(),
    email,
    password, // En producción NUNCA guardar contraseñas en texto plano
    nombre,
    fecha: new Date().toLocaleString('es-AR')
  };

  db.users.push(newUser);
  writeDB(db);

  res.json({ ok: true, message: 'Usuario registrado exitosamente', user: { id: newUser.id, nombre, email } });
});

// Login de usuario
app.post('/login', (req, res) => {
  const db = readDB();
  const { email, password } = req.body;

  const user = db.users.find(u => u.email === email && u.password === password);

  if (!user) {
    return res.json({ ok: false, message: 'Email o contraseña incorrectos' });
  }

  res.json({ ok: true, user: { id: user.id, nombre: user.nombre, email: user.email } });
});

// Obtener publicaciones
app.get('/posts', (req, res) => {
  const db = readDB();
  res.json(db.posts);
});

// Crear publicación
app.post('/posts', (req, res) => {
  const db = readDB();
  
  // DEBUG: Ver el body completo
  console.log('=== POST /posts ===');
  console.log('req.body keys:', Object.keys(req.body));
  console.log('Full req.body:', JSON.stringify(req.body, null, 2));
  
  const { titulo, precio, descripcion, categoria, usuarioId, usuarioNombre, imagenes } = req.body;

  if (!usuarioId) {
    return res.json({ ok: false, message: 'Debes estar registrado' });
  }

  const nuevoPost = {
    id: Date.now(),
    titulo,
    precio,
    descripcion,
    categoria,
    usuarioId,
    usuarioNombre,
    fecha: new Date().toLocaleString('es-AR'),
    imagenes: imagenes || []
  };

  db.posts.unshift(nuevoPost);
  writeDB(db);

  res.json({ ok: true, message: 'Publicación creada', post: nuevoPost });
});

// Eliminar publicación
app.delete('/posts/:id', (req, res) => {
  const db = readDB();
  const postId = parseInt(req.params.id);

  db.posts = db.posts.filter(p => p.id !== postId);
  writeDB(db);

  res.json({ ok: true, message: 'Publicación eliminada' });
});

const PORT = process.env.PORT || 3000;

app.listen(PORT, () => {
  console.log('🚗 Servidor corriendo en http://localhost:' + PORT);
});

