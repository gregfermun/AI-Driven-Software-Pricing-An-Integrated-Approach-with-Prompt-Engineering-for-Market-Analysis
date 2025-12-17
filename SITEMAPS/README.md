# AI-Driven Software Pricing (Sitemap + Prompt Engineering) — Replication Guide

Este repositório documenta um protocolo **reprodutível** para estimar **faixas de valor de mercado** de softwares por **analogia competitiva**, usando **sitemaps oficiais** como proxy de “contorno funcional” e **LLMs** (ChatGPT Thinking, Gemini 3 Pro e DeepSeek-V3.2) com um **prompt padronizado**.

> **Resumo do fluxo**: (1) coletar sitemap oficial → (2) salvar em PDF com metadados → (3) executar 1 chat por software×IA (chats isolados) → (4) extrair funcionalidades (somente PDF) + benchmarking (web) → (5) verificação humana → (6) normalização (base anual + moeda) → (7) indicadores relativos e Wilcoxon.

---

## 1) Pré-requisitos

### Contas e interfaces
- **ChatGPT Thinking** (com upload de PDF e navegação web habilitável).
- **Gemini 3 Pro** (interface web).
- **DeepSeek-V3.2** (interface web).
- Um navegador com suporte a **Imprimir → Salvar como PDF**.

### Ferramentas para análise (opcional, mas recomendado)
- Uma planilha (Excel/Google Sheets/LibreOffice) **ou** Python/R para cálculos e testes estatísticos.
- Python 3.10+ (se for replicar por código):
  - `pandas`, `numpy`, `scipy`

---

## 2) Estrutura recomendada do repositório

Sugerimos padronizar pastas/nomes para facilitar auditoria:

```
.
├─ prompts/
│  ├─ prompt_base_pt.md
│  └─ prompt_base_en.md   (opcional)
├─ data/
│  ├─ sitemaps_pdfs/
│  ├─ sitemaps_metadata/
│  ├─ references_mkt/     (valores de referência públicos/anonimizados)
│  └─ currency_rates/     (cotações usadas na conversão, se aplicável)
├─ runs/
│  ├─ transcripts_anon/
│  ├─ screenshots/
│  └─ session_log.csv
├─ outputs/
│  ├─ raw_tables/         (CSV/ODS)
│  └─ consolidated/       (CSV/ODS)
└─ analysis/
   ├─ indicators.xlsx      (ou .ods/.csv)
   └─ wilcoxon.ipynb       (ou .R)
```

---

## 3) Coleta e atualização dos sitemaps (download → PDF)

### 3.1 Sitemaps utilizados no artigo (URLs oficiais)
- INTEGRA (Rede INTEGRA): `https://redeintegra.mec.gov.br/sitemap.xml`
- Salesforce: `https://www.salesforce.com/br/sitemap.xml`
- HYPE Innovation: `https://www.hypeinnovation.com/sitemap.xml`
- IdeaScale: `https://ideascale.com/post-sitemap.xml`
- Viima / HYPE Boards: `https://www.viima.com/sitemap.xml`
- Qmarkets: `https://www.qmarkets.net/post-sitemap1.xml`

> **Se você for replicar com novos softwares**, substitua apenas as variáveis no prompt (Seção 6) e mantenha o fluxo idêntico.

### 3.2 Como gerar o PDF (procedimento base)
Para cada URL de sitemap:

1. Abra a URL do sitemap no navegador.
2. Use **Imprimir → Salvar como PDF** (garanta que o PDF contém a listagem completa de URLs/descritores).
3. Nomeie o arquivo seguindo a convenção:
   - `sitemap_<SOFTWARE>_YYYY-MM-DD.pdf`
   - Ex.: `sitemap_INTEGRA_2025-08-13.pdf`

### 3.3 Metadados mínimos (obrigatório)
Para cada PDF, registre:
- `captured_at` (data/hora local)
- `sitemap_url` (URL exata)
- `pdf_filename`
- `sha256` (hash do arquivo, recomendado)
- `pages` (número de páginas do PDF)
- `approx_urls` (estimativa do número de URLs)

Guarde esse registro em `data/sitemaps_metadata/metadata.csv` (ou JSON).

> Dica: o hash SHA-256 é importante para demonstrar que *o mesmo insumo* foi usado em diferentes rodadas/LLMs.

---

## 4) Preparação do ambiente experimental

### 4.1 Isolamento dos experimentos (1 chat por combinação)
Crie chats **independentes** para cada combinação:

- 6 softwares × 3 IAs = **18 execuções**
- Em **cada chat**, carregue **apenas 1 PDF** (o sitemap do software-alvo).

Nomeie cada execução com um ID padronizado, por exemplo:
- `RUN_001_INTEGRA_CHATGPT`
- `RUN_002_INTEGRA_GEMINI`
- `RUN_003_INTEGRA_DEEPSEEK`
- ...
- `RUN_018_QMARKETS_DEEPSEEK`

Registre esse ID no `runs/session_log.csv`.

### 4.2 Parâmetros de inferência (documentar como evidência)
Documente (prints + log) os parâmetros utilizados:

**ChatGPT 5.1 Thinking**
- temperatura = 0.2
- top_p = 1.0
- n = 1
- frequency penalty = 0.0
- presence penalty = 0.0

**Gemini 3 Pro (configuração padrão/balanceada)**
- temperatura = 1.0
- top_p = 0.95
- n = 1
- frequency penalty = 0.0
- presence penalty = 0.0

**DeepSeek-V3.2 (configuração padrão/balanceada)**
- temperatura = 0.7
- top_p = 0.9–0.95
- n = 1
- frequency penalty = 0.0
- presence penalty = 0.0

> Observação: interfaces web podem não expor todos os detalhes internos (prompt de sistema, cache, limites exatos de contexto). Por isso, registre tudo o que é **observável**.

---

## 5) Execução do protocolo (LLM) — passo a passo

### 5.1 Etapa A — Extração de funcionalidades (modo “somente arquivo”)
Objetivo: inferir funcionalidades **exclusivamente** do texto do PDF do sitemap.

1. Abra um chat novo (sem histórico).
2. Faça upload do PDF do sitemap do software-alvo.
3. Cole o **prompt padronizado** (Seção 6) preenchendo:
   - `{{NOME_DO_SOFTWARE}}`
   - `{{FOCO_DA_COMPARACAO}}`
   - `{{URL_SITEMAP}}`
4. **Instrução operacional**: nesta etapa, valide que o modelo não está usando fontes externas; se a plataforma permitir, trabalhe “somente arquivo”.

**Saídas esperadas (mínimo):**
- Lista estruturada de funcionalidades / módulos inferidos do sitemap.

### 5.2 Etapa B — Benchmarking competitivo (modo com navegação web, quando aplicável)
Objetivo: identificar concorrentes comparáveis e coletar faixas/modelos de preço em fontes públicas.

1. Habilite navegação web (quando a plataforma oferecer).
2. Reforce no prompt/execução a necessidade de:
   - citar URLs;
   - informar modelo de licenciamento (por usuário, por instância, enterprise etc.);
   - priorizar fontes **oficiais**; usar agregadores apenas quando não houver fonte primária.

**Saídas esperadas (mínimo):**
- Tabela de benchmarking (software, empresa, modelo, preços/faixas, fonte).
- Tabela comparativa de funcionalidades (sitemap vs docs públicas).
- Faixa anual estimada para o software-alvo (com justificativa e âncoras de comparação).
- Lista final de links consultados.

### 5.3 Registro dos artefatos (obrigatório)
Para cada execução (RUN):

- Exporte a transcrição do chat (ou copie para `runs/transcripts_anon/`).
- Salve capturas de tela:
  - modelo/versão exibidos na interface;
  - parâmetros (quando visíveis);
  - blocos de resposta contendo fontes (URLs).
- Preencha o `runs/session_log.csv` com:
  - `run_id`, `software`, `llm`, `date_time`, `pdf_filename`, `model_version_displayed`
  - `web_enabled` (sim/não)
  - `notes` (observações de divergência, travamentos etc.)

---

## 6) Prompt padronizado (template)

Salve este texto em `prompts/prompt_base_pt.md` e use-o **sem alterações**, trocando apenas as variáveis.

```text
Estimar o valor de mercado do software {{NOME_DO_SOFTWARE}}, posicionando-o e comparando-o diretamente com outras soluções e plataformas disponíveis no mercado, com foco em {{FOCO_DA_COMPARACAO}}.

Contexto: A IA atuará como um analista de produtos, com a tarefa de avaliar as funcionalidades do software {{NOME_DO_SOFTWARE}} (disponíveis publicamente através do sitemap) e realizar um benchmarking competitivo. O objetivo final é determinar um valor de mercado estimado para o {{NOME_DO_SOFTWARE}}, baseando-se nos preços e modelos de negócio de softwares concorrentes especializados em {{FOCO_DA_COMPARACAO}}.

Caminhos da IA (Passos a Serem Percorridos):

1) Análise e Extração de Funcionalidades do {{NOME_DO_SOFTWARE}}:
Ação: Analisar o sitemap ({{URL_SITEMAP}}) e identificar especificamente os módulos e funcionalidades que o {{NOME_DO_SOFTWARE}} oferece.
Output: Criar uma lista clara e estruturada das funcionalidades que o {{NOME_DO_SOFTWARE}} oferece, servindo como base para a comparação.

2) Pesquisa de Mercado Focada em Softwares de {{FOCO_DA_COMPARACAO}}:
Ação: Realizar uma pesquisa de mercado para encontrar softwares comerciais (SaaS ou licenciados) concorrentes diretos/análagos.
Termos de busca (exemplos):
- "Software {{FOCO_DA_COMPARACAO}} preço"
- "Plataforma para {{FOCO_DA_COMPARACAO}} custo"
- "Concorrentes {{NOME_DO_SOFTWARE}}"
- "{{FOCO_DA_COMPARACAO}} software pricing"
- "Comparativo {{FOCO_DA_COMPARACAO}} empresarial"

3) Análise Comparativa e Coleta de Dados de Preços:
Ação: Para cada concorrente encontrado, coletar (de fontes públicas) nome, modelo de precificação, valores/faixas e funcionalidades-chave.
Output: Uma tabela de benchmarking comparando o {{NOME_DO_SOFTWARE}} com os softwares encontrados.

4) Tabela Comparativa de Funcionalidades:
Ação: Comparar funcionalidades do {{NOME_DO_SOFTWARE}} (somente sitemap) com concorrentes (documentação pública).
Output: Tabela comparativa clara e organizada.

5) Estimativa de Custo por Analogia de Mercado:
Ação: Formular uma estimativa anual (faixa mínima e máxima) e justificar com âncoras comparativas.

Delimitação do que a IA entregará:
- Lista de funcionalidades (do sitemap)
- Tabela de benchmarking competitivo (com fontes)
- Tabela comparativa de funcionalidades
- Estimativa de valor de mercado (faixa anual) com justificativa
- Lista de links utilizados

A IA NÃO deverá:
- Utilizar Pontos de Função (APF).
- Incluir softwares que não sejam primariamente focados em {{FOCO_DA_COMPARACAO}}.
- Fazer suposições sobre custo de desenvolvimento/infraestrutura.
```

---

## 7) Verificação humana (conferência manual das respostas)

A verificação humana é **parte do protocolo** e não deve ser omitida.

### 7.1 O que verificar
Para cada RUN:

1. **Funcionalidades**: checar se o que foi descrito está de fato presente no sitemap e/ou páginas institucionais relevantes.
2. **Concorrentes**: checar se são realmente comparáveis ao foco do software-alvo.
3. **Preços**: checar:
   - se os valores constam na página acessada;
   - se o modelo de licenciamento está correto (por usuário/instância/enterprise);
   - se a periodicidade (mensal/anual) foi interpretada corretamente.
4. **Fontes**: rejeitar links sem referência primária quando houver fonte oficial disponível.

### 7.2 O que fazer quando houver discrepância
- Registrar a discrepância no `runs/session_log.csv` (campo `notes`).
- **Descartar ou repetir** a rodada.
- Quando repetir, aplicar apenas ajustes *pontuais* de reforço (ex.: “usar somente fontes oficiais” e “obrigatório listar URLs”).

---

## 8) Normalização e indicadores relativos de validação (AI vs MKT)

### 8.1 Padronizações (base comum)
Antes de comparar estimativas com referências:

1. **Faixa → ponto médio**  
   - `AI_mid = (AI_min + AI_max) / 2`
2. **Mensal → anual** (quando aplicável)  
   - `annual = monthly * 12`
3. **Conversão cambial** (quando necessário)  
   - Documente a taxa e a data de referência (salve em `data/currency_rates/`)

### 8.2 Indicadores (por software×IA)
Defina `AI` (valor estimado, já normalizado) e `MKT` (valor de referência, já normalizado) na **mesma moeda e horizonte temporal**.

Calcule:

- Diferença: `diff = AI - MKT`
- Diferença absoluta: `abs_diff = |AI - MKT|`
- Desvio relativo: `rel_diff = (AI - MKT) / MKT`
- Erro percentual absoluto: `APE(%) = (|AI - MKT| / MKT) * 100`

> Estes indicadores podem ser computados em planilha automatizada (linha a linha) e exportados para `outputs/consolidated/validation_indicators.csv`.

### 8.3 Template de planilha (colunas sugeridas)
- `software`, `llm`, `run_id`
- `ai_min`, `ai_max`, `ai_mid`, `ai_currency`, `ai_basis` (usuário/instância/enterprise)
- `mkt_value`, `mkt_currency`, `mkt_basis`, `mkt_source_type`
- `conversion_notes` (mensal→anual, câmbio etc.)
- `diff`, `abs_diff`, `rel_diff`, `ape_percent`

---

## 9) Teste estatístico (Wilcoxon signed-rank) — AI vs MKT

Objetivo: verificar se há diferença sistemática entre `AI` e `MKT` em amostras pareadas (n pequeno).

- Hipótese nula: `mediana(AI - MKT) = 0`
- Hipótese alternativa (bicaudal): `mediana(AI - MKT) ≠ 0`
- Nível de significância: `α = 0.05`

### 9.1 Exemplo em Python (SciPy)

Crie `analysis/wilcoxon.py`:

```python
import pandas as pd
from scipy.stats import wilcoxon

df = pd.read_csv("outputs/consolidated/validation_indicators.csv")

# exigir apenas pares completos
pairs = df.dropna(subset=["ai_mid", "mkt_value"]).copy()

diff = pairs["ai_mid"] - pairs["mkt_value"]

# teste bicaudal
stat, p = wilcoxon(diff, alternative="two-sided", zero_method="wilcox")

print(f"W = {stat:.4f} | p-value = {p:.6f} | n = {len(diff)}")
```

### 9.2 Interpretação
- Se `p < 0.05`: rejeita-se H0 (evidência de diferença sistemática).
- Se `p ≥ 0.05`: não se rejeita H0 (não há evidência suficiente de diferença sistemática).

---

## 10) Replicação técnica (controle de robustez)

Para cada software×IA, execute uma **segunda rodada**:

- Novo chat
- Mesmo PDF
- Mesmo prompt
- Mesma regra de navegação (somente arquivo vs web)

Compare:
- Similaridade das listas de funcionalidades
- Estabilidade das faixas (ordem de grandeza)
- Coincidência das fontes públicas citadas

Registre divergências e como foram tratadas (releitura de sitemap/páginas e repetição/descarto).

---

## 11) Checklist rápido (antes de publicar resultados)

- [ ] PDFs salvos com convenção de nomes e metadados (data/hora + URL + hash)
- [ ] 1 chat por software×IA (sem histórico)
- [ ] Prompt base sem alterações (apenas variáveis)
- [ ] Evidências arquivadas (transcrição + links + screenshots)
- [ ] Verificação humana registrada (OK/erro + decisão: manter/descartar)
- [ ] Normalização (ponto médio, anualização, câmbio documentado)
- [ ] Indicadores relativos calculados
- [ ] Wilcoxon executado sobre pares válidos

---

## 12) Licença e ética

Este repositório deve manter:
- apenas **artefatos não confidenciais**;
- transcrições **anonimizadas** quando necessário;
- referências de mercado públicas ou valores anonimizados, quando houver restrições.

