# Observabilidade — Raspberry Pi 5

## Pré-requisitos

```bash
# Docker e Docker Compose instalados
# Usuário no grupo docker
sudo usermod -aG docker $USER
```

## Subir a stack

```bash
cd ~/pi-observability
docker compose up -d

# Ver status
docker compose ps

# Ver logs de um serviço
docker compose logs -f grafana
```

## Acessar

| Serviço    | URL                          | Usuário/Senha        |
|------------|------------------------------|----------------------|
| Grafana    | http://<IP-DO-PI>:3000       | admin / changeme123  |
| Prometheus | http://<IP-DO-PI>:9090       | sem autenticação     |
| cAdvisor   | http://<IP-DO-PI>:8080       | sem autenticação     |

## Primeiro acesso ao Grafana

1. Abra http://<IP-DO-PI>:3000 no browser
2. Login: admin / changeme123
3. Datasources já estarão configurados (Prometheus + Loki)
4. Importar dashboards prontos:
   - Node Exporter Full: ID 1860
   - Docker cAdvisor:    ID 14282
   - Loki Logs:          ID 13639

## Ajustar o volume de logs do PM2

No docker-compose.yml, linha do promtail:
```yaml
- /home/SEU_USUARIO/.pm2/logs:/pm2-logs:ro
```

## Reload do Prometheus sem restart

```bash
curl -X POST http://localhost:9090/-/reload
```
