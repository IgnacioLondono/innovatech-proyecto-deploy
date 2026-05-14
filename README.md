# Innovatech — Despliegue (DevOps)

Monorepo con frontend React (Vite), dos APIs REST en Spring Boot (despachos y ventas), MySQL y orquestación con Docker Compose. Incluye flujos de CI/CD en GitHub Actions hacia AWS (ECR + EC2 vía SSM).

**Repositorio:** [Innovatech — código en GitHub](https://github.com/IgnacioLondono/innovatech-proyecto-deploy)

---

## Contenido del repositorio

| Componente | Descripción |
|------------|-------------|
| `front_despacho/` | SPA React + Vite, servida con Nginx en contenedor |
| `back-Despachos_SpringBoot/Springboot-API-REST-DESPACHO/` | API de despachos |
| `back-Ventas_SpringBoot/Springboot-API-REST/` | API de ventas |
| `docker-compose.yml` | MySQL + backends + frontend en red `innovatech2-network` |
| `.github/workflows/` | Build, push a ECR y despliegue en EC2 (rama `deploy`) |

---

## Arquitectura local (Docker Compose)

```
                    ┌─────────────────┐
                    │   Navegador     │
                    └────────┬────────┘
                             │ localhost
         ┌───────────────────┼───────────────────┐
         ▼                   ▼                   ▼
   ┌───────────┐      ┌─────────────┐     ┌─────────────┐
   │ Frontend  │      │ API         │     │ API         │
   │ Nginx :80 │      │ Despachos   │     │ Ventas      │
   │ → :3000   │      │ → :8081     │     │ → :8082     │
   └─────┬─────┘      └──────┬──────┘     └──────┬──────┘
         │                   │                   │
         └───────────────────┼───────────────────┘
                             ▼
                    ┌─────────────────┐
                    │ MySQL 8.0       │
                    │ puerto 3306     │
                    └─────────────────┘
              red Docker: innovatech2-network
```

Dentro de la red Docker los servicios se resuelven por nombre: `mysql`, `backend-despachos`, `backend-ventas`, `frontend-despacho`.

---

## Requisitos

- [Docker Desktop](https://www.docker.com/products/docker-desktop/) (incluye `docker compose`)
- Git
- ~4 GB de RAM libres recomendados para levantar todos los contenedores

---

## Inicio rápido

```bash
git clone https://github.com/IgnacioLondono/innovatech-proyecto-deploy.git
cd innovatech-proyecto-deploy
docker compose up -d --build
```

Comprobar estado:

```bash
docker compose ps
```

| Servicio | URL / host |
|----------|------------|
| Frontend | http://localhost:3000 |
| API Despachos | http://localhost:8081 |
| API Ventas | http://localhost:8082 |
| MySQL | `localhost:3306` (usuario `admin` / BD `innovatech_db` según `docker-compose.yml`) |

Documentación OpenAPI (Swagger UI) suele estar en rutas tipo `/swagger-ui.html` según cada API.

Detener y opcionalmente borrar volúmenes:

```bash
docker compose down
docker compose down -v   # elimina datos de MySQL en el volumen
```

---

## Estructura de carpetas

```
innovatech-proyecto-deploy/
├── .github/workflows/          # deploy-front, deploy-despachos, deploy-ventas
├── back-Despachos_SpringBoot/
│   └── Springboot-API-REST-DESPACHO/
├── back-Ventas_SpringBoot/
│   └── Springboot-API-REST/
├── front_despacho/
├── docker-compose.yml
└── README.md
```

---

## Variables de entorno

En local, credenciales y JDBC se definen en `docker-compose.yml` (`MYSQL_*`, `SPRING_DATASOURCE_*`). Los `application.properties` de Spring permiten sobreescritura con variables (`DB_ENDPOINT`, `DB_PORT`, etc.) para entornos como RDS.

**Producción:** no versiones secretos reales; usa GitHub Secrets y variables en el servidor o en AWS.

---

## CI/CD (GitHub Actions)

Los workflows se disparan con **push a la rama `deploy`** y rutas filtradas por carpeta:

- `front_despacho/**` → build, push a ECR y comando SSM en instancia de frontend
- Cambios bajo cada backend → pipeline equivalente para esa API

Secrets típicos: `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, `AWS_SESSION_TOKEN` (si aplica), `AWS_REGION`, URLs de ECR, IDs de instancia EC2, etc. (ver cada YAML en `.github/workflows/`).

---

## Comandos útiles

```bash
docker compose logs -f backend-despachos
docker compose logs -f mysql
docker compose build --no-cache
docker compose restart backend-ventas
```

---

## Documentación por módulo

- [Backend Despachos](back-Despachos_SpringBoot/Springboot-API-REST-DESPACHO/README.md)
- [Backend Ventas](back-Ventas_SpringBoot/Springboot-API-REST/README.md)
- [Frontend](front_despacho/README.md)

---

## Licencia y uso

Proyecto académico / Innovatech. Ajusta licencia y créditos según lo exija tu institución o equipo.

---

*Última revisión del README: mayo de 2026.*
