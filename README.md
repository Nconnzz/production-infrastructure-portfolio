# Production Infrastructure Portfolio - Monitoring & Communication Stack

## Descripcion

Este repositorio documenta la implementacion de un stack completo de infraestructura de produccion orientado al monitoreo y la comunicacion de equipos. El proyecto demuestra competencias en:

- **Observabilidad**: Implementacion de un stack de monitoreo con Prometheus y Grafana para visualizacion de metricas en tiempo real
- **Containerizacion**: Orquestacion de servicios con Docker Compose siguiendo mejores practicas de DevOps
- **Alertas proactivas**: Sistema de alertas configurado para detectar problemas antes de que impacten en produccion
- **Comunicacion de equipo**: Integracion de Rocket.Chat como plataforma de comunicacion interna

### Finalidad

Este proyecto fue desarrollado como parte de mi portfolio de infraestructura para demostrar habilidades practicas en:
- Diseno e implementacion de soluciones de monitoreo enterprise
- Configuracion de dashboards profesionales con metricas clave del sistema
- Gestion de contenedores Docker y persistencia de datos
- Automatizacion de alertas para mantenimiento preventivo
- Documentacion tecnica profesional

## Arquitectura

```
┌─────────────────────────────────────────────────────────────────┐
│                        MONITORING STACK                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────────────┐ │
│  │   Grafana   │◄───│  Prometheus │◄───│   Node Exporter     │ │
│  │   :3001     │    │    :9091    │    │      :9101          │ │
│  └─────────────┘    └──────┬──────┘    │  (System Metrics)   │ │
│                            │           └─────────────────────┘ │
│                            │                                    │
│                            │           ┌─────────────────────┐ │
│                            └───────────│     cAdvisor        │ │
│                                        │      :8080          │ │
│                                        │ (Container Metrics) │ │
│                                        └─────────────────────┘ │
├─────────────────────────────────────────────────────────────────┤
│                     COMMUNICATION STACK                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌─────────────┐    ┌─────────────┐                            │
│  │ Rocket.Chat │◄───│   MongoDB   │                            │
│  │    :3101    │    │   (rs0)     │                            │
│  └─────────────┘    └─────────────┘                            │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

## Componentes

| Servicio | Version | Puerto | Descripcion |
|----------|---------|--------|-------------|
| Grafana | 11.4.0 | 3001 | Dashboard de visualizacion de metricas |
| Prometheus | v3.1.0 | 9091 | Recolector y almacen de metricas |
| Node Exporter | v1.8.2 | 9101 | Exportador de metricas del sistema |
| cAdvisor | v0.51.0 | 8080 | Metricas de contenedores Docker |
| Rocket.Chat | 7.5.0 | 3101 | Plataforma de comunicacion de equipo |
| MongoDB | 6.0 | 27017 | Base de datos para Rocket.Chat |

## Requisitos

- Docker Engine 20.10+
- Docker Compose v2.0+
- 4GB RAM minimo
- 20GB espacio en disco

## Instalacion

1. Clonar el repositorio:
```bash
git clone https://github.com/Nconnzz/production-infrastructure-portfolio.git
cd production-infrastructure-portfolio
```

2. Iniciar los servicios:
```bash
docker compose up -d
```

3. Verificar que todos los contenedores esten corriendo:
```bash
docker compose ps
```

## Acceso a Servicios

| Servicio | URL | Credenciales Default |
|----------|-----|---------------------|
| Grafana | http://localhost:3001 | admin / ******** |
| Prometheus | http://localhost:9091 | - |
| Rocket.Chat | http://localhost:3101 | admin / ******** |
| Node Exporter | http://localhost:9101/metrics | - |
| cAdvisor | http://localhost:8080 | - |

> **Importante**: Cambiar las credenciales por defecto antes de usar en produccion.

## Dashboard de Grafana

El dashboard "Server Metrics" incluye 17 paneles organizados:

### Metricas Principales (Stats)
- CPU % - Uso actual del procesador
- RAM % - Uso actual de memoria
- Disco % - Uso del disco root
- Uptime - Tiempo desde el ultimo reinicio

### Graficos de Tendencia
- **CPU - Uso Detallado**: Total, Usuario, Sistema, IO Wait
- **RAM - Uso**: Porcentaje de memoria utilizada
- **RAM - Detalle**: Total, Usada, Disponible, Cache (en bytes)
- **Disco - Uso %**: Tendencia de uso del disco root

### Metricas de I/O
- **Disco Root - Gauge**: Indicador visual del espacio usado
- **Disco I/O**: Lectura y escritura por dispositivo
- **Network I/O**: Trafico de red RX/TX por interfaz

### Informacion del Sistema
- **System Load**: Load average 1m, 5m, 15m
- **CPU Cores**: Numero de nucleos
- **Total RAM**: Memoria total instalada
- **Total Disco**: Capacidad total del disco
- **Procesos Corriendo/Bloqueados**: Estado de procesos

## Sistema de Alertas

### Alertas de Servidor

| Alerta | Umbral | Severidad | Duracion |
|--------|--------|-----------|----------|
| High CPU Usage | > 80% | warning | 2m |
| High Memory Usage | > 85% | warning | 2m |
| High Disk Usage | > 90% | critical | 5m |
| High Load Average | > 4 | warning | 5m |
| Network Errors | > 10/s | warning | 5m |

### Alertas de Contenedores

| Alerta | Umbral | Severidad | Duracion |
|--------|--------|-----------|----------|
| Container High CPU | > 80% | warning | 5m |
| Container High Memory | > 1GB | warning | 5m |
| Prometheus Target Down | target = 0 | critical | 1m |

## Estructura del Proyecto

```
.
├── docker-compose.yml              # Orquestacion de servicios
├── prometheus/
│   └── prometheus.yml              # Configuracion de Prometheus
├── grafana/
│   └── provisioning/
│       ├── dashboards/
│       │   ├── dashboards.yml      # Proveedor de dashboards
│       │   └── server-metrics.json # Dashboard principal
│       ├── datasources/
│       │   └── datasources.yml     # Configuracion de Prometheus
│       └── alerting/
│           ├── rules.yml           # Reglas de alertas
│           ├── contactpoints.yml   # Puntos de contacto
│           └── policies.yml        # Politicas de notificacion
└── README.md
```

## Configuracion

### Prometheus

El scrape interval esta configurado a 15 segundos. Los jobs configurados:

- `prometheus` - Auto-monitoreo de Prometheus
- `node-exporter` - Metricas del sistema host
- `cadvisor` - Metricas de contenedores Docker

### Variables de Entorno

**Grafana:**
- `GF_SECURITY_ADMIN_USER` - Usuario admin
- `GF_SECURITY_ADMIN_PASSWORD` - Password admin
- `GF_USERS_ALLOW_SIGN_UP` - Deshabilitar registro

**Rocket.Chat:**
- `ROOT_URL` - URL base de la aplicacion
- `MONGO_URL` - Conexion a MongoDB
- `ADMIN_USERNAME/PASS/EMAIL` - Credenciales admin

## Operaciones Comunes

### Reiniciar servicios
```bash
docker compose restart
```

### Ver logs
```bash
docker compose logs -f [servicio]
```

### Detener todos los servicios
```bash
docker compose down
```

### Eliminar datos persistentes
```bash
docker compose down -v
```

### Actualizar imagenes
```bash
docker compose pull
docker compose up -d
```

## Metricas Recolectadas

### Node Exporter
- CPU (uso, frecuencia, estados)
- Memoria (total, disponible, cache, swap)
- Disco (espacio, I/O, filesystem)
- Red (bytes, paquetes, errores)
- Sistema (load, uptime, procesos)
- Temperatura (thermal zones)

### cAdvisor
- CPU por contenedor
- Memoria por contenedor
- Network I/O por contenedor
- Disk I/O por contenedor

## Personalizacion

### Agregar nuevas alertas
Editar `grafana/provisioning/alerting/rules.yml` y reiniciar Grafana.

### Modificar dashboard
El dashboard se puede editar desde la UI de Grafana. Para persistir cambios, exportar el JSON y reemplazar `server-metrics.json`.

### Agregar nuevos targets a Prometheus
Editar `prometheus/prometheus.yml` y reiniciar Prometheus o usar el endpoint `/-/reload`.

## Troubleshooting

**MongoDB no inicia correctamente:**
```bash
docker compose logs mongodb
# Esperar a que el replica set se inicialice (healthcheck)
```

**Grafana no muestra datos:**
1. Verificar que Prometheus este corriendo
2. Ir a Configuration > Data Sources > Prometheus > Test

**Alertas no funcionan:**
1. Verificar Contact Points en Grafana
2. Revisar que las reglas esten habilitadas

## Licencia

MIT License

## Autor

Nicolas Nunez

Repositorio: https://github.com/Nconnzz/production-infrastructure-portfolio
