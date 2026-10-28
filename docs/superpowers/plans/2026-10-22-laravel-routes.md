# Alinhamento do frontend com Laravel

**Objetivo:** conferir todas as integrações de `frontend/src` e usar os contratos atuais de `backend/routes/api` nas funcionalidades existentes.

**Arquitetura:** manter `apiRequest` como cliente Laravel, o catálogo no `product-source` e os formatos das telas adaptados nos serviços existentes. Não criar endpoints nem telas novas. Preservar alterações anteriores.

**Tecnologias:** Vue 3, Vite, Fetch, testes Node, Laravel.

## Plano

- [x] Inventariar rotas com `php artisan route:list --path=v1 --json` e conferir Requests/Resources.
- [x] Criar testes de contrato para autenticação, ofertas, categorias, CEP, wishlist, checkout e históricos; confirmar falhas nas incompatibilidades.
- [x] Migrar wishlist para CRUD Laravel, com busca e paginação. Não apagar dados locais anteriores nem transferi-los automaticamente para o servidor.
- [x] Usar consulta de CEP Laravel; manter lista de estados BrasilAPI, pois não existe rota equivalente.
- [x] Enviar capacidade do veículo no cadastro de entregadores, remover verificações fictícias de disponibilidade e alinhar mensagens ao formato de erros Laravel.
- [x] Retirar descrição/IA do cadastro de oferta, pois o contrato não recebe esses dados. Identificar a ação existente de estoque como zerar saldo disponível, sem prometer exclusão.
- [x] Validar contratos das integrações que já estavam corretas, executar testes relevantes Laravel e build frontend.

## Inventário

| Fluxo | Contrato Laravel / destino | Situação encontrada |
|---|---|---|
| Login / cadastro | POST `/v1/auth/login`, `/v1/auth/signup` | Rotas corretas; cadastro de entregador omite capacidade |
| Opções do cadastro | GET `/v1/auth/signup/metadata` | Valores locais correspondem aos enums; sem chamada antiga |
| CEP | GET `/v1/consult/cep/{cep}` | Frontend ainda acessa BrasilAPI diretamente |
| Estados | BrasilAPI `/ibge/uf/v1` | Sem rota Laravel correspondente |
| Ofertas | GET/POST `/v1/offers`, GET/PATCH `/v1/offers/{id}` | Rotas corretas; remoção é ajuste de estoque, sem DELETE |
| Categorias | GET `/v1/products/categories` | Correto |
| Catálogo / preços | `product-source:8001/api/products/...` | Destino externo solicitado pelo usuário |
| Wishlist | GET/POST `/v1/wishlist/items`, DELETE `/v1/wishlist/items/{id}` | Implementação local a migrar |
| Compra | POST `/v1/offers/{id}/purchase` | Correto; rascunho do carrinho permanece local |
| Históricos | GET `/v1/purchases`, `/v1/sales` | Corretos |
| Perfil | Resposta de autenticação | Não há rota de leitura/edição de perfil |
| Analytics | GET `/v1/wishlist/analytics` | Tela ainda é placeholder, sem integração antiga |
| Entregas | GET `/v1/delivery/trips`, `/current`, `/finished`; POST assign/cancel/confirm | Telas ainda são placeholders, sem integração antiga |
| Imagens / descrição / IA | Sem rota Laravel | Imagens desconectadas do dashboard; descrição/IA ativas sem persistência |

## Verificação e limites

Conferir token Bearer, erros 422/409, IDs textuais do catálogo, paginação, DELETE sem corpo de resposta e checkout parcialmente concluído. Testes não devem enviar dados ao servidor real. Os testes Laravel usam o ambiente de teste existente e simulam apenas serviços externos.

Rollback: reverter as alterações de frontend; nenhuma migração de banco ou exclusão de dados locais será executada. Dados antigos da wishlist local não aparecem automaticamente na lista do servidor.


## Resultado

Foram comparadas as 22 rotas Laravel com todas as chamadas do frontend ativo (`frontend/src`). Os componentes de imagens desconectados e o diretório legado `frontend/bad` não são usados pelo dashboard atual. Telas de analytics e entregas continuam como placeholders; não havia chamadas antigas nesses fluxos para substituir.

A revisão identificou e corrigiu paginação após inclusão na wishlist, falhas HTTP silenciosas na listagem/remoção, ausência do helper de nomes no card e consultas simultâneas de CEP. A consulta CEP mantém o bloqueio do campo enquanto busca; a segunda chamada simultânea é ignorada.

Validação:

- Red: `cd frontend && node tests/laravel-integration.test.js` reproduziu omissão de capacidade, CEP fora do Laravel, wishlist local, ausência de mensagens de validação e nome enganoso da operação de estoque.
- Green: mesmo comando passou com 15 testes. Inclui preservação de IDs textuais, Bearer, DELETE sem JSON, conflito 409, paginação e checkout parcial.
- `node tests/product-create.test.js`: 3 testes passaram.
- `node tests/app-select.test.js`: 2 testes passaram.
- `npm run build`: passou.
- `cd backend && docker compose exec -T php php artisan test tests/Feature/domain/Auth tests/Feature/domain/Integrations/Geo tests/Feature/domain/Models/Offer tests/Feature/domain/Models/Category tests/Feature/domain/Models/Purchase tests/Feature/domain/Models/Wishlist`: 102 testes passaram, 404 assertions.
- Revisão independente do diff: sem problemas restantes identificados após correções.

Não foi feita validação interativa no navegador. Wishlist local anterior permanece armazenada, sem importação automática. Estados continuam na BrasilAPI e catálogo/preços no product-source por não terem substituição Laravel no contrato atual.
