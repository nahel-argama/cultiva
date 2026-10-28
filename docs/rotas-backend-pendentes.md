# Rotas ausentes no backend Laravel

Conferência em 22/10/2026, comparando `backend/routes/api` com o frontend ativo em `frontend/src` e os serviços antigos em `frontend/bad`.

Este documento registra lacunas; não define um contrato aprovado nem implica que todas as funcionalidades antigas devam ser recriadas. Os caminhos com `/v1` abaixo são candidatos para Laravel. Quando existe uma rota antiga comprovada, ela aparece separadamente.

## 1. Imagens de produtos

Não existe nenhuma destas operações no Laravel atual. Os componentes de imagens em `frontend/src` estão desconectados do dashboard e importam um serviço inexistente, `@/services/defaultProductImages`. A implementação antiga desse serviço permanece em `frontend/bad`.

| Operação ausente | Rota candidata Laravel | Rota comprovada no serviço antigo | Entrada/saída esperada pelo frontend antigo |
|---|---|---|---|
| Listar imagens padrão | `GET /v1/default-product-images` | `GET /api/default-product-images/` | Query `page`; coleção paginada com identificação do produto, nome e URL da imagem |
| Enviar ou substituir imagem | `POST /v1/default-product-images/{product_id}/image` | `POST /api/default-product-images/{id}/image/` | `multipart/form-data`, campo `image`; o cliente antigo converte DataURL em arquivo |
| Excluir imagem do produto | `DELETE /v1/default-product-images/{product_id}/image` | `DELETE /api/default-product-images/{productExternalKey}/image/` | Identificador textual do catálogo; cliente não exige corpo na resposta |
| Consultar disponibilidade de geração por IA | `GET /v1/default-product-images/features` | `GET /api/default-product-images/features/` | JSON com `ai_generation_enabled: boolean` |
| Gerar imagem por IA | `POST /v1/default-product-images/{product_id}/generate-image` | `GET /api/default-product-images/{productExternalKey}/generate-image/` | Antigo cliente espera imagem binária (`Blob`) |

O `POST` sugerido para geração por IA é uma mudança em relação ao `GET` antigo, pois inicia processamento. Ainda é necessário decidir se a resposta será imagem binária, URL persistida ou tarefa assíncrona, e adaptar o frontend ao contrato escolhido. Também é necessário definir propriedade e permissões das imagens padrão.

Referências: [serviço antigo de imagens](../frontend/bad/services/defaultProductImages.js), [tela desconectada](../frontend/src/components/dashboard/views/ProductImagesPlaceholderView.vue) e [modal de imagem](../frontend/src/components/dashboard/views/product-images/ProductImageFormModal.vue).

## 2. Exclusão de oferta

| Operação ausente | Rota candidata Laravel | Situação atual |
|---|---|---|
| Excluir uma oferta do produtor | `DELETE /v1/offers/{offer}` | Não existe. O frontend oferece **zerar estoque disponível**, usando `GET` seguido de `PATCH` com `total_quantity = reserved_quantity`. A oferta e seu histórico continuam existentes. |

Antes de implementar, definir se a exclusão será lógica ou física, o comportamento para ofertas com reservas/compras e a preservação do histórico. Zerar saldo disponível não é equivalente a excluir.

Referências: [rotas de ofertas](../backend/routes/api/offers.php), [serviço de produtos](../frontend/src/services/product.js) e [card de estoque](../frontend/src/components/dashboard/views/stock/StockProductCard.vue).

## 3. Perfil e sessão

Estas chamadas existiam no frontend antigo e não têm equivalente no Laravel atual. O frontend ativo não as chama.

| Operação ausente | Rota candidata Laravel | Rota antiga | Situação/contrato a definir |
|---|---|---|---|
| Consultar perfil autenticado atualizado | `GET /v1/auth/me` | `GET /api/auth/me/` | Hoje o perfil usa a resposta armazenada do login/cadastro. Definir resposta atualizada com usuário, empresa, perfil e endereço. |
| Renovar token | `POST /v1/auth/refresh` | `POST /api/auth/refresh/` | Antigo cliente enviava `{ refresh }` e esperava `{ access }`. Laravel retorna `tokens.access_token` e `tokens.refresh_token` na autenticação, mas não possui endpoint de renovação. Contrato antigo não deve ser presumido válido para Sanctum. |
| Consultar disponibilidade de e-mail | `POST /v1/auth/check-email` | `POST /api/auth/check-email/` | Antigo corpo: `{ email }`; consumidor verificava `exists`. Hoje unicidade é validada no cadastro. |
| Consultar disponibilidade de documento | `POST /v1/auth/check-document` | `POST /api/auth/check-document/` | Antigo corpo: `{ documento }`; consumidor verificava `exists`. Hoje unicidade é validada no cadastro. |

Não foi encontrada uma chamada antiga de edição de perfil que justifique listar `PATCH /auth/me` como requisito existente. Os pré-checks de e-mail/documento são opcionais: a validação de unicidade no cadastro já existe e continua necessária.

Referências: [auth antigo](../frontend/bad/stores/auth.js), [perfil antigo](../frontend/bad/services/profile.js), [auth ativo](../frontend/src/composables/useAuth.js) e [rotas de autenticação](../backend/routes/api/auth.php).

## 4. Pedidos e pagamentos do fluxo antigo

O Laravel atual registra compras por oferta com `POST /v1/offers/{offer}/purchase` e lista históricos em `/v1/purchases` e `/v1/sales`. Não implementa o antigo agregado de pedido com carrinho persistido, confirmação, cancelamento e pagamento. O carrinho ativo é um rascunho local; finalizar registra uma compra por oferta.

As rotas abaixo são lacunas **do fluxo legado**, e não chamadas atualmente enviadas pelo dashboard. Os caminhos Laravel são candidatos apenas se esse fluxo voltar a ser necessário.

| Operação antiga sem equivalente completo | Rota candidata Laravel | Rota antiga | Entrada/uso do cliente antigo |
|---|---|---|---|
| Listar pedidos, inclusive pendentes | `GET /v1/orders` | `GET /api/orders/` | Filtros como `status=PENDING`, `producer`, `page`; históricos de compras/vendas já têm substitutos, mas pedidos pendentes persistidos não |
| Criar pedido/rascunho no servidor | `POST /v1/orders` | `POST /api/orders/` | `{ producer }` |
| Consultar pedido com itens | `GET /v1/orders/{order}` | `GET /api/orders/{order}/` | Pedido agregado com seus itens |
| Confirmar pedido | `POST /v1/orders/{order}/confirm` | `POST /api/orders/{order}/confirm/` | Sem corpo no cliente antigo |
| Cancelar pedido | `POST /v1/orders/{order}/cancel` | `POST /api/orders/{order}/cancel/` | Sem corpo no cliente antigo; também não existe cancelamento de compra no contrato atual |
| Adicionar item ao pedido | `POST /v1/orders/{order}/items` | `POST /api/orders/{order}/items/` | `{ product, quantity }` |
| Alterar quantidade de item | `PATCH /v1/orders/{order}/items/{item}` | `PATCH /api/orders/{order}/items/{item}/` | `{ quantity }` |
| Remover item do pedido | `DELETE /v1/orders/{order}/items/{item}` | `DELETE /api/orders/{order}/items/{item}/` | Identificadores do pedido e do item |
| Excluir pedido | `DELETE /v1/orders/{order}` | `DELETE /api/orders/{order}/` | Identificador do pedido |
| Processar pagamento | `POST /v1/orders/{order}/pay` | `POST /api/orders/{order}/pay/` | Antigo cliente enviava `{ payment_method, card? }`; contrato e integração de pagamento precisam ser definidos |

Registrar uma compra não comprova pagamento nem equivale a confirmar o pedido antigo. Cancelamento de atribuição de viagem (`/delivery/trips/{trip}/cancel`) também não cancela compra/pedido.

Referências: [serviço antigo de pedidos](../frontend/bad/services/ordersService.js), [carrinho atual](../frontend/src/stores/cart.js) e [rotas de compras/vendas](../backend/routes/api/purchases.php).

## 5. Descrição de oferta e geração de texto por IA

O cadastro antigo da interface apresentava descrição e geração por IA, mas a implementação ativa não tinha um endpoint real para geração. Os controles foram retirados porque o contrato Laravel não aceita/persiste descrição.

| Lacuna | Situação |
|---|---|
| Salvar descrição da oferta | `POST /v1/offers` e `PATCH /v1/offers/{offer}` já existem, mas seus contratos não incluem `description`. Trata-se de extensão de contrato e persistência, não de rota CRUD ausente. |
| Gerar descrição por IA | Não existe endpoint Laravel nem caminho antigo comprovado nos serviços auditados. Método, URL, entrada e resposta ainda precisam ser definidos se a funcionalidade for retomada. |

Referências: [validação de criação](../backend/domain/Models/Offer/Requests/StoreOfferRequest.php), [validação de atualização](../backend/domain/Models/Offer/Requests/UpdateOfferRequest.php) e [cadastro atual](../frontend/src/components/dashboard/views/stock/StockCreateModal.vue).

## 6. Itens que não são pendências de backend

| Funcionalidade | Implementação existente / motivo para não listar como rota pendente |
|---|---|
| Login e cadastro | `POST /v1/auth/login` e `POST /v1/auth/signup` já existem |
| Metadata do cadastro | `GET /v1/auth/signup/metadata` já existe; frontend usa opções locais |
| Consulta de CEP | `GET /v1/consult/cep/{cep}` já existe e é usado |
| Categorias e CRUD atual de ofertas | Rotas existentes, exceto DELETE da oferta |
| Wishlist | `GET/POST /v1/wishlist/items` e `DELETE /v1/wishlist/items/{id}` já existem |
| Analytics da wishlist | `GET /v1/wishlist/analytics` substitui a operação antiga `/api/wishlists/top-products/`; falta interface integrada, não endpoint |
| Entregas | As seis rotas de listar viagens, consultar viagem atual/histórico, assumir/cancelar atribuição e confirmar parada já existem; faltam telas integradas |
| Catálogo e preços | Usam `product-source:8001/api/products/search` e `/api/products/{id}/prices`, conforme integração atual; não são rotas Laravel faltantes |
| Lista de estados | Usa BrasilAPI `/api/ibge/uf/v1`; não existe equivalente Laravel, mas é uma integração externa funcional |

## Limites da conferência

Documento baseado nas rotas e nos consumidores do repositório. Formatos antigos são evidência histórica, não contratos Laravel aprovados. Não foram criadas rotas nem alterado código nesta tarefa. Decisões sobre exclusão, pagamento, autenticação e geração por IA permanecem abertas.
