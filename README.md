# PAYMENT_FEE_AUTOMATIC_INTEGRATED_MRLS - Automação de Pagamentos — Freelas de Picking/Packing

Notebook que automatiza o cálculo e o pagamento de freelas terceirizados que realizam tarefas de picking/packing em lojas parceiras. Construído em **PySpark**, rodando sobre dados do **BigQuery** e integrando com **Google Sheets**, **Google Drive** e **AppSheet**.

> ⚠️ **Todos os dados deste repositório são fictícios.** IDs de marca, nomes de loja, tokens, números de telefone e identificadores de planilha foram gerados aleatoriamente para fins de portfólio — nenhum dado real de nenhuma empresa aparece neste código.

## O que o notebook faz

1. **Busca os pedidos aprovados** no BigQuery, cruzando três origens (lojas grandes, pequenas e prime).
2. **Cruza com pagamentos já realizados**, pra não pagar a mesma tarefa duas vezes.
3. **Aplica regras de negócio**: bloqueio de lojas específicas em determinado canal, limite de itens por marca, bônus por quantidade.
4. **Calcula o valor do pagamento** direto de uma planilha de pricing (Google Sheets), com valor variável por **dia da semana, sexta, sábado, domingo e feriado** — feriados nacionais calculados automaticamente.
5. **Corrige o fuso horário** dos dados de origem (America/New_York) para o fuso de negócio (America/Sao_Paulo), respeitando o horário de verão.
6. **Sobe o resultado** numa planilha de auditoria e registra um log de execução.
7. **Prepara os dados** numa planilha ligada ao AppSheet, que é quem de fato dispara o pagamento para o freela (fora do escopo deste notebook).

No final há blocos de exemplo (não obrigatórios) mostrando como notificar um resumo do processamento por **Slack**, **e-mail (Gmail API)** e **WhatsApp Business API**.

## Stack

- **PySpark** — todo o processamento e cálculo de dados
- **Google BigQuery** — fonte dos pedidos e pagamentos
- **Google Sheets** (`gspread`, `gspread_dataframe`) — planilha de pricing, planilha de auditoria, log
- **Google Drive API** — cópia de planilhas, identificação de responsável
- **AppSheet** — camada final de aprovação e disparo do pagamento (fora do notebook)
- **holidays** — cálculo de feriados nacionais brasileiros

## Estrutura do pipeline

```
BigQuery (pedidos)  ─┐
                      ├─▶ cruzamento/dedupe ─▶ regras de negócio ─▶ classificação
BigQuery (pagamentos)─┘                                              do dia da semana
                                                                            │
                                                                            ▼
                                                   planilha de pricing ─▶ cálculo do
                                                   (Google Sheets)        pagamento
                                                                            │
                                                                            ▼
                                          planilha de auditoria ◀─ correção de zerados
                                                    │
                                                    ▼
                                     planilha ligada ao AppSheet
                                        (dispara o pagamento)
```

## Como rodar

1. Abra o notebook no Google Colab.
2. Rode a célula de instalação de bibliotecas.
3. Rode a célula de autenticação (`auth.authenticate_user()`) — precisa de uma conta Google com acesso às planilhas envolvidas.
4. Substitua os IDs fictícios de planilha (`PLANILHA_ID`, `PLANILHA_PRICING_`, etc.) pelos IDs reais das suas próprias planilhas, seguindo a mesma estrutura de colunas usada aqui.
5. Rode as células em ordem.

## Aviso

Este projeto é uma peça de portfólio. A lógica de negócio, o pipeline e a arquitetura são reais; os dados, IDs e nomes são inventados. Não use este código diretamente em produção sem revisar as regras de negócio para o seu próprio contexto.

## Licença

Sinta-se livre para usar este código como referência de aprendizado.
