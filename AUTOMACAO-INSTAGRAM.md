# Automação do Instagram — @direitobinario

Este repositório contém um fluxo de automação agendado para pesquisa, criação de artes e publicação no Instagram. **O código ainda não está operacional até que os secrets sejam adicionados e o primeiro teste seja verificado.** Não há tokens gravados nos arquivos.

## Agenda

O GitHub Actions usa cron em UTC; os horários abaixo correspondem ao horário de Brasília (America/Sao_Paulo, UTC-3):

- 08h: pesquisa de notícias, geração de três pautas/legendas e criação dos ativos visuais.
- 09h: imagem no Feed.
- 12h: Reel vertical, publicado também no Feed.
- 19h: Story.

O agendamento do GitHub pode sofrer atrasos ocasionais. O fluxo é diário e não garante execução no segundo exato.

## O que precisa ser configurado uma única vez

No repositório privado, abra **Settings → Secrets and variables → Actions → New repository secret** e adicione:

- `IG_USER_ID`: ID numérico da conta profissional do Instagram.
- `META_ACCESS_TOKEN`: token válido da Meta com permissão de publicação para a conta.
- `OPENAI_API_KEY`: chave da API da OpenAI. A geração de texto e imagem pode gerar custos de API.
- `CLOUDINARY_CLOUD_NAME`
- `CLOUDINARY_API_KEY`
- `CLOUDINARY_API_SECRET`

A chave do Instagram atualmente conectada ao ChatGPT não é exportada automaticamente para o GitHub. É necessário cadastrar um token próprio da Meta no secret `META_ACCESS_TOKEN`. Nunca cole tokens neste arquivo, em issues ou no chat.

Opcionalmente, em **Settings → Secrets and variables → Actions → Variables**, defina:
- `META_GRAPH_API_VERSION`: padrão `v26.0`.
- `OPENAI_TEXT_MODEL`: padrão `gpt-4.1-mini`.
- `OPENAI_IMAGE_MODEL`: padrão `gpt-image-1`.
- `META_GRAPH_API_BASE` não é consumida pelo workflow como variável atualmente; a base padrão do script é `https://graph.facebook.com`.

A conta precisa ser profissional e o token deve corresponder ao método de login/API e permissões habilitados no app Meta. A conta conectada foi verificada como Business pelo conector, mas isso não comprova que o token separado do GitHub já tenha permissão de publicação.

## Primeiro teste, sem publicar

1. Adicione os secrets acima.
2. Abra **Actions → Instagram Auto Publishing → Run workflow**.
3. Execute com `mode=healthcheck` e `dry_run=true`.
4. Confirme que o perfil e o limite de publicação são retornados sem erro.

O healthcheck só comprova leitura da conta e quota. A permissão de escrita só ficará comprovada após uma publicação real, que deve ser conferida no Instagram e no log da execução.

## Ativação e segurança

- Depois de configurar os secrets, o cron executa automaticamente: primeiro prepara a fila e depois publica nos três horários.
- O primeiro teste de publicação pode ser iniciado manualmente selecionando `mode=publish` e `dry_run=false`; isso publica conteúdo real. Alternativamente, aguarde o próximo horário programado.
- Antes de publicar, o script registra um estado `publishing`. Se a API responder de forma ambígua ou houver timeout, o slot não é repetido automaticamente, para reduzir o risco de duplicação. Verifique a conta e o arquivo `state/YYYY-MM-DD.json` antes de qualquer nova tentativa.
- A fila diária e o estado de publicação são gravados no repositório privado; os ativos são hospedados no Cloudinary para que a Meta consiga buscá-los.
- O fluxo gera três peças por dia. Reels são montados como vídeo vertical curto a partir da arte, com movimento leve; não há trilha sonora automática.
- O conteúdo usa notícias de RSS e links de fonte; o texto deve ser tratado como resumo editorial, não aconselhamento jurídico individual. Confira os resultados iniciais antes de confiar na operação contínua.

## Limitações conhecidas

- O cron do GitHub Actions não garante execução exata ao minuto.
- Se a pesquisa não encontrar três pautas adequadas, a preparação falha em vez de inventar notícias.
- Tokens da Meta podem expirar ou perder permissões. O fluxo não pode ser considerado ativo até o healthcheck passar e uma publicação real ser confirmada.
- Stories via API dependem de conta profissional compatível e permissões adequadas.
