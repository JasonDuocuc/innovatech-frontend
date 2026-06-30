# Innovatech - Frontend Dashboard

Dashboard React (Vite + Tailwind) para visualizar ventas y gestionar despachos, consumiendo los microservicios Ventas y Despachos del sistema Innovatech Chile.

## Arquitectura

- **Contenedor**: imagen Docker (Nginx sirviendo el build estático), almacenada en Amazon ECR como `innovatech-frontend`
- **Orquestación**: AWS ECS Fargate, cluster `innovatech-cluster-v2`, servicio `svc-frontend`
- **Red**: subred pública, con IP pública directa asignada por ECS (sin Application Load Balancer en esta versión)
- **Autoscaling**: Target Tracking en CPU, objetivo 50%, mínimo 1 / máximo 3 tareas
## Variables de entorno

Se inyectan en **build time**, porque Vite las compila dentro del bundle estático que después sirve Nginx (no son variables de runtime del contenedor):

| Variable | Descripción |
|---|---|
| VITE_API_VENTAS | URL pública del backend Ventas (DNS del ALB público + puerto 8080) |
| VITE_API_DESPACHOS | URL pública del backend Despachos (DNS del ALB público + puerto 8081) |

Ejemplo de archivo `.env`:
```
VITE_API_VENTAS=http://alb-ventas-publico-xxxxxxx.us-east-1.elb.amazonaws.com:8080
VITE_API_DESPACHOS=http://alb-despachos-publico-xxxxxxx.us-east-1.elb.amazonaws.com:8081
```
## Cómo ejecutar localmente

```bash
npm install
npm run build
docker build -t innovatech-frontend .
docker run -p 80:80 innovatech-frontend
```

## Pipeline CI/CD

Cada push a `main` dispara `.github/workflows/deploy.yml`, que ejecuta:

1. Checkout del código
2. Build de la imagen Docker con el código ya compilado por Vite
3. Push de la imagen a Amazon ECR (`innovatech-frontend`)
4. `aws ecs update-service --cluster innovatech-cluster-v2 --service svc-frontend --force-new-deployment`

Las credenciales AWS se gestionan vía GitHub Secrets (`AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, `AWS_SESSION_TOKEN`). Por ser sesión temporal de AWS Academy, estas credenciales expiran cada pocas horas y deben renovarse manualmente en los Secrets del repositorio.
## Funcionalidad

- Tabla de órdenes de compra, consumiendo el endpoint de Ventas
- Tabla de despachos, consumiendo el endpoint de Despachos
- Generación de despacho a partir de una orden de compra existente
- Comunicación con ambos backends vía peticiones HTTP (Axios) configuradas con las variables `VITE_API_*`
