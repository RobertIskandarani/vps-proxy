# vps-proxy — reverse proxy central del VPS

Caddy único en `:80/:443` que rutea por dominio a todas las apps del VPS. Es
**neutral**: no pertenece a ninguna app. Cada app se une a la red externa
`edge` y este proxy la alcanza por nombre de contenedor.

## Estándar multi-app (para todo proyecto en este VPS)

1. **Red neutral compartida** (una vez): `docker network create edge`.
2. **Este proxy** corre solo, dueño del 80/443, en la red `edge`.
3. **Cada app**:
   - `web` y `api` se unen a la red externa `edge`.
   - `postgres`/`redis`/`worker`/`migrate` quedan en una red `internal` privada.
   - **sin** Caddy propio y **sin** publicar puertos al host.
4. Agregar una app = sumar su bloque en `Caddyfile` (por nombre de contenedor,
   p. ej. `mi-app-web-1:3000`) y recargar el proxy.

## Uso en el VPS

```bash
docker network create edge 2>/dev/null || true   # una vez
git clone https://github.com/RobertIskandarani/vps-proxy.git /root/proxy
cd /root/proxy
docker compose up -d
```

Recargar tras editar el `Caddyfile` (sin reiniciar el contenedor):
```bash
git pull
docker exec proxy-caddy-1 caddy reload --config /etc/caddy/Caddyfile
```

## Requisitos
- El DNS de cada dominio apunta a la IP del VPS (Caddy emite el TLS solo).
- Cada app ya está levantada y unida a la red `edge`.
