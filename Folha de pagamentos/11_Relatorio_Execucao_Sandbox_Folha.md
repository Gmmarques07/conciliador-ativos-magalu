# 📊 Relatório de Execução — Motor de Conciliação de Folha (`EPC-3DF6`)

* **Modo de Execução:** `SANDBOX`
* **Data/Hora do Processamento:** `23/11/2026 10:22:31`
* **Total de Chaves Contábeis Avaliadas:** `9`
* **Chamados Pré-Montados p/ API do Sistema Kirk:** `4`

## 1. Resumo por Status Homologado

| Status do Motor | Qtd. Chaves | Ação Operacional |
| :--- | :---: | :--- |
| **`ALERTA_LANCAMENTO_MANUAL`** | 1 | 🟠 Auditar ajuste manual/planilha (`Manual` / `Spreadsheet`) na conta de folha. |
| **`ALERTA_RUBRICA_SEM_DEPARA`** | 1 | 🔴 Notificar CSC DP + Controladoria e regularizar conta transitória `2103010004` via Kirk. |
| **`ALERTA_TRAVAMENTO_SOA`** | 1 | 🔴 Atuar nos erros de interface (`xxml_gl_api_lines_error`) / abrir chamado Kirk. |
| **`CONCILIADO`** | 3 | 🟢 Nenhuma ação humana (exato ou dentro de R$ 1,00). |
| **`DIVERGENTE`** | 1 | 🔴 Divergência > R$ 1,00 — submeter ajuste ao Kirk. |
| **`EM_TRANSITO`** | 1 | 🔵 Aguardar janela normal de até 48h úteis. |
| **`PENDENTE_INTEGRACAO_ORIGEM`** | 1 | 🔵 Baixa antecipada aguardando lote APDATA (até 48h úteis). |

---

## 2. Detalhamento Analítico por Chave Contábil Canônica

| Competência | Filial | Conta (`SEGMENT3`) | Nível / CR | Lado A (APDATA) | Lado B (EBS) | Delta (R$) | Status | Diagnóstico Automático |
| :---: | :---: | :--- | :---: | ---: | ---: | ---: | :--- | :--- |
| `2026-07` | `0088` | `2103010003` (RESCISOES A PAGAR) | N2 / `000000` | R$ 0.00 | R$ -6,420.00 | R$ -6,420.00 | **`PENDENTE_INTEGRACAO_ORIGEM`** | Movimento no Razão aguardando chegada do lote APDATA (16h úteis <= janela de 48h úteis). |
| `2026-07` | `0120` | `3301010001` (SALARIOS FIXOS) | N1 / `040305` | R$ 54,320.00 | R$ 0.00 | R$ -54,320.00 | **`ALERTA_TRAVAMENTO_SOA`** | Divergência de R$ 54,320.00 na Filial 0120: Lote retido na interface xxml_gl_api_lines_error por diferença de R$ 0.02. |
| `2026-07` | `0205` | `3301010003` (HORAS EXTRAS) | N1 / `140802` | R$ 18,400.00 | R$ 0.00 | R$ -18,400.00 | **`EM_TRANSITO`** | Lote gerado pelo APDATA há 18h úteis (dentro do SLA de 48h úteis). |
| `2026-07` | `8068` | `2103010001` (SALARIOS E ORDENADOS A PAGAR) | N2 / `000000` | R$ -6,355,436.24 | R$ -6,355,436.24 | R$ 0.00 | **`CONCILIADO`** | Batimento exato entre APDATA e Razão Geral (R$ 0,00). |
| `2026-07` | `8068` | `2103010004` (RETORNO BANCÁRIO - FOLHA) | N2 / `000000` | R$ 0.00 | R$ -1,250.40 | R$ -1,250.40 | **`ALERTA_RUBRICA_SEM_DEPARA`** | Saldo pendente de R$ 1,250.40 na transitória 2103010004 (RETORNO BANCÁRIO - FOLHA) na Filial 8068. Notificar CSC DP e Controladoria. |
| `2026-07` | `8068` | `2103020001` (INSS A RECOLHER) | N2 / `000000` | R$ -210,000.00 | R$ -210,000.00 | R$ 0.00 | **`CONCILIADO`** | Batimento exato entre APDATA e Razão Geral (R$ 0,00). |
| `2026-07` | `8068` | `2103020002` (FGTS A RECOLHER) | N2 / `000000` | R$ 0.00 | R$ 0.00 | R$ 0.00 | **`ALERTA_LANCAMENTO_MANUAL`** | Ajuste manual/planilha (ORIGEM Manual/Spreadsheet) de R$ 3,800.00 detectado na conta 2103020002 (FGTS A RECOLHER) na Filial 8068. |
| `2026-07` | `8068` | `3301010001` (SALARIOS FIXOS) | N1 / `140802` | R$ 1,054,320.50 | R$ 1,054,320.12 | R$ -0.38 | **`CONCILIADO`** | Conciliado dentro da tolerância oficial de R$ 1.00 (desvio de R$ 0.38 p/ arredondamento). |
| `2026-07` | `8068` | `3301080004` (FGTS (DESPESA)) | N1 / `140906` | R$ 84,320.00 | R$ 80,000.00 | R$ -4,320.00 | **`DIVERGENTE`** | Diferença de R$ -4,320.00 acima da tolerância de R$ 1.00 (APDATA: R$ 84,320.00 vs. Razão Oracle: R$ 80,000.00). |

---

## 3. Fila de Payloads Canônicos Gerados para a API do Sistema Kirk (OAuth 2.0)

```json
[
  {
    "sistema_origem": "HUB_CONTABIL",
    "tipo_solicitacao": "AJUSTE_FOLHA",
    "competencia": "2026-07",
    "empresa": "001",
    "filial": "0120",
    "conta_debito": "3301010001",
    "conta_credito": "2103010004",
    "centro_resultado": "040305",
    "valor": 54320.0,
    "motivo": "Divergência de R$ 54,320.00 na Filial 0120: Lote retido na interface xxml_gl_api_lines_error por diferença de R$ 0.02.",
    "trilha_auditoria_id": "MTR-202607-00981"
  },
  {
    "sistema_origem": "HUB_CONTABIL",
    "tipo_solicitacao": "AJUSTE_FOLHA",
    "competencia": "2026-07",
    "empresa": "012",
    "filial": "8068",
    "conta_debito": "2103010001",
    "conta_credito": "2103010004",
    "centro_resultado": "000000",
    "valor": 1250.4,
    "motivo": "Regularização de saldo transitório / rubrica pendente Folha",
    "trilha_auditoria_id": "MTR-202607-00982"
  },
  {
    "sistema_origem": "HUB_CONTABIL",
    "tipo_solicitacao": "AJUSTE_FOLHA",
    "competencia": "2026-07",
    "empresa": "012",
    "filial": "8068",
    "conta_debito": "2103020002",
    "conta_credito": "2103010004",
    "centro_resultado": "000000",
    "valor": 3800.0,
    "motivo": "Ajuste manual/planilha (ORIGEM Manual/Spreadsheet) de R$ 3,800.00 detectado na conta 2103020002 (FGTS A RECOLHER) na Filial 8068.",
    "trilha_auditoria_id": "MTR-202607-00983"
  },
  {
    "sistema_origem": "HUB_CONTABIL",
    "tipo_solicitacao": "AJUSTE_FOLHA",
    "competencia": "2026-07",
    "empresa": "012",
    "filial": "8068",
    "conta_debito": "3301080004",
    "conta_credito": "2103010004",
    "centro_resultado": "140906",
    "valor": 4320.0,
    "motivo": "Diferença de R$ -4,320.00 acima da tolerância de R$ 1.00 (APDATA: R$ 84,320.00 vs. Razão Oracle: R$ 80,000.00).",
    "trilha_auditoria_id": "MTR-202607-00984"
  }
]
```
