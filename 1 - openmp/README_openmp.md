# Projeto K-means 1D - Etapa 1: Paralelização com OpenMP

Este diretório contém a implementação da **Etapa 1** do projeto: a paralelização do K-means 1D usando OpenMP para CPUs de memória compartilhada.

O notebook `OpenMP.ipynb` gerencia todo o fluxo de trabalho, desde a preparação dos dados até a execução de um benchmark detalhado e a validação dos resultados.

## Estrutura do Notebook

O notebook `OpenMP.ipynb` está organizado nas seguintes seções:

1.  **Configuração:** Prepara os arquivos de entrada `dados.csv` (a partir do `city_temperature.csv`) e `centroides_iniciais.csv` (fixos para reprodutibilidade).
2.  **Implementação (Opção A: `reduction`):** Contém o código-fonte `kmeans_1d_naive_parallel.c`. Esta é a implementação principal, que paraleliza ambos os laços `assignment_step` e `update_step` usando a cláusula `reduction` do OpenMP para garantir a segurança das threads de forma eficiente.
3.  **Benchmark (Opção A):** Um script de shell robusto que compila e executa a versão "Reduction". Ele testa sistematicamente o desempenho em várias configurações.
4.  **Implementação (Opção B: `critical`):** Contém o código-fonte `kmeans_1d_naive_parallel_critical.c`. Esta é uma implementação alternativa para fins de comparação, que usa `#pragma omp critical` na etapa de `update_step`, criando um gargalo de serialização proposital.
5.  **Benchmark (Opção B):** O mesmo script de shell adaptado para testar e coletar métricas da versão "Critical".
6.  **Análise de Convergência:** Uma célula Python que plota os gráficos de SSE por iteração de todas as execuções, validando visualmente que todas as versões (sequencial e paralelas) convergem para o mesmo resultado numérico.

## Implementações Paralelas

Duas abordagens de paralelização para o `update_step` foram implementadas para análise comparativa:

* **`kmeans_1d_naive_parallel.c` (Opção A):** Utiliza `reduction(+:sum[0:K]) reduction(+:cnt[0:K])`. Esta é a abordagem de alta performance, que permite a cada thread ter suas próprias cópias locais dos vetores de soma/contagem, combinando-os apenas no final.
* **`kmeans_1d_naive_parallel_critical.c` (Opção B):** Utiliza `#pragma omp critical` em torno das operações `sum[a] += ...` e `cnt[a] += ...`. Esta abordagem força que apenas uma thread por vez possa executar aquele bloco de código, criando um gargalo de serialização que deve resultar em um desempenho inferior.

Ambas as versões utilizam `reduction(+:sse)` no `assignment_step`, pois é a solução mais eficiente para essa etapa.

## Script de Benchmark

O coração da análise de desempenho é o script de shell (célula `%%shell`). Ele automatiza os testes e a coleta de métricas:

1.  **Estabelece o Baseline:** Primeiro, ele executa a versão paralela com `OMP_NUM_THREADS=1` para obter o tempo sequencial ($T_{seq}$).
2.  **Itera sobre Parâmetros:** Ele testa uma matriz de configurações, variando:
    * **Número de Threads ($T$):** $\{2, 4, 8, 16\}$
    * **Agendamento (`schedule`):** `static`, `dynamic`
    * **Tamanho do Bloco (`chunk`):** $\{1, 10, 100, 1000\}$ (apenas para `dynamic`)
3.  **Coleta Métricas:** Para cada execução, ele extrai o tempo de execução e o SSE final.
4.  **Calcula Métricas de Desempenho:**
    * **Speedup:** $S = T_{seq} / T_{paralelo}$
    * **Eficiência:** $E = S / T$ (onde $T$ é o número de threads)
5.  **Gera Relatórios:** Salva todos os resultados em `resultados.csv` e `resultados_critical.csv` para fácil análise e plotagem.

## Como Compilar e Executar Manualmente

Embora o notebook automatize tudo, os códigos podem ser compilados e executados manualmente.

**1.1 Compilação (Opção A: Reduction)**
```bash
gcc -O2 -std=c99 -fopenmp kmeans_1d_naive_parallel.c -o kmeans_parallel_reduction -lm
```

**1.2 Compilação (Opção B: Critical)**
```bash
gcc -O2 -std=c99 -fopenmp kmeans_1d_naive_parallel_critical.c -o kmeans_parallel_critical -lm
```

**1.3 Execução**
```bash
# Define 8 threads e schedule dynamic com chunk de 100
export OMP_NUM_THREADS=8
export OMP_SCHEDULE="dynamic,100"

# Executa o programa
./kmeans_parallel_reduction dados.csv centroides_iniciais.csv 500 0.0001 sse_log.csv
```
