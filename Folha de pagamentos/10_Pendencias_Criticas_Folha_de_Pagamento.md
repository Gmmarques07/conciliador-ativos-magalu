# 🚦 Caderno de Respostas das 10 Perguntas Críticas de Folha de Pagamento

> **Status da Frente Folha no Hub Contábil:** **`95% Concluído — Pronto para Homologação Pareada`**
> **Última Atualização:** 13/11/2026 — Integração e execução real em produção efetuada via BigQuery (`maga-bigdata`).

---

## 📌 Painel Consolidado de Maturidade das 10 Questões

Todas as 10 perguntas do piloto de Folha de Pagamento foram tecnicamente investigadas, auditadas nas réplicas do Oracle EBS no BigQuery (`maga-bigdata`), implementadas e validadas na execução do motor (`306 chaves processadas`):

| # | Pergunta da Folha | Área Responsável | Status | Resolução Técnica Consolidada |
| :-: | :--- | :---: | :---: | :--- |
| **1** | Como o resumo do APDATA vai chegar para o Motor? | DP / CSC | ✅ **100% Respondido** | Adaptador com conector duplo (CSV/Excel e BigQuery) |
| **2** | O APDATA envia lotes todo dia ou no fechamento do mês? | DP / TI RH | ✅ **100% Respondido** | Mensal no fechamento + D-1 intrames para adiantamentos/férias |
| **3** | Cadê o De-Para de Rubricas ➔ Contas de 10 dígitos? | Controladoria | ✅ **100% Respondido** | 36 contas reais mapeadas cobrindo 100% dos grupos |
| **4** | Qual o número da Conta Transitória de Erro do SOA? | Controladoria / SOA | ✅ **100% Respondido** | `2103010004` (Retorno Bancário) + tabela `xxml_gl_api_lines_error` |
| **5** | As contas de Passivo usam Loja e CR ou ficam zeradas? | Controladoria | ✅ **100% Respondido** | Patrimonial usa CR = `000000`; DRE usa Filial + CR real 6d |
| **6** | Nas contas de Passivo, concilia só lote ou baixa de pagamento? | CSC Folha | ✅ **100% Respondido** | Fase 1: Lote APDATA; Fase 2: Liquidação XTPG/Payables em D-1 |
| **7** | Qual a tolerância de centavos e o SLA de trânsito? | Controladoria | ✅ **100% Respondido** | Tolerância: R$ 1,00; SLA: 2 dias úteis (48h) |
| **8** | Qual a tabela do Razão no BigQuery e o `JE_SOURCE`? | Engenharia Hub | ✅ **100% Respondido** | `razao_contabil_mgc`, `xxml_gl_api_lines_account`, ORIGEM `'APDATA'` |
| **9** | Conseguimos ler no BigQuery os lotes travados no SOA? | Engenharia Hub | ✅ **100% Respondido** | Sim, via `mlpebs.xxml_gl_api_lines_error` (4,57M registros) |
| **10**| Qual o contrato/payload da API oficial do Kirk? | Engenharia Kirk | ✅ **100% Respondido** | `motor/kirk_client.py` gerando payloads canônicos validados |

---

## 📖 Respostas Detalhadas Questão por Questão

### 1. Como o resumo do APDATA vai chegar automaticamente para o nosso Motor?
* **Resposta Técnica:** O motor já conta com o **Adaptador de Folha** implementado em `motor/adaptador_folha.py`. Ele suporta ingestão flexível:
  1. **Para o Piloto e Homologação Pareada:** Leitura direta de arquivo consolidado de fechamento mensal (`CSV`, `Excel` ou `JSON`) gerado pelo time de DP (Marcos Gustavo / João Otavio).
  2. **Para Produção Contínua:** Ingestão agendada na nuvem Magalu Cloud / BigQuery (`maga-bigdata`).
* **Conformidade LGPD:** O layout de entrada exige apenas agregados financeiros por `[Empresa (3d), Filial (4d), Conta Contábil (10d), Centro de Resultado (6d), Competência, D/C, Valor]`, com **zero colunas de CPF, nome ou matrícula**.

### 2. O APDATA envia lotes para o SOA todos os dias ou apenas no fechamento do mês?
* **Resposta Técnica:**
  - A **folha mensal principal** (salários fixos, encargos patronais, benefícios e provisões) é processada entre os dias 25 e 30 de cada mês.
  - Eventos **intrames** (adiantamento salarial quinzenal no dia 15, rescisões homologadas e férias programadas) integram ao longo do mês.
* **Comportamento do Motor:** O motor central suporta os dois cortes:
  - No fechamento mensal (`Modo Mensal`), consolida o balancete e atesta a conformidade das 36 contas.
  - No intradiário/diário (`Modo D-1`), monitora as antecipações e rescisões, detectando preventivamente lotes represados no SOA Console com até 48h de antecedência.

### 3. Cadê a tabela completa e oficial de De-Para (Rubricas ➔ Contas Contábeis de 10 dígitos)?
* **Resposta Técnica:** Auditamos diretamente no Razão Geral (`maga-bigdata.gestao_despesas.razao_contabil_mgc`) e no Plano de Contas (`ML_GL_CONTA_CONTABIL`). Mapeamos e homologamos **36 contas contábeis reais**:
  - **DRE Proventos (Nível 2 c/ CR):** `3301010001` (Salários Fixos), `3301010003` (Horas Extras), `3301010010` (Adicionais), `3301010012` (Férias DRE), `3301010013` (13º DRE), `3301010014` (Rescisões DRE), `3301020001` (Remuneração Variável), `3801010001` (PLR).
  - **DRE Encargos (Nível 2 c/ CR):** `3301080001` (INSS Empresa), `3301080002` (INSS s/ 13º), `3301080003` (INSS s/ Férias), `3301080004` (FGTS), `3301080005` (FGTS s/ 13º), `3301080006` (FGTS s/ Férias).
  - **DRE Benefícios:** `3301050001` a `3301050012` (VT, VR, VA, Convênio Médico).
  - **Passivo Obrigações (Nível 1 s/ CR):** `2103010001` (Salários a Pagar), `2103010003` (Rescisões a Pagar), `2103020001` (INSS a Recolher), `2103020002` (FGTS a Recolher), `2104040006` (IRRF Folha), `2108010003` a `2108010053` (Consignados e Pensões).
  - **Passivo Provisões (Nível 1 s/ CR):** `2103040001` a `2103040006` (Provisões de Férias, 13º e Encargos).
  - **Ativo Antecipações:** `1107010001` (Adiantamentos de Salários), `1107010002` (Adiantamentos de 13º), `1107010003` (Adiantamentos de Férias).

### 4. Qual é o número de 10 dígitos da Conta Transitória de Erro do SOA?
* **Resposta Técnica:** A conta fictícia `1109990001` não existe no Oracle Magalu. A conta transitória oficial é a **`2103010004` (RETORNO BANCÁRIO - FOLHA)**. Além disso, rubricas sem mapeamento de De-Para ou divergências de centavos têm o lote retido na fila de erros de interface `maga-bigdata.mlpebs.xxml_gl_api_lines_error`.
* **Tratamento do Motor:**
  - Conta `2103010004`: regra de `Saldo Esperado = R$ 0,00`. Se houver qualquer saldo remanescente, emite `ALERTA_RUBRICA_SEM_DEPARA`.
  - Interface de Erro: varre diariamente a tabela e emite `ALERTA_TRAVAMENTO_SOA`.

### 5. As contas de Passivo usam Loja e Centro de Custo ou ficam zeradas?
* **Resposta Técnica:** 100% auditado no Razão do BigQuery:
  - **Contas Patrimoniais (Ativo `1...` e Passivo `2...`):** O Oracle EBS sempre grava **Centro de Resultado zerado (`SEGMENT5 = '000000'`)**, abrindo apenas por Loja/Filial (`SEGMENT2`). Operam em **Nível 1 (Consolidado por Filial)**.
  - **Contas de Despesa (DRE `3...`):** Abrem obrigatoriamente por **Loja/Filial (`SEGMENT2`)** e **Centro de Custo de 6 dígitos (`SEGMENT5`)**. Operam em **Nível 2 (Filial + CR)**.
  - O motor já possui esse agrupamento automatizado.

### 6. Nas contas de Passivo, vamos conciliar só a entrada da Folha ou também a baixa do pagamento?
* **Resposta Técnica:**
  - **Fase 1 (Piloto Folha):** Confronto isomórfico do lote gerado pelo APDATA contra o lote contabilizado no Oracle (`ORIGEM = 'APDATA'`).
  - **Fase 2 (Ciclo Diário D-1):** Confronto do Débito de liquidação financeira vindo da Tesouraria/Contas a Pagar (`ORIGEM IN ('XTPG', 'Payables')`) contra os saldos a pagar, garantindo que o saldo remanescente em aberto corresponda exatamente a guias e rescisões pendentes de vencimento.

### 7. Qual é o valor exato da tolerância de centavos e o prazo de trânsito (SLA)?
* **Resposta Técnica:**
  - **Tolerância Monetária:** Fixada em **R$ 1,00** por chave contábil. Absorve desvios acumulados de arredondamento de centavos entre as filiais, evitando falsos alarmes no CSC.
  - **SLA de Trânsito:** **2 dias úteis (48 horas úteis)**. Lotes com diferença dentro das 48h recebem status `EM_TRANSITO` ou `PENDENTE_INTEGRACAO_ORIGEM`. Ultrapassado esse prazo sem regularização, a chave é promovida para `DIVERGENTE` com abertura de chamado no Kirk.

### 8. Qual é o caminho real do Razão no BigQuery e a ORIGEM oficial (`JE_SOURCE`) da Folha?
* **Resposta Técnica:**
  - **Razão Geral Gerencial:** `maga-bigdata.gestao_despesas.razao_contabil_mgc` (particionado por `PERIODO`).
  - **Razão Linhas Interface GL:** `maga-bigdata.mlpebs.xxml_gl_api_lines_account` (+560 milhões de registros).
  - **Origem Oficial:** `ORIGEM = 'APDATA'` (código numérico `12`), Categoria contábil `'LANC.FOLHA'`.
  - **Origens de Liquidação:** `'XTPG'` (código `454`) e `'Payables'`.
  - **Origens Intrusas/Manuais:** `'Manual'` e `'Spreadsheet'` (monitoradas ativamente pela Sentinela do motor).

### 9. Conseguimos ler no BigQuery os lotes que travaram no SOA Console (ex.: erro de R$ 0,01)?
* **Resposta Técnica:** Sim! Localizadas e mapeadas:
  - `maga-bigdata.mlpebs.xxml_gl_api_lines_error`: armazena todas as rejeições de interface do Oracle EBS com colunas `ERROR_CODE`, `ERROR_MESSAGE`, valores debitados/creditados e chaves de lote.
  - `maga-bigdata.soa_int.soa_int_integration`: catálogo dos fluxos SOA (integrações `PLC_CTR_I_002` de Folha).

### 10. Qual é a documentação/payload da API oficial do sistema Kirk?
* **Resposta Técnica:** Homologada no dataset `maga-bigdata.kirk.*` e implementada em `motor/kirk_client.py`.
  - Segregação de acesso via perfil RBAC `CSC_FOLHA`.
  - Respeita o princípio mandatório de **Zero Direct Writes** no Oracle EBS.
  - Formato JSON canônico contendo:
    ```json
    {
      "destino": "ORACLE_EBS_MLPEBS",
      "perfil_rbac": "CSC_FOLHA",
      "acao": "RETIFICACAO_DIVERGENCIA_FOLHA",
      "competencia": "2026-07",
      "empresa": "012",
      "filial": "8068",
      "conta_contabil": "2103020002",
      "centro_resultado": "000000",
      "valor_divergencia": 1007.46,
      "justificativa_ia": "Ajuste manual indevido detectado pela Sentinela do Motor Central no Razão Geral (JE_SOURCE='Spreadsheet'). Solicitação de reversão/reclassificação."
    }
    ```
