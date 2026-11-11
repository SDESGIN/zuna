# Testes da ZUNA V21

- `node --check` em `netlify/functions/api.mjs`, `netlify/lib/security.mjs` e `web/app.js`: **aprovados**.
- `node --test tests/*.test.mjs`: **13/13 testes aprovados** (regras unitárias e checagens estáticas).
- Tentativa de abertura com Chromium headless neste ambiente: **não concluiu** (travamento do navegador). Portanto **não** foi considerada validação visual aprovada.
- Não foi possível instalar/executar Netlify Database e Blobs com credenciais de produção nesta sessão. Os novos fluxos do banco e do armazenamento **não passaram** por teste de integração real.
- Não foram executados testes com vídeo de 3 GiB, transcodificação, produção ou múltiplos dispositivos.

Não compare os 13 testes novos aos 144 testes da V20: a V21 usa arquitetura e conjunto de testes diferentes, e não executa a antiga suíte de integrações com Supabase/Cloudflare.
