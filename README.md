# Uptime Kuma (uptime de 24h do painel)

O Kuma roda na VPS e checa os serviços 24/7, mesmo com o painel fechado. O painel
lê a Status Page pública dele e mostra o uptime real de 24h.

**Fallback:** se o Kuma não responder, ou se o heartbeat mais novo tiver mais de
`VITE_KUMA_STALE_MS` (3 min por padrão), o painel volta sozinho a checar direto
do navegador, como antes, e mostra o aviso "Monitor 24h indisponível". Quando o
Kuma volta, o painel volta a usá-lo no ciclo seguinte.

## 1. Subir na VPS

Pré-requisitos: Docker com Compose, portas 80/443 livres e um DNS
(ex.: `status.mambaads.com.br`) apontando para a VPS.

```bash
cd infra/uptime-kuma
cp .env.example .env   # preencha KUMA_DOMAIN e PAINEL_ORIGIN
docker compose up -d
```

Se a VPS já tiver Nginx/Caddy nas portas 80/443, remova o serviço `caddy` do
compose, exponha o Kuma com `ports: ["127.0.0.1:3001:3001"]` e replique no seu
proxy o que o `Caddyfile` faz: proxy para a porta 3001 (com WebSocket) e o
header `Access-Control-Allow-Origin` nas rotas `/api/status-page/*`.

## 2. Configurar o Kuma (uma vez)

1. Acesse `https://<KUMA_DOMAIN>` e crie o usuário admin.
2. Para cada card do painel, crie um monitor **HTTP(s)**:
   - **Friendly Name igual ao título do card** (ex.: `API MeLi`, `Renê GO`). É
     assim que o painel casa card e monitor, sem diferenciar maiúsculas. Se
     preferir outro nome, coloque `kumaId: <id do monitor>` no card em `src/App.vue`.
   - URL: a mesma do card.
   - Heartbeat Interval: 60s. Retries: 2 (evita falso positivo).
3. Crie uma **Status Page** com slug `painel` e adicione todos os monitores.
4. (Opcional) Configure notificações (Slack etc.) em Settings > Notifications.

Cards sem monitor correspondente no Kuma continuam sendo checados pelo navegador.

## 3. Painel (Render)

Configure as variáveis de build e faça um novo deploy:

```
VITE_KUMA_URL=https://status.mambaads.com.br
VITE_KUMA_SLUG=painel
```

Deixar `VITE_KUMA_URL` vazio desliga o Kuma: o painel volta ao modo local.

## Backup

Todo o histórico fica em `infra/uptime-kuma/data` (SQLite). Inclua essa pasta no
backup da VPS.
