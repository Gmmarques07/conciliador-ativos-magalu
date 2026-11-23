# 📘 Manual Oficial de Contabilização e Conciliação de Folha de Pagamento — Magalu

> **Documento Oficial Corporativo de Contas Contábeis e Processos**
> * **Preparado por:** Sóstenes C. de Paula
> * **Validado por:** Gustavo Ribeiro Costa
> * **Revisado por:** Karina Aparecida da Silva
> * **Aprovado por:** Renato Andrade
> * **Data de Homologação:** 13/2026

Este manual documenta a mecânica contábil exata, as verbas do sistema **APDATA**, os relatórios operacionais auxiliares, os lançamentos contábeis no **Oracle EBS (`mlpebs`)** e as regras de prova matemática utilizadas pelo **CSC Magazine Luiza**.

---

## 1. Matriz Consolidada de Contas, Verbas APDATA e Lançamentos Contábeis

| Conta Contábil (10d) | Descrição Oficial | Grupo Contábil | Natureza | Verbas APDATA Envolvidas | Relatório Auxiliar APDATA | Lançamento Contábil no Oracle EBS |
| :--- | :--- | :---: | :---: | :--- | :--- | :--- |
| **`2103010001`** | `SALARIOS E ORDENADOS A PAGAR` | Passivo Circulante | Credora | **Vb. 0001** (Salário)<br>**Vb. 0500** (INSS)<br>**Vb. 0078** (Adto Salário Extra inicial)<br>**Vb. 0360** (Ex: Dif. Licença-Paternidade)<br>**Vb. 0568 / 0122** (Desconto metodológico de Extra) | **Resumo da Folha / Líquidos de Folha** + **3 Planilhas de Extras** (Remuneração, Ponto, Folha) + **Retornos até Dia 27** | **Proventos:** D `3301010001` / C `2103010001` (`APDATA`)<br>**Descontos:** D `2103010001` / C `2103020001` (`APDATA`)<br>**Liquidação Líquido:** D `2103010001` / C `1101020035` (`XTPG` / `Finet`)<br>**Exceção Estagiários Loja:** D `2103010001` / C Caixa (autorizado por Honório) |
| **`2103010002`** | `HONORÁRIOS ADMINISTRACAO PAGAR` | Passivo Circulante | Credora | **Vb. 0002** (Pró-labore)<br>**Vb. 0502** (INSS Pró-labore) | **Resumo de Pró-labore** | **Provisão:** D `3301010004` / C `2103010002` (`APDATA`)<br>**Descontos:** D `2103010002` / C `2103020001` (`APDATA`)<br>**Liquidação Líquido:** D `2103010002` / C `1101020041` (`XTPG`) |
| **`2103010003`** | `RESCISOES A PAGAR` | Passivo Circulante | Credora | **Vb. 0502** (Líquido Rescisão TRCT) | **Relatório de Rescisões Líquidas** | **Reconhecimento TRCT:** D `2103010001` / C `2103010003` (`APDATA`)<br>**Liquidação (até 10 dias):** D `2103010003` / C `1101020041` (`XTPG`) |
| **`2103040001`** | `PROVISAO FERIAS` | Passivo Circulante | Credora | **Vb. 1202** (Constituição 1/12+1/3)<br>**Vb. 0150, 0180** (Baixa gozo) | **Relatório de Provisão de Férias Acumuladas** | **Constituição:** D `3301010012` / C `2103040001` (`APDATA`)<br>**Baixa no Gozo:** D `2103040001` / C `2103010001` (`APDATA`) |
| **`2103040002`** | `PROVISAO 13º SALARIO` | Passivo Circulante | Credora | **Vb. 1212** (Constituição 1/12)<br>**Vb. 0012, 0175** (Baixa parcelas) | **Relatório de Provisão de 13º Salário** | **Constituição:** D `3301010013` / C `2103040002` (`APDATA`)<br>**Baixa Parcelas:** D `2103040002` / C `2103010001` (`APDATA`) *(Zera em Dezembro)* |
| **`2103040003`** | `PROVISAO INSS S/ FERIAS` | Passivo Circulante | Credora | **Vb. 1242** (Encargo patronal)<br>**Vb. 1243** (Baixa no gozo) | **Relatório de Encargos sobre Provisão de Férias** | **Constituição:** D `3301080003` / C `2103040003` (`APDATA`)<br>**Baixa no Gozo:** D `2103040003` / C `3301080001` ou `2103020001` (`APDATA`) |
| **`2103040004`** | `PROVISAO INSS S/ 13º SALARIO` | Passivo Circulante | Credora | **Vb. 1252** (Encargo patronal)<br>**Vb. 1253** (Baixa Dezembro) | **Relatório do APDATA de encargos sobre 13º** | **Constituição:** D `3301080002` / C `2103040004` (`APDATA`)<br>**Baixa Dezembro:** D `2103040004` / C `3301080001` (`APDATA`) *(Zera em Dezembro)* |
| **`2103040005`** | `PROVISAO FGTS S/ FERIAS` | Passivo Circulante | Credora | **Vb. 1222** (8% legal s/ férias)<br>**Vb. 1223** (Baixa gozo/rescisão) | **Relatório de Encargos FGTS s/ Férias** | **Constituição:** D `3301080006` / C `2103040005` (`APDATA`)<br>**Baixa no Gozo:** D `2103040005` / C `3301080004` (`APDATA`) |
| **`2103040006`** | `PROVISAO FGTS S/ 13º SALARIO` | Passivo Circulante | Credora | **Vb. 1232** (8% legal s/ 13º)<br>**Vb. 1233** (Baixa parcelas) | **Relatório de encargos FGTS s/ 13º** | **Constituição:** D `3301080005` / C `2103040006` (`APDATA`)<br>**Baixa Parcelas:** D `2103040006` / C `3301080004` (`APDATA`) *(Zera em Dezembro)* |
| **`1107010001`** | `ANTECIPACOES SALARIAIS` | Ativo Circulante | Devedora | **Vb. 0520** (Desconto vale quinzenal)<br>**Vb. 0240** (Líquido negativo Mês 1)<br>**Vb. 0640** (Desconto negativo Mês 2) | **Relatório de Adiantamentos / Papel de Trabalho de Estouro de Mês** | **Pagamento Vale:** D `1107010001` / C Banco (`XTPG`)<br>**Desconto Folha:** D `2103010001` / C `1107010001` (`APDATA`)<br>**Estouro Mês 1:** D `1107010001` / C `2103010001` (`APDATA`)<br>**Compensação Mês 2:** D `2103010001` / C `1107010001` (`APDATA`) |
| **`1107010002`** | `ANTECIPACOES 13º SALARIO` | Ativo Circulante | Devedora | **Vb. 0069** (Adto Fixo)<br>**Vb. 0076** (Adto Variáveis)<br>**Vb. 0521** (Baixa Vb. 0069)<br>**Vb. 0569** (Baixa Vb. 0076)<br>**Vb. 1656** (Baixa XTPG Banco)<br>**Vb. 0242/2652** x **Vb. 0642** | **Relatório Analítico por Colaborador** | **Geração Folha:** D `1107010002` / C `2103010001` (`APDATA`)<br>**Geração Banco:** D `1107010002` / C Banco (`XTPG`)<br>**Baixa 2ª Parcela (Dezembro):** D `2103010001` / C `1107010002` (`APDATA`) |
| **`2104010008`** | `CONTRIBUICAO CONFEDERATIVA RECOLHER` | Passivo Circulante | Credora | Descontos sindicais em folha | **Relatório de Retenções Sindicais** | **Retenção Folha:** D `2103010001` / C `2104010008` (`APDATA`)<br>**Repasse Guia:** D `2104010008` / C `1101020001` (Banco C/C Movimento) |
| **`2104010009`** | `CONTRIBUICAO ASSISTENCIAL RECOLHER` | Passivo Circulante | Credora | Descontos assistenciais sindicais | **Relatório de Retenções Sindicais** | **Retenção Folha:** D `2103010001` / C `2104010009` (`APDATA`)<br>**Repasse Guia:** D `2104010009` / C `1101020001` (Banco C/C Movimento) |

---

## 2. Contas Bancárias Oficiais do Magalu Mapeadas no Processo

Conforme homologado no manual, os pagamentos financeiros de folha via **`XTPG`** utilizam as seguintes contas correntes:

* **`1101020035` — BANCO ITAU - C/C 415-3:** Pagamento do valor líquido da folha de pagamento geral aos colaboradores.
* **`1101020041` — BANCO ITAU - C/C 219:** Quitação bancária de Honorários de Diretoria (Pró-labore) e Termos de Rescisão de Contrato de Trabalho (TRCT).
* **`1101020001` — BANCO CONTA MOVIMENTO:** Quitação de repasses e guias sindicais (Confederativa e Assistencial).

---

## 3. Regras Matemáticas de Auditoria Automatizada no Motor

Com base nas definições dos validadores da Controladoria (Gustavo Ribeiro Costa e Karina Aparecida da Silva), o motor de conciliação executa as seguintes **provas reais automáticas**:

### Prova 1: Provisão de FGTS (Férias e 13º Salário)
$$\text{Saldo Conta } 2103040005 = \text{Saldo Conta } 2103040001 \times 8\% \quad (\text{tolerância para Aprendizes a } 2\%)$$
$$\text{Saldo Conta } 2103040006 = \text{Saldo Conta } 2103040002 \times 8\%$$

### Prova 2: Provisão de INSS s/ Férias
$$\text{Saldo Conta } 2103040003 = \text{Saldo Conta } 2103040001 \times (\% \text{ INSS Patronal Efetivo com FAP/RAT})$$

### Prova 3: Batimento de Saldo Negativo (Estouro de Mês)
O saldo da conta `1107010001` no encerramento do Mês 1 é composto exclusivamente pelos valores da **Verba 0240**. No Mês 2, o motor valida o batimento de baixa:
$$\sum \text{Verba 0240 (Mês 1)} = \sum \text{Verba 0640 (Mês 2)}$$
*Divergências indicam geralmente transferência de funcionários entre filiais/empresas do grupo Magalu.*

### Prova 4: Batimento por Pares na Antecipação de 13º Salário (`1107010002`)
A conciliação deve checar o fechamento estrito dos seguintes pares contábeis:
* $\text{Débitos da Verba } 0069 \iff \text{Créditos da Verba } 0521$
* $\text{Débitos da Verba } 0076 \iff \text{Créditos da Verba } 0569$
* $\text{Débitos pagos via Banco (XTPG)} \iff \text{Créditos da Verba } 1656$
* $\text{Débitos da Verba } 0242 / 2652 \iff \text{Créditos da Verba } 0642$

### Prova 5: Zeramento Obrigatório de Contas em Dezembro
As seguintes contas patrimoniais devem **zerar no final do exercício civil (Dezembro)**:
* `2103040002` (PROVISAO 13º SALARIO) — exceto afastados;
* `2103040004` (PROVISAO INSS S/ 13º SALARIO) — guia DCTFWeb com vencimento em 20/12;
* `2103040006` (PROVISAO FGTS S/ 13º SALARIO);
* `1107010002` (ANTECIPACOES 13º SALARIO) — todos os adiantamentos baixados na 2ª parcela.

### Prova 6: Reconciliação de Pagamentos Extras Fora de Ciclo na Conta `2103010001` (Alinhamento 13/11/2026)
Conforme detalhado por **Mateus Facioli** no 1º Alinhamento de Salários e Ordenados, os **Pagamentos Extras** (ex.: comissões esquecidas, diferenças de licença) saem pelo banco (`XTPG` / `Finet` — Itaú C/C `415` / `1101020035`) no mês $M$, mas sobem na competência da folha do mês subsequente ($M+1$):
$$\text{Saldo Suporte } 2103010001_{(M)} = \text{Líquido Folha APDATA}_{(M)} - \text{Extras Pagos via XTPG}_{(M \to M+1)} + \text{Retornos Bancários até Dia 27}_{(M+1)}$$
* **Mecânica no APDATA:** O pagamento extra inicia na **Verba `0078`** (Adiantamento de Salário, não contabilizada diretamente) e, no fechamento da folha, converte-se na verba definitiva (ex.: **Verba `0360`** — Diferença de Licença-Paternidade) pareada com o desconto na **Verba `0568`** (ou **Verba `0122`**) para evitar duplicidade.
* **Travas de Auditoria no Motor:**
  1. Verificar se todo extra com **Verba `0078`** possui o par de desconto **`0568` / `0122`** (atenção especial a colaboradores desligados antes do processamento final);
  2. Garantir que extras de **13º Salário** não caiam em `2103010001` (devem ir para `1107010002`);
  3. Validar que 100% das filiais com movimentação no APDATA estejam parametrizadas nas tabelas temporárias do **ODI (`CTR`)** antes da geração do diário contábil (evitando o gap ocorrido nas duas filiais da **Magalog**).
