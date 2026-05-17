# Sumário Executivo — Revisão Integrativa da Literatura
**Versão:** 1.1 — CLEAN ROOM  
**Data:** Maio de 2026  
**Status:** [AGUARDANDO TRIAGEM]  

---

## Identificação do Estudo

| Campo | Conteúdo |
|---|---|
| **Título provisório** | Uso de Tecnologias Digitais para o Exercício Físico (2018–2026): Uma Revisão Integrativa |
| **Tipo de estudo** | Revisão Integrativa da Literatura |
| **Método** | Mendes, Silveira e Galvão (2008) — 6 etapas |
| **Período de busca** | 2018 – 2026 |
| **Bases de dados** | Web of Science / Scopus (via Portal CAPES/RNP) |
| **Idioma primário** | Inglês (+ Português como complemento) |
| **Periódico alvo** | Internacional (ex: *JMIR mHealth and uHealth*, *Frontiers in Digital Health*) |

---

## Pergunta Norteadora

> *O que se tem produzido academicamente acerca da relação entre exercício físico e tecnologias digitais no período de 2018 a 2026, sobretudo no que se refere às principais áreas e assuntos investigados (engajamento, gamificação, telemetria)?*

Esta revisão se posiciona como **atualização metodológica rigorosa** do estudo de referência:
> Oliveira et al. (2020). *Uso das tecnologias digitais para o exercício físico: uma revisão integrativa.* Conexões: Educ. Fís., Esporte e Saúde, v.18, e020002. Período coberto: 2013–2018.

*Nota: O cenário tecnológico analisado é restrito ao estado da arte atual (2018-2026).*

---

## Configuração Dinâmica do Projeto

> [!IMPORTANT]
> **Zero Hardcoding:** Todos os parâmetros operacionais deste projeto (nomes de pastas, strings de conexão, endpoints, nomes de modelos, critérios de elegibilidade) são definidos e mantidos em `criteria_config.yaml`. O conteúdo desse arquivo é gerado interativamente pelo workflow `/integrative-review` e aprovado pelo pesquisador responsável antes de qualquer processamento em lote.

### Domínios de Alto Valor (Mapeamento de Tendências)
O projeto possui interesse especial no mapeamento de tendências na intersecção de:
- **Educação Física Escolar** (Tecnologias digitais aplicadas ao ambiente escolar — crianças e adolescentes).

Artigos neste domínio receberão classificação `TRENDING_TOPIC` no Log PRISMA, permitindo análise isolada de tendências emergentes.

### Fontes de Dados da Amostra de Calibração
| Origem | Tipo | Pasta |
|---|---|---|
| Web of Science | CSV (metadados + abstracts) | `amostra/webofscience/` |
| Scopus | PDFs (full-text) | `amostra/scopus/` |

A pasta raiz de amostra e todos os demais caminhos são definidos em `criteria_config.yaml`.

---

## Critérios de Elegibilidade

> [!NOTE]
> Os critérios de Inclusão e Exclusão abaixo são propostos como ponto de partida e serão **refinados e aprovados** pelo pesquisador após a análise da amostra de calibração pelo workflow `/integrative-review`.

### Critérios de Inclusão (Preliminares)
- Estudos que investigam a relação direta entre tecnologias digitais e exercício físico (uso, eficácia, adesão).
- Estudos que aplicam tecnologias digitais ao aprimoramento do trabalho de profissionais de Educação Física.
- Estudos no domínio de Educação Física Escolar que utilizem tecnologias digitais.

### Critérios de Exclusão (Preliminares)
- Ênfase em questões empresariais (marketing esportivo, gestão de clubes).
- Estudos com animais ou sem sujeitos humanos (in vitro).
- Mídias tradicionais sem componente digital/interativo (TV, rádio).
- Protocolos de estudo sem resultados publicados.

---

## Estado Atual do Corpus Documental (PRISMA)

| Etapa PRISMA | Quantidade |
|---|---|
| Arquivo Bruto Exportado (WoS/Scopus) | > 50.000 artigos |
| Amostra de Calibração (Fase 0) | A depositar em `amostra/` |
| Após Screening Fase 1 (título + resumo) | A confirmar |
| Após leitura na íntegra (Fase 2) | A confirmar |
| **Artigos incluídos na revisão** | **0 (Estaca Zero)** |
| Fichamentos concluídos | 0 |

---

## Próximos Marcos

| Marco | Responsável | Status |
|---|---|---|
| Depositar amostra em `amostra/webofscience/` e `amostra/scopus/` | Usuário | PENDENTE |
| Executar `/integrative-review` e aprovar painel `review_setup.md` | Usuário + IA | PENDENTE |
| Gerar `criteria_config.yaml` e aprovar critérios | Usuário | PENDENTE |
| Screening PRISMA Fase 1 (lotes de 1.000 via Ollama local) | IA (Ollama local) | AGUARDANDO CONFIG |
| Download PDFs aprovados (multi-fonte: Unpaywall, PMC, EuropePMC) | IA + Usuário | - |
| Fichamentos (leitura integral) | IA + Revisor Humano | - |
| Síntese e Draft do Artigo | IA + Conselho | - |
