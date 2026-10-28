# Consumo dos contratos atuais pelo frontend

Objetivo: consumir uma página Laravel por navegação e refletir somente filtros, campos e operações aceitos pelo backend atual. Preservar as correções anteriores. O usuário autorizou explicitamente implementar pesquisa no backend durante a execução.

1. Criar provas de regressão para metadata, navegação, respostas fora de ordem, inclusão/atualização de estoque e validações divergentes.
2. Mapear `meta.current_page`, `meta.per_page`, `meta.last_page` e `meta.total` nos serviços de ofertas, históricos e wishlist. Eliminar carregamento de todas as páginas e paginação local.
3. Compartilhar controle de carregamento entre as cinco telas paginadas: uma requisição por página, descarte de respostas antigas, correção da página caso total diminua e erro visível.
4. Usar paginação Laravel em estoques, ofertas, wishlist, compras e vendas. Recarregar após criar/alterar; zerar estoque não deve simular exclusão.
5. Preservar os campos de busca e adicionar `search` opcional no Laravel, com máximo de 255 caracteres. Ofertas filtram nome do produto; históricos filtram nome do produto ou ID textual exato do registro. Aplicar filtro SQL antes da paginação e preservar escopos de usuário/visibilidade.
6. Alinhar campos existentes: limites de cadastro, endereço alfanumérico, estoque inicial zero, saldo disponível retornado e quantidade inteira na compra.
7. Executar testes frontend, build, suíte completa Laravel e revisão do diff.

Limites: sem endpoints novos, sem implementar telas de entregas/analytics/imagens, sem alterar banco, sem testes de pagamento real. A pesquisa de ofertas/históricos está incluída por autorização explícita; filtros por fornecedor, região, pagamento e status não estão incluídos.


## Resultado e comprovação

- Cinco telas usam uma página real da API, com `meta.current_page`, `meta.per_page`, `meta.last_page` e `meta.total`. Não baixam todas as páginas nem repaginam no navegador.
- Pesquisa é enviada em `search`, preservada ao navegar e reinicia na primeira página ao mudar. Campos limitados a 255 caracteres.
- `usePaginatedList` compartilha descarte de respostas antigas, erro visível e recuperação da última página após exclusões.
- Cadastro e alteração de estoque recarregam a página. Zerar saldo emite atualização, sem simular exclusão.
- Cadastro aceita nomes com dígitos e número de endereço alfanumérico, aplica limites do Laravel e não inventa referência de endereço. Oferta permite estoque inicial zero e limita colheita à data mínima UTC do backend. Compra usa saldo retornado e quantidade inteira.

Provas:

- Frontend Red: `node tests/pagination.test.js` reproduziu ausência de metadata, acréscimo local de estoque e navegação durante carregamento. Testes adicionais reproduziram validações divergentes e falta da data mínima.
- Frontend Green: `node tests/pagination.test.js` — 18 testes; `node tests/laravel-integration.test.js` — 15; `node tests/product-create.test.js` — 3; `node tests/app-select.test.js` — 2.
- `npm run build` passou. Repositórios sem erros em `git diff --check`.
- Backend Red: `docker compose exec -T php php artisan test --filter=test_should_search` — quatro testes novos falharam por totais não filtrados.
- Backend Green: mesmo comando — 6 testes, 47 assertions; suíte `docker compose exec -T php php artisan test` — 198 testes, 725 assertions.
- Revisão focada não encontrou bugs bloqueantes; limites dos campos de pesquisa e da data de colheita foram alinhados após revisão.

Sem alterações de schema, dependências ou rotas novas. As três rotas GET existentes passaram a aceitar `search`. Não houve validação interativa no navegador nem implementação das funcionalidades ausentes listadas em `docs/rotas-backend-pendentes.md`.
