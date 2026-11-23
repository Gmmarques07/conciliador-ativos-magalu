# 📑 Relatório Suporte Automatizado — Conta `2103010001` (Salários e Ordenados a Pagar)

> **Gerado pelo Sub-Motor de Salários e Ordenados ([conciliador_salarios_2103010001.py](file:///c:/Users/GAM_MARQUES/Downloads/Sinergia%20-%20Inteligencia/motor/conciliador_salarios_2103010001.py))**
> Execução real sobre os meses já fechados no Oracle EBS (`maga-bigdata.gestao_despesas.razao_contabil_mgc` — Empresa `012` / Filial `8068`), demonstrando como o Motor isola automaticamente **Folha Normal (`APDATA`)**, **Pagamentos Bancários (`XTPG`)**, **Pagamentos Extras**, **Retornos Bancários** e **13º Salário**.

---

## 1. Decomposição Automática dos Meses Fechados no Oracle (`2103010001`)

| Competência | Emp / Filial | Pagto Folha Normal (`XTPG` Débito) | Devoluções no Lote Folha (`XTPG` Crédito) | Reenvio Retorno Bancário (`XTPG` Débito) | Prova Real Retornos ($\Delta$) | Pagamentos Extras no Banco (`XTPG` Débito) | Ajustes Manuais / Reclass de Extra | Status & Diagnóstico Automático do Motor |
| :---: | :---: | ---: | ---: | ---: | :---: | ---: | ---: | :--- |
| **`2026-07`** | `012` / `8068` | R$ 3.273.998,75 | R$ 13.958,04 | R$ 13.958,04 *(R$ 7.306,54 + R$ 6.651,50)* | 🟢 **R$ 0,00** | **R$ 0,04** *(15/07 e 17/07)* | R$ 0,00 | 🟢 **`CONCILIADO`**: Retornos bancários (`08/07` e `10/07`) zeraram ao centavo com o estorno da folha (`06/07`). |
| **`2026-06`** | `012` / `8068` | R$ 3.338.556,50 *(+ R$ 2.365.228,25 Adto 13º)* | R$ 9.084,34 *(R$ 3.819,02 + R$ 5.265,32)* | R$ 9.084,34 *(19/06/2026)* | 🟢 **R$ 0,00** | **R$ 21.535,85** *(11/06/2026)* | **R$ 19.887,90** *(Verba 1796 ILP)* | 🟠 **`ALERTA_EXTRA_E_13`**:<br>1. Extra de `R$ 21.535,85` teve `R$ 19.887,90` reclassificado manualmente (`Verba 1796`), restando **`R$ 1.647,95`** de extra líquido.<br>2. Lote `ANTECIPACAO 1305062026` (`R$ 2.365.228,25`) transitou na `2103010001`. |
| **`2026-05`** | `012` / `8068` | R$ 3.297.313,00 | R$ 0,00 | R$ 0,00 | 🟢 **R$ 0,00** | **R$ 0,06** *(15/05/2026)* | R$ 0,00 | 🟢 **`CONCILIADO`**: Sem retornos bancários pendentes; extra residual de `R$ 0,06` isolado. |
| **`2026-04`** | `012` / `8068` | R$ 3.160.517,00 | R$ 0,00 | R$ 0,00 | 🟢 **R$ 0,00** | **R$ 0,03** *(15/04/2026)* | R$ 0,24 *(Kirk IDK 19396571)* | 🟢 **`CONCILIADO`**: Ajuste residual de `R$ 0,24` regularizado via chamado oficial **Kirk (`IDK: 19396571`)**. |

---

## 2. Duas Leis Contábeis Descobertas no Oracle que o Motor Já Faz Sozinho

Ao inspecionar as partidas fechadas de Abril a Julho/2026 na `2103010001`, o Motor provou matematicamente dois batimentos que antes eram feitos à mão em tabela dinâmica:

### Lei 1 — Casamento Exato dos Retornos Bancários (`XTPG`)
Quando um pagamento de salário volta do banco (rejeição bancária), o `XTPG` lança um **Crédito** na `2103010001` com a descrição `"PAGAMENTO DE FOLHA<DDMMYYYY>"`, e quando a tesouraria reenvia o pagamento até o dia 27, o `XTPG` lança um **Débito** com a descrição `"Retorno Bancario Folha<DDMMYYYY>"`:
* **Prova Real em Julho/2026 (`2026-07`):**
  $$\text{Crédito Devolvido (`06/07/2026`)} = \text{R\$ } 13.958,04$$
  $$\text{Débitos de Retorno (`08/07` R\$ } 7.306,54 + \text{`10/07` R\$ } 6.651,50) = \text{R\$ } 13.958,04 \implies \mathbf{\Delta = \text{R\$ } 0,00!}$$
* **Prova Real em Junho/2026 (`2026-06`):**
  $$\text{Créditos Devolvidos (`05/06` Folha R\$ } 3.819,02 + \text{`05/06` 13º R\$ } 5.265,32) = \text{R\$ } 9.084,34$$
  $$\text{Débito de Retorno (`19/06/2026`)} = \text{R\$ } 9.084,34 \implies \mathbf{\Delta = \text{R\$ } 0,00!}$$

### Lei 2 — Isolamento Automático dos Pagamentos Extras (`PAGAMENTO EXTRA<DDMMYYYY>`)
Em **Junho/2026 (`2026-06`)**, o Motor isolou exatamente o ciclo de vida de um **Pagamento Extra**:
1. Em `11/06/2026`, saiu pelo banco (`ORIGEM = 'XTPG'`) o lote `"PAGAMENTO EXTRA11062026 / BANCO 341 6625 000015241"` a Débito de **`R$ 21.535,85`**.
2. Na conferência manual de fechamento (em `07/07` e `09/07`), a contabilidade identificou que **`R$ 19.887,90`** desse extra referia-se à **Verba `1796` (ILP - Labs)** e lançou ajuste manual (`"BAIXA EM 11/06/26 - LABS - ILP - MS2025 - VERBA 1796"`).
3. A diferença líquida (**`R$ 1.647,95`**) é justamente o **Pagamento Extra de Folha** que subiu no holerite da competência seguinte (Verba `0078 -> 0568/0122`)!

---

## 3. Como a Planilha de Extras que Você Solicitou Vai Plugar Aqui

Assim que o time enviar a **Planilha de Extras**, nosso Motor vai ler apenas **3 colunas** dela (`Filial`, `Data do Pagamento Extra`, `Valor / Competência da Folha`) para cruzar automaticamente com a coluna **`Pagamentos Extras (XTPG)`** da tabela acima:
1. Se o valor do `PAGAMENTO EXTRA<DDMMYYYY>` no Oracle bater com a soma da Planilha de Extras + Desconto `Vb. 0568 / 0122` no mês seguinte ➔ **`CONCILIADO AUTOMÁTICO`**.
2. Se parte do `PAGAMENTO EXTRA` for de outra rubrica (como foi o ILP `Verba 1796` de `R$ 19.887,90`) ou de colaborador desligado ➔ **O Motor já gera o payload de reclassificação para o sistema Kirk sem ninguém fazer lançamento `Manual`!**
