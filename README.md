# Eval3 Orquestación y Automatización en AWS ECS

## Descripción
Proyecto de orquestación de contenedores en AWS ECS Fargate con pipeline CI/CD automatizado usando GitHub Actions. Despliega una aplicación compuesta por tres servicios: Backend Python (Flask), Backend JavaScript (Node.js/Express) y Frontend (Java/Maven + Nginx), con base de datos MySQL.

## Arquitectura

```
Internet
    ↓
[Frontend - Nginx:80]
    ↓              ↓
[BackendPy:8082]  [BackendJS:8081]
         ↓
    [MySQL:3306]
         ↓
   [Amazon ECR]
         ↓
 [ECS Fargate Cluster]
```

## Servicios desplegados

| Servicio | Tecnología | Puerto | Imagen ECR |
|---------|-----------|--------|------------|
| frontend | Java/Maven + Nginx | 80 | `339712924855.dkr.ecr.us-east-1.amazonaws.com/frontend:latest` |
| backend-py | Python/Flask | 8082 | `339712924855.dkr.ecr.us-east-1.amazonaws.com/backend-py:latest` |
| backend-js | Node.js/Express | 8081 | `339712924855.dkr.ecr.us-east-1.amazonaws.com/backend-js:latest` |
| mysql | MySQL 8.0 | 3306 | `mysql:8.0` (imagen pública) |

## Infraestructura AWS

- **Cluster ECS:** `ev3_cluster` (AWS Fargate)
- **Región:** `us-east-1`
- **VPC:** `eval3-vpc-vpc`
- **Subredes:** 2 subredes públicas
- **Security Group:** `grupo-ecs-eval3` (All traffic inbound/outbound)
- **IAM Role:** `LabRole`
- **Registro de imágenes:** Amazon ECR (3 repositorios privados)
- **Logs:** Amazon CloudWatch (`/ecs/backend-py`, `/ecs/backend-js`, `/ecs/frontend`, `/ecs/mysql`)

## Pipeline CI/CD

El pipeline se encuentra en `.github/workflows/deploy.yml` y se activa automáticamente con cada `push` a la rama `main`.

### Flujo del pipeline:
```
push a main
    ↓
Build imagen Docker
    ↓
Push a Amazon ECR
    ↓
Actualizar Task Definition
    ↓
Deploy a ECS Fargate
```

### Jobs paralelos:
- `deploy-backend-py` — Build y deploy del backend Python
- `deploy-backend-js` — Build y deploy del backend Node.js
- `deploy-frontend` — Build y deploy del frontend (depende de los backends)

## Autoscaling

Configurado con **Target Tracking Scaling** al **50% de CPU** para los 3 servicios:

| Servicio | Min tareas | Max tareas | Métrica |
|---------|-----------|-----------|---------|
| backend-py | 1 | 3 | CPU 50% |
| backend-js | 1 | 3 | CPU 50% |
| frontend | 1 | 3 | CPU 50% |

**Justificación del umbral 50% CPU:** Se eligió 50% para mantener margen de respuesta ante picos de carga, permitiendo que el sistema escale antes de llegar al límite y evitar degradación del servicio.

## Variables de entorno

### Backend Python y Backend JS
```env
DB_HOST=<IP privada del contenedor MySQL>
DB_PORT=3306
DB_USER=appuser
DB_PASSWORD=apppass123
DB_NAME=products_db
PORT=8082  # o 8081 para backend-js
```

### MySQL
```env
MYSQL_ROOT_PASSWORD=rootpass123
MYSQL_DATABASE=products_db
MYSQL_USER=appuser
MYSQL_PASSWORD=apppass123
```

## Secrets de GitHub Actions

Configurar en **Settings → Secrets and variables → Actions**:

| Secret | Descripción |
|--------|-------------|
| `AWS_ACCESS_KEY_ID` | Credencial AWS Academy |
| `AWS_SECRET_ACCESS_KEY` | Credencial AWS Academy |
| `AWS_SESSION_TOKEN` | Token de sesión AWS Academy |

> ⚠️ Las credenciales de AWS Academy cambian con cada sesión del laboratorio. Actualizar los secrets en GitHub cada vez que se reinicie el lab.

## Cómo desplegar manualmente

### Requisitos
- AWS CLI instalado y configurado
- Docker instalado
- Git instalado

### Pasos

1. Clonar el repositorio:
```bash
git clone https://github.com/FabianValenciaPizarro/Eval3.git
cd Eval3
```

2. Autenticarse en ECR:
```bash
aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 339712924855.dkr.ecr.us-east-1.amazonaws.com
```

3. Build y push de imágenes:
```bash
docker build -t backend-py ./backend-py
docker tag backend-py:latest 339712924855.dkr.ecr.us-east-1.amazonaws.com/backend-py:latest
docker push 339712924855.dkr.ecr.us-east-1.amazonaws.com/backend-py:latest

docker build -t backend-js ./backend-js
docker tag backend-js:latest 339712924855.dkr.ecr.us-east-1.amazonaws.com/backend-js:latest
docker push 339712924855.dkr.ecr.us-east-1.amazonaws.com/backend-js:latest

docker build -t frontend ./frontend
docker tag frontend:latest 339712924855.dkr.ecr.us-east-1.amazonaws.com/frontend:latest
docker push 339712924855.dkr.ecr.us-east-1.amazonaws.com/frontend:latest
```

4. Actualizar servicios ECS:
```bash
aws ecs update-service --cluster ev3_cluster --service backend-py-task-service-23gyiv0n --force-new-deployment
aws ecs update-service --cluster ev3_cluster --service backend-js-task-service-m80kdffi --force-new-deployment
aws ecs update-service --cluster ev3_cluster --service frontend-task-service-xq848awe --force-new-deployment
```

## Estructura del repositorio

```
Eval3/
├── .github/
│   └── workflows/
│       └── deploy.yml          # Pipeline CI/CD
├── backend-py/
│   ├── app.py                  # Aplicación Flask
│   ├── requirements.txt        # Dependencias Python
│   ├── Dockerfile              # Imagen Docker
│   └── .env.example            # Variables de entorno ejemplo
├── backend-js/
│   ├── server.js               # Servidor Express
│   ├── package.json            # Dependencias Node.js
│   ├── Dockerfile              # Imagen Docker
│   └── .env.example            # Variables de entorno ejemplo
├── frontend/
│   ├── src/                    # Código fuente Java
│   ├── pom.xml                 # Configuración Maven
│   └── Dockerfile              # Imagen multi-stage (Maven + Nginx)
└── README.md
```

## Logs y monitoreo

Los logs de cada servicio están disponibles en **CloudWatch Logs**:
- `/ecs/backend-py`
- `/ecs/backend-js`
- `/ecs/frontend`
- `/ecs/mysql`

Para ver logs en tiempo real desde CloudShell:
```bash
aws logs tail /ecs/backend-py --follow
aws logs tail /ecs/backend-js --follow
```

## Autores
- Fabian Valencia Pizarro