# Data Pipelines com GenAI
- Author: Prof. Barbosa  
- Contact: infobarbosa@gmail.com  
- Github: [infobarbosa](https://github.com/infobarbosa)

Este repositório é um guia passo a passo para criação de um projeto do zero utilizando IA generativa.

---

## Introdução

A Inteligência Artificial (IA) Generativa é um ramo da inteligência artificial focado na **criação de novos conteúdos originais**, em vez de apenas analisar ou agir sobre dados existentes. Esses conteúdos podem incluir texto, imagens, música, áudio, vídeos e até mesmo código de software.

Funciona através de modelos de *machine learning* (como Redes Adversariais Generativas - GANs, ou Modelos de Linguagem Grandes - LLMs, baseados em arquiteturas como *Transformers*) que são treinados em vastos conjuntos de dados. Ao aprender os padrões e estruturas subjacentes nesses dados, a IA generativa torna-se capaz de gerar novas saídas semelhantes, mas únicas.

---

### Qual a sua importância?

A importância da IA Generativa reside no seu potencial transformador em diversas áreas, impulsionado principalmente pela sua capacidade de:

* **Aumentar a Produtividade:** Automatiza tarefas repetitivas e demoradas (como escrever rascunhos, criar designs básicos ou gerar código), liberando profissionais humanos para se concentrarem em atividades mais estratégicas, criativas e complexas.
* **Permitir a Personalização em Escala:** Pode gerar conteúdo altamente personalizado para usuários individuais, desde recomendações de produtos e campanhas de marketing até experiências de usuário adaptativas em software.
* **Democratizar a Criação:** Reduz a barreira técnica para a criação de conteúdo. Pessoas sem habilidades profundas em design, música ou programação podem usar ferramentas de IA generativa para criar trabalhos sofisticados.
* **Inovar em Pesquisa e Desenvolvimento:** É usada para simular cenários complexos, projetar novas moléculas para medicamentos, criar novos materiais ou otimizar designs de engenharia.
* **Melhorar a Eficiência Operacional:** Pode automatizar a geração de relatórios, resumos de reuniões, respostas de atendimento ao cliente e análise de grandes volumes de dados, reduzindo custos e acelerando processos.

---

### Qual o impacto em produtividade em geração de código de sistemas?

O impacto na produtividade da engenharia de software é um dos mais significativos e imediatos. Ferramentas de IA generativa, como assistentes de codificação (a exemplo do GitHub Copilot ou Amazon CodeWhisperer), atuam como "programadores em par" virtuais, acelerando drasticamente o ciclo de desenvolvimento.

Os principais impactos incluem:

1.  **Geração Rápida de Código:** Desenvolvedores podem descrever uma função ou lógica em linguagem natural (ex: "crie uma função em Python para ler um arquivo CSV e retornar a média da coluna 'idade'") e a IA gera o código correspondente em segundos.
2.  **Autocompletar Inteligente:** Vai além do simples autocompletar de nomes de variáveis, sugerindo blocos inteiros de código com base no contexto do projeto, reduzindo o tempo de digitação e a carga cognitiva.
3.  **Automatização de Tarefas Repetitivas:** Gera automaticamente código "boilerplate" (estruturas repetitivas), testes unitários, documentação (explicando o que uma função faz) e configurações de projetos.
4.  **Auxílio na Depuração e Refatoração:** A IA pode analisar blocos de código, identificar possíveis bugs, sugerir otimizações (refatoração) ou até mesmo explicar o funcionamento de um código complexo que o desenvolvedor não escreveu.
5.  **Tradução entre Linguagens:** Facilita a modernização de sistemas legados ao ajudar a traduzir código de uma linguagem de programação antiga (como COBOL) para uma mais moderna (como Java ou Python).
6.  **Redução da Curva de Aprendizado:** Desenvolvedores podem aprender novas linguagens ou *frameworks* mais rapidamente, pois a IA pode fornecer exemplos e gerar código funcional instantaneamente.

Estudos e relatórios de mercado, como os da Bain & Company e Gartner, indicam que equipes de desenvolvimento que utilizam assistentes de IA generativa podem experimentar **aumentos de produtividade entre 20% e 35%** em tarefas específicas de codificação. A projeção é que, nos próximos anos, a grande maioria dos engenheiros de software utilizará assistentes de IA como ferramenta padrão de trabalho.

---

## Engenharia de Prompt

### O que é a Engenharia de Prompt?

A **Engenharia de Prompt** (ou *Prompt Engineering*) é a disciplina de desenhar, refinar e otimizar as instruções (os "prompts") fornecidas a modelos de Inteligência Artificial Generativa, como os LLMs (Large Language Models), para obter os resultados mais precisos, relevantes e úteis possíveis.

Pense nisso como a "arte" e a "ciência" de fazer as perguntas certas da maneira certa. Um modelo como o GPT-4 ou o Gemini é um motor de inferência probabilística; ele não "entende" o mundo, mas é excelente em prever a próxima palavra com base no contexto fornecido.

O **prompt é esse contexto**. Um prompt mal formulado leva a respostas genéricas, incorretas ou inúteis. Um prompt bem-estruturado e rico em contexto guia o modelo para a "região" de conhecimento correta, forçando-o a gerar uma saída de alta qualidade e específica para o seu problema.

Para um engenheiro, isso deixa de ser uma conversa casual e se torna um **processo iterativo de design de instruções**, muito similar a escrever um código ou uma query.

---

### Tópicos de Engenharia de Prompt Essenciais para Engenheiros de Dados

#### 1. Técnicas Fundamentais de Formulação

* **Atribuição de Persona (Persona Setting):**
    * **O que é:** Iniciar o prompt definindo "quem" a IA deve ser. Isso restringe o espaço de busca do modelo e ajusta seu "tom" e nível técnico.
    * **Exemplo:** 
    ```
    Você é um engenheiro de dados sênior, especialista em otimização de performance no Apache Spark. 
    Você escreve código PySpark limpo, idiomático e com alta performance.
    ```
        

* **Aprendizado no Contexto (In-Context Learning):**
    * **Zero-Shot:** Fazer um pedido direto sem exemplos. Funciona para tarefas simples.
    * **Exemplo:** 
    ```
    Escreva um script PySpark para ler um arquivo Parquet.

    ```

    * **Few-Shot (O mais importante para código):** Fornecer um ou mais exemplos de "entrada -> saída" antes de fazer o pedido final. Isso ensina o modelo o *padrão* exato que você deseja.
    * **Exemplo:** 
    ```
    Eu quero converter queries SQL para código PySpark. 
    *Exemplo 1 (SQL):* `SELECT nome, idade FROM pessoas WHERE idade > 18` 
    *Exemplo 1 (PySpark):* `df.filter(col("idade") > 18).select("nome", "idade")` 
    *Exemplo 2 (SQL):* `SELECT ...` 
    *Exemplo 2 (PySpark):* `...` 
    *Agora, converta a seguinte query:* `[SUA QUERY COMPLEXA]` 
    ```

---
#### 2. Fornecimento de Contexto Detalhado

O erro mais comum é assumir que a IA sabe do que você está falando. Um engenheiro de dados deve aprender a "alimentar" o prompt com o máximo de contexto relevante.

* **Fornecimento de Schema:** Nunca peça para a IA escrever uma query ou transformação sem fornecer o schema dos dados.
    * **Exemplo:** 
    ```
    Dado este schema de DataFrame Spark: `[nome: string, data_nascimento: timestamp, vendas_totais: double]`, 
    escreva um script PySpark que calcule a idade do cliente e 
    o agrupe por década de nascimento, somando as vendas totais.
    ```

* **Fornecimento de Stack Trace (Rastreamento de Pilha):** Para debugging, não basta dizer "meu código deu erro".
    * **Exemplo:**
    ```
    Meu script PySpark falhou com o seguinte erro. 
    Analise o *stack trace* e me dê 3 possíveis causas e como corrigi-las. 
    [COLE O STACK TRACE COMPLETO AQUI]
    ```

* **Restrições e Requisitos:** Seja explícito sobre as "regras do jogo".
    * **Exemplo:** 
    ```
    Gere uma função em Python que receba um DataFrame PySpark. 
    A função deve ser puramente funcional (sem efeitos colaterais) e 
    deve incluir *docstrings* no formato Google e *type hints*. 
    Não use UDFs do Python, prefira as funções nativas do Spark.
    ```

---
#### 3. Controle do Formato de Saída

A IA pode gerar a resposta correta no formato errado (ex: prosa em vez de código).

* **Solicitação de Formato Específico:** Diga explicitamente como você quer a resposta.
    * **Exemplo para Documentação:** 
    ```
    Gere a documentação para este projeto no formato `README.md`. 
    Inclua as seções: 'Objetivo', 'Como Instalar' (usando pip) e 'Exemplo de Uso'.
    ```
    * **Exemplo para Testes:** 
    ```
    Gere a saída em um bloco de código Python. A resposta deve ser 
    um arquivo `test_pipeline.py` completo, usando `pytest` e `pytest-spark`.
    ```
    * **Exemplo para Dados:** 
    ```
    Gere 5 linhas de dados sintéticos (mock data) para este schema. 
    A saída deve ser no formato JSON."
    ```

#### 4. Técnicas de Raciocínio (Debugging e Otimização)

* **Cadeia de Pensamento (Chain-of-Thought - CoT):** Para problemas complexos, em vez de pedir a resposta final, peça para a IA "pensar passo a passo". Isso força o modelo a detalhar sua lógica, o que geralmente leva a uma resposta melhor e permite que você identifique onde o raciocínio falhou.
    * **Exemplo:** 
    ```
    Estou tentando otimizar esta query Spark SQL: 
    `[QUERY LENTA]` 
    
    Pense passo a passo: 
    1. Analise o que a query faz. 
    2. Identifique potenciais gargalos de performance (ex: joins cartesianos, falta de filtros). 
    3. Sugira uma versão otimizada da query e explique *por que* ela é mais rápida."
    ```

* **Explicação de Código (Code Elucidation):** é o processo de usar um modelo de GenAI para traduzir um bloco de código complexo, obscuro ou desconhecido em uma explicação clara, concisa e em linguagem natural.
    * **Exemplo 1:**
    ```
    Explique este trecho de código linha por linha.
    ```
    * **Exemplo 2:**
    ```
    Você é um especialista em otimização de Apache Spark. 
    Analise este plano de execução físico de uma query Spark. 
    Identifique os principais gargalos de performance (ex: Shuffles, Scans) 
    e explique, em termos de negócio, por que a query está lenta 
    e sugira uma otimização no código PySpark original.

    == Physical Plan ==
    *(5) SortMergeJoin [id#10], [id#25], Inner
    :- *(2) Sort [id#10 ASC NULLS FIRST], false, 0
    :  +- Exchange hashpartitioning(id#10, 200), true, [id=#101]
    :     +- *(1) FileScan parquet [id#10,nome#11] ...
    +- *(4) Sort [id#25 ASC NULLS FIRST], false, 0
    :  +- Exchange hashpartitioning(id#25, 200), true, [id=#111]
    :     +- *(3) FileScan parquet [id#25,venda#26] ...
    ```
    * **Exemplo 3**: 
    ```
    Quebre esta expressão regular em partes e explique em detalhes o 
    que ela está tentando validar ou extrair: 
    `(\d{4}-\d{2}-\d{2})T(\d{2}:\d{2}:\d{2})Z\s+\[(ERROR|WARN)\]\s+(.*)`
    ```

#### 5. Iteração e Refinamento

O primeiro prompt quase nunca é o final. A engenharia de prompt é um **diálogo iterativo**.

* **Refinamento Pós-Resposta:** Usar a resposta da IA como parte do próximo prompt.
    * **Exemplo 1:** 
    ```
    O código que você gerou está bom, mas você usou um RDD. 
    Por favor, refatore para usar exclusivamente a API de DataFrames.
    ```
    * **Exemplo 2:** 
    ```
    A documentação está incompleta. Adicione uma seção sobre como executar os testes automatizados.
    ```


---

# Laboratório

## Passo 1: Configuração Inicial

Antes de começar, prepare seu ambiente:

ATENÇÃO! Se estiver utilizando AWS Cloud9, utilize esse [tutorial](https://github.com/infobarbosa/data-engineering-cloud9).

1.  **Crie uma pasta para o projeto:**
    ```bash
    mkdir -p data-pipeline-com-genai/
    mkdir -p data-pipeline-com-genai/data/input
    mkdir -p data-pipeline-com-genai/data/output
    
    ```
    
    ```bash
    cd data-pipeline-com-genai
    
    ```

2.  **Crie o arquivo `requirements.txt` para gestão das dependências:**
    ```bash
    touch requirements.txt
    
    ```

    ```bash
    echo 'pyspark==4.0.1' >> requirements.txt
    
    ```

    ```bash
    pip install -r requirements.txt
    
    ```

    **ATENÇÃO!** `requirements.txt` tem valor apenas temporário. A gestão de dependências mais adequada seria via `pyproject.toml`, a qual será vista mais tarde nesse tutorial.

3.  **Baixe os datasets:**
    Execute o script para baixar os dados necessários para a pasta `data/`.
    
    **Clientes**
    ```bash
    curl -L -o ./data/input/clientes.gz https://raw.githubusercontent.com/infobarbosa/dataset-json-clientes/main/data/clientes.json.gz
    
    ```

    Um olhada rápida no arquivo de clientes
    ```bash
    gunzip -c data/input/clientes.gz | head -n 5

    ```

    **Pedidos**
    ```bash
    curl -L -o ./data/input/pedidos.gz https://raw.githubusercontent.com/infobarbosa/datasets-csv-pedidos/main/data/pedidos/pedidos-2024-01.csv.gz

    ```

    Uma olhada rápida no arquivo de pedidos
    ```bash
    gunzip -c ./data/input/pedidos.gz | head -n 5
    
    ```
---

## Passo 2: 

### Exemplos de prompt
1. Atribuição de Persona (Persona Setting):

    ```
    Você é um engenheiro de dados sênior, especialista em otimização de performance no Apache Spark. 
    Você escreve código PySpark limpo, idiomático e com alta performance.
    ```

2. Aprendizado no Contexto (In-Context Learning):
    * **Zero-Shot**:
    * **Exemplo:**
    ```
    Escreva um script PySpark para ler os arquivos na pasta ./data/input,
    transformar em Parquet e armazenar na pasta ./data/output.
    ```

    * **Few-Shot**: Fornecer um ou mais exemplos de "entrada -> saída" antes de fazer o pedido final. Isso ensina o modelo o *padrão* exato que você deseja.
    * **Exemplo:**
    ```
    Escreva um script PySpark que leia os arquivos na pasta ./data/input,
    transforme em Parquet e armazene o resultado na pasta ./data/output.
    - Dataset de clientes
    - Localização: ./data/input/clientes.json.gz
    - Formato: JSON
    - Compressão: GZIP
    - Leiaute:

        | Atributo        | Tipo      | Obs                                               |
        | ---             | ---       | ---                                               |
        | ID              | long      | O identificador da pessoa                         |
        | NOME            | string    | O nome da pessoa                                  |
        | DATA_NASC       | date      | A data de nascimento da pessoa                    |
        | CPF             | string    | O CPF da pessoa                                   |
        | EMAIL           | string    | O email da pessoa                                 |
        | INTERESSES      | object    | Um array com a lista de interesses da pessoa      |

    - Sample:
        { "id": 1, "nome": "Isabelly Barbosa", "data_nasc": "1963-08-15", "cpf": "137.064.289-03", "email": "isabelly.barbosa@gmail.com", "interesses": ["Política", "Economia"] }
        { "id": 2, "nome": "Larissa Fogaça", "data_nasc": "1933-09-29", "cpf": "703.685.294-10", "email": "larissa.fogaca@example.com", "interesses": ["Música", "Viagens", "Gastronomia"] }
        { "id": 3, "nome": "João Gabriel Silveira", "data_nasc": "1958-05-27", "cpf": "520.179.643-52", "email": "joao.gabriel.silveira@example.com", "interesses": ["Ciências", "Inteligência Artificial"] }
    
    - Dataset de pedidos de venda
    - Localização: ./data/input/pedidos.gz
    - Separador: ";"
    - Header: True
    - Compressão: gzip
    - Leiaute:
        | Atributo        | Tipo      | Obs                                               | 
        | ---             | ---       | ---                                               |
        | ID_PEDIDO       | string    | O identificador do pedido                         | 
        | PRODUTO         | string    | O nome do produto no pedido                       | 
        | VALOR_UNITARIO  | float     | O valor unitário do produto no pedido             | 
        | QUANTIDADE      | long      | A quantidade do produto no pedido                 | 
        | DATA_CRIACAO    | timestamp | A data da criação do pedido                       | 
        | UF              | string    | A sigla da unidade federativa (estado) no Brasil  | 
        | ID_CLIENTE      | long      | O identificador do cliente                        | 
        
        **Atenção!** `ID_PEDIDO` é definido como `string` mas sua formatação segue a especificação UUID.

    - Sample

        id_pedido;produto;valor_unitario;quantidade;data_criacao;uf;id_cliente

        fdd7933e-ce3a-4475-b29d-f239f491a0e7;MONITOR;600;3;2024-01-01T22:26:32;RO;12414
        fe0f547a-69f3-4514-adee-8f4299f152af;MONITOR;600;2;2024-01-01T16:01:26;SP;11750
        fe4f2b05-1150-43d8-b86a-606bd55bc72c;NOTEBOOK;1500;1;2024-01-01T06:49:21;RR;1195
        fe8f5b08-160b-490b-bcb3-c86df6d2e53b;GELADEIRA;2000;1;2024-01-01T04:14:54;AC;8433
        feaf3652-e1bd-4150-957e-ee6c3f62e11e;HOMETHEATER;500;2;2024-01-01T10:33:09;SP;12231
        feb1efc5-9dd7-49a5-a9c7-626c2de3e029;CELULAR;1000;2;2024-01-01T13:48:39;SC;2340
        ff181456-d587-4abd-a0ac-a8e6e33b87d5;TABLET;1100;1;2024-01-01T21:28:47;RS;12121
        ff3bc5e0-c49a-46c5-a874-3eb6c8289fd1;HOMETHEATER;500;1;2024-01-01T22:31:35;SC;6907
        ff4fcf5f-ca8a-4bc4-8d17-995ecaab3110;SOUNDBAR;900;3;2024-01-01T19:33:08;RJ;9773
        ff703483-e564-4883-bdb5-0e25d8d9a006;NOTEBOOK;1500;3;2024-01-01T00:22:32;RN;2044
        ffe4d6ad-3830-45af-a599-d09daaeb5f75;HOMETHEATER;500;3;2024-01-01T02:55:59;MS;3846

    ```

---

## Passo 3: Schemas esplícitos

### Exemplos de prompt
1. Clientes
    ```
    DEFINA o schema para o DataFrame de clientes em PySpark. 
    Gere o código Python que define um objeto StructType chamado schema_clientes com os seguintes campos: 
    - id (LongType)
    - nome (StringType)
    - data_nasc (DateType)
    - cpf (StringType)
    - email (StringType)
    - interesses (ArrayType de StringType). 
    
    Inclua os imports necessários.
    ```

2. Pedidos
    ```
    DEFINA o schema para o DataFrame de pedidos em PySpark. 
    Gere o código PySpark para um StructType chamado schema_pedidos que contenha os campos: 
    - id_pedido (StringType) 
    - produto (StringType) -
    - valor_unitario (FloatType)
    - quantidade (LongType)
    - data_criacao (TimestampType)
    - uf (StringType)
    - id_cliente (LongType)."
    ```

---

## Passo 4: Configurações centralizadas

### Exemplo de prompt
```
Seu objetivo é ter uma única base de código PySpark que possa ser executada em múltiplos
ambientes (ex: desenvolvimento, homologação, produção) sem qualquer alteração no
código-fonte.
As diferenças (como caminhos de dados, credenciais de banco de dados, nomes de tabelas,
recursos do cluster Spark, etc.) devem ser inteiramente gerenciadas por arquivos de
configuração externos.

- Forneça um módulo dedicado a configurações
- Sugira um formato de arquivo de configuração robusto (ex: YAML ou TOML, que são 
mais legíveis que JSON para configurações complexas).
- Mostre um exemplo de estrutura de arquivo (config.yaml) que:
  1. Seja seccionado por ambiente (ex: dev, prod).
  2. Inclua configurações comuns (compartilhadas) e configurações específicas de cada ambiente.
  3. Contenha exemplos de parâmetros de pipeline, como:
    a. Configurações da SparkSession (ex: appName, spark.sql.shuffle.partitions).
    b. Definições de origem de dados (ex: source_path, format, options).
    c. Definições de destino (ex: target_path, save_mode).
    d. Parâmetros de lógica de negócios (ex: limiar_de_filtro, colunas_para_selecionar).    
    e. Forneça uma classe ou módulo Python (ex: ConfigLoader) responsável por carregar o arquivo de configuração.
    f. Este módulo deve determinar qual ambiente carregar (ex: a partir de uma variável de ambiente como APP_ENV ou um argumento de linha de comando como --env).
    g. Importante: Demonstre como "mesclar" as configurações comuns com as configurações específicas do ambiente selecionado.
    h. Sugira bibliotecas Python recomendadas para esta tarefa (como PyYAML para parsing e Pydantic para validação e tipagem da configuração, garantindo que os tipos de dados esperados estejam corretos).    
```

---

## Passo 5: Sessão Spark

### Exemplo de prompt

Sua tarefa é projetar e implementar um módulo Python para o gerenciamento centralizado da `SparkSession` em um projeto de engenharia de dados em larga escala.

O design deve priorizar:
1.  **Reutilização:** A solução deve ser facilmente importada e utilizada por múltiplos scripts de ETL e aplicações.
2.  **Centralização:** Deve haver um único ponto de verdade para a configuração e obtenção da sessão Spark.
3.  **Testabilidade:** Este é o requisito mais crítico. A arquitetura deve permitir que os testes unitários (usando `pytest`) "mockem" a `SparkSession` com facilidade. A solução NÃO deve instanciar uma sessão global no momento da importação.
4.  **Flexibilidade:** A sessão não deve ter configurações "hardcoded" (como `appName` ou `master`).

**Requisitos Técnicos Específicos:**

1.  **Use um Padrão de Design:** Implemente a lógica usando uma Classe (ex: `SparkManager`) ou um padrão de fábrica (Factory). Evite o padrão Singleton global, pois ele dificulta os testes.
2.  **Injeção de Dependência para Configuração:** A função ou método que cria/obtém a sessão deve aceitar um objeto de configuração (ex: um dicionário Python ou um objeto `Config`) como parâmetro. Isso permite injetar diferentes configurações para `dev`, `prod` e `test`.
3.  **Lógica "Get-or-Create" Inteligente:** O método principal (ex: `get_session`) deve ser idempotente, ou seja, se uma sessão ativa com a mesma configuração já existir, retorne-a; caso contrário, crie uma nova.
4.  **Boas Práticas de Código Limpo:**
    * Inclua type hints (`typing`) em todas as assinaturas de métodos e variáveis.
    * Use Docstrings claras (padrão Google ou reST) para a classe e seus métodos.
    * Integre logging (`logging`) para informar quando uma sessão está sendo criada ou reutilizada.
5.  **Gerenciamento de Encerramento:** Forneça um método explícito para encerrar a sessão Spark (`stop_session`).

**Formato da Resposta:**

1.  Forneça o código Python completo para o módulo (ex: `utils/spark_manager.py`).
2.  Mostre um exemplo claro de como um script de ETL (`meu_job.py`) importaria e usaria este gerenciador.
3.  Mais importante: forneça um exemplo de como um teste unitário (`test_meu_job.py`) usando `pytest` e `pytest-mock` poderia "mockar" a sessão Spark para testar a lógica de negócios sem iniciar um cluster Spark real.
4.  Justifique brevemente sua escolha de design (ex: por que uma classe de fábrica é melhor que um singleton global para testabilidade).

---

## Passo 6: Leitura e Escrita de Dados (I/O)

### Exemplo de prompt

Sua tarefa é projetar e implementar um módulo Python de I/O (Data Access Layer) genérico, configurável e testável para PySpark. O objetivo principal é **abstrair completamente** a lógica de leitura (`.read`) e escrita (`.write`) dos scripts de ETL.

**Princípios de Design Obrigatórios:**

1.  **Abstração (Ocultação de Informação):** Os scripts de ETL (consumidores do módulo) devem operar apenas com **identificadores lógicos** (ex: "clientes_bronze", "vendas_finais"). Eles NUNCA devem especificar caminhos (paths), formatos (parquet, delta, csv) ou opções de leitura/escrita.
2.  **Orientado à Configuração (Config-Driven):** O módulo deve ser "alimentado" por um catálogo de dados centralizado (ex: um arquivo YAML ou dicionário Python). Este catálogo é o que mapeia os IDs lógicos (ex: "clientes_bronze") para suas configurações físicas (ex: `path: "s3://..."`, `format: "parquet"`).
3.  **Extensibilidade (Padrão de Design):** A arquitetura deve ser extensível. Use um **Padrão de Fábrica (Factory Pattern)** ou **Estratégia (Strategy Pattern)** para que adicionar um novo formato (ex: `JDBC`, `BigQuery`) ou uma nova fonte (ex: `GCS`, `AzureBlob`) seja fácil, sem modificar a classe principal.
4.  **Testabilidade Extrema:** A arquitetura deve permitir que os testes unitários (`pytest`) mockem facilmente as operações de I/O. O teste de um ETL deve ser capaz de "injetar" um DataFrame de teste quando o ETL chamar `data_io.read("clientes_bronze")`, sem NENHUMA I/O real.

**Requisitos Técnicos Específicos:**

1.  **Interface da Classe Principal:** Crie uma classe principal (ex: `DataIOManager`).
    * O construtor (`__init__`) deve receber o "catálogo de dados" (ex: um `dict`) como dependência.
    * Deve expor dois métodos principais:
        * `read_data(spark: SparkSession, source_id: str, **kwargs) -> DataFrame`: Busca a configuração de `source_id` no catálogo e lê os dados. `kwargs` podem ser usados para passar opções dinâmicas (ex: filtros de partição).
        * `write_data(df: DataFrame, target_id: str, mode: str = "overwrite", **kwargs)`: Busca a configuração de `target_id` no catálogo e escreve o DataFrame.

2.  **Padrão de Design (Strategy/Factory):**
    * Crie "handlers" (estratégias) de I/O específicos por formato (ex: `ParquetIO`, `DeltaIO`, `CsvIO`).
    * A classe `DataIOManager` deve delegar a chamada (`read`/`write`) para o handler correto com base no `format` especificado no catálogo.

3.  **Código Limpo:**
    * Use type hints (`typing`) de forma rigorosa.
    * Inclua logging (`logging`) detalhado (ex: "Lendo 'clientes_bronze' do formato 'parquet' em 's3://...'").
    * Inclua tratamento de erros (ex: `SourceNotFoundInCatalogError`).

**Formato da Resposta:**

1.  **Exemplo do Catálogo:** Primeiro, mostre como seria a estrutura do catálogo de configuração (ex: `catalog.yaml` ou `catalog.py` como um `dict`).
2.  **Código Completo do Módulo:** O código Python completo para o módulo `data_io.py`, incluindo a classe `DataIOManager` e os handlers de estratégia.
3.  **Exemplo de Uso (O ETL):** Um script `meu_job.py` que importa e usa este módulo. O script deve receber o `SparkSession` e o `DataIOManager` (por injeção de dependência, se possível) e *apenas* usar IDs lógicos.
4.  **Exemplo de Teste (O Ponto Crítico):** Um `test_meu_job.py` usando `pytest` e `pytest-mock` (ou o `unittest.mock`) que testa a lógica de negócio do `meu_job.py`. O teste DEVE mockar o `DataIOManager.read_data` para retornar um DataFrame de teste criado manualmente.
5.  **Justificativa:** Explique brevemente por que o padrão Factory/Strategy é superior a um `if/elif` gigante dentro do método `read_data`.

---

## Passo 7: Lógica de Negócio e Orquestração

### Exemplo de prompt

Sua tarefa é projetar e implementar um módulo Python para a lógica de negócios (transformação) de um pipeline de ETL. O design deve seguir rigorosamente o princípio da **Separação de Preocupações**, onde a lógica de transformação é 100% isolada das camadas de I/O (leitura/escrita).

**Contexto da Lógica de Negócios:**

O objetivo é calcular o "Top 10 Clientes por Valor Total de Vendas".

* **Input DataFrame 1 (Clientes):** `df_clientes`
    * Colunas relevantes: `ID` (long), `NOME` (string), `EMAIL` (string).
* **Input DataFrame 2 (Pedidos):** `df_pedidos`
    * Colunas relevantes: `VALOR_UNITARIO` (float), `QUANTIDADE` (long), `ID_CLIENTE` (long).
* **Output DataFrame (Relatório):** `df_top_10_clientes`
    * Colunas: `id_cliente` (long), `nome_cliente` (string), `email` (string), `valor_total_vendas` (double).
    * Regras:
        1.  Calcular o valor total por pedido (`QUANTIDADE * VALOR_UNITARIO`).
        2.  Agregar o valor total por cliente.
        3.  Juntar com os dados do cliente (nome, email).
        4.  Ordenar por `valor_total_vendas` em ordem decrescente.
        5.  Selecionar os 10 primeiros registros.

**Requisitos Arquiteturais (Obrigatório):**

1.  **Função Pura:** A lógica DEVE ser implementada como uma função pura (ou um método de classe estático).
2.  **Assinatura Clara:** A função DEVE ter uma assinatura clara, recebendo os DataFrames de entrada como parâmetros e retornando o DataFrame transformado. 
    * Exemplo: `def calcular_top_10_clientes(df_clientes: DataFrame, df_pedidos: DataFrame) -> DataFrame:`
3.  A função NÃO DEVE ter configurações "hardcoded" e que deve apenas chamar os DataIOManager e a função de transformação.    
4.  **Sem Efeitos Colaterais (Side-effects):** A função NÃO DEVE conter NENHUMA lógica de I/O (`.read` ou `.write`). Ela não deve importar ou usar o `SparkManager` ou o `DataIOManager`.
5.  **Código Limpo e Manutenível:**
    * Use type hints (`pyspark.sql.DataFrame`, `pyspark.sql.functions`).
    * Use `pyspark.sql.functions` (F) para todas as transformações (evite UDFs).
    * **Crucial:** Use uma classe de constantes (ou Enum) para todos os nomes de colunas ("magic strings"). Isso é vital para refatoração e prevenção de bugs.
    * A função deve ser idempotente.

**Formato da Resposta:**

1.  **Código do Módulo de Transformação:**
    * Forneça o código Python completo para o módulo (ex: `transforms/vendas_transforms.py`).
    * Este arquivo deve conter a classe de constantes para os nomes de colunas e a função `calcular_top_10_clientes`.

2.  **Código do Job Orquestrador:**
    * Mostre como um "job" separado (ex: `jobs/run_top_10_clientes.py`) usaria os módulos que criamos anteriormente (`SparkManager`, `DataIOManager`) e este novo módulo de transformação.
    * Este script será o "cola":
        1.  Obtém a sessão Spark (via `SparkManager`).
        2.  Lê `df_clientes` (via `DataIOManager.read_data("clientes_bronze")`).
        3.  Lê `df_pedidos` (via `DataIOManager.read_data("pedidos_bronze")`).
        4.  **Chama a função `calcular_top_10_clientes(df_clientes, df_pedidos)`**
        5.  Escreve o resultado (via `DataIOManager.write_data(df_report, "top_10_clientes_silver")`).

3.  **Exemplo de Teste Unitário:**
    * Forneça um `tests/test_vendas_transforms.py` usando `pytest`.
    * O teste NÃO DEVE usar o `DataIOManager` ou ler arquivos.
    * O teste DEVE gerar 3 conjuntos de **dados sintéticos de exemplo** (Input 1: 2 clientes, 3 pedidos; Input 2: caso de borda com um cliente sem pedidos; Input 3: caso em que o cliente 10 e o cliente 11 têm o mesmo valor total de vendas, para testar a estabilidade da ordenação) no formato spark.createDataFrame(). 
    * O teste deve:
        1.  Criar uma `SparkSession` local (apenas para o teste).
        2.  Criar um `df_clientes_teste` (com 2-3 linhas) usando `spark.createDataFrame()`.
        3.  Criar um `df_pedidos_teste` (com 4-5 linhas) usando `spark.createDataFrame()`.
        4.  Chamar a função `calcular_top_10_clientes` com esses DataFrames de teste.
        5.  Coletar o resultado (`.collect()`) e fazer asserções (`assert`) para verificar se o cálculo, o join, a ordenação e o limite (top N) estão corretos.

4.  **Justificativa:** Explique por que essa separação entre a lógica de transformação pura e o "job orquestrador" de I/O é a base do TDD em PySpark.

---

## Passo 8: Clean Code

### Exemplo de prompt

Sua tarefa final é **integrar** todos esses módulos em um projeto coeso, pronto para produção, aplicando os princípios de **Clean Architecture** e **InjeIção de Dependência (DI)**. O objetivo é o **desacoplamento total**.

**Requisitos Arquiteturais Críticos:**

1.  **Ponto de Entrada (Composition Root):**
    * O projeto deve ter um ponto de entrada principal (ex: `main.py` ou `app.py`).
    * Este `main.py` é o **único** local responsável por **construir** os objetos de dependência (o "Composition Root"). Ele irá:
        1.  Carregar a configuração central.
        2.  Instanciar o `SparkManager`.
        3.  Instanciar o `DataIOManager` (injetando o catálogo de dados nele).
        4.  Obter a `SparkSession` do `SparkManager`.

2.  **Injeção de Dependência (DI) Explícita:**
    * O "script do job" (`jobs/run_top_10_clientes.py`) que contém o *fluxo* do ETL (ler -> transformar -> escrever) **NÃO DEVE** instanciar o `SparkManager` ou o `DataIOManager`.
    * Refatore-o para que ele exponha uma função (ex: `run_job`) que **recebe** suas dependências como parâmetros.
    * **Assinatura esperada:** `def run_job(spark: SparkSession, io_manager: DataIOManager, config: dict):`
    * O `main.py` será responsável por *chamar* `run_job` e *passar* as dependências instanciadas para ele.

3.  **Gerenciamento de Configuração Centralizado:**
    * Crie um módulo (ex: `core/config.py`) responsável por carregar configurações de um arquivo (`config.yaml`) ou variáveis de ambiente.
    * Isso deve incluir o `appName` do Spark e o "catálogo de dados" (o `dict` que o `DataIOManager` usa).
    * O `main.py` usará este módulo para obter a configuração e injetá-la onde for necessário.

4.  **Logging Centralizado:**
    * Crie um módulo (ex: `utils/logging_setup.py`) que forneça uma única função para configurar o `logging` (nível, formato) para toda a aplicação.
    * Esta função deve ser chamada *apenas uma vez* no `main.py`.
    * Todos os outros módulos (`spark_manager`, `data_io`, `transforms`) devem obter seu logger de forma padrão (ex: `logging.getLogger(__name__)`).

5.  **Tratamento de Erros Robusto:**
    * Crie um módulo (ex: `core/exceptions.py`) para exceções customizadas (ex: `DataPipelineError`, `ConfigNotFoundError`).
    * O `main.py` (o ponto de entrada) deve envolver a chamada `run_job` em um bloco `try/except` global.
    * Em caso de falha, ele deve logar o erro de forma clara (usando o logger configurado) e sair com um código de status diferente de zero (ex: `sys.exit(1)`).

**Formato da Resposta:**

1.  **Árvore de Diretórios Final:** Mostre a estrutura completa de pastas e arquivos do projeto (ex: `core/`, `utils/`, `jobs/`, `transforms/`, `tests/`, `main.py`, `config.yaml`).
2.  **Módulos de "Cola" (Obrigatório):**
    * `main.py` (O Composition Root que constrói e injeta tudo).
    * `core/config.py` (O carregador de configuração).
    * `core/exceptions.py` (Exceções customizadas).
    * `utils/logging_setup.py` (O configurador de logging).
3.  **Lógica de negócios:**
    * `jobs/run_top_10_clientes.py`, mostrando a função `run_job` que agora recebe suas dependências (DI).
4.  **Exemplo de `config.yaml`:** Mostre um exemplo de arquivo de configuração que o `core/config.py` leria.

---

## Passo 9: Empacotamento

### Exemplo de prompt

**Contexto:**
Temos um projeto PySpark multi-módulo (`core/`, `transforms/`, `jobs/`, `utils/`, `main.py`) que segue os princípios de Clean Architecture. O projeto tem dependências de produção (ex: `pyspark`, `pyyaml`) e dependências de desenvolvimento (ex: `pytest`, `pytest-mock`, `black`).

**Sua Tarefa:**
Criar todos os artefatos de configuração e scripts necessários para **empacotar e distribuir** esta aplicação de forma profissional, reprodutível e segura, pronta para ser integrada a um pipeline de CI/CD e executada em um ambiente orquestrado (ex: Kubernetes, Databricks, Airflow).

**Requisitos Técnicos Críticos:**

1.  **Gerenciamento de Dependências Moderno:**
    * Utilize um único arquivo `pyproject.toml` para gerenciar o projeto.
    * Este arquivo deve definir:
        * Metadados do projeto (nome, versão, autor).
        * Dependências de produção (na seção `[project.dependencies]`). As dependências de produção são pyspark, pyyaml e pydantic.
        * Dependências de desenvolvimento (em uma seção opcional `[project.optional-dependencies]`, grupo "dev"). As dependências de desenvolvimento incluem pytest, pytest-mock, black, flake8 e build.

2.  **Empacotamento como Wheel (`.whl`):**
    * O `pyproject.toml` deve estar configurado para permitir que o projeto seja "buildado" como um pacote Python (wheel) usando ferramentas padrão (ex: `build`). Isso é superior a enviar arquivos `.py` soltos ou `.zip`.

3.  **Containerização (A Prática Recomendada):**
    * Forneça um `Dockerfile` de **multi-stage build** otimizado para produção.
    * **Estágio 1 ("builder"):**
        * Comece com uma imagem Python completa (ex: `python:3.10-builder`).
        * Instale as ferramentas de build (ex: `pip`, `build`).
        * Copie o `pyproject.toml` e instale **todas** as dependências (incluindo as de "dev" para testes).
        * Copie o código-fonte.
        * *Execute os testes* (`pytest`) DENTRO do container para validar o build.
        * Construa o arquivo `.whl` da nossa aplicação.
    * **Estágio 2 ("runtime"):**
        * Comece com uma imagem Python *mínima* (ex: `python:3.10-slim`).
        * **NÃO** copie o código-fonte. Copie **apenas** o `.whl` gerado no Estágio 1.
        * Copie os arquivos de configuração necessários (ex: `config.yaml`).
        * Instale **apenas** as dependências de *produção* e o nosso `.whl`.
        * Configure o `ENTRYPOINT` ou `CMD` para executar a aplicação (ex: `CMD ["python", "-m", "nome_do_modulo_principal.main"]` ou `CMD ["python", "main.py"]`).
    * **Justificativa implícita:** Isso cria uma imagem final pequena, segura (sem `pytest`, `black`, ou ferramentas de build) e otimizada.

4.  **Automação de Build (O "Roteiro"):**
    * Forneça um `Makefile` simples e idiomático.
    * O `Makefile` deve conter *targets* claros para:
        * `lint`: Rodar `black` e `flake8`.
        * `test`: Rodar `pytest`.
        * `build`: Construir a imagem Docker (ex: `docker build -t meu-pipeline .`).
        * `package`: (Alternativa) Construir apenas o `.whl` localmente.
        * `clean`: Remover artefatos (`__pycache__`, `dist/`, `build/`).

5.  **Arquivos de "Ignorar":**
    * Forneça um `.gitignore` robusto para um projeto Python/Data.
    * Forneça um `.dockerignore` espelhado no `.gitignore` para garantir que o *build context* do Docker seja mínimo e rápido (ignorando `.venv`, `__pycache__`, `.pytest_cache`, etc.).

**Formato da Resposta:**

1.  O conteúdo do arquivo `pyproject.toml`.
2.  O conteúdo do `Dockerfile` (multi-stage).
3.  O conteúdo do `Makefile`.
4.  O conteúdo do `.gitignore`.
5.  O conteúdo do `.dockerignore`.
6.  Forneça exemplos de comandos para o build
7.  Forneça exemplos de submissão do job (`spark-submit ...`) em um ambiente de produção.

---

## Passo 10: README.md

### Exemplo de prompt

Sua tarefa é gerar o arquivo `README.md` completo (em Markdown) para o projeto PySpark que acabamos de arquitetar. O tom deve ser profissional, claro e direto.

**Contexto do Projeto (para você se basear):**
* **Propósito:** Pipeline de ETL que calcula o "Top 10 Clientes por Valor de Vendas".
* **Arquitetura:** Clean Architecture, com Separação de Preocupações (I/O vs. Transformações), Injeção de Dependência (DI) e um "Composition Root" em `main.py`.
* **Módulos Principais:** `core` (config, exceptions), `utils` (logging), `spark_manager`, `data_io`, `transforms`.
* **Gerenciamento:** `pyproject.toml` (com dependências "main" e "dev") e um `Makefile` para comandos comuns.
* **Artefatos de Build:** O projeto é empacotado como um Wheel (`.whl`) e como uma imagem Docker (via `Dockerfile` multi-stage).
* **Configuração:** O pipeline é "config-driven" por um `config.yaml` externo.

**Seções Obrigatórias para o `README.md`:**

1.  **Título e Visão Geral:**
    * Um título claro e um parágrafo conciso explicando o objetivo de negócio do projeto (o "Top 10 Clientes").

2.  **Filosofia da Arquitetura:**
    * Uma breve explicação (em *bullet points*) sobre os princípios de design:
        * Clean Architecture (separação de I/O, Lógica e Orquestração).
        * Testabilidade (Lógica de negócios pura em `transforms`).
        * Orientado à Configuração (Lógica de I/O abstraída pelo `data_io` e `config.yaml`).
        * Injeção de Dependência (Como `main.py` constrói e injeta dependências).

3.  **Estrutura do Projeto:**
    * Uma breve visualização em `tree` das pastas mais importantes (`core/`, `jobs/`, `transforms/`, `utils/`, `tests/`) com uma descrição de uma linha para cada.

4.  **Pré-requisitos:**
    * Lista de ferramentas que o desenvolvedor precisa ter instaladas (ex: Python 3.10+, Docker, Make).

5.  **Configuração do Ambiente de Desenvolvimento:**
    * Passo a passo claro:
        1.  Clonar o repositório.
        2.  Criar o ambiente virtual (ex: `python -m venv .venv`).
        3.  Instalar as dependências (incluindo as de "dev") usando o `pyproject.toml` (ex: `pip install -e .[dev]`).  As dependências de produção são pyspark, pyyaml e pydantic. As dependências de desenvolvimento incluem pytest, pytest-mock, black, flake8 e build
        4.  Copiar o `config.yaml.template` para `config.yaml` (se houver).

6.  **Comandos Principais (Interface do `Makefile`):**
    * Como executar as tarefas mais comuns (use blocos de código):
        * Rodar Testes Unitários (`make test`).
        * Rodar Linters e Formatadores (`make lint`).

7.  **Empacotamento (Build) para Produção:**
    * Instruções claras sobre como gerar os artefatos de produção:
        * **Opção 1: Imagem Docker:** `make build` (explicando que isso roda os testes e cria a imagem final de produção).
        * **Opção 2: Pacote Wheel:** `python -m build` (explicando que isso gera o `.whl` na pasta `dist/`).

8.  **Execução e Submissão em Produção (O Ponto Crítico):**
    * Mostre os dois principais cenários de execução:
    * **Cenário A: Executando como um Container Docker**
        * Mostre o comando `docker run`.
        * **Importante:** Mostre como montar o `config.yaml` externo como um volume (ex: `-v $(pwd)/config.yaml:/app/config.yaml`).
    * **Cenário B: Submetendo via `spark-submit` (usando o Wheel)**
        * Mostre um exemplo de comando `spark-submit` que usa o pacote `.whl` que construímos.
        * O comando deve incluir flags essenciais como:
            * `--master <yarn/k8s/etc>`
            * `--deploy-mode <cluster/client>`
            * `--py-files dist/nosso_projeto-0.1.0-py3-none-any.whl` (para distribuir o pacote para os executores).
            * O script principal a ser executado (ex: `main.py`).
        * Explique que o `main.py` e o `config.yaml` precisam estar acessíveis no nó de submissão.

9.  **Configuração:**
    * Explique brevemente como o `config.yaml` funciona e quais são as principais chaves (ex: `appName`, `data_catalog`).

Gere o `README.md` completo com base nessas diretrizes.



---
## Playbooks de GenAI para Projetos PySpark: Automatizando o Ciclo de Vida de Desenvolvimento

No universo de desenvolvimento de software, especialmente em projetos que utilizam PySpark para processamento de grandes volumes de dados, os **playbooks para uso de GenAI** emergem como guias estratégicos e práticos para a automação de diversas fases do ciclo de vida de desenvolvimento. Estes playbooks não são ferramentas únicas, mas sim um conjunto de diretrizes, melhores práticas e fluxos de trabalho que integram modelos de linguagem de grande escala (LLMs) e outras técnicas de Inteligência Artificial Generativa para otimizar a criação de código, testes automatizados e a geração de massa de dados para testes.

A adoção de GenAI em projetos PySpark visa aumentar a produtividade dos desenvolvedores, melhorar a qualidade do código, acelerar a detecção de bugs e garantir uma cobertura de testes mais robusta, tudo isso enquanto lida com a complexidade inerente ao processamento de dados distribuídos.

### Geração Automática de Código PySpark

A geração automática de código com GenAI em PySpark funciona como um assistente de programação avançado. Desenvolvedores podem descrever a lógica de transformação de dados em linguagem natural e receber como resultado o código PySpark correspondente.

**Como funciona em um playbook:**

* **Prompts Estruturados:** O playbook define templates de prompts que guiam o desenvolvedor a fornecer o contexto necessário para a GenAI. Por exemplo, um prompt poderia incluir: o esquema do DataFrame de entrada, o esquema desejado para o DataFrame de saída e uma descrição detalhada da transformação ("Agrupar por 'categoria' e calcular a média da 'venda'").
* **Ferramentas e Bibliotecas:** Recomenda-se o uso de ferramentas específicas como o `pyspark-ai`, uma biblioteca que integra LLMs diretamente no ambiente Spark, permitindo a geração de código a partir de comandos em inglês. Outras ferramentas como o Copilot da GitHub e modelos avançados como o GPT-4 da OpenAI e o DBRX da Databricks também são peças centrais nesses playbooks.
* **Refinamento Iterativo:** O processo é interativo. O código gerado pela IA pode não ser perfeito na primeira tentativa. O playbook orienta o desenvolvedor a revisar, depurar e refinar o código, fornecendo feedback à GenAI para melhorar iterações futuras.
* **Migração de Código:** Playbooks também podem abordar a migração de código de outras linguagens (como SQL ou Pandas) para PySpark, utilizando a GenAI para traduzir a lógica de forma eficiente.

**Exemplo de caso de uso:** Um analista de dados precisa limpar e transformar um grande conjunto de dados. Em vez de escrever manualmente o código PySpark, ele pode instruir a GenAI: "Carregue o arquivo parquet 'dados_brutos'. Renomeie a coluna 'id_cliente' para 'identificador_cliente'. Converta a coluna 'data_transacao' para o formato de data. Remova linhas onde a coluna 'valor' seja nula."

### Geração de Testes Automatizados

A GenAI pode acelerar significativamente a criação de testes unitários e de integração para o código PySpark, garantindo que as transformações de dados se comportem como o esperado.

**Como funciona em um playbook:**

* **Análise de Código:** O playbook sugere o uso de ferramentas de GenAI que podem analisar uma função PySpark existente e gerar casos de teste que cubram diferentes cenários, incluindo casos de borda (edge cases).
* **Geração de Asserções:** A IA pode criar as asserções necessárias para verificar o resultado de uma transformação. Por exemplo, após uma operação de agregação, a GenAI pode gerar um teste que verifica se o número de linhas do DataFrame resultante está correto.
* **Criação de Mocks:** Para testes de integração que dependem de fontes de dados externas, a GenAI pode auxiliar na criação de *mocks* (objetos simulados) para isolar o código que está sendo testado.
* **Elaboração de Planos de Teste:** Em um nível mais alto, a GenAI pode ajudar a gerar planos de teste completos a partir de documentos de requisitos ou especificações funcionais, delineando os diferentes cenários que precisam ser validados.

### Geração de Massa de Testes (Dados Sintéticos)

Em ambientes de big data, ter acesso a dados de teste realistas e em volume adequado é um desafio constante, especialmente devido a questões de privacidade e sensibilidade dos dados de produção. A GenAI oferece uma solução poderosa para a criação de dados sintéticos.

**Como funciona em um playbook:**

* **Perfilamento de Dados:** O playbook recomenda iniciar com uma análise do esquema e da distribuição dos dados de produção. Ferramentas de GenAI podem ser alimentadas com esses metadados.
* **Geração de Dados Realistas:** A partir do perfil dos dados, a IA pode gerar um volume massivo de dados sintéticos que mantêm as mesmas características estatísticas e relacionamentos dos dados reais, sem expor informações sensíveis. Isso é crucial para testar o desempenho e a escalabilidade dos jobs PySpark.
* **Criação de Cenários Específicos:** É possível instruir a GenAI a gerar dados que simulem cenários de erro ou condições de borda específicas que podem não estar presentes no conjunto de dados de produção, permitindo testes mais abrangentes.
* **Integração com Ferramentas de Teste:** O playbook orienta como integrar a geração desses dados sintéticos diretamente nos pipelines de CI/CD (Integração Contínua/Entrega Contínua), garantindo que os testes sejam sempre executados com uma massa de dados relevante e atualizada.

### Estrutura Típica de um Playbook de GenAI para PySpark

Um playbook bem estruturado geralmente é dividido nas seguintes seções:

1.  **Visão Geral e Objetivos:** Explica os benefícios esperados da adoção de GenAI no projeto PySpark.
2.  **Ferramentas e Configuração do Ambiente:** Detalha as bibliotecas, APIs e extensões de IDE a serem utilizadas.
3.  **Fluxos de Trabalho por Atividade:**
    * **Desenvolvimento:** Passos e exemplos de prompts para geração e refatoração de código.
    * **Testes:** Diretrizes para a geração de testes unitários e de integração.
    * **Dados de Teste:** Processo para a criação de massa de dados sintéticos.
4.  **Melhores Práticas e Padrões:**
    * **Engenharia de Prompt:** Como escrever prompts eficazes para obter os melhores resultados.
    * **Segurança e Governança:** Como utilizar a GenAI de forma segura, garantindo que código e dados sensíveis não sejam expostos.
    * **Revisão Humana:** A importância da supervisão e validação do código e dos testes gerados pela IA.
5.  **Métricas de Sucesso:** Como medir o impacto da implementação da GenAI, por exemplo, através da redução do tempo de desenvolvimento, aumento da cobertura de testes e diminuição do número de bugs em produção.

Em suma, os playbooks de GenAI para projetos PySpark são um roteiro para a modernização do desenvolvimento de software de dados, capacitando as equipes a entregar soluções mais robustas e inovadoras em um ritmo acelerado.

# Parabéns
Parabéns por dominar a programação moderna em Engenharia de Dados, integrando de PySpark e Spark SQL ao poder da GenAI. Lembre-se que a Engenharia de Prompt é agora sua ferramenta mais estratégica para codificar, depurar e documentar com velocidade. Continue aprendendo e aplique essa produtividade para construir o futuro dos dados.<br>
Bons projetos pra você! ;)
