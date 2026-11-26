# Bot de Cadastro Gocase

Cadastra produtos Gocase no Spree e acompanha cada um até ele estar pronto no Factory,
com estampa, e, se você pedir, publicado no site. Você monta o lote numa página (o
**lobby**) e uma **extensão do Chrome** faz os cliques no admin por você.

- **Lobby:** https://bot-cadastro.devgogroup.com
- **Extensão:** baixada no próprio lobby, na aba **Extensão**
- **Versão atual:** extensão 0.11.0 (26/11/2026)
- **Responsável:** João Wellington

---

## Por que é uma extensão, e não um robô no servidor

Criar produto em produção não tem volta e aparece no site. Por isso o bot:

- age **com a sua sessão** no Spree, no Factory e no Catalog. Ele não guarda senha
  e não tem credencial própria;
- **nunca digita o código do autenticador** do Spree. Se a tela do código aparecer,
  ele para e avisa;
- antes de enviar, confere se todos os campos existem na tela do admin. Se a tela
  mudou, ele para em vez de chutar;
- pede confirmação a cada "Cadastrar", e oferece um **Testar sem criar**, que mostra
  exatamente o que seria enviado sem enviar nada.

Tudo que o bot fizer fica registrado no Spree com o seu usuário, como se você tivesse
feito à mão.

---

## Instalar a extensão (uma vez)

1. No lobby, aba **Extensão**, clique em **Baixar a extensão (.zip)** e descompacte
   numa pasta que você não vá apagar.
2. No Chrome, abra `chrome://extensions` e ligue o **Modo do desenvolvedor** (canto
   superior direito).
3. Clique em **Carregar sem compactação** e escolha a pasta descompactada.

Quando sair versão nova, o lobby avisa. Baixe o zip de novo, troque a pasta e aperte o
↻ da "Bot de Cadastro Gocase" em `chrome://extensions`.

Funciona também no Opera.

## Antes de cadastrar

- Esteja **logado** no admin do Spree (com o código do autenticador já digitado), no
  Factory e no Catalog v3.
- **Deixe a aba do lobby aberta** enquanto um card roda. É o lobby que decide o próximo
  passo. Se a aba fechar, o card para onde estava.

---

## Como cadastrar

O lobby tem uma aba por linha: **Autoral**, **Licenciamento** e **Futebol**. As três
usam o mesmo montador, em três passos:

1. **Quem:** o ilustrador, a licença ou o time.
2. **Estampas:** uma ou várias. Dá para digitar, colar um CSV (`name, identifier`, e
   opcionalmente `sku`, `illustrator_name`, `commissionable_sku`) ou usar
   **Puxar do Catalog**, que lista as artes novas do Catalog v3 (artes `-b2b` nunca
   aparecem).
3. **Produtos:** marque os produtos com foto. O bot já sabe o protótipo, a categoria,
   o prefixo do nome, o modelo de SKU e as taxons de cada um.

Antes de enviar, você vê a **prévia** de cada card (nome, SKU, identifier, taxons). O
SKU pode ser editado ali. Depois é só **Cadastrar** e confirmar.

Opções do card:

- **Fora do site: cadastrar sem publicar.** O produto é criado com "Disponível em"
  31/12/2099. Serve para teste e para coleção que só vai ao ar depois. Quando chegar a
  hora, use **Publicar no site** no card.
- **Copiar.** Abre um card novo com os mesmos dados.
- **Cancelar.** Para no próximo passo; o que já foi feito no Spree e no Factory fica.
- **Testar sem criar.** Faz todo o caminho e mostra o CSV e os campos que seriam
  enviados, sem criar nada. Use sempre que for um produto novo para o bot.
- **Formulário manual.** Para produto que não está no catálogo do montador.

### Aba Ilustradores

Mostra as estampas mais recentes de cada ilustrador no Catalog e diz o que já virou
produto (no Factory, num card do lobby, à venda no site ou fora do site). Os botões
**Autoral / Licenciamento / Futebol** abrem o montador já preenchido com a estampa, o
time e o ilustrador.

---

## O que o bot faz depois do "Cadastrar"

1. Confere se você está logado no Spree e no Factory.
2. Cria o produto pela **criação em massa** do Spree (um CSV de uma linha).
3. Espera o produto chegar no Factory. Se demorar, aperta **Sincronizar Produto** no
   Spree.
4. No Factory, segue o processo do time: **Adaptar Estampas** e depois **Sincronizar
   Customizações**.
5. Quando a estampa padrão aparece, volta ao Spree: **Sincronizar Produto** e
   **Atualizar**.
6. Se o card não for "fora do site": coloca **"Disponível em" = agora − 5 min** (fuso
   de Fortaleza) e salva. É isso que leva o produto ao site.
7. Avisa por notificação do Chrome, e o card ganha a foto do mockup e o link do site.

### Quando o card fica "pronto"

| ponto | verde quando |
|---|---|
| Spree | o SKU existe no Spree |
| Factory | o SKU existe no Factory |
| Identifier | o identifier no Factory é o do card |
| Material | o Factory associou pelo menos um material |
| Estampa padrão | o Factory gerou a estampa padrão |
| Estampa no Spree | a estampa chegou ao Spree |
| Cores | os térmicos ficaram só com as cores escolhidas |
| No site | o produto está à venda (ou fora do site, se foi pedido assim) |

Tempos normais: o Spree cria o produto em segundos, mas Spree → Factory já levou de 1 a
16 minutos. O Adaptar trabalha uns 5 minutos em segundo plano. O mockup aparece uns 2
minutos depois da estampa. Um card completo costuma levar de 10 a 30 minutos.

---

## Regras fixas

Estas regras valem para todo produto criado pelo bot:

- **Só loja BR**, com **"Bloquear produto na substore US"** marcado.
- **Todo produto tem ilustrador.** Sem ilustrador, o Spree recusa o lote inteiro.
- **Licenciamento precisa de `commissionable_sku`.** O time usa o próprio SKU.
- **Cores dos térmicos:** o produto nasce com todas as cores do protótipo e o bot deixa
  só as que você escolheu. Menos cores também significa menos materiais no Factory.
- **Frente e verso** só em Fresh, Urban, Cerveja, Flow, Moove, Garrafa Fun e Taça.
  **Nunca** em Flip Pro, Magsafe ou Copo Life. Vale daqui para frente; os produtos
  antigos não foram regerados.
- **Taxons de futebol:** cada produto entra na coleção do time e na taxon do tipo
  ("Fresh Flamengo" etc.). O bot **não cria taxon** no Spree; se precisar de outra, use
  o campo de taxons extras.
- **Nada muda no Catalog.** O bot só lê o Catalog.

## Corrigir taxons dos times (futebol antigo)

Cerca de 1.470 produtos de futebol ativos estão sem a coleção do time (Flamengo 234,
Corinthians 186, CBF 150…). No montador de Futebol, **Corrigir taxons dos times** monta
os grupos e a extensão envia uma **Atualização em massa** do Spree por taxon. Ainda não
foi rodado em produção.

---

## Problemas comuns

| o que aparece | o que fazer |
|---|---|
| "Entre no Catalog" | O login do Catalog vale umas 12 h. Abra o Catalog v3, entre de novo e recarregue o lobby. |
| O bot parou na tela do código | Digite o código do autenticador no Spree (ou no Factory) e aperte **Tentar de novo** no card. |
| "Atualize a extensão" | Baixe o zip na aba Extensão, troque a pasta e aperte o ↻ em `chrome://extensions`. |
| Card falhou em "Envio" depois de 2 min | O Spree recusou o lote sem avisar. Quase sempre falta ilustrador ou `commissionable_sku`. |
| "Sem arte no Catalog" | Aquela família (térmicos, têxteis, pet) não tem arte no Catalog para a estampa. O produto fica de fora em vez de esperar 20 min por uma estampa que não vem. |
| Card com erro | A mensagem do card diz o motivo (o detalhe fica em **Log**). Aperte **Tentar de novo**: o bot continua de onde parou, sem refazer o que já foi feito. |
| Card "executando" parado | A aba do lobby foi fechada no meio. Depois de 15 min aparece **Retomar**. |

## O que ainda não faz

- Não preenche as **traduções** do nome (`english_name` etc.).
- Não lê o **Checklist** do Factory (estampa faltando por molde).
- **Copo Flow** e **Moove** não têm taxon de tipo em nenhum time; entram só na coleção.
- No **Licenciamento**, o SKU real é o código sequencial da licença (ex.: `dsn-gtf-96`).
  O montador sugere um modelo e você troca na prévia ou manda pelo CSV.
- Alguns produtos do catálogo de Futebol (Food Jar, Marmita, Mochila/Tote Care,
  Mochila/Estojo Dots) nunca tiveram versão de futebol. Rode **Testar sem criar** antes.

---

## Para quem mexe no código

- **Código:** GitHub `jwfilho1512/bot-cadastro` (o README explica o fluxo técnico,
  os arquivos e o deploy).
- **Lobby:** app GoDeploy `2d270010`. O deploy tem que subir **todos** os arquivos
  (`subir.sh`).
- **Mapa das telas do admin** (campos da criação em massa, botões, consultas de
  auditoria): [MAPA-ADMIN-SPREE.md](MAPA-ADMIN-SPREE.md).
- Ao mudar a extensão, suba a versão em `extensao/manifest.json` e em
  `VERSAO_EXTENSAO` (`app.js`), e avise quem já instalou.
