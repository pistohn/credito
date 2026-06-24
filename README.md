# VerifiCrédito v2.0

Sistema de gestión de créditos con flujo Asesor → Analista → Verificador, medición de tiempos, mapa en vivo y auditoría completa.

---

## 🚀 Instalación paso a paso

### 1. Crear proyecto en Supabase

1. Ve a [supabase.com](https://supabase.com) → **Start your project**
2. Crea una cuenta (gratis)
3. Click **New project**
   - Name: `verificredito`
   - Database Password: (guarda esta contraseña)
   - Region: `South America (São Paulo)` — la más cercana a Honduras
4. Espera ~2 minutos a que se levante el proyecto

---

### 2. Ejecutar el SQL de base de datos

En Supabase: **SQL Editor → New query**

**Primero** pega y ejecuta el archivo `supabase_setup.sql`

**Luego** pega y ejecuta este SQL adicional (función de login):

```sql
-- Función de autenticación segura con bcrypt
CREATE OR REPLACE FUNCTION login_usuario(p_usuario TEXT, p_password TEXT)
RETURNS TABLE(id UUID, nombre TEXT, usuario TEXT, rol TEXT, activo BOOLEAN)
LANGUAGE plpgsql SECURITY DEFINER AS $$
BEGIN
  RETURN QUERY
  SELECT u.id, u.nombre, u.usuario, u.rol, u.activo
  FROM usuarios u
  WHERE u.usuario = p_usuario
    AND u.password = crypt(p_password, u.password);
END;
$$;
```

---

### 3. Obtener las claves de API

En Supabase: **Settings → API**

Copia estos dos valores:
- **Project URL** → algo como `https://abcdefghij.supabase.co`
- **anon public** → una clave larga que empieza con `eyJ...`

---

### 4. Configurar los archivos HTML

Abre `panel.html` y `verificador.html` con cualquier editor de texto (Notepad, VS Code, etc.)

Busca estas dos líneas **en ambos archivos** y reemplaza con tus datos:

```javascript
const SUPABASE_URL  = 'https://TU_PROYECTO.supabase.co';   // ← tu Project URL
const SUPABASE_ANON = 'TU_ANON_KEY';                        // ← tu anon public key
```

---

### 5. Subir a GitHub Pages

1. Ve a [github.com](https://github.com) → crea una cuenta si no tienes
2. **New repository** → nombre: `verificredito` → Public → **Create**
3. Sube los archivos:
   - `panel.html`
   - `verificador.html`
4. Ve a **Settings → Pages**
5. Source: **Deploy from a branch** → Branch: `main` → `/root`
6. Click **Save**
7. En ~1 minuto tendrás las URLs:
   - `https://TU_USUARIO.github.io/verificredito/panel.html`
   - `https://TU_USUARIO.github.io/verificredito/verificador.html`

---

## 👥 Usuarios de prueba (creados por el SQL)

| Usuario | Contraseña | Rol |
|---|---|---|
| `admin` | `Admin2025!` | Admin (acceso total) |
| `asesor1` | `Asesor123!` | Asesor |
| `analista1` | `Analista123!` | Analista |
| `verif1` | `Verif123!` | Verificador |

> ⚠️ **Cambia estas contraseñas inmediatamente** después del primer acceso.
> Para cambiar: Supabase → **Table Editor → usuarios** → editar el campo `password` con:
> `crypt('NuevaContraseña', gen_salt('bf'))`

---

## ➕ Crear nuevos usuarios

En Supabase **SQL Editor**:

```sql
INSERT INTO usuarios (nombre, usuario, password, rol)
VALUES ('Juan Pérez', 'juanp', crypt('SuContraseña123!', gen_salt('bf')), 'verificador');
```

Roles disponibles: `asesor`, `analista`, `verificador`, `admin`

---

## 🔄 Flujo del sistema

```
ASESOR                 ANALISTA              VERIFICADOR
  │                       │                       │
  │ 1. Registra           │                       │
  │    cliente + monto    │                       │
  │                       │                       │
  │ 2. Envía a análisis ──►                       │
  │                       │                       │
  │                       │ 3. Evalúa 6 criterios │
  │                       │    (todos deben pasar)│
  │                       │                       │
  │                       │ 4a. Aprueba ──────────►
  │                       │ 4b. Rechaza           │ 5. Inicia verificación
  │                       │ 4c. Cancela           │    (captura GPS trabajo
  │                       │                       │     y domicilio)
  │                       │                       │
  │                       │                       │ 6a. Aprueba → VERIFICADO
  │                       │                       │ 6b. Rechaza → RECHAZADO
```

---

## ⏱ Medición de tiempos

Cada solicitud registra:
- **Tiempo total**: desde creación hasta cierre
- **Tiempo de análisis**: desde que analista toma hasta que decide
- **Tiempo de verificación**: desde inicio en campo hasta cierre

Los timers se muestran en color:
- 🟢 Verde: < 2 horas
- 🟡 Amarillo: 2–4 horas  
- 🔴 Rojo: > 4 horas (con pulso animado)

---

## 📍 Mapa en vivo

- Los verificadores comparten su GPS automáticamente cada 30 segundos
- El panel muestra su posición en tiempo real (Leaflet + OpenStreetMap, sin costo)
- Se marca como inactivo si no envía ubicación en +10 minutos

---

## 📋 Módulos del Panel (`panel.html`)

| Módulo | Descripción |
|---|---|
| **Kanban** | Vista de todas las solicitudes del día por columnas |
| **Historial** | Tabla con filtros, búsqueda y exportación a CSV |
| **Mapa** | Verificadores en campo con ubicación en tiempo real |

---

## 📱 App Verificador (`verificador.html`)

- Login exclusivo para verificadores
- Lista de solicitudes asignadas con timers
- Captura GPS de trabajo y domicilio
- Checklist de verificación
- Historial personal
- Notificaciones push (vía Supabase Realtime)

---

## 🗃️ Auditoría completa

Cada acción queda registrada en la tabla `historial_solicitud` con:
- Timestamp exacto
- Usuario que hizo la acción
- Rol del usuario
- Estado anterior y nuevo
- Notas de la acción

---

## 🛠️ Mantenimiento

**Ver todos los registros:**
```sql
SELECT * FROM solicitudes ORDER BY fecha_creacion DESC;
```

**Ver historial de una solicitud:**
```sql
SELECT * FROM historial_solicitud WHERE solicitud_id = 'UUID_AQUI' ORDER BY timestamp;
```

**Estadísticas del día:**
```sql
SELECT * FROM vista_estadisticas;
```

**Solicitudes activas con tiempo transcurrido:**
```sql
SELECT * FROM vista_solicitudes_hoy;
```
