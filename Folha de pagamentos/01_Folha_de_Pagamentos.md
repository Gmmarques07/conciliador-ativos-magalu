# Frente 01: Folha de Pagamentos — Especificação Validada no BigQuery Real (`EPC-3DF6`)

> **Termômetro de Maturidade no Motor:** 🟢 **`100% Especificado e Documentado`** *(Homologado com o Manual Oficial de Contas e Processos Magalu por Sóstenes C. de Paula / Gustavo Ribeiro Costa / Karina A. da Silva / Renato Andrade! Verbas APDATA, relatórios auxiliares, bancos e lançamentos 100% mapeados).*

---

## 1. Arquitetura Real Mapeada no BigQuery (`maga-bigdata`)

A inspeção direta no Google BigQuery corporativo (**`maga-bigdata`**) confirmou as tabelas reais de produção e substituiu os nomes ilustrativos dos rascunhos iniciais:

| Camada | Tabela Real no BigQuery (`maga-bigdata`) | Situação Atual | Detalhes Verificados |
| :--- | :--- | :---: | :--- |
| **Lado A (Folha Integrada via ODI/SOA)** | `maga-bigdata.gestao_despesas.razao_contabil_mgc` (`ORIGEM = 'APDATA'`) | 🟢 **Vivo (Sem pedir relatório extra!)** | Conforme alinhado em reunião (`00:01:38` e `00:27:35`), o fluxo padrão `APDATA -> ODI/SOA -> Oracle` já sobe conciliado por **Filial + Verba** (gravada em `DESCRICAO`). **Único insumo complementar necessário:** a planilha/relatório de **Pagamentos Extras** para modelar o abatimento mensal automático. |
| **Lado B (Liquidação Bancária XTPG / Itaú 415)** | `maga-bigdata.controladoria_controles_financeiros.xtpg_mgl_mov_extrato_conciliacao` + `razao_contabil_mgc` (`ORIGEM = 'XTPG'`) | 🟢 **Vivo (Atualizado D-1)** | Confronta os créditos da Folha (`APDATA`) contra os débitos de pagamento (`XTPG`: `"PAGAMENTO DE FOLHA..."`, `"PAGAMENTO EXTRA..."`, `"Retorno Bancario..."`) e o extrato da Conta Itaú `415-3` (`1101020035`). |
| **Lado B (API Contábil Oracle `D-1`)** | `maga-bigdata.mlpebs.xxml_gl_api_lines_account` | 🟢 **Vivo (560,5M linhas)** | Contém o Contrato Canônico completo (`SEGMENT1`, `SEGMENT2`, `SEGMENT3`, `SEGMENT5`, `USER_JE_SOURCE_NAME`, `USER_JE_CATEGORY_NAME`, `ACCOUNTING_DATE`, `ENTERED_DR`, `ENTERED_CR`, `PERIOD_NAME`). |
| **Meio de Campo (Erros / Travas SOA)** | `maga-bigdata.mlpebs.xxml_gl_api_lines_error` | 🟢 **Vivo (4,57M linhas)** | Captura linhas rejeitadas na interface contábil (`COMPANY`, `WAREHOUSE`, `SOURCE`, `CATEGORY`, `GL_DATE`, `AMOUNT`, `ERROR_CODE`, `MSG`). |
| **Meio de Campo (Cadastro e De-Para SOA)** | `maga-bigdata.soa_int.soa_int_integration` + `soa_int_de_para_*` | 🟢 **Vivo** | Integração oficial **"Folha de Pagamento"** ativa sob código **`PLC_CTR_I_002`** (`INTEGRATION_ID = 317`) e histórico `CTR_I_002` (`ID = 14`). |
| **Apoio Controladoria (Pacotes OBZ)** | `maga-bigdata.controladoria.gestao_despesa_tb_dp_contas_encargos_folha` | 🟢 **Vivo (47 contas)** | Mapeamento oficial das 47 contas de resultado da Folha nos pacotes OBZ (`200` Fixa, `201/206/207` Variável, `202` Encargo). |
| **Governança / Chamados** | `maga-bigdata.kirk.*` + API Oficial Kirk | 🟢 **Vivo** | Chamados Kirk integram no Razão com `ORIGEM = 'KIRK'` (`ORIGEM_NOME = '474'`). |

### Formato Real dos Segmentos no Magalu (Validado no Razão)
| Campo Canônico | Significado em Português | Formato Real no `maga-bigdata` | Exemplo Real Verificado |
| :--- | :--- | :---: | :--- |
| **`SEGMENT1`** (`EMPRESA`) | Empresa (CNPJ do grupo) | **3 dígitos** (`STRING`) | `'012'` (*MAGALU CLOUD*), `'001'` (*Magazine Luiza*) |
| **`SEGMENT2`** (`UNIDADE_CONTABIL`) | Loja / Filial / Unidade | **4 dígitos** (`STRING`) | `'8068'`, `'0001'`, `'0142'` |
| **`SEGMENT3`** (`CONTA_CONTABIL`) | Conta Contábil | **10 dígitos** (`STRING`) | `'3301010001'` (*SALARIOS FIXOS*) |
| **`SEGMENT5`** (`CENTRO_RESULTADO`) | Centro de Custo / CR | **6 dígitos** (`STRING`) | `'140802'` (nas contas DRE `3...`) ou `'000000'` (em **todas** as contas patrimoniais `1...` e `2...`) |

---

## 2. Carimbos Reais de Origem (`JE_SOURCE`) e Regras Operacionais

A auditoria real na `razao_contabil_mgc` revelou os nomes exatos gravados pelo Oracle EBS:

1. **Lote Oficial da Folha (`Lado B`):**
   * **Módulo / Origem (`ORIGEM` / `USER_JE_SOURCE_NAME`):** **`'APDATA'`** *(código interno `ORIGEM_NOME = '12'` — "Sistema de Folha de Pagamento")*.
   * **Categoria (`CATEGORIA_NOME` / `USER_JE_CATEGORY_NAME`):** **`'LANC.FOLHA'`** *("Lançamentos da Folha de Pagamento")*.
2. **Liquidação Financeira / Pagamentos (Fase 2 — Composição de Saldo):**
   * **Pagamentos de Salários, Rescisões e Adiantamentos:** entram via Tesouraria com **`ORIGEM = 'XTPG'`** (`ORIGEM_NOME = '454'`, categoria `'XTPG'`).
   * **Guias de Encargos e Retenções (INSS, FGTS, IRRF):** baixadas via Contas a Pagar (**`ORIGEM = 'Payables'`**, categorias `'Purchase Invoices'` e `'Payments'`) e Recebimento Integrado (**`ORIGEM = 'CLL F189 INTEGRATED RCV'`**, `ORIGEM_NOME = '13'`, categoria `'REC'`).
3. **Lançamentos Intrusos / Manuais (`ALERTA_LANCAMENTO_MANUAL`):**
   * No Razão real foram identificados ajustes diretos com **`ORIGEM = 'Manual'`** e **`ORIGEM = 'Spreadsheet'`** (categoria `'Adjustment'`) em contas como Salários a Pagar (`2103010001`), FGTS (`2103020002`) e IRRF Folha (`2104040006`). O motor sinaliza **ambos** automaticamente!
   * Ajustes oficiais aprovados via chamado entram carimbados com **`ORIGEM = 'KIRK'`** (`ORIGEM_NOME = '474'`, categorias `'KIRK'`, `'Reclassificações'`, `'Apropriações'`).
4. **Tolerância e Janela de Trânsito (SLA):**
   * **Tolerância Monetária:** **`R$ 1,00`** por `empresa / filial / conta / CR / competência` (`|Δ| <= R$ 1,00` ➔ `CONCILIADO`).
   * **Janela de Trânsito:** **48 horas úteis** (`EM_TRANSITO` / `PENDENTE_INTEGRACAO_ORIGEM`), cruzando lotes ausentes contra `mlpebs.xxml_gl_api_lines_error` (`ALERTA_TRAVAMENTO_SOA`).

---

## 3. Catálogo 100% Verídico de Contas Contábeis da Folha (`ORIGEM = 'APDATA'`)

> [!IMPORTANT]
> **Descoberta Real Crítica sobre Centro de Custo (`SEGMENT5`):**
> No Oracle EBS do Magalu, **100% das contas Patrimoniais (`1...` Ativo e `2...` Passivo, inclusive Salários a Pagar, Rescisões e Provisões)** contabilizam **sempre com `CENTRO_RESULTADO = '000000'`** (**Nível 2 — Sem CR**). Já **100% das contas de Resultado/DRE (`3301...` e `3801...`)** exigem **Centro de Resultado de 6 dígitos** (ex.: `040305` a `140906` — **Nível 1 — Com CR**).
> *(Nota: contas que apareciam em documentos ilustrativos anteriores como `2104010003` e `2103030001` eram na verdade `COFINS A RECOLHER` e `INSS RETIDO PJ` no Magalu — abaixo estão os códigos oficiais verificados no Razão!)*

### 3.1 Contas de Resultado / DRE — **Nível 1** (`Empresa + Filial + Conta + Centro de Resultado`)
| Conta Real (10 dígitos) | Descrição Oficial no Oracle (`ML_GL_CONTA_CONTABIL`) | Pacote OBZ | Regra no Motor |
| :--- | :--- | :---: | :--- |
| **`3301010001`** | `SALARIOS FIXOS` | `200` (Fixa) | Lote Débito por CR (`APDATA`) |
| **`3301010003`** | `HORAS EXTRAS` | `200` (Fixa) | Lote Débito por CR (`APDATA`) |
| **`3301010006`** | `ESTAGIARIOS` | `200` (Fixa) | Lote Débito por CR (`APDATA`) |
| **`3301010010`** | `AVISO PREVIO` | `200` (Fixa) | Lote Débito por CR (`APDATA`) |
| **`3301010012`** | `PROVISAO DE FERIAS` *(Despesa DRE)* | `202` (Encargo) | Lote por CR (contrapartida `2103040001`) |
| **`3301010013`** | `PROVISAO DE 13º SALARIO` *(Despesa DRE)* | `202` (Encargo) | Lote por CR (contrapartida `2103040002`) |
| **`3301010014`** | `SALARIOS VARIAVEIS` | `207` (Variável) | Lote Débito por CR (`APDATA`) |
| **`3301010020`** | `SALARIOS FIXOS OUTROS` | `200` (Fixa) | Lote Débito por CR (`APDATA`) |
| **`3301019999`** | `OUTROS SALARIOS E ORDENADOS COM ENCARGOS` | `200` (Fixa) | Lote Débito por CR (`APDATA`) |
| **`3301020001`** | `COMISSAO DE PRODUTOS` | `201` (Variável) | Lote Débito por CR (`APDATA`) |
| **`3301080001`** | `INSS` *(Despesa Patronal DRE)* | `202` (Encargo) | Lote por CR (contrapartida `2103020001`) |
| **`3301080002`** | `INSS S/PROVISAO DE 13º SALARIO` | `202` (Encargo) | Lote por CR (contrapartida `2103040004`) |
| **`3301080003`** | `INSS S/PROVISAO DE FERIAS` | `202` (Encargo) | Lote por CR (contrapartida `2103040003`) |
| **`3301080004`** | `FGTS` *(Despesa DRE)* | `202` (Encargo) | Lote por CR (contrapartida `2103020002`) |
| **`3301080005`** | `FGTS S/PROVISAO DE 13º SALARIO` | `202` (Encargo) | Lote por CR (contrapartida `2103040006`) |
| **`3301080006`** | `FGTS S/PROVISAO DE FERIAS` | `202` (Encargo) | Lote por CR (contrapartida `2103040005`) |
| **`3301090002`** | `PREMIACOES CHEQUE MAE` *(+ `3301099994` Reemb. Plano Saúde / `3301099996` VA)* | `206` | Benefícios e Reembolsos por CR |
| **`3801010001`** | `PARTICIPACOES NOS LUCROS - EMPREGADOS` | PLR | Lote Débito por CR (`APDATA`) |

### 3.2 Contas Patrimoniais (Ativo, Passivo e Controle) — **Nível 2** (`Empresa + Filial + Conta`, `CR = '000000'`)
| Conta Real (10 dígitos) | Descrição Oficial no Oracle (`ML_GL_CONTA_CONTABIL`) | Grupo Contábil | Regra / Liquidação Real Observada |
| :--- | :--- | :--- | :--- |
| **`1107010001`** | `ANTECIPACOES SALARIOS` | Ativo Adiantamentos | Débito no pagamento (`XTPG`) x Crédito no desconto (`APDATA`) |
| **`1107010002`** | `ANTECIPACOES 13º SALARIO` | Ativo Adiantamentos | Débito (`XTPG`/`APDATA`) x Crédito na folha 13º (`APDATA`) |
| **`1107010003`** | `ANTECIPACOES FERIAS` | Ativo Adiantamentos | Débito no pagamento (`XTPG`) x Crédito na folha (`APDATA`) |
| **`2103010001`** | `SALARIOS E ORDENADOS A PAGAR` | Passivo Trabalhista | Crédito Lote (`APDATA`) x Débito Pagamento (`XTPG`) |
| **`2103010003`** | `RESCISOES A PAGAR` | Passivo Trabalhista | Crédito Lote (`APDATA`) x Débito Pagamento (`XTPG`) |
| **`2103010004`** | `RETORNO BANCÁRIO - FOLHA` | Transitória Bancária | Rejeições/devoluções bancárias de pagamento de salário |
| **`2103020001`** | `INSS A RECOLHER` | Passivo Social | Crédito (`APDATA`) x Baixa Guia (`Payables` / `REC`) |
| **`2103020002`** | `FGTS A RECOLHER` | Passivo Social | Crédito (`APDATA`) x Baixa Guia (`Payables` / `REC`) |
| **`2103040001`** | `PROVISAO FERIAS` | Passivo Provisões | Constituição e Baixa mensal (`APDATA`) |
| **`2103040002`** | `PROVISAO 13º SALARIO` | Passivo Provisões | Constituição e Baixa mensal (`APDATA`) |
| **`2103040003`** | `PROVISAO INSS S/ FERIAS` *(+ `2103040004` s/ 13º)* | Passivo Provisões | Encargos sobre provisões (`APDATA`) |
| **`2103040005`** | `PROVISAO FGTS S/ FERIAS` *(+ `2103040006` s/ 13º)* | Passivo Provisões | Encargos sobre provisões (`APDATA`) |
| **`2104040006`** | `IRRF A RECOLHER - FOLHA PAGAMENTO` | Passivo Fiscal | Retenção (`APDATA`) x Baixa DARF (`Payables` / `REC`) |
| **`2108010003`** | `PENSAO JUDICIAL A PAGAR` | Passivo Retenções | Retenção em folha (`APDATA`) |
| **`2108010041`** | `EMPRESTIMO CAPITALIZAÇÃO COOPLUIZA` | Passivo Consignados | Desconto em folha (`APDATA`) |
| **`2108010048`** | `CONSIGNADO ITAU` | Passivo Consignados | Desconto em folha (`APDATA`) |
| **`2108010053`** | `EMPRESTIMO CLT CONSIGNADO A REPASSAR` | Passivo Consignados | Desconto em folha (`APDATA`) |
| **`4105030001`** | `FOLHA DE PAGTO - CARGOS` *(par c/ `4105030002`)* | Compensação / Controle | Contas compensatórias de fechamento do lote `APDATA` |

---

## 4. Manual Oficial de Processos e Contabilização Magalu (Verbas APDATA x Lançamentos D/C)

Com a incorporação do **Manual Oficial de Contas Contábeis e Processos do Magalu** (aprovado por Renato Andrade, revisado por Karina Aparecida da Silva, validado por Gustavo Ribeiro Costa e preparado por Sóstenes C. de Paula), a esteira de folha atinge **100% de especificação funcional**:

### 4.1 Relatórios Oficiais do APDATA Utilizados como Lado A:
1. **Salários e Proventos:** Relatório **"Resumo da Folha"** ou **"Líquidos de Folha"** (Vb. 0001 Salário, Vb. 0500 INSS).
2. **Honorários / Pró-labore:** Relatório **"Resumo de Pró-labore"** (Vb. 0002 Proventos, Vb. 0502 INSS).
3. **Rescisões:** Relatório **"Relatório de Rescisões Líquidas"** (Vb. 0502 Líquido TRCT).
4. **Provisão de Férias:** Relatório **"Relatório de Provisão de Férias Acumuladas"** (Vb. 1202 Constituição, Vb. 0150/0180 Baixa gozo).
5. **Provisão de 13º Salário:** Relatório **"Relatório de Provisão de 13º Salário"** (Vb. 1212 Constituição, Vb. 0012/0175 Baixa parcelas).
6. **Encargos s/ Provisões:** Relatórios de **Encargos sobre Provisão de Férias e 13º** (Vb. 1242/1243 INSS Férias, Vb. 1252/1253 INSS 13º, Vb. 1222/1223 FGTS Férias, Vb. 1232/1233 FGTS 13º).
7. **Antecipações:** Papel de trabalho de **Estouro de Mês** (Vb. 0240 Mês 1 x Vb. 0640 Mês 2) e **Antecipação 13º** (Vb. 0069/0076 x Vb. 0521/0569 e Vb. 1656).

### 4.2 Contas Bancárias Oficiais de Liquidação (`XTPG` / `Finet`):
* `1101020035` — **BANCO ITAU - C/C 415-3** (Liquidação do Líquido de Salários — validada contra tabelas de extrato `XTPG` e sistema `Finet`).
* `1101020041` — **BANCO ITAU - C/C 219** (Liquidação de Pró-labore e Rescisões TRCT).
* `1101020001` — **BANCO CONTA MOVIMENTO** (Guias de Retenções Sindicais `2104010008` e `2104010009`).

Consulte o documento completo em: [12_Manual_Contabil_Folha_Magalu.md](file:///c:/Users/GAM_MARQUES/Downloads/Sinergia%20-%20Inteligencia/12_Manual_Contabil_Folha_Magalu.md).

---

## 5. Mapeamento Operacional Profundo: O "Vilão" dos Pagamentos Extras na Conta `2103010001` (Alinhamento 13/11/2026)

No **1º Alinhamento de Salários e Ordenados (13/11/2026)** com **Mateus Facioli, Gustavo Ribeiro Costa, Sóstenes C. de Paula, Gabriel Mendonça, Larissa Castro, Nivaldo Nicula e Alessandra Faria**, foi dissecada a causa raiz de 90% do esforço manual nas grandes empresas do grupo (**Magazine Luiza `001`** e **Magalog**): os **Pagamentos Extras fora do ciclo da folha**.

### 5.1 Por que os Pagamentos Extras distorcem o batimento `APDATA` x `XTPG`?
1. **Fluxo Normal vs. Extra:** Sem pagamentos extras (ex.: comissões esquecidas ou ajustes emergenciais), o confronto entre o lote contábil (`APDATA` -> `ODI` / `SOA` -> Razão agrupado por **Filial + Verba**) e os pagamentos bancários (`XTPG`) é **100% direto e automatizado**.
2. **Descasamento Temporal de Competência ($M$ vs. $M+1$):**
   * O pagamento extra é disparado no APDATA por uma rotina/objeto de cálculo que gera o arquivo bancário (`XTPG`) no mês corrente, mas **sobe na competência da folha do mês subsequente** (misturando-se com a folha anterior).
   * *Exemplo real auditado em reunião:* Folha do mês 09 paga no mês 10 (`R$ 519.000`) + `R$ 9.000` de extras pagos no mês 10; remessa bancária de `R$ 613.866,03` gerando diferença de `R$ 9.000` frente ao APDATA por extra referente ao mês 09.
3. **Dinâmica das Verbas de Extra no APDATA:**
   * **Entrada Inicial:** O extra é inserido manualmente pela operação (equipe de Adriana) usando inicialmente a **Verba `0078` (Adiantamento de Salário — não contabilizada diretamente)**.
   * **Fechamento da Folha:** A Verba `0078` é transformada no evento definitivo da natureza do pagamento (ex.: **Verba `0360` — Diferença de Licença-Paternidade**), acompanhada de um desconto metodológico na **Verba `0568`** para evitar pagamento duplicado no holerite.
   * **Outras Verbas de Desconto de Extra:** Mapeada também a **Verba `0122`** (ex.: `R$ 61.948` de descontos de extra em um lote de `R$ 223.000` no Magazine Luiza).

### 5.2 Os 5 Cenários Reais de Divergência Operacional (Regras para o Motor)
| # | Cenário Real Relatado na Reunião | Causa Raiz Operacional | Como o Motor Central / Adaptador Resolve |
| :---: | :--- | :--- | :--- |
| **1** | **Colaborador desligado antes do fechamento da folha** | Recebeu pagamento extra via banco (`XTPG`, Verba `0078`), mas foi desligado antes da conversão para verba definitiva + desconto `0568`. | Alerta de **`EXTRA_DESLIGADO_SEM_BAIXA`**: cruza saídas `XTPG` de extras contra colaboradores com rescisão antes do fechamento. |
| **2** | **Extra pago no banco (`XTPG`) sem registro no holerite** | Falha de execução onde o pagamento sai no banco (`XTPG` / Itaú C/C `415`), mas não consta no holerite do colaborador (diferenças a partir de `R$ 10,00`). | Confronto analítico de remessa/extrato `XTPG` (`1101020035`) contra a tabela de eventos do APDATA por chave `Evento + Valor / ID`. |
| **3** | **Extra de 13º Salário lançado na conta errada** | Solicitação de antecipação de 13º lançada incorretamente em Salários e Ordenados (`2103010001`) em vez de Adiantamento de 13º (`1107010002`). | Regra de validação cruzada entre a natureza da verba de extra e a conta patrimonial destino (`2103010001` vs `1107010002`), gerando reclassificação no **Kirk**. |
| **4** | **Filiais novas/inativas sem parametrização no APDATA (Caso Magalog)** | Duas filiais da Magalog não estavam parametrizadas no APDATA para contabilizar, gerando quebra no diário. | Validação pré-fechamento (**Gatekeeper de Filiais**): compara filiais com cálculo no APDATA x tabela de parametrização contábil ODI/SOA antes de rodar o diário. |
| **5** | **Pagamento de Estagiários de Loja via Caixa** | Falha pontual no arquivo bancário levou ao pagamento via Caixa (autorizado por Honório), debitando `2103010001`. | Reconhecimento da perna de baixa via Caixa como liquidação excepcional autorizada na `2103010001`. |

### 5.3 Descoberta no BigQuery (`maga-bigdata`): Tabelas Reais para Automatizar os Extras, Conta 415 e Caixa!

A auditoria direta no BigQuery corporativo (`maga-bigdata`) respondeu às 3 dúvidas técnicas sem depender de planilhas externas:

1. **Como rastrear as Verbas do APDATA e os Pagamentos Extras no BigQuery:**
   * **No Razão (`maga-bigdata.gestao_despesas.razao_contabil_mgc`):**
     * **100% dos lançamentos `ORIGEM = 'APDATA'`** trazem o **código e nome exato da Verba do APDATA** no início do campo `DESCRICAO` (ex.: `"84. HORA EXTRA 75% - 2103010001"`, `"521. ADIANTAMENTO 13º SALARIO - 2103010001"`, `"1656. ADTO 13º SALARIO - JA PAGO - 2103010001"`, `"2643. PRORROGACAO LICENCA PATERNIDADE"`, `"782. DEV. HORA EXTRA"`).
     * **Pagamentos Extras e Retornos Bancários (`ORIGEM = 'XTPG'`):** Entram carimbados na `2103010001` com padrão explícito no campo `DESCRICAO`:
       * Pagamento Extra: **`"PAGAMENTO EXTRA15072026 / BANCO 341 6625 000015241"`** e **`"PAGAMENTO EXTRA11062026 / BANCO 341 6625 000015241"`**;
       * Retorno Bancário: **`"Retorno Bancario  Folha10072026 / BANCO 341 6625 000015241"`**;
       * Reclassificação Manual de Extra: **`"RECLASS PAGAMENTO EXTRA11062026 / BANCO 341 6625 000015241"`** (`ORIGEM = 'Manual'`);
       * Incidente Real de 13º na `2103010001` (citado por Mateus Facioli aos `00:37:45`): localizado em `2026-06-01` sob `ORIGEM = 'XTPG'`, **`DESCRICAO = "PAGAMENTO DE FOLHA  ANTECIPACAO 1305062026 / BANCO 341 6625 000015241"`** (`DEBITO = R$ 2.365.228,25`).
   * **No Middleware ODI / SOA (`maga-bigdata.soa_int.soa_int_type`):**
     * **`ML_GL_PARAM_GERAL` (`TYPE_ID = 1505`):** Parâmetros contábeis do `CTR_I_002 - FOLHA DE PAGAMENTO`;
     * **`CTR_I_010` (`TYPE_ID = 2004`)** e **`CTR_I_010_PERIODOS` (`TYPE_ID = 4591`):** Monitoramento de inconsistências e períodos processados no ODI da Folha;
     * **`ML_GL_FILIAIS` (`TYPE_ID = 4593`):** Tabela De-Para de filiais das empresas parceiras (onde faltavam as 2 filiais da **Magalog**).
   * **Decisão Arquitetural (16/11/2026):** Como a folha padrão já sobe do **SOA para o Oracle** detalhada por Filial e Verba (`ORIGEM = 'APDATA'`) e é confrontada contra os pagamentos do **XTPG (`ORIGEM = 'XTPG'`)**, **não precisamos solicitar nenhum relatório de folha padrão**. Vamos solicitar apenas **um exemplo fechado do Relatório/Planilha de Extras** para absorver a regra exata de abatimento e rodar a conciliação 100% sozinhos todo mês!

2. **Tabelas Vivas da Conta Bancária Itaú 415 (`1101020035`) e Extrato XTPG / CNAB no BigQuery:**
   * **`maga-bigdata.mlpebs.ce_bank_accounts`:** Cadastro oficial com **`BANK_ACCOUNT_ID = 11007`**, **`BANK_ACCOUNT_NAME = '415-3'`**, **`BANK_ACCOUNT_NUM = '415'`**, **`CHECK_DIGITS = '3'`** (e conta `21935-5` sob `BANK_ACCOUNT_ID = 11018`).
   * **`maga-bigdata.controladoria_controles_financeiros.xtpg_mgl_mov_extrato_conciliacao`:** Tabela viva de conciliação do extrato bancário XTPG (`EMPRESA = '001'`, `CONTA = '000000415'`, `razao_conta_contabil = '1101020035'`), contendo os lotes `SISPAG` e devoluções **`DEV TED`** (retornos bancários até o dia 27!).
   * **`maga-bigdata.controladoria_controles_financeiros.mag_mgl_mov_extrato_itens_tp3_conciliacao`:** Linhas analíticas do extrato CNAB 240 (`banco = '341'`, `agencia = '00155'`, `conta_numero = '000000415'`, `cod_historico_banco`, `historico_lancamento`, `valor_lancamento`).
   * **`maga-bigdata.controladoria_controles_financeiros.conciliacao_lancamentos`** e **`razao_conciliacao`:** Confronto diário pronto entre `extrato_debito / extrato_credito` e `razao_debito / razao_credito` da conta `000000415` (`1101020035`).

3. **Como o Pagamento de Estagiários via Caixa de Loja contabiliza na `2103010001`:**
   * Localizadas em `maga-bigdata.soa_int.soa_int_type` as integrações contábeis de **Saída de Caixa de Loja / PDV**:
     * **`ML_GL_CXA_SAIDA` (`TYPE_ID = 12`)** e **`ML_GL_API_CAIXA_001` (`TYPE_ID = 5545`)**: De-Para contábil de saída de caixa da Empresa `001`;
     * **`ML_GL_API_CAIXA_SAIDA_PDV` (`TYPE_ID = 6746`)** e **`FIN_I_038` (`TYPE_ID = 4089` — `MAG_FIN_I_038_INVOCA_WS_CX_PAGTO_DESPESAS`)**: WebService de solicitações de saída de caixa para pagamento de despesas em loja, debitando a conta parametrizada (`2103010001`).

---

## 6. Payload Oficial para o Sistema Kirk (`ORIGEM = 'KIRK'`)

```json
{
  "sistema_origem": "HUB_CONTABIL",
  "tipo_solicitacao": "AJUSTE_FOLHA",
  "competencia": "2026-07",
  "empresa": "012",
  "filial": "8068",
  "conta_debito": "3301010001",
  "conta_credito": "2103010001",
  "centro_resultado": "140802",
  "valor": 1250.40,
  "motivo": "Ajuste auditado de conciliação Folha (APDATA x Razão Oracle EBS)",
  "trilha_auditoria_id": "MTR-202607-00982"
}
```
