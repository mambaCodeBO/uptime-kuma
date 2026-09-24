# Uptime Kuma

Monitoramento 24/7 dos serviços da Mamba, rodando na VPS. Checa cada serviço a
cada 60s, guarda o histórico e calcula o uptime real de 24h, mesmo sem ninguém
com o painel aberto.

Quem consome: o [PainelHealthCheck](https://github.com/mambaCodeBO/PainelHealthCheck),
que lê a Status Page pública do Kuma e mostra o "Uptime 24h" em cada card. Se o
Kuma cair ou travar, o painel volta sozinho a checar direto do navegador.

```
Dokploy: Traefik (HTTPS + CORS) → Uptime Kuma → SQLite (volume kuma-data)
                                      ▲
Painel (Render) ── GET /api/status-page/heartbeat/painel
```

## Subir no Dokploy

1. **DNS:** crie um registro A `status.mambaads.com.br` → IP da VPS do Dokploy.
2. **Dokploy:** Project → Create Service → **Compose** (tipo Docker Compose).
   - Provider: GitHub, este repo, branch `main`, Compose Path `./docker-compose.yml`.
3. **Environment:** cole o conteúdo do `.env.example` com os valores reais
   (`KUMA_DOMAIN` e `PAINEL_ORIGIN`).
4. **Deploy.** Não use a aba Domains: o domínio, o HTTPS (Let's Encrypt) e o
   CORS já estão nas labels do Traefik no `docker-compose.yml`.
5. Abra `https://<KUMA_DOMAIN>`. O certificado pode levar alguns segundos na
   primeira vez.

Fora do Dokploy (Docker puro), é preciso outro proxy na frente fazendo o mesmo
que as labels: HTTPS, proxy para a porta 3001 (com WebSocket) e o header
`Access-Control-Allow-Origin: <PAINEL_ORIGIN>` em `/api/status-page/*`.

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

- **Backup:** todo o histórico fica no volume `kuma-data` (SQLite). Configure o
  backup do volume no Dokploy (Volume Backups) ou no backup da VPS.
- **Atualizar:** Redeploy no Dokploy (a tag `:1` puxa a versão 1.x mais nova).
- **Logs:** aba Logs do serviço no Dokploy.
- **Kuma fora do ar:** o painel mostra "Monitor 24h indisponível" e segue
  funcionando no modo local até o Kuma voltar.
