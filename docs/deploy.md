# Deploy e infraestrutura — POC v2

## Onde roda

Esta POC é um **Cloudflare Worker** que serve assets estáticos (apenas `index.html`).

- URL de produção: `https://exame-video-poc-v2.luanleitecardoso.workers.dev`
- Conta Cloudflare: `luanleitecardoso@gmail.com`
- Worker: `exame-video-poc-v2`

---

## Deploy automático via GitHub

Qualquer push para a branch `main` dispara o deploy automaticamente. Não é necessário rodar `wrangler deploy` manualmente.

**Pipeline:**
```
git push origin main
  → Cloudflare Workers CI (cloudflare-workers-and-pages[bot])
  → npx wrangler deploy
  → novo conteúdo em produção (~1–2 min)
```

**Configuração do pipeline** (fica no Cloudflare Dashboard, não no repositório):

| Campo | Valor |
|---|---|
| Git repository | `luancardosolc/exame-video-poc-v2` |
| Production branch | `main` |
| Build command | *(nenhum)* |
| Deploy command | `npx wrangler deploy` |
| Root directory | `/` |
| Build watch paths | `*` |

> **Por que não existe `wrangler.toml` no repo?** A configuração de build e deploy fica inteiramente no dashboard da Cloudflare. O repo precisa apenas do `index.html`.

---

## Cache: não há cache HTTP

O Worker serve assets estáticos com o header:

```
cache-control: public, max-age=0, must-revalidate
```

Isso significa:
- O edge CDN da Cloudflare pode armazenar a resposta localmente, mas **deve revalidar a cada request**.
- Após um novo deploy, a Cloudflare invalida o cache do edge automaticamente. O próximo request já recebe o novo `index.html`.

### O que é o `cf-cache-status: HIT`?

O header `cf-cache-status: HIT` aparece nas respostas mesmo assim. Isso **não indica conteúdo desatualizado** — é comportamento normal do CDN: a resposta estava no cache do edge, foi revalidada e retornada (ou reusada dentro do `max-age=0` da mesma janela de request). Após um novo deploy, o primeiro request post-deploy já recebe o conteúdo novo.

### Build cache (CI)

Nas settings do Worker há a opção "Build cache: Enabled". Isso é um **cache de artefatos de CI** — acelera builds futuros guardando resultados intermediários (similar ao cache do npm entre pipelines). Não tem nenhuma relação com o conteúdo servido aos usuários.

---

## Verificar o deploy atual

### Via curl

```bash
curl -sI https://exame-video-poc-v2.luanleitecardoso.workers.dev | grep -E 'cf-cache-status|cache-control|etag|last-modified'
```

### Via Cloudflare Dashboard

1. Acessar `https://dash.cloudflare.com` → Workers & Pages → `exame-video-poc-v2`
2. Aba **Deployments** → ver "Active deployment" com Version ID, timestamp e % de tráfego
3. A coluna "Version History" lista todos os deploys com a mensagem do commit e autor

---

## Limpar cache manualmente (se necessário)

Em casos extremos (ex: Cloudflare com comportamento inesperado), é possível limpar o build cache:

1. Dashboard → Worker `exame-video-poc-v2` → aba **Settings** → seção **Build**
2. Clicar em **Clear Cache** ao lado de "Build cache: Enabled"

Para purge do edge CDN, acessar **Caching** → **Cache Rules** na conta Cloudflare. Atualmente nenhuma regra está configurada, então o comportamento é o padrão descrito acima.

---

## Navegação

- [Voltar ao README](../README.md)
- [Contrato da POC v2](contract.md)
