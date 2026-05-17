# Revisão Integrativa — Sistema de IA para Pesquisa Científica

> **Manual Técnico do Sistema:** Este documento descreve como configurar, operar e manter o pipeline completo de Revisão Integrativa da Literatura — desde o clone inicial até a finalização do artigo.

---

## 🏗️ Arquitetura do Sistema (As Três Camadas)

Antes de começar, entenda que este repositório é composto por **três camadas independentes**, cada uma com seu próprio ciclo de vida e repositório remoto.

```
revisao/                          ← ESTE REPOSITÓRIO (dados e trabalho da revisão)
├── kit/                          ← CAMADA 1: Kit de Agentes (vitalia-agent-kit)
│   └── [agents, skills, ...]        Submodule Git — atualizações independentes
├── .agent/                       ← CAMADA 2: Sistema de Contexto (vitalia-01-context)
│   └── session/                     Submodule Git — memória persistente entre sessões
│       ├── CONTEXT.md               Estado atual da revisão
│       └── SESSION_HISTORY.md       Histórico cronológico
├── inicio/                       ← CAMADA 3: ESTE PROJETO (dados da revisão)
│   ├── criteria_config.template.yaml
│   ├── 00_SUMARIO_EXECUTIVO.md
│   └── ROTEIRO_IA_REVISAO.md
├── amostra/                      ← Pasta criada por você (não versionada)
│   ├── scopus/                      PDFs baixados da plataforma Scopus
│   └── webofscience/                CSVs exportados da Web of Science
└── lotes/                        ← Pool de CSVs brutos para triagem em massa
```

| Camada                | Repositório Remoto                     | Para que serve                                                           |
| --------------------- | -------------------------------------- | ------------------------------------------------------------------------ |
| 🛠️ **Kit de Agentes** | `vitalia-platform/vitalia-agent-kit`   | Agentes, skills, regras e workflows. Compartilhado entre projetos.       |
| 🧠 **Contexto**       | `vitalia-platform/vitalia-01-context`  | Memória persistente da sessão de trabalho da IA entre máquinas.          |
| 📚 **Revisão**        | `vitalia-platform/revisao-integrativa` | Os dados, configurações e artefatos desta revisão científica específica. |

---

## 1. Pré-requisitos

### Ferramentas necessárias

- **Git** 2.28+ — com suporte a submodules
- **Python** 3.10+
- **Chave SSH** configurada com acesso à organização `vitalia-platform` no GitHub
- **Antigravity** (IDE / Gemini CLI) configurada para utilizar os agentes do kit
- **Servidor Local (opcional, mas recomendado):** Stack Docker com Ollama para processamento de lotes massivos (ver `Hardware.md`)

### Gerando sua chave SSH (se ainda não tiver)

```bash
# Gera uma chave ED25519 (recomendado)
ssh-keygen -t ed25519 -C "seu-email@instituicao.br"

# Exibe a chave pública — copie e adicione em github.com/settings/keys
cat ~/.ssh/id_ed25519.pub

# Testa a conexão com o GitHub
ssh -T git@github.com
```

---

## 2. Setup Inicial (Primeira Vez em uma Máquina)

Execute os passos abaixo **em sequência** na sua máquina local.

### 2.1 Clonar o repositório com submodules

```bash
# Clone o repositório principal E os submodules em um único comando
git clone --recurse-submodules git@github.com:vitalia-platform/revisao.git

cd revisao
```

> [!NOTE]
> Se você já clonou sem `--recurse-submodules`, execute:
> `git submodule update --init --recursive`

### 2.2 Instalar o Kit de Agentes

```bash
# Este script cria a pasta .agent/ e configura os symlinks para o kit
bash kit/scripts/install.sh
```

Durante a instalação, o script pedirá a URL do seu repositório de contexto. Informe:

```
git@github.com:vitalia-platform/vitalia-01-context.git
```

### 2.3 Sincronizar o contexto da nuvem

```bash
# Resolve o repositório de sessão e importa o contexto mais recente
bash kit/scripts/session-resolve.sh

# Execute o install.sh novamente para garantir que tudo está ligado
bash kit/scripts/install.sh
```

### 2.4 Verificar a instalação

```bash
# Audita os symlinks e valida o estado do ETag de concorrência
python3 .agent/scripts/validate-kit.py --target .
```

Se tudo estiver correto, você verá o contexto da última sessão disponível e estará pronto para trabalhar.

---

## 3. Iniciando uma Sessão de Trabalho

**Sempre** comece cada sessão com o comando abaixo no seu chat com a IA:

```
/session-start
```

O agente `session-manager` lerá o `CONTEXT.md`, apresentará o estado atual do projeto e o próximo passo prioritário.

---

## 4. Iniciando a Revisão Integrativa (`/integrative-review`)

Este é o ponto de entrada principal do sistema de revisão. Ele abre o **Painel Interativo de Setup** e configura toda a infraestrutura antes de qualquer processamento.

### 4.1 Preparar a pasta de amostra

Antes de chamar o workflow, deposite os dados de calibração:

```bash
mkdir -p amostra/scopus amostra/webofscience lotes saida fichamentos
```

- Em `amostra/scopus/`: coloque os PDFs dos artigos mais relevantes (baixados do portal Scopus).
- Em `amostra/webofscience/`: coloque o CSV de metadados exportado da Web of Science.
- Em `lotes/`: coloque os arquivos `lote_001.csv`, `lote_002.csv`, etc. (os 50.000+ artigos particionados).

### 4.2 Chamar o workflow

```
/integrative-review
```

O agente abrirá o arquivo `review_setup.md` como um painel lateral de formulário. **Preencha todos os campos.** O sistema solicitará:

| Campo                            | O que informar                                            |
| -------------------------------- | --------------------------------------------------------- |
| Título provisório do estudo      | Ex: "Tecnologias Digitais e Exercício Físico (2018-2026)" |
| Autores e filiação institucional | Nomes e instituição (para o artigo final)                 |
| Periódico alvo                   | Ex: _JMIR mHealth and uHealth_                            |
| Recorte temporal                 | Ex: 2018 – 2026                                           |
| Pasta de amostra                 | `amostra/`                                                |
| Pastas por origem                | `amostra/scopus/`, `amostra/webofscience/`                |
| Pool de lotes                    | `lotes/`                                                  |
| Pasta de saída PRISMA            | `saida/`                                                  |
| Pasta de fichamentos             | `fichamentos/`                                            |
| URL da API Ollama local          | Ex: `http://ip-do-servidor:11434`                         |
| Modelo Ollama                    | Ex: `llama3.2:3b`                                         |
| Modelo de embeddings             | Ex: `nomic-embed-text`                                    |
| Tamanho do lote                  | Ex: `1000`                                                |
| Domínios de alto valor           | Ex: "Educação Física Escolar", "Gamificação para Adesão"  |

### 4.3 Análise da Amostra e Critérios

Após o preenchimento, a IA lerá a pasta `amostra/` (PDFs e CSVs), mapeará os temas recorrentes e **sugerirá ativamente** os critérios de Inclusão e Exclusão. Você pode:

- Aprovar as sugestões;
- Modificar os critérios pelo chat;
- Fornecer um artigo de exemplo ("Padrão Ouro") para refinamento.

### 4.4 Geração do arquivo de configuração

Ao aprovar, o agente gerará o `criteria_config.yaml` na raiz do projeto. **Este arquivo é o cérebro de todo o processamento posterior.** Nenhum script ou agente usa valores fixos — tudo é lido daqui.

---

## 5. O Pipeline PRISMA (Fases da Triagem)

Após o setup, o fluxo segue de forma semi-automatizada:

```
Amostra → [IA calibra critérios] → Aprovação Humana
    ↓
Lotes CSV → [Ollama local: triagem por abstract] → LOG_PRISMA_FASE1.csv
    ↓
Revisão Humana da amostra → Aprovação Humana
    ↓
DOIs aprovados → [Script multi-fonte: Unpaywall, PMC, EuropePMC, Crossref] → PDFs
    ↓
PDFs → [IA: leitura integral] → Fichamentos em /fichamentos/
    ↓
Fichamentos → [IA: síntese + categorias emergentes] → Draft do Artigo
    ↓
Draft → [Colegiado: N reuniões de revisão] → Artigo Final
```

> [!IMPORTANT]
> **HITL (Human-in-the-Loop):** O pesquisador humano é obrigatório nos pontos de aprovação do PRISMA. Nenhuma fase avança sem sua validação explícita. Isso garante a rastreabilidade exigida para publicação em periódicos de alto impacto.

---

## 6. Sincronização Entre Máquinas

O sistema foi projetado para trabalho em múltiplos computadores sem conflito.

### Ao encerrar a sessão (em qualquer máquina):

```
/session-end
```

Ou manualmente:

```bash
bash .agent/scripts/session-sync.sh "Mensagem descrevendo o que foi feito"
```

### Ao retomar em outra máquina:

```bash
# 1. Atualiza o repositório principal (dados da revisão)
git pull origin main

# 2. Atualiza o contexto da sessão
bash .agent/scripts/session-sync.sh
```

### Regra de Concorrência (ETags)

O sistema usa um Guardião de Concorrência (`.sync_lock`). Se duas máquinas tentarem sincronizar ao mesmo tempo, a segunda receberá um erro de bloqueio. **Sempre encerre a sessão antes de trocar de máquina.**

---

## 7. Mantendo o Kit de Agentes Atualizado

O Kit de Agentes em `kit/` é um submodule independente que recebe atualizações centrais.

```bash
# Atualiza o kit para a versão mais recente
cd kit && git pull origin main && cd ..

# Reinstala os symlinks após atualizar
bash kit/scripts/install.sh

# Registra a atualização no repositório principal
git add kit
git commit -m "chore(kit): update agent kit to latest version"
git push origin main
```

---

## 8. Troubleshooting

### Contexto não sincroniza (erro de rebase com unstaged changes)

```bash
# Commita tudo localmente primeiro
git add . && git commit -m "wip: salvar estado antes de sincronizar"

# Depois sincroniza
bash .agent/scripts/session-sync.sh "Sua mensagem"
```

### Repositório de contexto corrompido / errado

```bash
# Opção 1: Reconfigura o remote e faz pull
cd .agent/session
git remote set-url origin git@github.com:vitalia-platform/vitalia-01-context.git
git pull origin main --rebase
cd ../..
```

```bash
# Opção 2 (reset total): Remove e reinstala o contexto
rm -rf .agent/session
bash kit/scripts/install.sh
```

### Symlinks do kit quebrados

```bash
python3 .agent/scripts/validate-kit.py --target .
bash kit/scripts/install.sh
```

### API do Ollama não responde

Verifique se o servidor Docker está rodando e se o endereço no `criteria_config.yaml` está correto. Consulte `Hardware.md` para a configuração do servidor local.

---

## 9. Estrutura de Branches

| Branch               | Propósito                                       |
| -------------------- | ----------------------------------------------- |
| `main`               | Estado estável, aprovado pelo colegiado.        |
| `draft/fase-N`       | Rascunhos da fase N da triagem ou do artigo.    |
| `review/colegiado-N` | Branch para revisão e comentários do colegiado. |

---

## 10. Referências

| Documento                    | Localização                                           | Propósito                                                  |
| ---------------------------- | ----------------------------------------------------- | ---------------------------------------------------------- |
| Manual do Kit de Agentes     | `kit/MANUAL.md`                                       | Catálogo completo de agentes, skills e workflows           |
| Sumário Executivo            | `export_novo_repo/00_SUMARIO_EXECUTIVO.md`            | Identidade do estudo, critérios e estado PRISMA            |
| Roteiro de Trabalho da IA    | `export_novo_repo/ROTEIRO_IA_REVISAO.md`              | Instruções detalhadas por fase para os agentes             |
| Template de Configuração     | `export_novo_repo/criteria_config.template.yaml`      | Modelo base para o `criteria_config.yaml`                  |
| ADR-001 (Rigor Metodológico) | `export_novo_repo/ADR-001-Rigor-Revisao-Academica.md` | Decisões arquiteturais que regem a Clean Room              |
| Roteiro de Busca CAPES       | `export_novo_repo/ROTEIRO_BUSCA_CAPES.md`             | Passo a passo para exportar dados via acesso institucional |
| Configuração do Hardware     | `Hardware.md`                                         | Especificações do servidor local de inferência             |

---

> _Este repositório opera sob a metodologia Mendes, Silveira e Galvão (2008) e segue as diretrizes PRISMA para máxima rastreabilidade e publicabilidade._
