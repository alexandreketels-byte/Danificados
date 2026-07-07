# Portal de Danificados — Guia de Configuração

## ⚠️ Se você já tinha o portal rodando (atualização importante)

Esta versão mudou duas coisas estruturais:

1. **Identificador único agora é a coluna OS, não mais PEDIDO.** Percebemos que um mesmo PEDIDO pode ter várias máquinas/itens (linhas) diferentes — só a OS é sempre única. Isso corrigia o bug das linhas em branco/duplicadas.
2. **Nova coluna DEFEITO** (logo depois de FOTOS).

Siga os passos abaixo, na ordem:

1. Cole o novo conteúdo do `Code.gs` no editor do Apps Script (substitua tudo).
2. No menu de funções, rode **`migrateAddDefeitoColumn`** uma vez — ela insere a coluna DEFEITO na planilha sem apagar nada.
3. **Importante:** vá em **Implantar → Gerenciar implantações → editar (ícone de lápis) → Versão: "Nova versão" → Implantar.** Só salvar o script não atualiza a URL do Web App que já está publicada — sem esse passo, o site continua usando o código antigo.
4. Dê uma limpada manual na aba "Danificados": pedidos como `460285N`, `460884N`, `458463N`, `474495N`, `473471N`, `422067N-A` e `404364N` têm mais de uma linha (isso é esperado agora, cada linha é uma máquina/item diferente) — mas se sobrou alguma linha **em branco** de testes de importação anteriores (da época em que o identificador ainda era PEDIDO), apague-a manualmente uma vez. Depois disso os próximos imports não vão mais duplicar nada.
5. Substitua o `index.html` no GitHub Pages pelo novo arquivo (mantendo sua `APPS_SCRIPT_URL` já configurada).

## 1. Criar a planilha (ou usar a mesma do Portal de Gestão)

1. Abra a planilha do Portal de Gestão (ou crie uma nova).
2. Você **não** precisa criar as abas manualmente — o script faz isso pra você.

## 2. Configurar o Apps Script

1. Na planilha, vá em **Extensões → Apps Script**.
2. Apague o conteúdo padrão e cole o conteúdo do arquivo `Code.gs`.
3. **Antes de publicar**, edite a linha dentro da função `setupAdmPin()`:
   ```js
   var PIN_DO_ADM = '9999'; // troque para o PIN real do ADM
   ```
4. No topo do editor, selecione a função `setupAdmPin` no menu de funções e clique em **Executar** (▶). Autorize as permissões pedidas.
5. Selecione a função `setupSheets` e clique em **Executar**. Isso cria as abas **Danificados** (com a coluna DEFEITO já incluída) e **Gestores** (já com os 7 gestores cadastrados).

## 3. Publicar como Web App

1. **Implantar → Nova implantação.**
2. Tipo: **App da Web**.
3. Executar como: **Eu**.
4. Quem pode acessar: **Qualquer pessoa**.
5. Clique em **Implantar** e copie a **URL do Web App** (termina em `/exec`).

**Sempre que editar o `Code.gs` depois disso**, volte em **Gerenciar implantações → editar → Nova versão → Implantar** — não basta salvar o script.

## 4. Configurar o frontend (index.html)

1. Abra o arquivo `index.html`.
2. Encontre a linha:
   ```js
   const APPS_SCRIPT_URL = 'COLE_AQUI_A_URL_DO_SEU_WEB_APP';
   ```
3. Substitua pela URL copiada no passo 3.5.
4. Suba o arquivo pro GitHub Pages.

⚠️ **localStorage**: esse portal usa um prefixo próprio (`portalDanificados_`) pra preferência de colunas, evitando colisão com os outros portais no mesmo domínio.

## 5. Importar sua planilha de danificados (.xlsm ou CSV)

Se seu arquivo é `.xlsm`/`.xlsx`, exporte a aba de dados como **CSV** primeiro (Arquivo → Fazer download → CSV, no Sheets; ou Salvar Como → CSV no Excel).

O importador reconhece automaticamente cabeçalhos com ou sem acento/espaço (ex: "DESCRIÇÃO", "PREÇO CUSTO", "NF fabricante", "% desconto") e também detecta sozinho se o arquivo está em UTF-8 ou Windows-1252 (padrão do Excel BR) — não precisa se preocupar em converter nada.

Cabeçalhos esperados (podem vir com acento, sem acento, com espaço — o sistema normaliza):
```
PEDIDO, OS, OS2, FABRICANTE, STATUS, DESCRIÇÃO, COD, BTUS, NF fabricante,
PREÇO CUSTO, preço de venda2, % desconto, liquidação, SETOR, FOTOS, DEFEITO
```

- Itens com **OS** já existente são **atualizados** (só os dados originais — as 6 colunas de interação não são mexidas).
- Itens com OS nova são **inseridos** com as 6 colunas de interação em branco.
- A importação pede o PIN do ADM.

## 6. Como o fluxo funciona no dia a dia

1. **Gestor** clica em "Reservar" na linha do item → marca "Reservar" (e opcionalmente "Converter OS"), digita o PIN dele, salva. A linha passa a mostrar o nome dele em "Reservado Por".
2. Se ele errou, ele mesmo clica em "Desfazer" e confirma com o próprio PIN — libera a reserva (mas mantém cód. convertido / nº pedido / status, caso o ADM já tenha avançado nessas etapas).
3. **ADM** clica em "ADM: Cód." e, com o PIN do ADM, grava o código danificado convertido — texto livre, sem formato fixo.
4. Assim que esse código existe, aparece o botão "Nº Pedido" para o gestor que reservou lançar o número do pedido, novamente com o PIN dele.
5. **ADM** pode marcar o status como Aberto/Finalizado a qualquer momento pelo botão "ADM: Status".
6. No painel **Área ADM**, o ADM adiciona/remove gestores sem mexer no código.

## 7. Busca, filtros e ordenação

- **Busca geral** (campo no topo da página): procura o termo em todas as colunas ao mesmo tempo.
- **Filtro por coluna**: cada cabeçalho tem um campo de texto — digite parte do valor pra filtrar (não precisa ser exato), e o campo sugere os valores existentes conforme você digita.
- **Ordenar**: clique no ícone ↕ ao lado do nome da coluna pra alternar entre crescente/decrescente (funciona com números, preços e texto).
- **Limpar Filtros**: botão no topo limpa busca geral, filtros de coluna e ordenação de uma vez.

## 8. Testando

Abra o `index.html` (local ou publicado) e confira se a tabela carrega. Erro de CORS geralmente significa que a implantação não está com acesso "Qualquer pessoa" ou que você copiou a URL errada (precisa terminar em `/exec`).

## Observações / pontos que assumi

- O ADM é identificado por um único PIN fixo, guardado nas Propriedades do Script.
- Ao desfazer uma reserva, o código convertido / nº pedido / status **não** são apagados — só a reserva em si.
- A importação de CSV exige PIN do ADM.
- **OS é o identificador único de cada linha** — nunca vai haver duas linhas com a mesma OS, mas o mesmo PEDIDO pode aparecer várias vezes (uma por máquina/item).

