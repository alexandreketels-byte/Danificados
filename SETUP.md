# Portal de Danificados — Guia de Configuração

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
4. No topo do editor, selecione a função `setupAdmPin` no menu de funções e clique em **Executar** (▶). Autorize as permissões pedidas. Isso grava o PIN do ADM de forma segura (fora da planilha).
5. Selecione a função `setupSheets` e clique em **Executar**. Isso cria:
   - Aba **Danificados** com os cabeçalhos corretos (15 colunas originais + 6 colunas de interação).
   - Aba **Gestores** já populada com os 7 gestores que você passou (Judyson, Kaique, Paulo, Altair, Yan, Carlos, Rogério).
6. Depois de rodar as duas funções uma vez, você pode apagar o PIN de dentro de `setupAdmPin()` do código (ele já está salvo nas Propriedades do Script) — ou deixar, não tem problema de segurança porque só quem tem acesso ao Apps Script vê isso.

## 3. Publicar como Web App

1. No editor do Apps Script, clique em **Implantar → Nova implantação**.
2. Tipo: **App da Web**.
3. Executar como: **Eu**.
4. Quem pode acessar: **Qualquer pessoa**.
5. Clique em **Implantar** e copie a **URL do Web App** (termina em `/exec`).

## 4. Configurar o frontend (index.html)

1. Abra o arquivo `index.html`.
2. Encontre a linha:
   ```js
   const APPS_SCRIPT_URL = 'COLE_AQUI_A_URL_DO_SEU_WEB_APP';
   ```
3. Substitua pela URL copiada no passo 3.5.
4. Suba o arquivo pro GitHub Pages (mesmo repositório dos outros portais, ou um novo — só atenção ao ponto abaixo).

⚠️ **Importante sobre localStorage**: como você já teve colisão de chaves entre o SAC e o Portal de Ocorrências no mesmo domínio do GitHub Pages, esse portal já usa um prefixo próprio (`portalDanificados_`) pra guardar a preferência de colunas visíveis — não deve colidir com os outros.

## 5. Importar a planilha de danificados existente (a da imagem que você mandou)

O importador de CSV espera as colunas com estes cabeçalhos (pode exportar sua planilha atual como CSV e ajustar o cabeçalho se necessário):

```
PEDIDO,OS,OS2,FABRICANTE,STATUS,DESCRICAO,COD,BTUS,NF_FABRICA,PRECO_CUSTO,PRECO_VENDA2,PCT_DESCONTO,LIQUIDACAO,SETOR,FOTOS
```

- Itens com `PEDIDO` já existente na planilha são **atualizados** (só os dados originais — as 6 colunas de interação não são mexidas).
- Itens novos são **inseridos** com as 6 colunas de interação em branco.
- A importação pede o PIN do ADM.

## 6. Como o fluxo funciona no dia a dia

1. **Gestor** clica em "Reservar" na linha do item → marca "Reservar" (e opcionalmente "Converter OS"), digita o PIN dele, salva. A linha passa a mostrar o nome dele em "Reservado Por".
2. Se ele errou, ele mesmo clica em "Desfazer" e confirma com o próprio PIN — libera a reserva (mas mantém cód. convertido / nº pedido / status, caso o ADM já tenha avançado nessas etapas).
3. **ADM** clica em "ADM: Cód." e, com o PIN do ADM, grava o código danificado convertido — texto livre, sem formato fixo.
4. Assim que esse código existe, aparece o botão "Nº Pedido" para o gestor que reservou lançar o número do pedido, novamente com o PIN dele.
5. **ADM** pode marcar o status como Aberto/Finalizado a qualquer momento pelo botão "ADM: Status".
6. No painel **Área ADM** (topo da tela), o ADM consegue adicionar ou remover gestores sem mexer no código — só entrar com o PIN do ADM.

## 7. Testando

Depois de configurar a URL, abra o `index.html` localmente no navegador (ou publique no GitHub Pages) e confira se a tabela carrega. Se der erro de CORS, confirme que a implantação está com acesso "Qualquer pessoa" e que copiou a URL terminada em `/exec` (não a de edição do script).

## Observações / pontos que assumi

- O ADM é identificado por um único PIN fixo (não por nome), guardado nas Propriedades do Script — se no futuro você quiser mais de um ADM, dá pra adaptar fácil.
- Ao desfazer uma reserva, o código convertido / nº pedido / status **não** são apagados — só a reserva em si. Se quiser que tudo reinicie ao desfazer, é uma mudança pequena no `Code.gs` (função `desfazerReserva`).
- A importação de CSV também exige PIN do ADM (não defini isso na conversa, assumi que faz sentido tratar como ação estrutural).
