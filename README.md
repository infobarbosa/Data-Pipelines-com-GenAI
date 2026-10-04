# Data Pipelines com GenAI

> Construindo um pipeline PySpark **dirigido por especificação** (Spec-Driven Development) com um agente de IA local (OpenCode + Ollama).

- Author: Prof. Barbosa
- Contact: infobarbosa@gmail.com
- Github: [infobarbosa](https://github.com/infobarbosa)

---
![Data Pipelines com GenAI](./img/geracao-automatica-01.png)

## Objetivo do hands-on

Ao final deste laboratório você terá construído, **sem escrever código manualmente**, um pipeline PySpark que calcula os **Top 10 Clientes** de um e-commerce por volume total de compras — seguindo os princípios de **Clean Architecture**, **Clean Code** e **Spec-Driven Development (SDD)**.

A jornada deste laboratório é **incremental**:
1. Começamos com um **prompt ad-hoc inicial** como linha de base para observar as limitações de gerar código sem restrições: scripts monolíticos, caminhos hardcoded e, principalmente, **regras de negócio ambíguas e não-determinísticas**.
2. Em seguida, praticamos o **Shift-Left**: formalizamos as regras de negócio e contratos de dados antes de codificar (`AGENTS.md` v1).
3. Elevamos a régua da engenharia de software adicionando **Clean Architecture e POO** (`AGENTS.md` v2).
4. Fechamos o ciclo de confiabilidade transformando regras de negócio em **testes automatizados com PyTest** e empacotamento com **Makefile** (`AGENTS.md` v3).

A spec (`AGENTS.md`) é a **fonte da verdade**; o código é uma consequência.

---

### O que você vai aprender

- **O perigo do Prompt Ad-hoc:** Por que a IA toma decisões de negócio perigosas e silenciosas quando não é balizada por uma especificação.
- **Spec-Driven Development (SDD) & Shift-Left:** Como escrever regras de negócio determinísticas e contratos de dados *antes* do código.
- **Evolução Arquitetural Incremental:** Como guiar o agente na transição de um script simples para um projeto robusto em Clean Architecture e POO.
- **Agente de IA Local:** Como operar o **OpenCode** integrado ao **Ollama** utilizando o modelo **Gemma 4** (`gemma4:cloud`) sem custo de API.
- **Critérios de Aceite como Testes:** Como validar formalmente a entrega através de asserções automatizadas no PySpark.

### Pré-requisitos

- Docker instalado (ou acesso a uma instância AWS EC2 com o container oficial do laboratório).
- Navegador web moderno (para acesso à IDE `code-server`).
- Conhecimentos básicos de PySpark, terminal e Git.

> **Trilha do laboratório:**  
> Conceitos (Parte 1) → Setup (Parte 2) → Ponto de Partida: O Teste do Prompt Ad-hoc (Parte 3) → SDD Fase 1: Regras e Contratos (Parte 4) → SDD Fase 2: Clean Architecture (Parte 5) → SDD Fase 3: Confiabilidade e Testes (Parte 6) → Trilha Avançada de Specs Modulares (Parte 7) → Apêndices.

---

## Parte 1 — Conceitos essenciais

### 1.1 GenAI na engenharia de dados: Velocidade vs. Precisão

A IA Generativa reduz drasticamente o tempo de digitação de código. Para o engenheiro de dados, isso significa transformar descrições em linguagem natural em transformações Spark, testes e configurações. No entanto, ganhos reais de produtividade só acontecem quando a geração é **correta e reproduzível**. O maior risco em pipelines de dados não é um erro de sintaxe (que falha imediatamente), mas **um erro silencioso de regra de negócio** que polui o data lake.

### 1.2 Por que prompts ad-hoc falham em produção?

No fluxo tradicional (ad-hoc), o desenvolvedor conversa com o chat da IA em linguagem livre: *"faz um top 10 clientes para mim"*.  
O problema: a IA precisa preencher todas as lacunas que você não especificou. Ela decide arbitrariamente:
- Como desempatar clientes com valores idênticos.
- Se clientes sem compras entram ou não no ranking.
- Quais os nomes exatos das colunas.
- Onde os arquivos são salvos e como o código é estruturado.

O resultado é um script descartável, não-determinístico e sem testes.

### 1.3 O que é Spec-Driven Development (SDD) e o Shift-Left

No **Spec-Driven Development**, invertemos a dinâmica:

| Prompt Ad-hoc (Linguagem Solta) | Spec-Driven Development (SDD) |
| :--- | :--- |
| Regras de negócio implícitas e adivinhadas | Regras, critérios de desempate e schemas **explícitos** |
| Código monolítico com caminhos hardcoded | Princípios de Clean Architecture e Config-Driven |
| Conhecimento se perde no chat | Especificação versionada no repositório (`AGENTS.md`) |
| Difícil de reproduzir e validar | Reprodutível e com Critérios de Aceite testáveis |

Aplicamos o princípio do **Shift-Left**: esforço de engenharia concentrado na **especificação formal** antes de disparar a geração do código. Quando o requisito muda, nós evoluímos a spec — o agente apenas reconcilia a implementação.

---

## Parte 2 — Setup do ambiente

Para garantir paridade total, reprodutibilidade e eliminar atritos na instalação de Java 21, PySpark, Python e runtimes de IA em diferentes sistemas operacionais, o ambiente oficial deste laboratório é executado através de container Docker.

> 💡 **Provisionamento Automatizado na AWS (Recomendado para Aulas):**  
> Se você estiver utilizando o **AWS Academy** ou uma conta AWS própria, execute o script de provisionamento "one-liner" no **AWS CloudShell** via repositório [opencode-lab-aws](https://github.com/infobarbosa/opencode-lab-aws):
> ```sh
> curl -sS https://raw.githubusercontent.com/infobarbosa/opencode-lab-aws/main/launch-lab.sh | bash
> ```
> O script cria a instância EC2 `m5.large`, configura o Security Group e inicia o container automaticamente.

### 2.1 Ambiente Padrão: Container Docker `opencode-lab-docker-image`

A imagem oficial está hospedada publicamente no GitHub Container Registry (GHCR):  
**`ghcr.io/infobarbosa/opencode-lab-docker-image:latest`**

Componentes pré-instalados:
- **`code-server`**: IDE Visual Studio Code acessível diretamente no seu navegador (porta 8080).
- **`ollama`**: Servidor de LLM local já ativo em background (porta 11434).
- **`opencode`**: CLI oficial do agente de IA configurado no `PATH`.
- **`PySpark` e Java 21 Headless**: Prontos para execução dos jobs Spark.

#### Como iniciar o container

Execute o comando abaixo no terminal da sua máquina host (ou instância EC2):

```sh
docker run -d \
  --name opencode-lab \
  -p 8080:8080 \
  -p 11434:11434 \
  ghcr.io/infobarbosa/opencode-lab-docker-image:latest
```

> **Dica para persistência:** Se desejar mapear uma pasta local da sua máquina para dentro do container, adicione a flag de volume:  
> `-v $(pwd)/workspace:/home/barbosa/project`

#### Acessando a IDE

1. Abra o navegador web e acesse: `http://localhost:8080` (ou o IP público da sua instância AWS EC2 na porta `8080`).
2. O ambiente carregará diretamente no VS Code (`code-server`) em modo sem senha.
3. Abra o terminal integrado no menu: **Terminal -> New Terminal** (ou atalho ``Ctrl + ` `` / ``Cmd + ` ``).
4. Todo o restante do laboratório será executado dentro deste terminal integrado.

### 2.2 Login no Ollama e Verificação

O modelo adotado como padrão neste laboratório é o **Gemma 4** (`gemma4:cloud`). Para que o Ollama possa acessá-lo via nuvem, faça a autenticação uma única vez no terminal:

```sh
ollama login
```

Siga as instruções exibidas para concluir a autenticação. Após autenticar, valide a conectividade da API local:

```sh
curl http://localhost:11434/api/tags
```

### 2.3 Criar a pasta do projeto

Dentro do terminal integrado do container (no diretório de trabalho `/home/barbosa/project`), crie um diretório para o projeto e entre nele:

```sh
mkdir -p top-10-clientes && cd top-10-clientes
```

### 2.4 Instalar dependências de suporte

O PySpark já se encontra instalado no ambiente. Instale as bibliotecas complementares para formatação, validação e testes:

```sh
pip install pyyaml pytest ruff black build
```

### 2.5 Baixar os datasets de exemplo

Crie os diretórios de dados de entrada e saída:

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

---

## Parte 3 — Ponto de Partida: O Teste do Prompt Ad-hoc

Antes de construirmos a especificação formal (SDD), vamos estabelecer uma linha de base observando o comportamento do modelo ao receber um prompt direto e sem restrições explícitas.

### 3.1 Inicialize o OpenCode

No terminal, inicialize o OpenCode conectado ao modelo Gemma 4:

```sh
ollama launch opencode --model gemma4:cloud
```

### 3.2 Executando o Prompt Inicial

No prompt do OpenCode, envie o seguinte comando:

```text
Elabore um projeto pyspark que gere um relatório dos top 10 clientes com base no valor total dos pedidos.
```

Autorize o modelo a gerar os arquivos sugeridos e, quando ele concluir, saia com:

```text
/exit
```

### 3.3 Avaliação do Código Gerado

Abra o arquivo gerado (normalmente um `main.py` ou `top10.py` solto). Se você rodar com `spark-submit`, ele pode até executar e exibir um DataFrame no terminal.  
No entanto, uma análise criteriosa revela premissas e decisões ocultas.

### 3.4 Análise Crítica: 4 Questões Essenciais de Regra de Negócio

Analise o código gerado à luz de quatro questões fundamentais que a IA precisou decidir de forma arbitrária:

#### 1. "Qual a regra de desempate se duas pessoas tiverem o mesmo valor total?"
- **O que a IA geralmente fez:**  
  `df.groupBy("id_cliente").agg(sum("total")).orderBy(col("total").desc()).limit(10)`
- **O risco técnico:**  
  No Apache Spark, um `orderBy` sobre valores idênticos em partições distribuídas é **não-determinístico**! Se dois clientes empatarem no 10º lugar com R$ 5.000 gastos, a cada execução do pipeline um cliente diferente pode entrar no Top 10.  
  *Um relatório gerencial confiável não pode produzir resultados diferentes a cada execução.*

#### 2. "E os clientes que nunca compraram nada?"
- **O que a IA geralmente fez:**  
  Fez um `left_join` ou cruzou sem validar compras ativas.
- **O problema:**  
  Se a base tiver menos de 10 compradores, clientes com R$ 0 gastos poderiam figurar no relatório de "Maiores Compradores", o que desvirtua o objetivo analítico.

#### 3. "Quais os nomes e tipos exatos das colunas?"
- O relatório tem `id_cliente` ou `ID_CLIENTE`? Tem `nome` ou `nome_cliente`?  
- Em produção, tabelas e dashboards esperam contratos de schema estritos. Variações imprevistas quebram integrações downstream.

#### 4. "Onde está a arquitetura e a testabilidade?"
- Os caminhos dos arquivos estão fixados diretamente no corpo do código (*hardcoded*).
- O código está concentrado em um bloco procedural único (impossível de testar sem instanciar recursos de disco/cluster).
- Não há testes automatizados nem rotinas de validação de estilo.

> **Conclusão:** Quando regras de negócio e restrições arquiteturais não são explicitadas, o agente preenche as lacunas com premissas próprias e silenciosas. Esse comportamento evidencia a necessidade do Spec-Driven Development.

---

## Parte 4 — SDD Fase 1: Formalizando Regras de Negócio e Contratos de Dados (`AGENTS.md` v1)

Agora vamos aplicar o **Spec-Driven Development**. Em vez de tentar "consertar" o código via chat com 10 prompts soltos, nós vamos **criar a primeira versão da especificação**.

### 4.1 O que colocar na v1 da spec?
- **Persona:** Engenheiro de dados sênior.
- **Regras de Negócio Inequívocas:**
  - Fórmula explícita do cálculo: `SUM(VALOR_UNITARIO * QUANTIDADE)`.
  - **Critério Determinístico de Desempate:** Ordenar por `valor_total_gasto` DESC e, em caso de empate, por `id_cliente` ASC.
  - Apenas clientes com compras ativas (Inner Join).
  - Schema de saída padronizado: `id_cliente` (Long), `nome_cliente` (String), `valor_total_gasto` (Double).
- **Config-Driven:** Caminhos lidos obrigatoriamente de `config/config.yaml`.

### 4.2 Crie o `AGENTS.md` (v1)

Crie o arquivo `AGENTS.md` na raiz do seu projeto `top-10-clientes`:

```markdown
# AGENTS.md — Versão 1: Regras de Negócio e Contratos de Dados

## 1. Persona e Contexto
Você é um **Engenheiro de Dados Sênior** especialista em Apache Spark. Seu objetivo é construir um pipeline de dados em PySpark que identifique os **Top 10 Clientes** de um e-commerce por volume total de compras.

## 2. Regras de Negócio e Critérios de Aceite (Mandatórios)
1. **Métrica de Ranqueamento:**
   - O valor de cada item de pedido é dado por: `VALOR_UNITARIO * QUANTIDADE`.
   - O total do cliente é a soma (`SUM`) de todos os seus pedidos válidos no período.
2. **Esquema e Nomenclatura da Saída:**
   - O DataFrame de saída deve conter EXATAMENTE as seguintes colunas:
     - `id_cliente` (Long)
     - `nome_cliente` (String)
     - `valor_total_gasto` (Double)
3. **Determinismo e Regra de Desempate (Crítico):**
   - No Apache Spark distribuído, empates de valores geram rankings não-determinísticos se não houver um critério de desempate explícito.
   - O ranking DEVE ordenar por:
     - 1º critério: `valor_total_gasto` em ordem DECRESCENTE (`DESC`).
     - 2º critério (desempate determinístico): `id_cliente` em ordem CRESCENTE (`ASC`).
4. **Filtros e Integridade de Dados:**
   - Clientes sem pedidos registrados NÃO devem constar no ranking (apenas clientes com compras ativas).
   - Pedidos cujo `ID_CLIENTE` não possua registro correspondente na base de clientes devem ser descartados.
5. **Volume de Saída:**
   - O relatório deve conter exatamente os 10 maiores clientes (ou menos, caso haja menos de 10 clientes com pedidos válidos).

## 3. Diretriz de Configuração (Zero Hardcoding)
Nenhum caminho de arquivo ou parâmetro deve estar fixado ("hardcoded") no código. Utilize um arquivo de configuração **`config/config.yaml`** para definir os caminhos dos datasets de entrada e da pasta de saída.

## 4. Datasets de Entrada
- **Clientes (JSON comprimido):** `./data/input/dataset-json-clientes/data/clientes.json.gz`
- **Pedidos (CSV comprimido, sep ';'):** `./data/input/datasets-csv-pedidos/data/pedidos/pedidos-2026-01.csv.gz`

## 5. Definição de Pronto (DoD v1)
O pipeline deve ler a configuração de `config/config.yaml`, processar os dados brutos e salvar o ranking determinístico em `./data/output/top_10_clientes`.
```

*(Nota: este checkpoint está disponível no repositório em `specs/checkpoints/AGENTS-v1-regras-e-contratos.md`).*

### 4.3 Execute o OpenCode com a Spec v1

Inicie o OpenCode:

```sh
ollama launch opencode --model gemma4:cloud
```

Envie o prompt contextual referenciando o arquivo `@AGENTS.md`:

```text
Verifique o arquivo @AGENTS.md. Exponha seu plano de implementação e, após aprovação, reestruture o projeto para atender rigorosamente às regras de negócio e ao arquivo de configuração definidos.
```

Saia do OpenCode (`/exit`) e inspecione o código.  
**Resultado da Fase 1:** O código agora respeita as regras de negócio, o desempate é determinístico e não há caminhos hardcoded. Mas como está a arquitetura de software?

---

## Parte 5 — SDD Fase 2: Elevando a Régua: Clean Architecture & POO (`AGENTS.md` v2)

Um script único, mesmo que acerte as regras de negócio, é um pesadelo de manutenção. Como testar regras de negócio sem ler arquivos do disco? Como trocar o formato de entrada sem reescrever a lógica analítica?

### 5.1 Princípios de Clean Architecture Aplicados
Agora evoluímos a spec para exigir:
- **Paradigma Orientado a Objetos (POO).**
- **Separação de Camadas:**
  - `src/core/`: Leitura de configuração e exceções.
  - `src/utils/`: Factory da SparkSession e logs.
  - `src/data_io/`: Abstração de leitura e escrita (Strategy Pattern).
  - `src/transforms/`: **Transformações Puras** (funções/classes que recebem DataFrames e retornam DataFrames, sem tocar no disco).
  - `src/jobs/`: Orquestração do pipeline.
  - `src/main.py`: **Composition Root** (injeta as dependências e inicia o job).

### 5.2 Atualize o `AGENTS.md` para a v2

Substitua o conteúdo de `AGENTS.md` pelo checkpoint v2:

```markdown
# AGENTS.md — Versão 2: Clean Architecture e POO

## 1. Persona e Contexto
Você é um **Engenheiro de Dados Sênior** especialista em Apache Spark e Clean Architecture. Seu objetivo é construir um pipeline de dados em PySpark modular e orientado a objetos que identifique os **Top 10 Clientes** de um e-commerce por volume total de compras.

## 2. Princípios Arquiteturais (Mandatórios)
* **Paradigma:** Orientação a Objetos (POO).
* **Clean Architecture:** Separação total entre lógica de configuração, I/O (leitura/escrita), lógica pura de transformação e orquestração do pipeline.
* **Injeção de Dependência:** O script `main.py` atua como *Composition Root*, instanciando e injetando as dependências (`SparkManager`, `DataIOManager`) no job de orquestração.
* **Config-Driven:** Nenhum caminho físico deve estar no código. Utilize o arquivo **`config/config.yaml`**.
* **Transformações Puras:** As classes em `src/transforms/` devem conter métodos que recebem DataFrames e retornam DataFrames, sem executar leitura de disco nem escrita de arquivos.

## 3. Estrutura de Pastas Esperada
```text
.
├── config/             # Arquivo de configuração YAML
│   └── config.yaml
└── src/                # Código-fonte da aplicação
    ├── core/           # ConfigLoader e Exceções customizadas
    ├── utils/          # SparkManager (Factory) e LoggingSetup
    ├── data_io/        # DataIOManager (Strategy / Abstração de I/O)
    ├── transforms/     # Lógica pura de transformação (Top 10)
    ├── jobs/           # Orquestração do pipeline (run_top_10.py)
    └── main.py         # Composition Root e ponto de entrada
```

## 4. Regras de Negócio e Critérios de Aceite (Mandatórios)
1. **Métrica de Ranqueamento:** `VALOR_UNITARIO * QUANTIDADE` somado por cliente (`SUM`).
2. **Esquema de Saída:** Exatamente as colunas `id_cliente` (Long), `nome_cliente` (String) e `valor_total_gasto` (Double).
3. **Determinismo e Desempate:** Ordenação decrescente por `valor_total_gasto`, com desempate determinístico crescente por `id_cliente`.
4. **Filtros:** Apenas clientes com compras ativas. Pedidos sem correspondência em clientes devem ser descartados.
5. **Volume de Saída:** Limite de 10 clientes no ranking final.

## 5. Datasets de Entrada
- **Clientes (JSON comprimido):** `./data/input/dataset-json-clientes/data/clientes.json.gz`
- **Pedidos (CSV comprimido, sep ';'):** `./data/input/datasets-csv-pedidos/data/pedidos/pedidos-2026-01.csv.gz`

## 6. Definição de Pronto (DoD v2)
O código deve estar desacoplado nas camadas de `src/`, executando via `spark-submit ./src/main.py` e gerando o resultado em `./data/output/top_10_clientes`.
```

*(Nota: este checkpoint está disponível em `specs/checkpoints/AGENTS-v2-clean-architecture.md`).*

### 5.3 Reconcilie o Projeto com o OpenCode

Inicie o OpenCode:

```sh
ollama launch opencode --model gemma4:cloud
```

Solicite a refatoração orientada pela spec:

```text
O arquivo @AGENTS.md foi atualizado para a Versão 2. Refatore o projeto para adotar a Clean Architecture, POO e a estrutura modular em src/, preservando todas as regras de negócio já estabelecidas.
```

Saia do OpenCode (`/exit`).  
Execute o pipeline refatorado:

```sh
export PYTHONPATH=$(pwd)
spark-submit ./src/main.py
```

Confira a saída em `./data/output/top_10_clientes`.  
**Resultado da Fase 2:** Arquitetura limpa, código modular, testável e manutenível. Mas como provamos que as regras de negócio não quebram em uma alteração futura?

---

## Parte 6 — SDD Fase 3: Confiabilidade e Automação: Regras de Negócio viram Testes (`AGENTS.md` v3)

O ápice do Spec-Driven Development acontece quando os **Critérios de Aceite** definidos na especificação se transformam diretamente em **Asserções de Testes Automatizados**.

### 6.1 Da Regra de Negócio ao Teste Unitário

| Regra de Negócio na Spec | Como o PyTest valida em `tests/test_vendas_transforms.py` |
| :--- | :--- |
| **Cálculo da Métrica** | Cria pedidos sintéticos e valida se o valor total confere com `Σ(preco * qtd)`. |
| **Desempate Determinístico** | Cria dois clientes com R$ 1.000 gastos (IDs 5 e 2) e valida se o ID 2 surge antes do ID 5. |
| **Exclusão de Inativos** | Cria um cliente sem nenhum pedido associado e valida que ele NÃO consta no output. |
| **Tamanho do Ranking** | Cria 15 clientes com compras e valida se o resultado final tem exatamente 10 linhas. |

Como a camada `src/transforms/` é composta por **transformações puras**, os testes usam `spark.createDataFrame` em memória — rodando em segundos **sem precisar de dados no disco**.

### 6.2 Atualize o `AGENTS.md` para a v3 (Produção e DoD Completa)

Substitua o conteúdo de `AGENTS.md` pela versão final:

```markdown
# AGENTS.md — Versão 3: Produção, Qualidade e Automação (DoD Completa)

## 1. Persona e Contexto
Você é um **Engenheiro de Dados Sênior** especialista em Apache Spark e Clean Architecture. Seu objetivo é construir um pipeline de dados em PySpark profissional, modular e testável que identifique os **Top 10 Clientes** de um e-commerce por volume total de compras.

## 2. Princípios Arquiteturais (Mandatórios)
* **Paradigma:** Orientação a Objetos (POO).
* **Clean Architecture:** Separação total entre lógica de configuração, I/O (leitura/escrita), lógica pura de transformação e orquestração do pipeline.
* **Injeção de Dependência:** O script `main.py` atua como *Composition Root*, instanciando e injetando as dependências (`SparkManager`, `DataIOManager`) no job de orquestração.
* **Config-Driven:** Nenhum caminho físico ou parâmetro deve estar hardcoded. Utilize o arquivo **`config/config.yaml`**.
* **Transformações Puras:** As classes em `src/transforms/` devem conter métodos que recebem DataFrames e retornam DataFrames, sem executar leitura de disco nem escrita de arquivos.

## 3. Estrutura de Pastas Esperada
```text
.
├── config/             # Configuração (RAIZ)
│   └── config.yaml
├── src/                # Código-fonte da aplicação
│   ├── core/           # ConfigLoader e Exceções customizadas
│   ├── utils/          # SparkManager (Factory) e LoggingSetup
│   ├── data_io/        # DataIOManager (Strategy / Abstração de I/O)
│   ├── transforms/     # Lógica pura de transformação (Top 10)
│   ├── jobs/           # Orquestração do pipeline (run_top_10.py)
│   └── main.py         # Composition Root e ponto de entrada
├── tests/              # Testes unitários com pytest
├── pyproject.toml      # Gestão de dependências e metadados
└── Makefile            # Automação local (lint, test, package)
```

## 4. Regras de Negócio e Critérios de Aceite (Mandatórios)
1. **Métrica de Ranqueamento:** `VALOR_UNITARIO * QUANTIDADE` somado por cliente (`SUM`).
2. **Esquema e Nomenclatura da Saída:** Exatamente as colunas `id_cliente` (Long), `nome_cliente` (String) e `valor_total_gasto` (Double).
3. **Determinismo e Regra de Desempate (Crítico):**
   - 1º critério: `valor_total_gasto` em ordem DECRESCENTE (`DESC`).
   - 2º critério (desempate determinístico): `id_cliente` em ordem CRESCENTE (`ASC`).
4. **Filtros e Integridade de Dados:**
   - Apenas clientes com compras ativas no período (Inner Join).
   - Pedidos cujo `ID_CLIENTE` não conste na base de clientes devem ser descartados.
5. **Volume de Saída:** Exatamente os 10 maiores clientes (ou menos, se houver menos de 10 clientes válidos).

## 5. Qualidade e Automação Local
* **Testes Unitários:** Criar `tests/test_vendas_transforms.py`.
  - Utilizar `spark.createDataFrame` para gerar dados sintéticos em memória (sem depender do disco).
  - Cobrir explicitamente os critérios de aceite:
    - CA1: Cálculo e agregação do valor total.
    - CA2: Validação da regra de desempate determinístico por `id_cliente`.
    - CA3: Garantia de que clientes sem compras não figuram no resultado.
    - CA4: Limite de 10 linhas.
* **Makefile:** Fornecer os alvos:
  - `make lint`: Executar `black` e `ruff`.
  - `make test`: Executar `pytest`.
  - `make package`: Gerar o arquivo `.whl` na pasta `dist/` via ferramenta `build`.
* **Empacotamento:** Arquivo `pyproject.toml` configurado para empacotar o diretório `src/`.

## 6. Datasets de Entrada
- **Clientes (JSON comprimido):** `./data/input/dataset-json-clientes/data/clientes.json.gz`
- **Pedidos (CSV comprimido, sep ';'):** `./data/input/datasets-csv-pedidos/data/pedidos/pedidos-2026-01.csv.gz`

## 7. Definição de Pronto (DoD)
1. Todos os testes unitários passando (`make test`).
2. Código em conformidade com as regras de estilo (`make lint`).
3. Pacote distribuível gerado com sucesso em `dist/` (`make package`).
4. Pipeline executando ponta a ponta via `spark-submit ./src/main.py` e gerando o relatório final em `./data/output/top_10_clientes`.
```

*(Nota: este checkpoint está disponível no repositório em `specs/checkpoints/AGENTS-v3-producao.md`).*

### 6.3 Implementando a Suíte de Qualidade com o OpenCode

No terminal:

```sh
ollama launch opencode --model gemma4:cloud
```

Prompt:

```text
O arquivo @AGENTS.md foi atualizado para a Versão 3. Elabore a suíte completa de testes unitários em tests/test_vendas_transforms.py cobrindo os 4 critérios de aceite com dados sintéticos, e construa o Makefile e o pyproject.toml para atender à Definição de Pronto.
```

Saia do OpenCode (`/exit`).

### 6.4 Validando a Definição de Pronto (DoD)

Agora execute os comandos de validação profissional:

1. **Formatação e Estilo:**
   ```sh
   make lint
   ```
2. **Execução da Suíte de Testes:**
   ```sh
   make test
   ```
3. **Empacotamento do Pacote Python (`.whl`):**
   ```sh
   make package
   ```
4. **Execução do Pipeline em Produção:**
   ```sh
   export PYTHONPATH=$(pwd)
   spark-submit ./src/main.py
   ```

### 6.5 A Revisão Humana Continua Indispensável

Mesmo com 100% dos testes passando, a revisão humana de engenharia é o selo final de aprovação:
- As camadas estão verdadeiramente desacopladas?
- As docstrings e type hints explicam a intenção do código?
- O arquivo gerado em `./data/output/top_10_clientes` é condizente com os dados reais?

> **O agente acelera a escrita; você responde pela qualidade e pelo negócio.**

---

## Parte 7 — Trilha avançada (opcional): refatorando a spec em cadeia modular

Num único arquivo (`AGENTS.md`) convivem quatro preocupações distintas: intenção de negócio, regra analítica, contrato de dados e arquitetura de software.

No mundo corporativo ideal, essas definições nascem de atores diferentes:

| Ator | Spec Modular | Papel |
| :--- | :--- | :--- |
| **Área de Negócios** | `specs/01-brief.md` | *Por quê?* Para qual decisão estratégica o relatório serve? |
| **Analista de Negócios** | `specs/02-requisitos.md` | *O quê?* Critérios de aceite precisos e regras de desempate. |
| **Analista de Dados** | `specs/03-contrato-dados.md` | *Com quais dados?* Schemas, grãos, integridade referencial. |
| **Engenheiro de Dados** | `AGENTS.md` (enxuto) | *Como construir?* Clean Architecture, testes, build, DoD. |

### O Payoff: A Reconciliação em Cascata

Se amanhã a regra mudar de **Top 10** para **Top 20**, ou o critério de desempate mudar para *"data do último pedido"*, você não mexe no código. Você edita `specs/02-requisitos.md` e dispara no OpenCode:

```text
Verifique a cadeia de especificações: @AGENTS.md, @specs/02-requisitos.md e @specs/03-contrato-dados.md.
Os requisitos em @specs/02-requisitos.md mudaram. Reconcilie a implementação e as asserções de teste com a cadeia atualizada.
```

O agente atualiza a lógica, atualiza as asserções do `pytest`, e o pipeline continua íntegro. **Isso é Spec-Driven Development em sua plenitude.**

---

## Apêndice A — Executando o Laboratório com Aider (Fluxo Legado)

> **Nota:** Nas edições anteriores deste laboratório, o **Aider** foi utilizado como agente de IA de linha de comando padrão. Nesta versão do material, o **OpenCode** passou a ser a ferramenta padrão da aula. Mantemos este fluxo documentado como referência técnica e alternativa de estudo.

O **Aider** é um coding agent de terminal que conversa com modelos LLM (incluindo instâncias locais do Ollama) e possui como característica distintiva a realização de commits Git automáticos a cada alteração confirmada.

### 1. Instalação do Aider

Caso deseje testar o fluxo com o Aider, instale-o via `pip`:

```sh
python -m pip install aider-install
aider-install
```

### 2. Inicialização com o Ollama

Com o servidor do Ollama ativo, inicialize o Aider apontando para o modelo desejado (ex: `gemma4:cloud`):

```sh
aider --model ollama/gemma4:cloud
```

### 3. Inclusão da Spec no Contexto

No prompt interativo do Aider, adicione o arquivo de especificação:

```text
/add AGENTS.md
```

### 4. Solicitação do Plano e Implementação

Solicite a elaboração do plano:

```text
Verifique o conteúdo de AGENTS.md e me mostre o seu plano de trabalho para implementação.
```

Após aprovar o plano, autorize a escrita do código. O Aider gerará os diffs e registrará commits atômicos no repositório Git local. Para encerrar a sessão:

```text
/exit
```

---

## Apêndice B — Panorama de ferramentas e ecossistema de Coding Agents

O fluxo deste laboratório utiliza **OpenCode + Ollama** integrado ao container Docker oficial. No entanto, os mesmos princípios de Spec-Driven Development (SDD) se aplicam a todo o ecossistema moderno de ferramentas de IA para desenvolvimento:

1. **OpenCode:** Coding agent open source em terminal, altamente integrado ao Ollama e fluxo Spec-Driven.
2. **Aider:** Agente de terminal com automação forte de commits Git e edição contextual.
3. **VS Code / Cursor:** Extensões de assistência de código, chat inline e autocomplete.
4. **GitHub Copilot / Copilot Workspace:** Assistente de desenvolvimento e geração de tarefas dirigidas por spec.
5. **Antigravity / Antigravity CLI:** Agente de codificação avançado orientado a tarefas complexas e pair-programming.
6. **Claude Code:** Agente de linha de comando para raciocínio profundo e orquestração de projetos.
7. **Windsurf / Devin:** Ambientes agênticos autônomos para engenharia de software de ponta a ponta.

A interface e as mecânicas de interação mudam; a disciplina de engenharia — **especificação como fonte da verdade, plano antes de implementar, validação automatizada e revisão humana crítica** — permanece a mesma.

---

## Parabéns

Você dominou um fluxo moderno e profissional de engenharia de dados: do PySpark e Spark SQL ao **Spec-Driven Development** com um agente de IA local. Sua ferramenta mais estratégica agora não é só escrever código — é **escrever boas especificações** e **revisar criticamente** o que o agente produz.

Bons projetos pra você! ;)
