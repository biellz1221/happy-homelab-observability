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
| Grafana    | http://<IP-DO-PI>:3030       | admin / changeme123  |
| Prometheus | http://<IP-DO-PI>:9090       | sem autenticação     |
| cAdvisor   | http://<IP-DO-PI>:8081       | sem autenticação     |

> Em produção o Grafana é exposto via Cloudflare Tunnel em `status.happyhomelab.me`.
> A senha vem de `GRAFANA_ADMIN_PASSWORD` no `.env` (default `changeme123`).

## Primeiro acesso ao Grafana

Datasources **e** dashboards são provisionados automaticamente — não precisa
configurar nada na UI.

1. Abra http://<IP-DO-PI>:3030 e faça login (admin / `$GRAFANA_ADMIN_PASSWORD`)
2. Datasources já carregados: **Prometheus** (uid `prometheus`, default) e **Loki** (uid `loki`)
3. Dashboards já carregados na pasta **Homelab** (menu Dashboards):

| Dashboard            | Fonte                | Cobre                                      |
|----------------------|----------------------|--------------------------------------------|
| Node Exporter Full   | grafana.com ID 1860  | Host/Pi: CPU, RAM, disco, rede, temperatura |
| Docker / cAdvisor    | grafana.com ID 14282 | CPU/RAM/rede por container                  |
| Loki Logs            | grafana.com ID 13639 | Logs (syslog, Docker, PM2)                  |
| PostgreSQL           | grafana.com ID 9628  | Conexões, queries, locks                    |
| Redis                | grafana.com ID 763   | Memória, hits/misses, clients               |
| NestJS APIs          | custom (este repo)   | 4 APIs Node.js: heap, event loop, GC, CPU   |

## Como o provisionamento funciona

```
config/grafana/
├── provisioning/
│   ├── datasources/datasources.yml   # Prometheus + Loki (uid fixo)
│   └── dashboards/dashboards.yml      # provider: lê JSONs de /var/lib/grafana/dashboards
└── dashboards/*.json                  # dashboards versionados (montados no container)
```

- Os JSONs da comunidade têm o placeholder de datasource resolvido para os uids
  `prometheus` / `loki`, então funcionam sem prompt de import.
- O provider re-escaneia a pasta a cada 30s: editar/adicionar um JSON e salvar já
  reflete no Grafana, sem restart.
- `allowUiUpdates: true` — você pode ajustar painéis na UI sem o provisioning sobrescrever.

### Adicionar um novo dashboard da comunidade

```bash
# Baixa o JSON e resolve o datasource (ajuste ID e placeholder conforme o dashboard)
curl -sL https://grafana.com/api/dashboards/<ID>/revisions/latest/download \
  -o config/grafana/dashboards/<nome>.json
sed -i 's/${DS_PROMETHEUS}/prometheus/g; s/${DS_LOKI}/loki/g' config/grafana/dashboards/<nome>.json
# Aparece sozinho em até 30s (ou docker compose restart grafana)
```

## Ajustar o volume de logs do PM2

No docker-compose.yml, linha do promtail:
```yaml
- /home/SEU_USUARIO/.pm2/logs:/pm2-logs:ro
```

## Reload do Prometheus sem restart

```bash
curl -X POST http://localhost:9090/-/reload
```
