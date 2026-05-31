
# Data Pipelines com GenAI

> Construindo um pipeline PySpark **dirigido por especificação** (Spec-Driven Development) com um agente de IA local (Ollama + Aider).

- Author: Prof. Barbosa
- Contact: infobarbosa@gmail.com
- Github: [infobarbosa](https://github.com/infobarbosa)

---
![Data Pipelines com GenAI](./img/geracao-automatica-01.png)

## Objetivo do hands-on

Ao final deste laboratório você terá construído, **sem escrever código manualmente**, um pipeline PySpark que calcula os **Top 10 Clientes** de um e-commerce por volume total de compras — seguindo os princípios de **Clean Architecture** e **Spec-Driven Development (SDD)**.

A ideia central: você **não vai prompar o agente passo a passo**. Você vai entregar a ele uma **especificação** (o arquivo `AGENTS.md`) que descreve *o que* construir e *sob quais princípios*, e o agente implementa a partir dela. A spec é a **fonte da verdade**; o código é uma consequência.

### O que você vai aprender

- O que é **Spec-Driven Development** e por que ele muda a forma de trabalhar com agentes de IA.
- Como rodar um **agente de IA local** (Aider + Ollama) sem depender de APIs pagas.
- Como **operar** o agente contra uma spec: planejar → implementar → revisar → iterar.
- Como **validar** o que o agente gerou (lint, testes, empacotamento) e por que a **revisão humana** continua indispensável.

### Pré-requisitos

- Python 3.10+
- Git
- Acesso a um terminal Linux/macOS (ou **AWS Cloud9** — ver nota no setup)
- Noções de PySpark e linha de comando

> **Trilha do laboratório:** Conceitos (Parte 1) → Setup (Parte 2) → Lab Spec-Driven (Parte 3) → Validação (Parte 4). A **Parte 5** é uma trilha avançada opcional (refatorar a spec em cadeia); o **Apêndice** é leitura complementar.

---

## Parte 1 — Conceitos essenciais

### 1.1 GenAI na engenharia de dados (em 1 minuto)

A IA Generativa cria conteúdo novo — inclusive **código**. Para o engenheiro de dados, isso significa transformar descrições em linguagem natural em scripts PySpark, testes, configurações e documentação. Estudos de mercado (Bain, Gartner) apontam ganhos de produtividade de **20%–35%** em tarefas de codificação. O ponto de virada não é "a IA escreve código", e sim **como você a instrui**.

### 1.2 O que é Spec-Driven Development (SDD)

No fluxo tradicional com IA, o desenvolvedor conversa com o agente em prompts soltos e ad-hoc. O problema: o conhecimento do projeto fica espalhado no histórico de chat, é difícil de reproduzir e não sobrevive entre sessões.

No **Spec-Driven Development**, invertemos a ordem:

| Ad-hoc (prompt a prompt) | Spec-Driven (SDD) |
| --- | --- |
| Conhecimento no histórico de chat | Conhecimento numa **spec versionada** |
| Difícil de reproduzir | Reprodutível por qualquer pessoa |
| Agente "esquece" o contexto | Spec é recarregada a cada sessão |
| Decisões implícitas | Princípios e DoD **explícitos** |

A **spec** (no nosso caso, o arquivo **`AGENTS.md`**) descreve:
- a **persona** que o agente deve assumir,
- os **princípios arquiteturais** mandatórios,
- a **estrutura de pastas** esperada,
- os **requisitos** de implementação,
- a **Definição de Pronto (DoD)**.

O agente lê essa spec e produz o projeto. Quando algo precisa mudar, **você edita a spec** e pede o incremento — não reescreve prompts do zero. É exatamente o ciclo que praticaremos na Parte 3.

### 1.3 Engenharia de prompt aplicada ao SDD

Mesmo no SDD, saber escrever boas instruções importa. As técnicas que mais pesam:

- **Persona:** dizer *quem* o agente deve ser ("Engenheiro de Dados Sênior, especialista em Spark e Clean Architecture") restringe o espaço de respostas e ajusta o nível técnico.
- **Contexto e schema:** nunca peça uma transformação sem informar o **schema** e uma **amostra** dos dados.
- **Restrições explícitas:** declare as regras do jogo (ex.: "sem UDFs Python; use funções nativas do Spark; inclua type hints e docstrings").
- **Formato de saída:** diga *como* quer a resposta (arquivo `.py`, bloco de teste `pytest`, etc.).
- **Cadeia de pensamento (CoT):** para problemas complexos, peça ao agente para "pensar passo a passo" antes da resposta final.

No SDD, a maior parte dessas técnicas já vive **dentro do `AGENTS.md`** — por isso a spec é tão poderosa.

---

## Parte 2 — Setup do ambiente

> **AWS Cloud9:** se estiver usando Cloud9, prepare o ambiente com este [tutorial de Cloud9](https://github.com/infobarbosa/data-engineering-cloud9) antes de continuar.

### 2.1 Instalar o Ollama

O Ollama executa modelos de linguagem **localmente**, sem custo de API.

```sh
curl -fsSL https://ollama.com/install.sh | sh
```

### 2.2 Login no Ollama

```sh
ollama login
```

### 2.3 Verificar a conexão com o Ollama

O Aider usará a API local do Ollama. Garanta que ela responde:

```sh
curl http://localhost:11434/api/tags
```

### 2.4 Clonar o projeto base

```sh
git clone https://github.com/infobarbosa/data-eng-genai-demo
```

```sh
cd data-eng-genai-demo
```

Este projeto base já contém a spec **`AGENTS.md`** que dirigirá o agente — você vai conhecê-la em detalhe na Parte 3.

### 2.5 Criar o ambiente virtual e dependências

```sh
python3 -m venv .venv
```

```sh
source .venv/bin/activate
```

```sh
python -m pip install --upgrade pip
```

Crie o `requirements.txt`:

```sh
touch ./requirements.txt
```

Conteúdo do `requirements.txt`:

```text
pyspark==4.1.1
pyyaml==6.0.3
pytest==9.0.3
ruff==0.12.9
black==25.1.0
build==1.3.0
```

Instale:

```sh
pip install -r ./requirements.txt
```

### 2.6 Instalar o Aider

O **Aider** é o agente de IA de linha de comando que conversa com o Ollama e edita seu repositório (inclusive fazendo commits automáticos).

```sh
python -m pip install aider-install
```

```sh
aider-install
```

### 2.7 Baixar os datasets

```sh
mkdir -p ./data/{input,output}
```

**Clientes** (JSON comprimido):

```sh
git clone https://github.com/infobarbosa/dataset-json-clientes ./data/input/dataset-json-clientes
```

```sh
zcat ./data/input/dataset-json-clientes/data/clientes.json.gz | head -5
```

**Pedidos** (CSV comprimido):

```sh
git clone https://github.com/infobarbosa/datasets-csv-pedidos ./data/input/datasets-csv-pedidos
```

```sh
zcat ./data/input/datasets-csv-pedidos/data/pedidos/pedidos-2026-01.csv.gz | head -5
```

#### Anatomia dos dados

**`clientes.json.gz`** — um objeto JSON por linha:

```json
{"id": 1, "nome": "Isabel Abreu", "data_nasc": "1982-10-26", "cpf": "512.084.739-05", "email": "isabel.abreu@outlook.com", "interesses": ["Filmes"], "carteira_investimentos": {"FIIs": 11533.69, "CDB": 26677.01}}
```

**`pedidos-2026-01.csv.gz`** — separador `;`, com header:

```text
ID_PEDIDO;PRODUTO;VALOR_UNITARIO;QUANTIDADE;DATA_CRIACAO;UF;ID_CLIENTE
f198e8f7-033d-414d-b032-20975e84edde;LIQUIDIFICADOR;300.0;1;2026-01-05T18:36:28;MG;8409
97969db5-9304-4b80-b19e-3a9d60ce6520;CELULAR;1000.0;3;2026-01-01T11:58:48;DF;934
```

> O cálculo do Top 10 usa essencialmente `VALOR_UNITARIO * QUANTIDADE` agregado por `ID_CLIENTE`, cruzado com o nome do cliente.

---

## Parte 3 — Laboratório Spec-Driven

Esta é a parte central. Aqui você opera o agente **contra a spec**, em quatro movimentos: **conhecer a spec → planejar → implementar → iterar**.

### 3.1 Conheça a spec (`AGENTS.md`)

Abra e leia o `AGENTS.md` na raiz do projeto. Ele é o **contrato** que o agente vai cumprir. Suas seções:

| Seção da spec | O que define |
| --- | --- |
| **1. Persona e Contexto** | Quem o agente é (Eng. de Dados Sênior, Spark + Clean Architecture) e o objetivo (Top 10 Clientes). |
| **2. Princípios Arquiteturais** | POO, Clean Architecture, Injeção de Dependência, Config-Driven (nada hardcoded). |
| **3. Estrutura de Pastas** | `config/`, `src/` (com `core/`, `utils/`, `data_io/`, `transforms/`, `jobs/`, `main.py`), `tests/`, `pyproject.toml`, `Makefile`. |
| **4. Requisitos de Implementação** | Config em `config/config.yaml`; I/O por IDs lógicos; transformações puras e testáveis. |
| **5. Qualidade e Automação** | `tests/test_vendas_transforms.py`; `Makefile` com `lint`, `test`, `package`. |
| **6. Arquivos de Exemplo** | Os datasets `clientes` e `pedidos` (os mesmos da Parte 2). |
| **7. Definição de Pronto (DoP)** | Pipeline roda e salva o ranking em `/data/output/top_10_clientes`. |

> **Por que isso importa:** repare que a spec **não diz como escrever cada linha de código**. Ela define *princípios* e *resultado esperado*. O agente preenche os detalhes. Esse é o coração do SDD.

### 3.2 Inicialize o agente Aider

```sh
aider --model ollama/gemma4:31b-cloud
```

Quando o chat do Aider abrir, adicione a spec ao contexto da conversa:

```text
/add AGENTS.md
```

### 3.3 Peça o plano (planejar antes de implementar)

Antes de deixar o agente escrever qualquer arquivo, peça que ele exponha o **plano de trabalho**. Dentro do chat do Aider:

```text
Verifique o conteúdo de AGENTS.md e me mostre o seu plano de trabalho para implementação.
```

**Revise o plano criticamente.** Ele cobre todos os módulos da estrutura esperada? Respeita a Clean Architecture? Vai criar os testes? Se algo estiver faltando ou divergente, **corrija pela conversa** antes de autorizar a implementação. Esse momento de revisão é o que separa "usar IA" de "engenheirar com IA".

### 3.4 Deixe o agente implementar

Com o plano aprovado, autorize a implementação. O Aider vai criar os arquivos e, a cada mudança, **fazer commits automáticos** no repositório. Acompanhe os diffs que ele apresenta.

Quando ele terminar, saia do chat:

```text
/exit
```

### 3.5 Rode o pipeline

```sh
export PYTHONPATH=$(pwd)
```

```sh
spark-submit ./src/main.py
```

Confira a saída em `./data/output/top_10_clientes`. Esse é o critério da **Definição de Pronto** da spec.

> **Quando der erro (e vai dar):** se o `spark-submit` falhar, não decifre o stack trace sozinho — entregue-o ao agente. Volte ao Aider e use um prompt assim:
>
> ```text
> Meu spark-submit falhou com o stack trace abaixo. Analise-o e me dê as 3 causas
> mais prováveis e como corrigir cada uma, respeitando os princípios do AGENTS.md.
> [COLE O STACK TRACE COMPLETO]
> ```

### 3.6 Itere a spec (o ciclo SDD na prática)

A spec **não menciona empacotamento** (gerar um `.whl` distribuível). Em vez de improvisar prompts, vamos **evoluir a spec** e pedir o incremento ao agente.

Volte ao Aider:

```sh
aider --model ollama/gemma4:31b-cloud
```

```text
/add AGENTS.md
```

```text
Elabore os artefatos para empacotamento do projeto (pyproject.toml e os targets de
Makefile para build do .whl em dist/), mantendo os princípios já definidos no AGENTS.md.
```

> **Reflexão:** note o padrão — quando o requisito muda, a spec evolui e o agente reimplementa de forma consistente. Esse é o loop fundamental do Spec-Driven Development.

---

## Parte 4 — Validação e qualidade

Código gerado por IA **precisa ser validado**. O `Makefile` que o agente criou expõe os comandos:

```sh
make lint
```
Roda `black` e `flake8` — formatação e estilo.

```sh
make test
```
Roda `pytest` — valida a lógica de transformação (agregação e ranking) com dados sintéticos, **sem precisar de cluster Spark**.

```sh
make package
```
Gera o `.whl` em `dist/` — o artefato distribuível.

### A revisão humana é indispensável

Os testes verificam **correção da lógica**, não **correção do produto**. Antes de considerar a tarefa concluída, revise você mesmo:

- A separação de camadas (I/O × transformação × orquestração) foi respeitada?
- Há valores "hardcoded" que deveriam estar na configuração?
- Os testes cobrem casos de borda (cliente sem pedidos, empate no ranking)?
- O resultado em `/data/output/top_10_clientes` faz sentido com os dados de entrada?

> O agente acelera; **você responde** pela qualidade.

> **Funcionou? Ótimo — mas olhe de novo para o `AGENTS.md`.** Num único arquivo convivem a intenção de negócio, a regra do "top 10", o schema dos dados e a arquitetura de software. São **quatro vozes diferentes** comprimidas numa só. No mundo real, cada uma nasce de um ator distinto. Pronto para o próximo nível? Siga para a Parte 5.

---

## Parte 5 — Trilha avançada (opcional): refatorando a spec

> Esta parte é para quem já completou o lab e quer ver como o SDD funciona no "mundo perfeito". Você vai **refatorar a especificação como se refatora código**: quebrando o `AGENTS.md` monolítico em uma cadeia de specs com responsabilidade única — sem nunca quebrar o pipeline.

### 5.1 Por que refatorar uma spec que já funciona?

O `AGENTS.md` atual é a "god class" das especificações: faz tudo, mistura tudo. Na vida real, a jornada de *"a área de negócios quer saber os top 10 clientes"* até um pipeline rodando passa por vários atores, cada um produzindo seu próprio artefato:

| Ator | Artefato (spec) | Responde a... |
| --- | --- | --- |
| **Negócio** | `specs/01-brief.md` | *Por quê?* Para qual decisão isso serve? |
| **Analista de negócios** | `specs/02-requisitos.md` | *O quê?* Definição precisa + critérios de aceite |
| **Analista de dados** | `specs/03-contrato-dados.md` | *Com quais dados?* Schema, grão, métrica |
| **Engenheiro de dados** | `AGENTS.md` (enxuto) | *Como construir bem?* Arquitetura, testes, build |

A regra de ouro do refactoring vale aqui: **cada movimento preserva o comportamento** (o pipeline continua buildando e passando nos testes), mas melhora a estrutura (separa as preocupações em arquivos rastreáveis).

### 5.2 Movimento 1 — Extrair a intenção de negócio

Crie `specs/01-brief.md`. Tire o *porquê* da cabeça do `AGENTS.md` e dê a ele um dono:

```markdown
# Brief — Top 10 Clientes
**Dono:** Área de Negócios

## Contexto
Nosso e-commerce suspeita que a receita está concentrada em poucos clientes.

## Pergunta de negócio
Quais são os 10 clientes que mais compram conosco?

## Decisão que isso habilita
Priorizar relacionamento e retenção dos clientes de maior valor.

## Fora de escopo
Segmentação, previsão de churn, análise de cesta. (Por enquanto.)
```

### 5.3 Movimento 2 — Extrair requisitos e critérios de aceite

Crie `specs/02-requisitos.md`. Aqui o "top 10" deixa de ser vago e ganha **critérios de aceite testáveis**:

```markdown
# Requisitos — Top 10 Clientes
**Dono:** Analista de Negócios — deriva de [01-brief.md]

## Definição
"Top 10" = os 10 clientes com maior **valor total de compras** no período disponível.
valor_total = Σ (valor_unitário × quantidade) de todos os pedidos do cliente.

## Saída esperada
Ranking com: id_cliente, nome_cliente, valor_total — ordenado desc, 10 linhas.

## Critérios de aceite (viram testes!)
- CA1: cliente sem pedidos NÃO aparece no ranking.
- CA2: empate em valor_total → ordenação determinística (desempate por id_cliente).
- CA3: exatamente 10 linhas quando há ≥ 10 clientes com pedidos.
- CA4: valor_total confere com Σ(valor_unitário × quantidade).
```

> **Gancho com a Parte 4:** esses CAs são *exatamente* as asserções do `pytest`. É aqui que a intenção de negócio se conecta ao teste automatizado — o ciclo intenção → spec → teste → código se fecha.

### 5.4 Movimento 3 — Extrair o contrato de dados

Crie `specs/03-contrato-dados.md`. O `AGENTS.md` para de embutir layout de dataset:

```markdown
# Contrato de Dados — Top 10 Clientes
**Dono:** Analista de Dados — deriva de [02-requisitos.md]

## Fontes
- clientes: JSON.gz — grão: 1 linha por cliente
- pedidos:  CSV.gz (sep ";") — grão: 1 linha por item de pedido

## Schema relevante
- clientes: id (long), nome (string)
- pedidos:  valor_unitario (float), quantidade (long), id_cliente (long)

## Junção e métrica
- join: pedidos.id_cliente = clientes.id
- valor_total_cliente = SUM(valor_unitario * quantidade) GROUP BY id_cliente

## Qualidade
- pedido com id_cliente sem correspondência em clientes → descartado (atende CA1)
```

### 5.5 Movimento 4 — O que sobra é engenharia pura

Edite o `AGENTS.md`: remova intenção, regra de negócio e schema (agora moram nos specs) e **referencie** a cadeia. O que permanece é a *constituição de engenharia* — uma voz só:

```markdown
# AGENTS.md — Constituição de Engenharia
Implementa o pipeline definido em specs/01-brief.md, specs/02-requisitos.md
e specs/03-contrato-dados.md.

## Persona
Engenheiro de Dados Sênior — Spark + Clean Architecture.

## Princípios (inalterados)
POO, Clean Architecture, Injeção de Dependência, Config-Driven.

## Estrutura, testes e empacotamento
(como antes — arquitetura, src/, tests/, Makefile, pyproject.toml)
```

Repare: o `AGENTS.md` ficou **menor e mais coeso**. Cada detalhe de negócio/dado agora tem um lugar rastreável.

### 5.6 Movimento 5 — O payoff: a cascata

Aqui você sente *por que* tudo isso valeu a pena. Com a cadeia operável, dê ao agente o contexto completo:

```text
/add AGENTS.md
/add specs/02-requisitos.md
/add specs/03-contrato-dados.md
```

Agora **mude um requisito lá no topo** — por exemplo, troque "top 10" por "top 20", ou "maior valor total" por "maior número de pedidos". Edite apenas `specs/02-requisitos.md` (e o `03` se a métrica mudar) e peça:

```text
Os requisitos em specs/02-requisitos.md mudaram. Reconcilie a implementação e os
testes (asserções dos critérios de aceite) com a cadeia de specs atualizada.
```

Observe a **cascata**: requisito → contrato → código → testes, tudo reconciliado a partir de *uma* edição rastreável no topo — em vez de caçar regras espalhadas num arquivo monolítico. **Isso** é Spec-Driven Development no mundo perfeito.

> **Reflexão final:** você refatorou não o código, mas o *conhecimento* sobre o problema. A spec virou uma arquitetura limpa — com camadas, donos e rastreabilidade — exatamente como o software que ela gera.

---

## Apêndice — outras ferramentas de agente

O fluxo deste lab usa **Aider + Ollama** (local, gratuito). Os mesmos princípios de SDD se aplicam a outras ferramentas — vale conhecer:

1. **VSCode** — autocomplete
2. **GitHub Copilot**
3. **Antigravity**
4. **Codex**
5. **Devin**
6. **Windsurf**
7. **Gemini CLI**

A diferença está na interface; a disciplina — **spec como fonte da verdade, plano antes de implementar, revisão humana depois** — permanece.

---

## Parabéns

Você dominou um fluxo moderno de engenharia de dados: do PySpark e Spark SQL ao **Spec-Driven Development** com um agente de IA local. Sua ferramenta mais estratégica agora não é só escrever código — é **escrever boas especificações** e **revisar criticamente** o que o agente produz.

Bons projetos pra você! ;)
