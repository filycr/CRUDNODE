# CRUDNODE
# 🛒 CRUD Productos — Node.js + MySQL + HTML

## 📁 Estructura del proyecto

```
crud-productos/
├── public/
│   └── index.html        ← Frontend (HTML + CSS + JS)
├── routes/
│   └── productos.js      ← Rutas API REST
├── db.js                 ← Conexión a MySQL
├── server.js             ← Servidor Express
├── database.sql          ← Script SQL (BD + tabla + datos)
└── package.json


### 2. Instalar dependencias
  cd crud-productos Ubicarse en la carpeta e intalar
npm install



## puertos disponibles
netstat -aon | findstr LISTENING   (Ejecutar en CMD)

### 3 modificar el archivo server.js
// server.js — Servidor principal Express
const express    = require('express');
const cors       = require('cors'); // es necesario la instalacion del npm install cors
const path       = require('path');

const app  = express();
const PORT = process.env.PORT || 3000;

// ── Middlewares ──
app.use(cors());
app.use(express.json());
app.use(express.urlencoded({ extended: true }));
app.use(express.static(path.join(__dirname, 'public')));

// ── Rutas API ──
app.use('/api/productos', require('./routes/productos'));

// ── Ruta principal → sirve el HTML el public es la ubicación del directorio público donde esta el HTML ──
app.get('/', (req, res) => {
  res.sendFile(path.join(__dirname, 'public', 'index.html'));
});

// ── 404 para rutas desconocidas ──
app.use((req, res) => {
  res.status(404).json({ ok: false, mensaje: 'Ruta no encontrada' });
});

// ── Arrancar servidor ──
app.listen(PORT, () => {
  console.log(` Servidor corriendo en http://localhost:${PORT}`);
});

###4 Modificar db.js
// db.js — Configuración de conexión a MySQL   instalar  npm install cors mysql2
const mysql = require('mysql2/promise');

const pool = mysql.createPool({
  host:            process.env.DB_HOST     || 'localhost',
  user:            process.env.DB_USER     || 'root',
  password:        process.env.DB_PASSWORD || '',
  database:        process.env.DB_NAME     || 'tienda_db',
  port:            process.env.DB_PORT     || 3306,
  waitForConnections: true,
  connectionLimit:    10,
  queueLimit:         0,
  charset:         'utf8mb4',
});

// Verificar conexión al iniciar
(async () => {
  try {
    const conn = await pool.getConnection();
    console.log('Conectado a MySQL correctamente');
    conn.release();
  } catch (err) {
    console.error('Error al conectar a MySQL:', err.message);
    process.exit(1);
  }
})();

module.exports = pool;

### 6 modificamos productos.js
// routes/productos.js — Rutas CRUD para productos  productos.js
const express = require('express');
const router  = express.Router();
const db      = require('../db');

// ── GET /api/productos ── Listar todos
router.get('/', async (req, res) => {
  try {
    const { buscar, categoria } = req.query;
    let sql    = 'SELECT * FROM productos WHERE activo = 1';
    const args = [];

    if (buscar) {
      sql += ' AND (nombre LIKE ? OR descripcion LIKE ?)';
      args.push(`%${buscar}%`, `%${buscar}%`);
    }
    if (categoria) {
      sql += ' AND categoria = ?';
      args.push(categoria);
    }

    sql += ' ORDER BY creado_en DESC';
    const [rows] = await db.query(sql, args);
    res.json({ ok: true, total: rows.length, datos: rows });
  } catch (err) {
    console.error(err);
    res.status(500).json({ ok: false, mensaje: 'Error al obtener productos' });
  }
});

// ── GET /api/productos/:id ── Obtener uno
router.get('/:id', async (req, res) => {
  try {
    const [rows] = await db.query(
      'SELECT * FROM productos WHERE id = ? AND activo = 1',
      [req.params.id]
    );
    if (!rows.length)
      return res.status(404).json({ ok: false, mensaje: 'Producto no encontrado' });
    res.json({ ok: true, datos: rows[0] });
  } catch (err) {
    console.error(err);
    res.status(500).json({ ok: false, mensaje: 'Error al obtener producto' });
  }
});

// ── POST /api/productos ── Crear
router.post('/', async (req, res) => {
  try {
    const { nombre, descripcion, precio, stock, categoria, imagen_url } = req.body;

    if (!nombre || !precio)
      return res.status(400).json({ ok: false, mensaje: 'nombre y precio son obligatorios' });

    const [result] = await db.query(
      `INSERT INTO productos (nombre, descripcion, precio, stock, categoria, imagen_url)
       VALUES (?, ?, ?, ?, ?, ?)`,
      [nombre, descripcion || null, precio, stock || 0, categoria || null, imagen_url || null]
    );

    const [rows] = await db.query('SELECT * FROM productos WHERE id = ?', [result.insertId]);
    res.status(201).json({ ok: true, mensaje: 'Producto creado', datos: rows[0] });
  } catch (err) {
    console.error(err);
    res.status(500).json({ ok: false, mensaje: 'Error al crear producto' });
  }
});

// ── PUT /api/productos/:id ── Actualizar
router.put('/:id', async (req, res) => {
  try {
    const { nombre, descripcion, precio, stock, categoria, imagen_url } = req.body;

    if (!nombre || !precio)
      return res.status(400).json({ ok: false, mensaje: 'nombre y precio son obligatorios' });

    const [result] = await db.query(
      `UPDATE productos
       SET nombre = ?, descripcion = ?, precio = ?, stock = ?, categoria = ?, imagen_url = ?
       WHERE id = ? AND activo = 1`,
      [nombre, descripcion || null, precio, stock || 0, categoria || null, imagen_url || null, req.params.id]
    );

    if (!result.affectedRows)
      return res.status(404).json({ ok: false, mensaje: 'Producto no encontrado' });

    const [rows] = await db.query('SELECT * FROM productos WHERE id = ?', [req.params.id]);
    res.json({ ok: true, mensaje: 'Producto actualizado', datos: rows[0] });
  } catch (err) {
    console.error(err);
    res.status(500).json({ ok: false, mensaje: 'Error al actualizar producto' });
  }
});

// ── DELETE /api/productos/:id ── Eliminar (soft delete)
router.delete('/:id', async (req, res) => {
  try {
    const [result] = await db.query(
      'UPDATE productos SET activo = 0 WHERE id = ? AND activo = 1',
      [req.params.id]
    );
    if (!result.affectedRows)
      return res.status(404).json({ ok: false, mensaje: 'Producto no encontrado' });
    res.json({ ok: true, mensaje: 'Producto eliminado' });
  } catch (err) {
    console.error(err);
    res.status(500).json({ ok: false, mensaje: 'Error al eliminar producto' });
  }
});

module.exports = router;

## 7 agregar los archvis index.html, estilos.css

###8 Modificar la collumna Stok donde aparec  la cantidad de produtos y tiene la frase uds cambiarla a pzs
