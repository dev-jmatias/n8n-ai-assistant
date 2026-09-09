# n8n AI Assistant + Sandbox Stack

Stack Docker Compose preparada para deployment posterior pelo Portainer, usando um Traefik já existente.

## Serviços

- n8n 2.36.9
- PostgreSQL 17
- n8n external Task Runners para execução isolada dos Code nodes
- n8n Sandbox Service API
- n8n Sandbox Runner Docker-in-Docker
- bootstrap automático de certificados mTLS do Sandbox
- SearXNG com resposta JSON para pesquisa web do AI Assistant/agentes
- Valkey
- Traefik para n8n e SearXNG

## Arquitetura

Internet -> Traefik -> n8n

n8n -> PostgreSQL
n8n -> SearXNG -> Valkey
n8n -> external Task Runners
n8n -> Sandbox API -> Sandbox Runner -> sandboxes isolados

A Sandbox API e o Sandbox Runner NÃO possuem labels Traefik nem portas publicadas no host.

## Antes do Deploy no Portainer

Use a `.env` como modelo e configure os valores no Portainer. Troque todos os `CHANGE_ME`.

Variáveis principais:

- `N8N_HOST`
- `SEARXNG_HOST`
- `TRAEFIK_NETWORK`
- `TRAEFIK_HTTPS_ENTRYPOINT`
- `TRAEFIK_CERTRESOLVER`
- `POSTGRES_PASSWORD`
- `N8N_ENCRYPTION_KEY`
- `N8N_RUNNERS_AUTH_TOKEN`
- `SEARXNG_SECRET`
- `SANDBOX_API_KEYS`
- `N8N_SANDBOX_SERVICE_API_KEY`
- `SANDBOX_API_RUNNER_REGISTRATION_TOKEN`
- `SANDBOX_RUNNER_REGISTRATION_TOKEN`
- `SANDBOX_API_RUNNER_API_KEY`
- `SANDBOX_RUNNER_API_KEYS`
- `OPENAI_API_KEY`

Nunca coloque segredos reais neste repositório enquanto ele for público.

## Pares de segredos obrigatoriamente iguais

`SANDBOX_API_KEYS` = `N8N_SANDBOX_SERVICE_API_KEY`

`SANDBOX_API_RUNNER_REGISTRATION_TOKEN` = `SANDBOX_RUNNER_REGISTRATION_TOKEN`

`SANDBOX_API_RUNNER_API_KEY` = `SANDBOX_RUNNER_API_KEYS`

Gere um valor diferente para cada par:

```bash
openssl rand -hex 32
```

## Sandbox

A integração segue a arquitetura oficial atual do n8n Sandbox Service. A stack cria automaticamente o volume `sandbox_tls` e o serviço `sandbox-certs` gera os certificados mTLS usados entre a Sandbox API e o Runner.

O n8n usa internamente:

- `N8N_INSTANCE_AI_SANDBOX_ENABLED=true`
- `N8N_INSTANCE_AI_SANDBOX_PROVIDER=n8n-sandbox`
- `N8N_SANDBOX_SERVICE_URL=http://sandbox-api:8080`

O Sandbox Runner usa Docker-in-Docker. Nesta stack ele está configurado com `privileged: true`, compatível com o compose oficial simples. Para produção com isolamento mais forte em Linux, o projeto n8n Sandbox Service recomenda `sysbox-runc`. Não exponha as portas 8080, 9090 ou 9091 do Sandbox.

## SearXNG

O ficheiro `searxng-settings.yml` habilita os formatos HTML e JSON. O JSON é necessário para a pesquisa web usada pelo AI Assistant.

URL interna:

```text
http://searxng:8080
```

## Traefik

A stack pressupõe uma rede Docker externa já usada pelo Traefik. Confirme o nome no servidor:

```bash
docker network ls
```

Depois defina esse nome em `TRAEFIK_NETWORK`.

Somente n8n e SearXNG entram na rede externa do Traefik. PostgreSQL, Valkey, Task Runners e Sandbox permanecem na rede interna `backend`.

## Portainer

1. Abra **Stacks > Add stack**.
2. Escolha **Repository**.
3. Repository URL: `https://github.com/dev-jmatias/n8n-ai-assistant.git`
4. Compose path: `compose.yaml`
5. Configure as Environment variables com base na `.env`.
6. Substitua os domínios e todos os segredos.
7. Faça o deploy.

## DNS

Antes do deploy público, crie os registos DNS de `N8N_HOST` e `SEARXNG_HOST` apontando para o VPS.

## OpenAI / AI Agents

Depois do primeiro login no n8n, crie a credencial OpenAI no próprio n8n. A variável `OPENAI_API_KEY` também está disponível como placeholder, mas não deve conter uma chave real no GitHub.

Os recursos de Instance AI, SearXNG e Sandbox já ficam ligados pela stack. Os AI Agents normais do n8n podem usar modelos e ferramentas configurados nos workflows.

## Atualizações

`N8N_VERSION` mantém o n8n e os external Task Runners na mesma versão. Faça backup antes de atualizar. As imagens do Sandbox Service estão fixadas em `1.2.0` para evitar upgrades silenciosos da infraestrutura do sandbox.
