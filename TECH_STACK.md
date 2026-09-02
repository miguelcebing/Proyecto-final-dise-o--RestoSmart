# Stack Tecnológico y Herramientas para RestoSmart

Este documento describe el stack tecnológico recomendado y las herramientas que deberían establecerse para el desarrollo del proyecto RestoSmart.

## Stack Tecnológico Recomendado

### Frontend
- **Framework**: React 18+ con TypeScript
- **Estado**: Redux Toolkit o Zustand
- **Estilos**: Tailwind CSS o MUI (Material-UI)
- **Build**: Vite o Create React App
- **Testing**: Jest + React Testing Library
- **UI Components**: Biblioteca propia o basada en Material-UI/Ant Design

### Backend
- **Lenguaje**: Node.js (v18+) o Python (v3.10+)
- **Framework**: Express.js (Node) o FastAPI/Django (Python)
- **Base de Datos**: PostgreSQL o MySQL
- **ORM**: Sequelize/TypeORM (Node) o SQLAlchemy/Django ORM (Python)
- **API**: RESTful con documentación OpenAPI/Swagger
- **Autenticación**: JWT (JSON Web Tokens) + OAuth 2.0 para servicios externos
- **Testing**: Jest/Pytest + Supertest/Requests
- **Documentación**: Swagger UI o Redoc

### Infraestructura
- **Contenedores**: Docker + Docker Compose
- **Orquestación**: Kubernetes (para producción) o Docker Compose (para desarrollo)
- **CI/CD**: GitHub Actions o GitLab CI
- **Hosting**: AWS, Google Cloud o Azure (servicios gestionados)
- **CDN**: Cloudflare o AWS CloudFront
- **Monitoreo**: Prometheus + Grafana o similares
- **Logging**: ELK Stack (Elasticsearch, Logstash, Kibana) o similar

### Herramientas de Desarrollo
- **IDE**: VS Code con extensiones recomendadas
- **Control de Versiones**: Git (con GitHub/GitLab)
- **Gestión de Paquetes**: npm/yarn (Node) o pip/poetry (Python)
- **Formateo**: Prettier + ESLint
- **Testing de Cycles**: Cypress o Playwright para E2E
- **Documentación de API**: Swagger/OpenAPI
- **Diagramas**: Draw.io o Mermaid para documentación de arquitectura

## Configuración del Entorno de Desarrollo

### Prerrequisitos
1. Git instalado y configurado
2. Node.js v18+ y npm/yarn instalados
3. Python v3.10+ y pip instalados (si se elige backend en Python)
4. Docker y Docker Compose instalados
5. PostgreSQL o MySQL instalados y ejecutándose localmente
6. IDE configurado (VS Code recomendado)

### Pasos para Configurar el Entorno

#### 1. Clonar el Repositorio
```bash
git clone https://github.com/miguelcebing/Proyecto-final-dise-o--RestoSmart.git
cd Proyecto-final-dise-o--RestoSmart
```

#### 2. Configurar Ramas
```bash
# Asegurarse de estar en main
git checkout main
# Crear rama de desarrollo si no existe
git checkout -b develop
git push -u origin develop
```

#### 3. Configurar Variables de Entorno
Crear archivo `.env` en la raíz del proyecto:
```env
# Variables de entorno comunes
NODE_ENV=development
PORT=3000
FRONTEND_URL=http://localhost:3000
BACKEND_URL=http://localhost:5000

# Base de datos
DB_HOST=localhost
DB_PORT=5432
DB_NAME=restosmart
DB_USER=dev_user
DB_PASSWORD=dev_password

# JWT
JWT_SECRET=your_jwt_secret_here
JWT_EXPIRES_IN=24h

# Pasarelas de pago (modo sandbox/test)
STRIPE_PUBLISHABLE_KEY=pk_test_...
STRIPE_SECRET_KEY=sk_test_...
PAYPAL_CLIENT_ID=...
PAYPAL_CLIENT_SECRET=...

# Otros servicios externos
EMAIL_SERVICE_USER=...
EMAIL_SERVICE_PASS=...
```

#### 4. Instalar Dependencias
```bash
# Para frontend (asumiendo React)
cd frontend
npm install
# o
yarn install

# Para backend (asumiendo Node.js/Express)
cd ../backend
npm install
# o
yarn install
```

#### 5. Iniciar Servicios
```bash
# Iniciar base de datos (si se usa Docker)
docker-compose up -d db

# Iniciar backend
cd backend
npm run dev
# o
yarn dev

# En otra terminal, iniciar frontend
cd frontend
npm run dev
# o
yarn dev
```

#### 6. Ejecutar Pruebas
```bash
# Backend pruebas
cd backend
npm test
# o
yarn test

# Frontend pruebas
cd frontend
npm test
# o
yarn test
```

## Estructura de Carpetas Recomendada

```
Proyecto-final-dise-o--RestoSmart/
├── .github/                  # Configuración de GitHub Actions, plantillas de issue, etc.
├── .vscode/                  # Configuración de VS Code
├── docs/                     # Documentación del proyecto
├── frontend/                 # Código fuente del frontend
│   ├── public/               # Archivos estáticos
│   ├── src/                  # Código fuente
│   │   ├── components/       # Componentes reutilizables
│   │   ├── pages/            # Páginas de la aplicación
│   │   ├── services/         # Servicios API
│   │   ├── store/            # Gestión de estado
│   │   ├── utils/            # Utilidades
│   │   ├── hooks/            # Custom hooks
│   │   ├── styles/           # Estilos y temas
│   │   ├── tests/            # Pruebas unitarias y de integración
│   │   ├── App.tsx           # Componente raíz
│   │   └── main.tsx          # Punto de entrada
│   ├── tests/                # Pruebas E2E
│   ├── .env                  # Variables de entorno frontend
│   ├── package.json          # Dependencias y scripts
│   └── vite.config.ts        # Configuración de Vite (o equivalente)
├── backend/                  # Código fuente del backend
│   ├── src/                  # Código fuente
│   │   ├── controllers/      # Controladores de rutas
│   │   ├── models/           # Modelos de datos
│   │   ├── routes/           # Definición de rutas API
│   │   ├── middleware/       # Middleware personalizado
│   │   ├── services/         # Lógica de negocio
│   │   ├── utils/            # Utilidades
│   │   ├── config/           # Configuración
│   │   ├── tests/            # Pruebas unitarias y de integración
│   │   ├── migrations/       # Migraciones de base de datos
│   │   ├── seeds/            # Datos iniciales
│   │   └── server.ts         # Punto de entrada
│   ├── tests/                # Pruebas E2E y de contrato
│   ├── .env                  # Variables de entorno backend
│   ├── package.json          # Dependencias y scripts
│   └── tsconfig.json         # Configuración de TypeScript (si aplica)
├── database/                 # Scripts de base de datos
│   ├── schema.sql            # Esquema inicial
│   └── migrations/           # Scripts de migración
├── docker/                   # Configuración de Docker
│   ├── frontend.Dockerfile   # Dockerfile para frontend
│   ├── backend.Dockerfile    # Dockerfile para backend
│   └── docker-compose.yml    # Orquestación de servicios
├── requirements.md           # Requisitos funcionales y no funcionales
├── PROJECT_PLAN.md           # Plan de desarrollo
├── DEFINITION_OF_DONE.md     # Criterios de completado
└── README.md                 # Información general del proyecto
```

## Herramientas de Calidad y Seguridad

### Análisis Estático de Código
- **ESLint** + plugins para React/TypeScript o Python
- **Prettier** para formateo de código
- **SonarQube** o similar para análisis de calidad continua

### Seguridad
- **OWASP ZAP** o **Snyk** para escaneo de vulnerabilidades
- **Dependabot** o **Renovate** para actualización automática de dependencias
- **GitHub Advanced Security** para escaneo de código y secretos

### Rendimiento
- **Lighthouse** para auditorías de rendimiento frontend
- **WebPageTest** o similares para pruebas de carga
- **Artillery** o **k6** para pruebas de estrés de API

## Procesos Recomendados

### Flujo de Trabajo Git
1. Desarrollar en ramas feature/`nombre-feature`
2. Hacer pull request a `develop`
3. Revisión de código requerida (mínimo 1 aprobador)
4. Las pruebas deben pasar en CI antes del merge
5. Los releases se hacen desde `develop` a `main` con tagging

### Integración Continua (GitHub Actions)
- Ejecutar pruebas en cada push a cualquier rama
- Ejecutar pruebas y build en pull requests
- Desplegar automáticamente a staging en cada push a `develop`
- Desplegar a producción manualmente o con approval en cada tag en `main`

### Despliegue
- **Desarrollo**: Docker Compose local
- **Staging**: Entorno idéntico a producción en la nube
- **Producción**: Orquestación con Kubernetes o servicios gestionados
- **Blue/Green Deployment** o **Canary Releases** para minimizar riesgos

## Próximos Pasos para Establecer el Entorno

1. Crear el repositorio con la estructura inicial
2. Configurar GitHub Actions para CI/CD básico
3. Crear documentación de instalación y setup
4. Establecer plantillas de issue y pull request
5. Configurar protecciones de rama (require reviews, status checks)
6. Configurar Dependabot o similar para actualizaciones de dependencias
7. Configurar código de conducta y contribuir guidelines