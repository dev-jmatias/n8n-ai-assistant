# n8n AI Assistant Stack

Stack Docker Compose preparada para deployment posterior pelo Portainer, usando Traefik já existente.

## Serviços

- n8n
- PostgreSQL 17
- n8n Task Runners em modo externo (sandbox/isolamento do Code node)
- SearXNG para pesquisa web de agentes
- Valkey para o limiter do SearXNG
- Rede interna `backend`
- Rede externa do Traefik

O n8n já inclui os nós de Advanced AI/AI Agent. O SearXNG pode ser usado pelo nó SearXNG Tool/AI Agent ou via HTTP internamente em `http://searxng:8080`.

## Antes de fazer Deploy no Portainer

Edite/substitua as variáveis da `.env` (ou configure-as no Environment variables do Portainer):

- `N8N_HOST`
- `SEARXNG_HOST`
- `TRAEFIK_NETWORK`
- `TRAEFIK_HTTPS_ENTRYPOINT`
- `TRAEFIK_CERTRESOLVER`
- `POSTGRES_PASSWORD`
- `N8N_ENCRYPTION_KEY`
- `N8N_RUNNERS_AUTH_TOKEN`
- `SEARXNG_SECRET`
- `OPENAI_API_KEY`

Nunca coloque segredos reais no repositório se ele permanecer público.

## Gerar segredos

```bash
openssl rand -hex 32
```

Use valores diferentes para PostgreSQL, `N8N_ENCRYPTION_KEY`, runner token e SearXNG.

## Rede Traefik

A stack pressupõe que o Traefik já está em execução e conectado a uma rede Docker externa. Confirme o nome:

```bash
docker network ls
```

Depois coloque esse nome em `TRAEFIK_NETWORK`.

## Portainer

1. **Stacks > Add stack**.
2. Escolha **Repository**.
3. Repository URL: `https://github.com/dev-jmatias/n8n-ai-assistant.git`.
4. Compose path: `compose.yaml`.
5. Configure as variáveis da `.env` no Portainer antes do deploy, principalmente os segredos e domínios.
6. Deploy the stack.

## DNS

Antes do deploy público, crie os registos DNS para `N8N_HOST` e, se quiser expor a interface do SearXNG, `SEARXNG_HOST`, apontando para o VPS.

## AI / Agentes

Depois do primeiro login no n8n:

1. Crie a credencial OpenAI no n8n (recomendado em vez de depender apenas da variável de ambiente).
2. Crie um workflow com **AI Agent**.
3. Ligue um **OpenAI Chat Model** ao Agent.
4. Adicione ferramentas, incluindo **SearXNG Tool**, HTTP Request, Call n8n Workflow Tool ou outras ferramentas adequadas ao agente.
5. Para SearXNG dentro da stack, use `http://searxng:8080` quando o node/credential solicitar a URL da instância.

## Sandbox / Task Runners

O `n8n-runners` está separado do processo principal e o n8n está configurado com `N8N_RUNNERS_MODE=external`. Isto isola a execução suportada do Code node do container principal do n8n. Não exponha a porta broker `5679` ao host ou ao Traefik.

## Atualizações

A versão está definida em `N8N_VERSION` para manter n8n e runners na mesma versão. Atualize ambos através dessa única variável e faça backup antes de upgrades.
