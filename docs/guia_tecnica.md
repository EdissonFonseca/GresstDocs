# Guía Técnica

Esta guía está dirigida a desarrolladores, personal técnico y administradores del sistema Gresst.

---

## Arquitectura General

Para entender la arquitectura completa del sistema, consulta la [Arquitectura del Sistema](arquitectura.md).

La plataforma Gresst consta de **tres componentes principales**:
1. **Portal de Gestores** (Web)
2. **Portal de Generadores** (Web)
3. **App Móvil** (iOS/Android)

---

## Requerimientos del Sistema

### Portales Web

#### Requisitos del cliente:
- **Navegadores soportados:**
  - Google Chrome 90+
  - Mozilla Firefox 88+
  - Microsoft Edge 90+
  - Safari 14+ (macOS/iOS)
- **Resolución mínima:** 1280x720
- **Conexión a internet:** Banda ancha recomendada

#### Requisitos del servidor:
- **Node.js:** v16+ o v18+ (LTS recomendado)
- **Base de datos:** PostgreSQL 13+ o MySQL 8+
- **Memoria RAM:** Mínimo 2GB (4GB recomendado)
- **Almacenamiento:** 20GB+ para datos y documentos
- **Certificado SSL:** Requerido (Let's Encrypt gratuito)

### App Móvil

#### iOS:
- **Versión mínima:** iOS 13.0+
- **Dispositivos:** iPhone 6s o superior, iPad Air 2 o superior
- **Espacio:** 150MB mínimo

#### Android:
- **Versión mínima:** Android 8.0 (API 26)+
- **RAM:** 2GB mínimo
- **Espacio:** 150MB mínimo
- **Permisos requeridos:** Cámara, GPS, Almacenamiento

---

## Instalación y Despliegue

### Portal Web (Desarrollo)

```bash
# Clonar el repositorio
git clone https://github.com/TU-ORGANIZACION/gresst-portal.git
cd gresst-portal

# Instalar dependencias
npm install

# Configurar variables de entorno
cp .env.example .env
# Editar .env con tus credenciales

# Iniciar servidor de desarrollo
npm run dev

# Acceder a: http://localhost:3000
```

### Portal Web (Producción)

```bash
# Construir para producción
npm run build

# Iniciar servidor de producción
npm start

# O con PM2 (recomendado)
pm2 start npm --name "gresst-portal" -- start
pm2 save
pm2 startup
```

### Base de Datos

```sql
-- Crear base de datos
CREATE DATABASE gresst_db;

-- Crear usuario
CREATE USER gresst_user WITH PASSWORD 'tu_password_seguro';

-- Otorgar permisos
GRANT ALL PRIVILEGES ON DATABASE gresst_db TO gresst_user;

-- Ejecutar migraciones
npm run migrate
```

### Variables de Entorno

Archivo `.env` de ejemplo:

```bash
# Base de datos
DATABASE_URL=postgresql://usuario:password@localhost:5432/gresst_db

# JWT
JWT_SECRET=tu_secreto_muy_seguro_cambialo
JWT_EXPIRES_IN=24h

# API
API_PORT=3000
API_URL=https://api.gresst.com

# Servicios externos
GOOGLE_MAPS_API_KEY=tu_api_key
SMTP_HOST=smtp.gmail.com
SMTP_PORT=587
SMTP_USER=tu_email@gmail.com
SMTP_PASS=tu_password

# Almacenamiento
AWS_BUCKET=gresst-documents
AWS_REGION=us-east-1
AWS_ACCESS_KEY=tu_access_key
AWS_SECRET_KEY=tu_secret_key

# Notificaciones
FIREBASE_PROJECT_ID=gresst-app
FIREBASE_PRIVATE_KEY=tu_private_key
```

---

## Configuración de Servidores

### Nginx (Reverse Proxy)

```nginx
server {
    listen 80;
    server_name portal.gresst.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name portal.gresst.com;

    ssl_certificate /etc/letsencrypt/live/portal.gresst.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/portal.gresst.com/privkey.pem;

    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```

### Docker (Opcional)

```dockerfile
FROM node:18-alpine

WORKDIR /app

COPY package*.json ./
RUN npm ci --only=production

COPY . .
RUN npm run build

EXPOSE 3000

CMD ["npm", "start"]
```

```yaml
# docker-compose.yml
version: '3.8'
services:
  web:
    build: .
    ports:
      - "3000:3000"
    environment:
      - DATABASE_URL=postgresql://postgres:password@db:5432/gresst_db
    depends_on:
      - db
  
  db:
    image: postgres:15-alpine
    environment:
      - POSTGRES_DB=gresst_db
      - POSTGRES_PASSWORD=password
    volumes:
      - postgres_data:/var/lib/postgresql/data

volumes:
  postgres_data:
```

---

## API Endpoints

### Autenticación

```http
POST /api/auth/login
POST /api/auth/register
POST /api/auth/refresh
POST /api/auth/logout
```

### Órdenes de Recolección

```http
GET    /api/orders          # Listar órdenes
POST   /api/orders          # Crear orden
GET    /api/orders/:id      # Obtener detalle
PUT    /api/orders/:id      # Actualizar orden
DELETE /api/orders/:id      # Eliminar orden
```

### Usuarios y Permisos

```http
GET    /api/users           # Listar usuarios
POST   /api/users           # Crear usuario
GET    /api/users/:id       # Obtener usuario
PUT    /api/users/:id       # Actualizar usuario
DELETE /api/users/:id       # Eliminar usuario
```

Documentación completa de API: [Swagger/OpenAPI](https://api.gresst.com/docs)

---

## Seguridad

### Mejores Prácticas

1. **Autenticación:**
   - Usar JWT con expiración corta (24h)
   - Implementar refresh tokens
   - MFA para cuentas administrativas

2. **Encriptación:**
   - HTTPS obligatorio (TLS 1.3)
   - Encriptar datos sensibles en base de datos
   - Hash de contraseñas con bcrypt (10+ rounds)

3. **Validación:**
   - Sanitizar todas las entradas
   - Validar tipos de datos
   - Límites de rate limiting

4. **Auditoría:**
   - Log de acciones críticas
   - Monitoreo de intentos fallidos
   - Alertas automáticas de actividad sospechosa

---

## Monitoreo y Logs

### Herramientas recomendadas:

- **APM:** New Relic / Datadog
- **Logs:** ELK Stack (Elasticsearch, Logstash, Kibana)
- **Uptime:** UptimeRobot / Pingdom
- **Errors:** Sentry / Rollbar

### Logs de aplicación:

```javascript
// Ejemplo con Winston (Node.js)
const winston = require('winston');

const logger = winston.createLogger({
  level: 'info',
  format: winston.format.json(),
  transports: [
    new winston.transports.File({ filename: 'error.log', level: 'error' }),
    new winston.transports.File({ filename: 'combined.log' })
  ]
});
```

---

## Testing

### Unit Tests

```bash
# Ejecutar tests
npm test

# Con cobertura
npm run test:coverage
```

### Integration Tests

```bash
# Tests de integración
npm run test:integration
```

### E2E Tests

```bash
# Tests end-to-end (Cypress)
npm run test:e2e
```

---

## Respaldos (Backups)

### Base de datos automática:

```bash
#!/bin/bash
# backup.sh

DATE=$(date +%Y%m%d_%H%M%S)
BACKUP_DIR="/backups/postgres"
DB_NAME="gresst_db"

pg_dump $DB_NAME | gzip > $BACKUP_DIR/backup_$DATE.sql.gz

# Mantener solo los últimos 30 días
find $BACKUP_DIR -type f -mtime +30 -delete
```

Configurar cron:
```bash
# Backup diario a las 2 AM
0 2 * * * /scripts/backup.sh
```

---

## Solución de Problemas

### Portal no carga

1. Verificar logs: `tail -f logs/error.log`
2. Verificar proceso: `pm2 status`
3. Verificar puertos: `netstat -tlnp | grep 3000`
4. Reiniciar: `pm2 restart gresst-portal`

### Error de conexión a base de datos

1. Verificar PostgreSQL: `systemctl status postgresql`
2. Probar conexión: `psql -U gresst_user -d gresst_db`
3. Revisar credenciales en `.env`

### App móvil no sincroniza

1. Verificar conectividad de red
2. Revisar logs en Firebase Crashlytics
3. Forzar sincronización desde configuración
4. Limpiar cache de la app

---

## Actualizaciones

### Proceso de actualización:

```bash
# 1. Backup
npm run backup

# 2. Pull cambios
git pull origin main

# 3. Instalar dependencias
npm install

# 4. Ejecutar migraciones
npm run migrate

# 5. Construir
npm run build

# 6. Reiniciar
pm2 restart gresst-portal

# 7. Verificar
curl https://portal.gresst.com/health
```

---

## Soporte Técnico

- **Documentación:** [docs.gresst.com](https://docs.gresst.com)
- **Email:** soporte@gresst.com
- **Slack/Discord:** Canal #soporte-tecnico
- **Issues:** [GitHub Issues](https://github.com/TU-ORG/gresst/issues)

---

## Recursos Adicionales

- [Arquitectura del Sistema](arquitectura.md)
- [Guía de Usuario](guia_usuarios.md)
- [Procesos Operativos](procesos_operativos.md)
- [Changelog](https://github.com/TU-ORG/gresst/releases)
