# ZUNA V21 — Netlify + GitHub ONLY

**Nova edição independente**, reconstruída para usar **apenas recursos da Netlify** na execução. O GitHub é usado somente para versionamento e deploy do repositório. Não contém Supabase, Cloudflare Stream, Firebase, Redis, serviços de autenticação externos ou servidor Node para administrar.

## Arquitetura

| Responsabilidade | Serviço |
|---|---|
| Site/PWA | Netlify Hosting (`web/`) |
| Backend | Netlify Functions (`netlify/functions/api.mjs`) |
| Dados relacionais | **Netlify Database**, PostgreSQL gerenciado nativo (`@netlify/database`) |
| Arquivos | **Netlify Blobs** (`@netlify/blobs`) |
| Tarefas agendadas | Netlify Scheduled Functions (`cleanup.mjs`) |
| Repositório/deploy | GitHub + Netlify |

**Atenção:** Netlify Database exige um plano de cobrança por créditos e consome créditos; Functions e Blobs também podem gerar uso cobrado. Usar só Netlify e GitHub **não significa custo zero**.

## Configurar

1. Extraia o ZIP. Suba **todo o conteúdo da raiz** ao GitHub, incluindo `package.json`, `netlify.toml`, `web/` e `netlify/database/migrations/`.
2. Na Netlify, selecione **Add new project → Import from Git** e conecte o repositório GitHub.
3. Confira as configurações: build command `npm run check`, publish directory `web`, Functions `netlify/functions`. Instale dependências automaticamente do `package.json` no deploy.
4. O pacote `@netlify/database` e a migração SQL em `netlify/database/migrations/` integram o banco nativo; confira **Data & Storage → Database** na Netlify após publicar. Em caso de erro, use `netlify database init` com a Netlify CLI atualizada e siga a documentação oficial.
5. Confira o **Data & Storage → Blobs** da Netlify. O store `zuna-v21-media` é criado pelo uso das Functions quando o ambiente estiver configurado.
6. Teste `https://SEU-SITE.netlify.app/api/health` (resposta deve mostrar `version: 21.0`). Depois crie duas contas de teste, habilite perfil público, publique foto/vídeo, siga, curta e comente.
7. Para ter administrador, **crie primeiro a conta pelo aplicativo**, depois no editor SQL do Netlify Database execute, com o usuário certo: `UPDATE z_users SET role='admin' WHERE username='seu_usuario';`. Saia e entre novamente. **Não existem contas ou senhas administrativas padrão.**
8. Ajuste acesso do site para **piloto privado** e não disponibilize a menores. O campo de autodeclaração de idade não equivale a verificação de idade. Faça auditoria e testes antes de abrir para o público.

O site foi projetado para navegar por URLs e abrir `/api/*` usando a configuração do Netlify. Não serve como um site totalmente funcional se você apenas abrir `index.html` em `file://` ou hospedar os arquivos estáticos sem as Functions e o banco.

## Recursos no código

- Cadastro/login por `@usuário` e senha; sem obrigação de e-mail, com código de recuperação único.
- Hash de senha `scrypt`, código de recuperação armazenado por hash, cookies `HttpOnly + Secure + SameSite=Lax`, origem obrigatória nas operações de escrita e limitação básica de tentativas.
- Perfil, bio, foto, conta privada por padrão, mudança de @usuário (30 dias), bloqueio, seguidores (para perfis públicos), exportação JSON dos dados e encerramento de outras sessões.
- Feed público e de seguindo, vídeos no formato original e fotos/carrosséis (até 10 fotos), comentários, respostas, curtidas, favoritos e republicações; links por ID.
- Envio de fotos de até 12 MiB e vídeos MP4/WebM em blocos de 2 MiB, com alvo técnico de até 3 GiB por vídeo; **não é pipeline de transcodificação** e pode ser caro/impraticável para arquivos grandes.
- Histórias de 24h, mensagens entre contas que se seguem mutuamente, sequência 🔥 após trocas em 3 dias, restauração limitada a uma por usuário/mês e geração de um Story de foguinho.
- Playlists públicas ou privadas e painel do criador com contagem de posts, seguidores, curtidas e comentários reais.
- Notificações **dentro do aplicativo**, denúncias, painel administrativo de selos verificados e indicadores **promocionais** separados de métricas orgânicas, auditoria e diagnósticos básicos.
- Limpeza agendada de uploads abandonados (até 24h), sessões e Stories expirados.

## O que mudou em relação à V20

A V20 exigia Supabase e Cloudflare. A V21 **não é somente uma troca de credenciais**: o banco e o backend foram substituídos por uma arquitetura nova. Por isso, **não há migração automática de contas, posts ou mídias antigas da V20**. Faça backup antes de desativar a versão anterior e não apague a base da V20 antes de concluir um plano de migração de dados.

Esta edição implementa as funções sociais essenciais listadas acima, **mas não manteve integralmente** as funções avançadas da V20: lives reais, conversão/transcodificação de vídeos para 1080p/60 fps, Dueto/Costura renderizados como arquivo final, editor multiclipes completo, efeitos faciais, push com o navegador fechado, comunidades ainda exigiriam desenvolvimento específico para a arquitetura Netlify-only.

O novo banco é destinado a uma instalação nova; o SQL V9–V20 do Supabase **não deve ser aplicado** nele. Consulte `MATRIZ_V21.md`.

## Vídeos e limitações

- O sistema **aceita registrar arquivos de até 3 GiB** e divide cada um em blocos de 2 MiB para contornar o limite por requisição da Function. Há um limite de até 3 uploads simultaneamente pendentes e 10 GiB totais de arquivos por conta no piloto.
- A reprodução atende solicitações HTTP `Range` através das Functions, entregando blocos de até 2 MiB. A compatibilidade real deve ser testada em diversos navegadores e arquivos MP4/WebM, incluindo vídeos com metadados no final.
- Um arquivo já gravado em 1080p/60 fps **mantém essa codificação original**. Não há garantia de saída Full HD, 60 fps, streaming adaptativo, antivírus, miniaturas automáticas ou CDN de vídeo especializada.
- Arquivos de 3 GiB exigem **1536 requisições de upload** e muitas requisições de reprodução; esse tamanho **não foi testado** em produção e pode estourar créditos, tempo ou limites da plataforma. Comece com clipes pequenos.
- A segurança e privacidade de cada mídia são verificadas pela API antes de acessar os Blobs; não forneça chaves de blobs diretamente no frontend.

## Testes

Execute `npm run check` ou `npm test` com Node.js 22.13+. Os testes incluem regras de senha/recuperação, formatos/limites, `Range`, cookies e checagens estáticas da arquitetura. **Não equivalem** a testes de integração com Netlify Database/Blobs, publicação real, upload de 3 GiB ou uso em dispositivos móveis.

## Segurança e piloto

A ZUNA é um produto voltado futuramente para público jovem, mas esta implementação **restringe o cadastro do piloto a pessoas que afirmam ter 18 anos ou mais**, sem alegar autenticação etária comprovada. Para liberar crianças/adolescentes, é necessário implementar verificação de idade adequada, controles de privacidade, segurança, moderação e cumprimento das obrigações legais antes do lançamento. Não há políticas legais mágicas embutidas no código.

É necessário ainda um plano de backup/restauração, monitoramento de créditos, moderação humana, segurança de arquivos, controles anti-spam e testes reais de múltiplos dispositivos, conforme `CHECKLIST_LANCAMENTO_V21.md`.
