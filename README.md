# Data Pipelines com GenAI

> Construindo um pipeline PySpark **dirigido por especificação** (Spec-Driven Development) com um agente de IA local (OpenCode).

- Author: Prof. Barbosa
- Contact: infobarbosa@gmail.com
- Github: [infobarbosa](https://github.com/infobarbosa)

---
![Data Pipelines com GenAI](./img/geracao-automatica-01.png)

## Objetivo do hands-on

Ao final deste laboratório você terá construído, **sem escrever código manualmente**, um pipeline PySpark que calcula os **Top 10 Clientes** de um e-commerce por volume total de compras — seguindo os princípios de **Clean Architecture**, **Clean Code** e **Spec-Driven Development (SDD)**.

A jornada deste laboratório é **incremental e sem atrito**:
1. **Primeiro contato atômico ("Hello World"):** Um prompt simples digitado à mão (*"Crie um script que leia um arquivo csv e converta em parquet"*) para ambientar o aluno com o agente em 2 minutos, sem necessidade de logins ou downloads de modelos pesados.
2. **O Projeto Real e a Spec via `/init`:** Início do projeto `top-10-clientes` e criação da especificação inicial usando o comando nativo `/init` do OpenCode.
3. **Ponto de Partida (Prompt Ad-hoc):** Observação de como o agente se comporta sem regras explícitas e análise crítica das **decisões de negócio silenciosas e não-determinísticas**.
4. **Shift-Left e Evolução Mista da Spec:** O aluno faz o append das regras de negócio no editor visual do VS Code; nas etapas seguintes de Clean Architecture e testes, usa o próprio agente para editar o `AGENTS.md`.
5. **Modo Planejamento (Plan First):** Adoção sistemática da exigência de um plano passo a passo do agente antes da execução e alteração de arquivos de código.

A spec (`AGENTS.md`) é a **fonte da verdade**; o plano estrutura o raciocínio; o código é uma consequência.

---

### O que você vai aprender

- **Primeiros passos com Coding Agents:** Como interagir com o **OpenCode** de forma imediata via terminal.
- **O perigo do Prompt Ad-hoc:** Por que a IA toma decisões de negócio perigosas e silenciosas quando não é balizada por uma especificação.
- **Spec-Driven Development (SDD) & Shift-Left:** Como escrever regras de negócio determinísticas e contratos de dados *antes* do código.
- **Modo Planejamento (Plan Mode):** Como estruturar o fluxo de trabalho (Especificar ➔ Planejar ➔ Revisar ➔ Executar) para evitar alucinações e destruição de código.
- **Co-autoria da Spec com o Agente:** Como delegar ao agente a redação de seções técnicas no `AGENTS.md` (Clean Architecture e Testes) sob validação humana.
- **Critérios de Aceite como Testes:** Como transformar requisitos de negócio em asserções automatizadas no PySpark (`pytest`).
- **Ecossistema Flexível de Modelos:** Como começar com modelos rápidos integrados e, opcionalmente, conectar modelos locais ou cloud via **Ollama** (`gemma4:cloud`).

### Pré-requisitos

- Docker instalado (ou acesso a uma instância AWS EC2 com o container oficial do laboratório).
- Navegador web moderno (para acesso à IDE `code-server`).
- Conhecimentos básicos de PySpark, terminal e Git.

> **Trilha do laboratório:**  
> Conceitos (Parte 1) → Setup e "Hello World" (Parte 2) → Projeto Real e Spec com `/init` (Parte 3) → Ponto de Partida: Prompt Ad-hoc (Parte 4) → SDD Fase 1: Regras e Contratos (Parte 5) → SDD Fase 2: Clean Architecture (Parte 6) → SDD Fase 3: Confiabilidade e Testes (Parte 7) → Trilha Avançada de Specs Modulares (Parte 8) → Apêndices.

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

## Parte 2 — Setup do ambiente e primeiro contato ("Hello World")

Para garantir paridade total, reprodutibilidade e eliminar atritos na instalação de Java 21, PySpark, Python e runtimes de IA em diferentes sistemas operacionais, o ambiente oficial deste laboratório é executado através de container Docker.

> 💡 **Provisionamento Automatizado na AWS (Recomendado para Aulas):**  
> Se você estiver utilizando o **AWS Academy** ou uma conta AWS própria, execute o script de provisionamento "one-liner" no **AWS CloudShell** via repositório [opencode-lab-aws](https://github.com/infobarbosa/opencode-lab-aws):
> ```sh
> curl -sS https://raw.githubusercontent.com/infobarbosa/opencode-lab-aws/main/launch-lab.sh | bash
> ```
> O script cria a instância EC2 `m5.large`, configura o Security Group e inicia o container automaticamente.

### 2.1 Ambiente Padrão: Container Docker `opencode-lab-docker-image`

Caso esteja executando localmente na sua máquina, utilize a imagem pública hospedada no GitHub Container Registry (GHCR):  
**`ghcr.io/infobarbosa/opencode-lab-docker-image:latest`**

Componentes pré-instalados:
- **`code-server`**: IDE Visual Studio Code acessível diretamente no seu navegador (porta 8080).
- **`opencode`**: CLI oficial do agente de codificação configurado no `PATH`.
- **`ollama`**: Servidor de LLM local já ativo em background (porta 11434).
- **`PySpark` e Java 21 Headless**: Prontos para execução dos jobs Spark.

#### Como iniciar o container

Execute o comando abaixo no terminal da sua máquina host:

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

### 2.2 Instalar dependências de suporte

No terminal integrado, instale as bibliotecas complementares para formatação, validação e testes:

```sh
pip install pyyaml pytest ruff black build
```

### 2.3 O Primeiro Contato com o OpenCode (O "Hello World" de Dados)

Para que você experimente o fluxo do agente de forma imediata — sem necessidade de login prévio ou download de modelos pesados — o OpenCode já vem pronto para uso com seus modelos de entrada integrados.

1. No terminal integrado (em `/home/barbosa/project`), inicie o agente:
   ```sh
   opencode
   ```
2. Digite à mão no prompt do OpenCode uma tarefa atômica de engenharia de dados:
   ```text
   Crie um script que leia um arquivo csv e converta em parquet.
   ```
3. O OpenCode analisará o pedido, proporá o código (ex: `convert_csv_to_parquet.py`) e solicitará a sua confirmação para criar o arquivo.
4. Confirme a criação e, quando ele terminar, encerre a sessão do agente digitando:
   ```text
   /exit
   ```
5. Veja o arquivo criado no painel do VS Code.  
Pronto! Em menos de 2 minutos você validou o funcionamento do seu agente de IA.

---

## Parte 3 — O projeto real e a inicialização da spec com `/init`

Agora que você já teve o primeiro contato com a ferramenta, vamos iniciar o desafio real: **o pipeline analítico dos Top 10 Clientes**.

### 3.1 Criar a pasta do projeto

Dentro do terminal integrado, crie o diretório do projeto e acesse-o:

```sh
mkdir -p top-10-clientes && cd top-10-clientes
```

### 3.2 Baixar os datasets de exemplo

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

### 3.3 Inicializar a especificação com `/init`

Em ambientes baseados em navegador (`code-server`), copiar e colar textos grandes no terminal frequentemente falha por restrições de segurança do clipboard do browser. Por isso, a melhor prática com o OpenCode é utilizar o comando nativo `/init`:

1. No terminal, dentro da pasta `top-10-clientes`, execute:
   ```sh
   opencode
   ```
2. No prompt do OpenCode, digite o comando:
   ```text
   /init
   ```
3. O OpenCode varrerá o workspace e gerará automaticamente o arquivo **`AGENTS.md`** na raiz do projeto.
4. Encerre o OpenCode:
   ```text
   /exit
   ```

> 💡 **Ergonomia no VS Code:** A partir de agora, o arquivo `AGENTS.md` existe fisicamente no disco. Em vez de colar comandos gigantes no terminal, você simplesmente **abrirá o `AGENTS.md` no editor visual do VS Code** (clicando no arquivo na árvore lateral esquerda) para editá-lo confortavelmente.

---

## Parte 4 — Ponto de partida: O teste do prompt ad-hoc

Antes de customizarmos a especificação, vamos observar como o agente responde a um pedido direto de negócio sem restrições explícitas.

### 4.1 Executando o Prompt Ad-hoc Inicial

Inicie o OpenCode dentro de `top-10-clientes`:

```sh
opencode
```

Envie o prompt direto definindo a persona:

```text
Atuando como um(a) Engenheiro(a) de Dados Sênior especialista em Apache Spark, elabore um projeto PySpark que gere um relatório dos top 10 clientes com base no valor total dos pedidos.
```

Autorize a criação dos arquivos sugeridos e encerre a sessão com `/exit`.

### 4.2 Avaliação do Código Gerado

Abra o arquivo gerado (normalmente um `main.py` solto). Se você rodar com `spark-submit`, ele pode até executar e exibir um DataFrame no terminal.  
No entanto, uma análise criteriosa revela premissas e decisões ocultas — demonstrando que **apenas definir uma persona no prompt livre não substitui uma especificação formal**.

### 4.3 Análise Crítica: 4 Questões Essenciais de Regra de Negócio

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

> **Conclusão:** Mesmo quando atribuímos uma persona sênior, a ausência de regras de negócio e restrições arquiteturais explícitas faz com que o agente preencha as lacunas com premissas próprias e silenciosas. Esse comportamento evidencia a necessidade do Spec-Driven Development.

---

## Parte 5 — SDD Fase 1: Formalizando regras de negócio e contratos (`AGENTS.md`)

Agora aplicamos o **Shift-Left**: blindamos as regras de negócio e os contratos de dados antes de deixar a IA gerar ou refatorar código.

O comando `/init` que executamos na Parte 3 já criou o esqueleto básico do `AGENTS.md`. No entanto, nenhum modelo de IA é capaz de adivinhar os requisitos específicos do seu domínio de negócio. O papel do engenheiro é complementar essa especificação com clareza funcional.

### 5.1 Adicionando a Persona e as Regras de Negócio (Append no Editor Visual)

Abra o arquivo `AGENTS.md` na árvore lateral esquerda do VS Code.  
Role até o final do arquivo e **acrescente (append)** o seguinte bloco estruturado contendo a persona e as regras funcionais:

```markdown

## 1. Persona e Contexto
Você é um(a) **Engenheiro(a) de Dados Sênior** especialista em Apache Spark. Seu objetivo é construir um pipeline de dados analítico e determinístico em PySpark que identifique os **Top 10 Clientes** de um e-commerce por volume total de compras.

## 2. Regras de Negócio e Critérios de Aceite (Mandatórios)

### 1. Métrica de Ranqueamento
- O valor de cada item de pedido é calculado por: `VALOR_UNITARIO * QUANTIDADE`.
- O total gasto por cliente é a soma (`SUM`) de todos os seus pedidos válidos no período.

### 2. Esquema e Contrato de Saída
- O DataFrame final deve conter estritamente as seguintes colunas e tipos:
  - `id_cliente` (Long)
  - `nome_cliente` (String)
  - `valor_total_gasto` (Double)

### 3. Determinismo e Regra de Desempate (Crítico)
- Em pipelines distribuídos (Apache Spark), ordenações com valores idênticos geram resultados não-determinísticos se não houver um critério de desempate explícito.
- O ranking DEVE ordenar estritamente por:
  1. `valor_total_gasto` em ordem DECRESCENTE (`DESC`).
  2. `id_cliente` em ordem CRESCENTE (`ASC`) como critério determinístico de desempate.

### 4. Filtros e Integridade de Dados
- Apenas clientes com compras ativas no período devem constar no ranking (Inner Join). Clientes sem pedidos registrados NÃO devem aparecer no relatório.
- Pedidos cujo `ID_CLIENTE` não possua correspondência na base de clientes devem ser descartados.

### 5. Volume e Configuração (Zero Hardcoding)
- O relatório deve conter exatamente os 10 maiores clientes (ou menos, se houver menos de 10 clientes válidos).
- Nenhum caminho de arquivo deve estar fixado ("hardcoded") no código. Utilize um arquivo `config/config.yaml` para mapear os datasets de entrada e o diretório de saída `./data/output/top_10_clientes`.
```

Salve o arquivo (`Ctrl + S` / `Cmd + S`).

*(Nota: um gabarito desta primeira fase também está disponível para consulta em `specs/checkpoints/AGENTS-v1-regras-e-contratos.md`).*

---

### 5.2 Boas Práticas com Agentes: O Modo Planejamento (Plan Mode)

Antes de solicitar alterações de código, introduzimos aqui um princípio fundamental da engenharia de software com agentes de IA: **sempre exija um plano antes da execução**.

Quando você pede para um modelo modificar arquivos imediatamente, ele tende a assumir premissas implícitas, ignorar restrições sutis ou sobrescrever estruturas funcionais. Ao exigir um plano prévio, você força a IA a estruturar o raciocínio, permitindo que você aprove ou ajuste a abordagem antes que qualquer linha de código seja alterada.

O fluxo de trabalho profissional segue o ciclo:  
**Especificar (`AGENTS.md`) ➔ Planejar (Revisão Humana) ➔ Executar (Implementação).**

---

### 5.3 Solicitando o Plano no OpenCode

No terminal, inicie o OpenCode dentro de `top-10-clientes`:

```sh
opencode
```

Envie o prompt contextualizando a persona e solicitando exclusivamente o planejamento:

```text
Atuando como Engenheiro(a) de Dados Sênior especialista em Apache Spark, analise o arquivo @AGENTS.md com as novas regras de negócio e contratos de dados.
NÃO altere nenhum arquivo de código ainda.
Apresente primeiro um plano passo a passo detalhando as modificações necessárias no projeto para atender rigorosamente a essas regras, com atenção especial ao determinismo de desempate (ordenação composta), ao Inner Join para exclusão de inativos e ao arquivo config/config.yaml.
```

#### O que você deve avaliar no plano da IA:
1. Ela identificou a necessidade da ordenação composta (`valor_total_gasto DESC` seguido de `id_cliente ASC`)?
2. Ela propôs a criação do arquivo `config/config.yaml` em vez de caminhos hardcoded?
3. Ela garantiu que clientes inativos não serão inseridos no ranking?

### 5.4 Aprovando e Executando a Implementação

Se o plano estiver aderente à especificação, autorize a execução com um comando simples:

```text
Plano aprovado. Pode prosseguir com a implementação dos arquivos conforme planejado.
```

O OpenCode criará o arquivo `config/config.yaml` e atualizará o script PySpark.  
Após a conclusão, encerre o OpenCode com `/exit`.

Execute o pipeline para validar:

```sh
spark-submit ./main.py
```

Confira o diretório de saída `./data/output/top_10_clientes`.  
**Resultado da Fase 1:** As regras de negócio foram implementadas com precisão e determinismo. Mas o código ainda está concentrado em um único script.

---

## Parte 6 — SDD Fase 2: Elevando a régua arquitetural: Clean Architecture & POO via Agente

Com as regras de negócio blindadas, nosso próximo objetivo é transformar o script monolítico em uma base de código modular, orientada a objetos e estruturada segundo os princípios da Clean Architecture.

Em vez de você ter que redigir ou formatar dezenas de linhas de especificações arquiteturais manualmente no `AGENTS.md`, **vamos usar o próprio OpenCode como copiloto para atualizar a especificação**.

### 6.1 Solicitando à IA a atualização do `AGENTS.md`

No terminal, abra o OpenCode:

```sh
opencode
```

Instrua o agente a enriquecer a especificação com as diretrizes de arquitetura:

```text
Atualize o arquivo @AGENTS.md incorporando diretrizes de Clean Architecture e POO para PySpark:
1. Organize o projeto na estrutura src/ com separação em camadas:
   - src/core/ (ConfigLoader e exceções customizadas)
   - src/utils/ (SparkManager para gerenciar a SparkSession)
   - src/data_io/ (DataIOManager para abstração de leitura e escrita)
   - src/transforms/ (Classes com transformações puras que recebem e retornam DataFrames, sem I/O direto)
   - src/jobs/ (Orquestração do pipeline)
   - src/main.py (Composition Root / Ponto de entrada com injeção de dependência)
2. Preserve integralmente todas as regras de negócio, métricas, contratos de schema e critérios de desempate já existentes.
```

O OpenCode fará a edição direta do `AGENTS.md`.

### 6.2 Validação Humana da Especificação

Abra o arquivo `AGENTS.md` no VS Code.  
Como engenheiro responsável, verifique se:
- As diretrizes de Clean Architecture foram adicionadas de forma coerente.
- A regra de desempate determinístico e o contrato de dados continuam intactos.

*(Caso queira comparar com uma referência completa desta fase, consulte `specs/checkpoints/AGENTS-v2-clean-architecture.md`).*

### 6.3 Modo Planejamento para a Refatoração Arquitetural

Agora que a especificação arquitetural está formalizada, solicitamos o plano de refatoração ao OpenCode:

```text
Com base nas novas diretrizes de Clean Architecture do @AGENTS.md, apresente um plano de refatoração modular passo a passo antes de alterar os arquivos. Liste quais classes e módulos serão criados em src/ e como as dependências serão injetadas no main.py.
```

Avalie o plano:
- A camada `src/transforms/` receberá apenas funções/classes puras?
- O `main.py` atuará como *Composition Root*, orquestrando as dependências?

### 6.4 Aprovando e Executando a Refatoração

Após validar o plano:

```text
Plano aprovado. Execute a refatoração completa do projeto.
```

Encerre a sessão do OpenCode (`/exit`).  
Execute o pipeline refatorado a partir da nova raiz modular:

```sh
export PYTHONPATH=$(pwd)
spark-submit ./src/main.py
```

Confira os resultados em `./data/output/top_10_clientes`.  
**Resultado da Fase 2:** O projeto agora possui uma arquitetura desacoplada, profissional e modular.

---

## Parte 7 — SDD Fase 3: Confiabilidade e automação: Regras viram testes (PyTest & Makefile)

O ápice do Spec-Driven Development acontece quando os **Critérios de Aceite** definidos na especificação se transformam diretamente em **Asserções de Testes Automatizados**.

### 7.1 Da Regra de Negócio ao Teste Unitário

Como a camada `src/transforms/` é composta por **transformações puras**, os testes usam `spark.createDataFrame` com dados sintéticos em memória — rodando em segundos **sem precisar ler dados do disco**:

| Regra de Negócio na Spec | Como o PyTest valida em `tests/test_vendas_transforms.py` |
| :--- | :--- |
| **Cálculo da Métrica** | Cria pedidos sintéticos e valida se o valor total confere com `Σ(preco * qtd)`. |
| **Desempate Determinístico** | Cria dois clientes com R$ 1.000 gastos (IDs 5 e 2) e valida se o ID 2 surge antes do ID 5. |
| **Exclusão de Inativos** | Cria um cliente sem nenhum pedido associado e valida que ele NÃO consta no output. |
| **Tamanho do Ranking** | Cria 15 clientes com compras e valida se o resultado final tem exatamente 10 linhas. |

---

### 7.2 Solicitando à IA a inclusão dos Requisitos de Qualidade na Spec

Novamente, utilizamos o OpenCode para atualizar o `AGENTS.md` com a Definição de Pronto (DoD) e os requisitos de automação.

Inicie o OpenCode:

```sh
opencode
```

Envie o prompt para atualizar o `AGENTS.md`:

```text
Atualize o arquivo @AGENTS.md adicionando uma nova seção 'Qualidade e Automação Local' com:
1. Suíte de testes unitários com pytest em tests/test_vendas_transforms.py cobrindo os 4 critérios de aceite com dados sintéticos em memória (sem depender de arquivos em disco).
2. pyproject.toml para empacotamento do pacote Python a partir de src/.
3. Makefile com os seguintes alvos:
   - make lint: execução de black e ruff.
   - make test: execução de pytest.
   - make package: geração do pacote .whl na pasta dist/.
4. Definição de Pronto (DoD) formal exigindo que todos os testes passem (make test) e o código atenda aos linters (make lint).
```

Abra o `AGENTS.md` no VS Code e inspecione as adições.  
*(Gabarito de referência disponível em `specs/checkpoints/AGENTS-v3-producao.md`).*

---

### 7.3 Modo Planejamento para Testes e Automação

Com a especificação completa, solicite o plano de implementação dos testes e da automação:

```text
Apresente um plano de implementação para a suíte de testes unitários em tests/ e para a infraestrutura de automação (Makefile e pyproject.toml) conforme especificado no @AGENTS.md. Aguarde minha aprovação antes de criar os arquivos.
```

Avalie o plano:
- Os testes contemplam explicitamente o caso de teste para o desempate determinístico?
- O Makefile possui os alvos exigidos?

### 7.4 Aprovando e Executando a Implementação dos Testes

Após validar o plano:

```text
Plano aprovado. Crie os testes unitários e os arquivos de automação.
```

Saia do OpenCode (`/exit`).

---

### 7.5 Validando a Definição de Pronto (DoD)

Agora, valide a qualidade profissional do projeto executando os alvos do Makefile:

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

---

### 7.6 A Revisão Humana Continua Indispensável

Mesmo com 100% dos testes passando, a revisão humana de engenharia é o selo final de aprovação:
- As camadas estão verdadeiramente desacopladas?
- As docstrings e type hints explicam a intenção do código?
- O arquivo gerado em `./data/output/top_10_clientes` é condizente com os dados reais?

> **O agente acelera a escrita e a especificação técnica; você conduz o processo, define as regras de negócio e responde pela qualidade da entrega.**

---

## Parte 8 — Trilha avançada (opcional): refatorando a spec em cadeia modular

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

## Apêndice A — Utilizando o Ollama (Modelos Locais e Cloud com `gemma4:cloud`)

O container Docker do laboratório já possui o serviço do **Ollama** pré-instalado e rodando em background na porta local `11434`. Caso você queira utilizar modelos específicos gerenciados pelo Ollama:

### 1. Autenticação para Modelos Cloud (Gemma 4 Cloud)

Para modelos executados via Ollama Cloud (como o `gemma4:cloud`), faça login uma única vez no terminal:

```sh
ollama login
```

### 2. Inicializando o OpenCode Conectado ao Ollama

Inicie o OpenCode apontando diretamente para o modelo desejado no Ollama:

```sh
ollama launch opencode --model gemma4:cloud
```

*(Ou para modelos locais baixados na máquina, ex.: `ollama run qwen3.5`).*

---

## Apêndice B — Executando o Laboratório com Aider (Fluxo Legado)

> **Nota:** Nas edições anteriores deste laboratório, o **Aider** foi utilizado como agente de IA de linha de comando padrão. Nesta versão do material, o **OpenCode** passou a ser a ferramenta padrão da aula. Mantemos este fluxo documentado como referência técnica e alternativa de estudo.

O **Aider** é um coding agent de terminal que conversa com modelos LLM (incluindo instâncias locais do Ollama) e possui como característica distintiva a realização de commits Git automáticos a cada alteração confirmada.

### 1. Instalação do Aider

Caso deseje testar o fluxo com o Aider, instale-o via `pip`:

```sh
python -m pip install aider-install
aider-install
```

### 2. Inicialização com o Ollama

Com o servidor do Ollama ativo, inicialize o Aider apontando para o modelo desejado:

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

## Apêndice C — Panorama de ferramentas e ecossistema de Coding Agents

O fluxo deste laboratório utiliza **OpenCode** integrado ao container Docker oficial. No entanto, os mesmos princípios de Spec-Driven Development (SDD) se aplicam a todo o ecossistema moderno de ferramentas de IA para desenvolvimento:

1. **OpenCode:** Coding agent open source em terminal, altamente integrado ao ecossistema de modelos e fluxo Spec-Driven.
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
