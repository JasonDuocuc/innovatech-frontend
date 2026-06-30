# Innovatech - Frontend Dashboard

Dashboard React (Vite + Tailwind) para visualizar ventas y gestionar despachos, consumiendo los microservicios Ventas y Despachos.

## Arquitectura
- **Contenedor**: imagen Docker (Nginx sirviendo el build), en ECR (`innovatech-frontend`)
- **Orquestación**: ECS Fargate, cluster `innovatech-cluster-v2`, servicio `svc-frontend`
- **Red**: subred pública, IP pública directa (sin ALB en esta versión)
- **Autoscaling**: Target Tracking CPU 50%, mín 1 / máx 3

## Variables de entorno (build time)
