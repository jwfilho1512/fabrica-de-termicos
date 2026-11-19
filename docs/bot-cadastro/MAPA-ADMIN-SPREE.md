# Mapa do admin: cadastro de produto Gocase

Levantado em 18/11/2026, só lendo as telas, sem salvar nada. É a base da extensão
do bot de cadastro: o que ela preenche, onde clica e o que confere depois.

Este arquivo fica aqui até o bot ganhar repositório próprio.

## 1. Criação em massa no Spree

Tela: `https://checkout.gocase.com.br/admin/products/products_in_batch`
Envio: `POST /admin/products/create_products_in_batch` (`multipart/form-data`,
com `authenticity_token` da página).

### O CSV

Uma linha por estampa. O modelo oficial está na planilha linkada na tela
("Clique aqui"): `docs.google.com/spreadsheets/d/18OT6SFLtD2ScEMkfZlLpVwbelNnavE_XekzIxxqS5mY`.

| coluna | o que é | exemplo do modelo |
|---|---|---|
| `name` | nome da estampa (o prefixo do produto entra na frente) | Garota Mística |
| `identifier` | identifier da arte no Catalog (engine Prisma) | garota-mistica-oki |
| `sku` | SKU final do produto | mochila-soft-preto-v2-garota-mistica-oki |
| `commissionable_sku` | SKU de licenciamento | — |
| `illustrator_name` | ilustrador | — |
| `english_name` … `dutch_name` | nome **completo** traduzido (prefixo + estampa) | — |

Colunas de tradução: `english_name`, `spanish_name`, `german_name`,
`french_name`, `italian_name`, `portuguese_name`, `dutch_name`.

Segundo o Confluence (GN 20480001), o prefixo não é traduzido sozinho. Por isso cada
coluna de tradução leva o nome inteiro, prefixo incluído.

Texto da tela: "SKU - Apenas produtos que irão suportar o modelo".

### Configurações compartilhadas (valem para o lote inteiro)

| campo (`name`) | tipo | observação |
|---|---|---|
| `file` | arquivo | obrigatório |
| `store_ids[]` | multi | `1=br`, `3=global`, `4=lascapinhas` |
| `available_on` | texto (data) | |
| `commissionable_id` | select | 223 licenciamentos (Disney, Warner, times…) |
| `blocklisted_to_catalog_store` | checkbox | bloquear na substore US |
| `blocklisted_to_br_store` | checkbox | bloquear na BR (só US) |
| `generate_png_images` | checkbox | imagem frente e verso |
| `generate_gif_images` | checkbox | GIF 360 |
| `allow_covering_logo` | checkbox | logo sobre a estampa |

### Por subproduto (`products[N]`, botão "Adicionar Subproduto")

Cada subproduto é um tipo de produto. Na leitura da tela, **o mesmo CSV é
aplicado a cada subproduto** do lote: N estampas × M subprodutos. Ainda falta
confirmar isso com um lote de teste.

| campo | tipo | observação |
|---|---|---|
| `products[N][prototype_id]` | select | 440 protótipos (ex.: `[ENGINE] PRISMA - CASE - MARVEL`) |
| `products[N][meta_keywords]` | texto | iguais para todos os produtos criados |
| `products[N][product_type_code]` | select, **obrigatório** | 361 categorias de material (ex.: `case`, `backpack`) |
| `products[N][material_id]` | select | 10.786 materiais do Factory; pode ficar "Nenhum" |
| `products[N][prefix]` | texto | ex.: "Garrafa Square" |
| `products[N][sku_suffix]` | texto | |
| `products[N][price]` | texto | |
| `products[N][old_price]` | texto | só BRL (preço cortado) |
| `products[N][taxon_ids]` | hidden (select2 por ajax) | ids separados por vírgula, ex.: `1,30` |
| `products[N][available_to_offline]` | checkbox | offline sem customização |
| `products[N][send_ebook]` | checkbox | ebook após pagamento |

Para a extensão: os selects grandes (protótipo, material, licenciamento) devem
ser preenchidos **pelo value (id)**, não pelo texto. O lobby resolve o id antes,
pelo banco.

## 2. Produto individual no Spree

Edição: `/admin/products/<slug>/edit`. Abas: Detalhes, Imagens, Variantes,
Estoque, Customization, Turbo boost, Recomendação, Preços, Translations.

- **Sincronizar Produto (Spree → Factory)**: link `GET /admin/products/<slug>/factory_sync`
  (`#admin_sync_product`). É um GET simples que dispara o sync em background.
- **Customization**: `/admin/products/<slug>/product_customizations/<id>/edit`
  - `code` = `stamp`, `customization_type` = `engine`
  - `spree_engine_manager_id` = `1` → **Prisma (Produção) - 0.286.0**
    (`preferred_engine` = `https://unpkg.com/@gocasebr/prisma@0.286.0`)
  - `preferred_engine_identifier` = o identifier do Catalog
  - `preferred_required` = true
- Campos de detalhes relevantes: `taxon_ids`, `product_type_code`,
  `case_type_ids[]`, `unsupported_case_device_ids[]`, `store_ids[]`,
  `preferred_br_enable_govinci_listing`, `preferred_global_enable_govinci_listing`,
  `ai_keywords` (já existe campo de palavras-chave geradas por IA).

Tela em massa vizinha: "Sincronizar Materiais" em `/admin/products/sync_materials`.

## 3. Factory

O login é separado ("Entrar com Google", só @gocase.com). A extensão depende de a
pessoa estar logada nos dois sistemas.

### Página do produto: `https://factory.gocase.com.br/products/<id>`

Ações (todas são links GET simples, sem `data-method` e sem confirmação):

| botão | URL | quando a extensão usa |
|---|---|---|
| **Sincronizar Customizações** (Factory → Spree) | `/products/<id>/sync_customizations` | estampa padrão OK no Factory, mas sem imagem no Spree |
| Adaptar Estampas | `/products/<id>/adapt_to_engine` | ainda não sei o efeito; **não usar** sem falar com a Engenharia |
| Editar Produto | `/products/<id>/edit` | — |
| Upload de Estampas / de Personalização | `/products/<id>/upload_stamps`, `/upload_custom_stamps` | fora do escopo (fluxo sem engine) |
| Duplicar Customizações, Drag Customs | `/duplicate_customs`, `/drag_customs` | fora do escopo |

Campos mostrados em "Detalhes do Produto" (iguais às colunas do banco): Ativo,
SKU, Engine, Engine Identifier, **Tem estampa padrão**, Customizações pendentes,
**Customização do Mockup** (PT-BR, EN, fundo escuro PT-BR/EN), Customização default
para engines, Lojas habilitadas.

A seção **Checklist** lista as estampas por tamanho (largura × altura → modelos
de aparelho), com a coluna IMAGEM SIM/NÃO. Um "NÃO" ali é estampa faltando para
aquele molde, e entra como sexto ponto da auditoria. Pelo banco, é o mesmo que
cruzar `stamps` com os tamanhos dos materiais.

### Outras telas
- `/sync_processes`: log de cada sincronização (status, processo, origem,
  contagem de erros). Tem 31 milhões de linhas, a grande maioria de `orders`.
  Serve para a auditoria procurar o sync de um produto. Exporta CSV e JSON.
- `/create_product`: apesar do nome, **não cria produto**. Busca produto por
  SKU, customização (1.830) ou template de categoria (345), para o fluxo de
  estampa sem engine. Não entra no bot.

### Banco
- O banco já responde à auditoria pelo proxy de dados (fonte `factory`). Consulta
  validada com um produto real (`wanderlust-travel-girl-prisma` → Factory id 19915):

```sql
SELECT p.id, p.sku, p.engine, p.engine_identifier, p.engine_url, p.active,
       p.has_default_stamp_for_material, p.pending_stamps_count,
       p.pt_br_mockup_customization -> 'stamp' AS mockup_stamp,
       (SELECT count(*) FROM available_product_materials apm WHERE apm.product_id = p.id) AS n_materials,
       (SELECT count(*) FROM stamps s WHERE s.product_id = p.id) AS n_stamps
FROM products p
WHERE p.sku = '<sku>';
```

Como cada coluna vira um semáforo da auditoria:

| ponto | sinal verde |
|---|---|
| 1. existe no Factory | linha retornada, `deleted_at` nulo |
| 2. identifier certo | `engine_identifier` igual ao do CSV |
| 3. tem material | `n_materials > 0` |
| 4. estampa padrão OK | `mockup_stamp` contém `prisma-render/prod-v2/previews` |
| 5. nada pendente | `pending_stamps_count = 0` e `pending_customizations_count = 0` |

## 4. Ainda falta mapear

- O que o "Adaptar Estampas" do Factory faz de fato.
- Catalog v3: como conferir se um identifier existe, já que o banco do Catalog
  não está no proxy.
- Confirmar com um lote de teste (1 estampa × 1 subproduto) se o CSV se aplica
  mesmo a cada subproduto, e para onde a tela redireciona depois do envio.
