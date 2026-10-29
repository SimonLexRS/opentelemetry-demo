# Despliegue en Dokploy

Stack completo (equivale a `make start`): core + `compose.full.yaml` (Kafka, accounting, fraud-detection) + `compose.observability.yaml` (Jaeger, Grafana, Prometheus, OpenSearch).
`dokploy/compose.yaml` es un override: no se editan los archivos upstream, así que `git merge upstream/main` sigue limpio.

Requisitos: unos 8 GB de RAM libres en el servidor y un registro DNS A del dominio apuntando a su IP.

## Pasos

1. **Create Service → Compose**
   - Provider: GitHub, repo `SimonLexRS/opentelemetry-demo`, rama `main`.
   - Compose Path: `./dokploy/compose.yaml`. Así Dokploy escribe su `.env` en `dokploy/.env` y no pisa el `.env` de la raíz.
2. **Environment**: pega `dokploy/.env.example` con tu dominio real.
3. **Advanced → Command**: reemplaza el comando por defecto. Conserva el `-p <appName>` que muestra la UI:

   ```
   compose -p <appName> -f compose.yaml -f compose.full.yaml -f compose.observability.yaml -f dokploy/compose.yaml --env-file .env --env-file .env.override --env-file dokploy/.env up -d --no-build --remove-orphans
   ```

   El comando por defecto usa `--build` y compilaría los ~20 servicios. Con `--no-build` se usan las imágenes de `ghcr.io/open-telemetry/demo`.
4. **No** uses la pestaña Domains ni "Isolated Deployments": las labels de Traefik ya están en el override, y la opción isolated reescribe las redes.
5. Deploy.

## URLs

| Ruta | Qué es |
|---|---|
| `https://<dominio>/` | Tienda (Astronomy Shop) |
| `/grafana/` | Grafana |
| `/jaeger/ui/` | Jaeger |
| `/loadgen/` | Locust (generador de carga) |
| `/feature/` | flagd-ui (feature flags) |

Solo `frontend-proxy` (Envoy) es accesible desde fuera; el resto de servicios no publica puertos en el host.

## Probar en local

```bash
docker network create dokploy-network
cp dokploy/.env.example dokploy/.env
docker compose -p otel-demo -f compose.yaml -f compose.full.yaml -f compose.observability.yaml -f dokploy/compose.yaml --env-file .env --env-file .env.override --env-file dokploy/.env config
```
