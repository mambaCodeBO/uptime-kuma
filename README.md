# Uptime Kuma

Monitoramento 24/7 dos serviços da Mamba, rodando na VPS. Checa cada serviço a
cada 60s, guarda o histórico e calcula o uptime real de 24h, mesmo sem ninguém
com o painel aberto.

Quem consome: o [PainelHealthCheck](https://github.com/mambaCodeBO/PainelHealthCheck),
que lê a Status Page pública do Kuma e mostra o "Uptime 24h" em cada card. Se o
Kuma cair ou travar, o painel volta sozinho a checar direto do navegador.

```
VPS: Caddy (HTTPS + CORS) → Uptime Kuma → SQLite (./data)
                                 ▲
Painel (Render) ── GET /api/status-page/heartbeat/painel
```

## Subir na VPS

Pré-requisitos: Docker com Compose, portas 80/443 livres e um DNS
(ex.: `status.mambaads.com.br`) apontando para a VPS.

```bash
git clone <este repo> uptime-kuma && cd uptime-kuma
cp .env.example .env   # preencha KUMA_DOMAIN e PAINEL_ORIGIN
docker compose up -d
```

O Caddy emite o certificado HTTPS sozinho na primeira requisição.

**Já tem Nginx/Caddy nas portas 80/443?** Remova o serviço `caddy` do compose,
exponha o Kuma com `ports: ["127.0.0.1:3001:3001"]` e replique no seu proxy o
que o `Caddyfile` faz: proxy para a porta 3001 (com WebSocket) e o header
`Access-Control-Allow-Origin: <PAINEL_ORIGIN>` nas rotas `/api/status-page/*`.

## Configurar (uma vez)

1. Acesse `https://<KUMA_DOMAIN>` e crie o usuário admin.
2. Crie um monitor **HTTP(s)** para cada card do painel. Use Heartbeat Interval
   de 60s e Retries 2 (evita falso positivo).
3. Crie uma **Status Page** com slug `painel` e adicione todos os monitores.
4. (Opcional) Configure alertas em Settings > Notifications (Slack etc.).

O painel casa card e monitor **pelo nome**. O Friendly Name precisa ser igual ao
título do card (maiúsculas não importam). Para usar outro nome, coloque
`kumaId: <id do monitor>` no card em `src/App.vue` do painel.

| Friendly Name | URL |
|---|---|
| API MeLi | https://api-meli-590ef52def2f.herokuapp.com/health |
| CRM Service | https://crm-service-feb2ad0b78a9.herokuapp.com/ping |
| Link de pagamento | https://payments-api1-fc54d7729f5b.herokuapp.com/ping |
| Integração OMIE | https://api-financial-fe5ab7a5fe25.herokuapp.com/health |
| API Nutshell | https://webhooknutshell-8bb84dbf9d18.herokuapp.com/health |
| API ConnectDuo | https://connectduo-api-fb0284a0db8b.herokuapp.com/health |
| API Mamba Nexus | https://api.mambanexus.com.br/health |
| NestOps | https://internal-api-97783fd8f8da.herokuapp.com/health |
| Renê Academy | https://rene-academy-n8n.mambanexus.com.br/healthz |
| Renê Academy Painel | https://rene-academy-painel-api.mambanexus.com.br/health |
| Renê GO | https://rene-basic-n8n.mambanexus.com.br/healthz |
| Bryan Evolution | https://bryan-evolution.mambanest.com.br |
| Bryan Painel | https://bryan-painel-api.mambanest.com.br/health |
| Identity Service | https://identity-api.mambanest.com.br/actuator/health |

Card novo no painel = monitor novo aqui, com o mesmo nome. Enquanto não existir,
o card continua sendo checado pelo navegador.

## Ligar no painel

No Render (variáveis de build do painel) e depois um novo deploy:

```
VITE_KUMA_URL=https://status.mambaads.com.br
VITE_KUMA_SLUG=painel
```

Para testar a API: `curl https://<KUMA_DOMAIN>/api/status-page/heartbeat/painel`.

## Operação

- **Backup:** todo o histórico fica em `./data` (SQLite). Inclua essa pasta no
  backup da VPS.
- **Atualizar:** `docker compose pull && docker compose up -d`.
- **Logs:** `docker compose logs -f uptime-kuma`.
- **Kuma fora do ar:** o painel mostra "Monitor 24h indisponível" e segue
  funcionando no modo local até o Kuma voltar.
