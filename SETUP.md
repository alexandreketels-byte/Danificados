# Portal de Danificados — Guia de Configuração

## ⚠️ Atualização: máquina de estados de 7 fases (leia primeiro se já tinha o portal rodando)

Esta versão reformula o processo pra refletir o fluxo real da sua operação:

**Disponível → Reservado → Convertido → Pedido Lançado → Separação Solicitada → OS Encerrada → Finalizado**

Mudanças estruturais:
- As colunas antigas `RESERVADO` (Sim/Não), `CONVERTER_OS` (Sim/Não) e `STATUS_RESERVA` (Aberto/Finalizado) foram substituídas por **uma única coluna `STATUS`**, que sempre mostra a fase exata do item.
- A coluna original `STATUS` (que vinha da sua planilha, tipo "ABERTO") foi renomeada pra `STATUS_ITEM`, pra não confundir com a fase do processo.
- Nova aba **Historico**: toda transição de fase vira uma linha (data/hora, OS, ação, de → para, quem fez). Dá pra ver isso pelo botão "Histórico" em cada linha da tabela.
- Nova coluna `ATUALIZADO_EM` mostra a última movimentação do item.
- Dois passos novos que não existiam: **"ADM: Solicitar Separação"** e **"ADM: Confirmar OS Encerrada"**, entre o pedido lançado e a finalização.

### Passos pra atualizar (nessa ordem):

1. Cole o novo `Code.gs` no editor do Apps Script (substitua tudo).
2. Rode a função **`migrateToStatusMachine`** uma vez (menu de funções → selecionar → Executar). Ela lê os dados no formato antigo, calcula o `STATUS` correto pra cada linha e reescreve a aba "Danificados" já no novo modelo — sem perder nada. Também cria a aba "Historico" se não existir.
3. **Vá em Implantar → Gerenciar implantações → editar (lápis) → Nova versão → Implantar.** Sem isso o site continua rodando o código antigo.
4. Troque o `index.html` no GitHub Pages pelo novo arquivo (mantendo a `APPS_SCRIPT_URL` já configurada).

## 1. Criar a planilha (do zero)

1. Crie uma planilha nova (ou use uma existente).
2. Você não precisa criar as abas manualmente — o script faz isso.

## 2. Configurar o Apps Script

1. Na planilha, vá em **Extensões → Apps Script**.
2. Cole o conteúdo de `Code.gs`.
3. Edite a linha dentro de `setupAdmPin()`:
   ```js
   var PIN_DO_ADM = '9999'; // troque para o PIN real do ADM
   ```
4. Rode `setupAdmPin` uma vez (autorize as permissões pedidas).
5. Rode `setupSheets` uma vez — cria as abas **Danificados**, **Gestores** (já com os 7 gestores) e **Historico**.

## 3. Publicar como Web App

1. **Implantar → Nova implantação → App da Web.**
2. Executar como: **Eu**. Quem pode acessar: **Qualquer pessoa**.
3. Copie a URL terminada em `/exec`.
4. **Sempre que editar o `Code.gs`**, volte em Gerenciar implantações → editar → Nova versão → Implantar.

## 4. Configurar o frontend

1. No `index.html`, troque:
   ```js
   const APPS_SCRIPT_URL = 'COLE_AQUI_A_URL_DO_SEU_WEB_APP';
   ```
2. Suba pro GitHub Pages.

## 5. Importar sua planilha (.xlsm/.xlsx → CSV)

Exporte a aba de dados como CSV (com ou sem acento nos cabeçalhos, tanto faz — o sistema normaliza e detecta a codificação automaticamente). Cabeçalhos esperados:
```
PEDIDO, OS, OS2, FABRICANTE, STATUS, DESCRIÇÃO, COD, BTUS, NF fabricante,
PREÇO CUSTO, preço de venda2, % desconto, liquidação, SETOR, FOTOS, DEFEITO
```
(o "STATUS" do seu CSV vira `STATUS_ITEM` automaticamente, sem conflito com a fase do processo)

- Itens com **OS** já existente são atualizados (só os dados originais).
- Itens novos entram como **"Disponível"**.

## 6. O fluxo completo, fase por fase

| Fase | Quem aciona | O que acontece |
|---|---|---|
| Disponível | — | Item cadastrado, aguardando venda |
| Reservado | Gestor (PIN) | Gestor vendeu, informa a OS e reserva |
| Convertido | ADM (PIN) | Você converte a OS em código danificado |
| Pedido Lançado | Gestor (PIN, mesmo que reservou) | Gestor lança o pedido no sistema comercial e informa o nº |
| Separação Solicitada | ADM (PIN) | Você solicita o encerramento da OS e a separação do item |
| OS Encerrada | ADM (PIN) | Você confirma que a OS foi encerrada e o item separado |
| Finalizado | ADM (PIN) | Pedido atendido, processo fechado |

Cada fase só libera o botão da próxima — não dá pra pular etapa. "Desfazer" só funciona enquanto o item está em "Reservado" (se já foi convertido pra frente, não desfaz mais — nesse ponto seria um caso pra tratar manualmente).

O botão **"Histórico"** em cada linha mostra todas as transições daquele item, com data/hora e quem fez.

## 7. Busca, filtros e ordenação

- **Chips de status** no topo: clique pra filtrar rapidamente por fase (mostra a contagem de cada uma).
- **Busca geral**: procura o termo em todas as colunas.
- **Filtro por coluna**: campo de texto em cada cabeçalho, com sugestões.
- **Ordenar**: ícone ↕ ao lado do nome da coluna.
- **Limpar Filtros**: reseta tudo de uma vez.

## 9. Notificações por e-mail

A cada avanço de fase (Reservado, Convertido, Pedido Lançado, Separação Solicitada, OS Encerrada, Finalizado), o sistema dispara um e-mail pra você (ADM) e pro gestor responsável por aquele item, se ele tiver e-mail cadastrado.

Passos pra ativar:

1. No Apps Script, edite a função `setupEmailNotificacoes()`:
   ```js
   var EMAIL_DO_ADM = 'seu-email@exemplo.com'; // troque pelo seu e-mail
   var PORTAL_URL = ''; // opcional: link do site, aparece no corpo do e-mail
   ```
2. Rode `setupEmailNotificacoes()` uma vez.
3. Se sua aba "Gestores" já existia antes dessa atualização (sem a coluna EMAIL), rode `migrateAddEmailGestores()` uma vez — ela adiciona a coluna sem apagar nome/PIN.
4. Preencha o e-mail de cada gestor: pelo painel **Área ADM** no site (tem um campo de e-mail do lado do nome de cada gestor agora) ou direto na planilha, coluna C da aba "Gestores".
5. Redeploy de sempre: Implantar → Gerenciar implantações → Nova versão.

Detalhes:
- Se um gestor não tiver e-mail cadastrado, ele simplesmente não recebe nada — não dá erro, só pula.
- Se `ADM_EMAIL` não estiver configurado, nenhum e-mail é enviado (mas o sistema continua funcionando normalmente).
- O Gmail/Apps Script tem um limite diário de envio (100 e-mails/dia em conta pessoal do Gmail, bem mais em contas Google Workspace) — não deve ser problema no seu volume atual, mas se um dia isso virar gargalo (ex: lote com 50+ itens de uma vez), dá pra agrupar os e-mails de uma ação em lote num resumo único em vez de um e-mail por item.
- As ações em lote também disparam e-mail — um por item, então reservar 6 OS de uma vez gera 6 e-mails (pra cada gestor + você).

## 10. Testando

Abra o `index.html` e confira se a tabela carrega. Erro de CORS geralmente é implantação sem acesso "Qualquer pessoa" ou URL sem `/exec`.

## Observações / pontos que assumi

- O ADM é um PIN único, guardado nas Propriedades do Script.
- **OS é o identificador único** de cada linha (PEDIDO pode se repetir quando um pedido tem várias máquinas).
- A importação de CSV exige PIN do ADM.
- Não criei um jeito de "voltar" uma fase depois de "Convertido" (só até "Reservado") — se isso acontecer na prática com frequência, dá pra adicionar uma ação de reversão por fase, mas achei melhor não liberar isso de cara pra não virar bagunça no histórico.
