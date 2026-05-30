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
const express = require('express');
const app = express();
const path = require('path');
app.get('/', (req, res) => {
    res.sendFile(path.join(__dirname, "index.html"));
});
app.listen(3000, () => {
    console.log('Servidor escuchando en el puerto 3000   http://localhost:3000');
});
