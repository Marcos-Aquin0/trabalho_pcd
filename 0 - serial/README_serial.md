# Projeto K-means 1D - Versão Sequencial (Baseline)

Este diretório contém a implementação da versão sequencial (não paralela) do algoritmo K-means 1D, executada em um ambiente Google Colab (`Serial.ipynb`). O objetivo deste código é servir como a **linha de base (baseline)** para todas as medições de desempenho e cálculos de *speedup* das versões paralelizadas (OpenMP, CUDA, MPI) do projeto.

O fluxo de trabalho, contido no notebook `Serial.ipynb`, é responsável por:
1.  Pré-processar um grande conjunto de dados de temperatura (`city_temperature.csv`).
2.  Definir um conjunto fixo de centróides iniciais para garantir a reprodutibilidade.
3.  Executar o K-means sequencial e medir seu tempo de execução.
4.  Gerar um gráfico de convergência do SSE (Soma dos Erros Quadráticos).

## Pré-requisitos

Para executar este notebook em um ambiente como o Google Colab ou localmente, você precisará de:
* Um compilador C (ex: `gcc`).
* Python 3.
* Bibliotecas Python: `pandas` e `matplotlib`.
* O arquivo de dados original: `city_temperature.csv`.

## Fluxo de Execução do Notebook

O notebook `Serial.ipynb` é dividido nas seguintes etapas:

### 1. Preparação dos Dados

A primeira célula de código Python usa a biblioteca `pandas` para:
* Ler o arquivo `city_temperature.csv`.
* Selecionar a coluna `AvgTemperature`.
* Limpar os dados, tratando valores inválidos (`-99`) como `NA` e removendo-os.
* Salvar os dados limpos em `dados.csv`, que servirá como entrada para o programa em C.

### 2. Inicialização dos Centróides

Para garantir que todas as execuções (sequencial e paralelas) comecem das mesmas condições, um arquivo fixo de centróides é utilizado.

* **`inicializar_centroides.c`**: Um programa utilitário em C é fornecido para demonstrar como os centróides iniciais podem ser selecionados aleatoriamente a partir dos dados, usando uma semente fixa (`42`) para reprodutibilidade.
* **`centroides_iniciais.csv`**: Este arquivo é escrito diretamente pela célula do notebook e contém os 16 centróides iniciais que serão lidos pelo programa principal.

### 3. Código K-means Sequencial (`kmeans_1d_naive.c`)

Este é o núcleo do *baseline*. O programa `kmeans_1d_naive.c` é um código C99 que:
* Lê os dados de `dados.csv` e os centróides de `centroides_iniciais.csv`.
* Recebe argumentos de linha de comando para `max_iter`, `eps` e nomes de arquivos de saída.
* Implementa a lógica do K-means de forma iterativa:
    * `assignment_step_1d()`: Atribui cada ponto ao centróide mais próximo e calcula o SSE.
    * `update_step_1d()`: Recalcula a média de cada cluster para definir os novos centróides.
* Mede o tempo de execução total usando a função `clock()`.
* Salva o SSE de cada iteração no arquivo `sse_evolution_seq.csv` para análise de convergência.

### 4. Compilação e Execução

O código C é compilado e executado através de uma célula `%%shell`:

1.  **Compilação**: O código é compilado com o `gcc` e a flag de otimização `-O2`.
    ```bash
    gcc -O2 -std=c99 kmeans_1d_naive.c -o kmeans_1d_naive -lm
    ```
2.  **Execução**: O programa é executado passando os arquivos de entrada e parâmetros.
    ```bash
    ./kmeans_1d_naive dados.csv centroides_iniciais.csv 500 0.0001 assign.csv centroids.csv
    ```

### 5. Análise de Convergência

A célula final do notebook usa `pandas` e `matplotlib` para ler o arquivo `sse_evolution_seq.csv` e plotar um gráfico. Este gráfico mostra como o SSE (Soma dos Erros Quadráticos) diminui a cada iteração, validando visualmente que o algoritmo está convergindo para uma solução.

## Arquivos Gerados

* `dados.csv`: O conjunto de dados de entrada limpo (temperaturas 1D).
* `centroides_iniciais.csv`: Os 16 centróides iniciais fixos.
* `kmeans_1d_naive`: O executável C da versão sequencial.
* `assign.csv`: (Saída) O índice do cluster final para cada ponto de dado.
* `centroids.csv`: (Saída) As coordenadas finais dos 16 centróides após a convergência.
* `sse_evolution_seq.csv`: (Saída) O log do valor do SSE em cada iteração, usado para gerar o gráfico.
