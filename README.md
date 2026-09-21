# Dwarf Fortress — Backend

API REST hecha con Express, Prisma y SQLite. Gestiona el registro de personas (enanos) de la fortaleza: nombre, apellido, edad, fecha de llegada y estado laboral.

## Versiones verificadas

| Herramienta         | Versión  |
|----------------------|----------|
| Node.js              | 20.20.2  |
| npm                   | 10.9.2   |
| Prisma / Client       | 6.19.0   |
| TypeScript            | 5.9.3    |

## Instalación

```bash
npm install
```

## Preparar la base de datos

```bash
npx prisma generate
npm run db:init
```

- `prisma generate` crea el Prisma Client a partir de `prisma/schema.prisma`.
- `db:init` crea (o adapta) la tabla `Person` en SQLite y agrega datos de ejemplo si la base está vacía. Si ya se corrió el proyecto antes con una versión anterior del esquema, este script agrega las columnas nuevas sin borrar los datos existentes.

## Verificar que compila sin errores

```bash
npx tsc --noEmit
npm run build
```

## Correr el servidor

```bash
npm run dev
```

Dejar esta terminal abierta, el proceso queda escuchando en `http://localhost:3000` (o el puerto definido en `PORT`). Otros scripts disponibles: `build` (compila a `dist/`), `start` (corre la versión compilada con `node dist/index.js`).

## Endpoints

| Método | Ruta               | Descripción                          |
|--------|--------------------|---------------------------------------|
| GET    | /api/health        | Chequeo de que el servidor está vivo |
| GET    | /api/personas      | Lista todas las personas              |
| POST   | /api/personas      | Crea una persona nueva                |
| PATCH  | /api/personas/:id  | Actualiza campos de una persona       |
| DELETE | /api/personas/:id  | Elimina una persona                   |

### Modelo `Person`

| Campo         | Tipo      | Descripción                        |
|----------------|-----------|-------------------------------------|
| id             | Int       | Identificador autoincremental       |
| firstName      | String    | Nombre                              |
| lastName       | String    | Apellido                            |
| edad           | Int       | Edad                                |
| arrivalDate    | DateTime? | Fecha de llegada al fuerte          |
| isWorking      | Boolean   | Estado laboral (default: false)     |
| createdAt      | DateTime  | Fecha automática de alta            |

### Probar la API sin abrir la app (PowerShell)

```powershell
# Salud del servidor
Invoke-RestMethod http://localhost:3000/api/health

# Listar personas
Invoke-RestMethod http://localhost:3000/api/personas

# Crear una persona
$body = @{
  firstName = "Mina"
  lastName = "Miner"
  edad = 30
  arrivalDate = "2026-09-17"
  isWorking = $true
} | ConvertTo-Json
Invoke-RestMethod http://localhost:3000/api/personas -Method Post -ContentType "application/json" -Body $body

# Actualizar solo el estado laboral
$body = @{ isWorking = $false } | ConvertTo-Json
Invoke-RestMethod http://localhost:3000/api/personas/1 -Method Patch -ContentType "application/json" -Body $body

# Eliminar una persona
Invoke-RestMethod http://localhost:3000/api/personas/1 -Method Delete
```

## Errores habituales

- **Prisma no encuentra el cliente**: correr `npx prisma generate` dentro de `backend`. Si la tabla no existe, correr `npm run db:init`.
- **PATCH no actualiza**: comprobar que el `id` exista y que el body tenga al menos un campo.
- **Cannot find module o errores de tipos**: correr `npm install` dentro de esta carpeta (backend tiene su propio `package.json`, separado del de la app).
