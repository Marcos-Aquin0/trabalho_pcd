## K-means 1D - Etapa 2: Paralelização com CUDA

Este notebook e o código associado implementam a *Etapa 2* de um projeto de K-means 1D, focando na paralelização do algoritmo usando CUDA para GPUs. O objetivo é demonstrar a aceleração computacional obtida com o uso da GPU em comparação com uma versão sequencial (CPU).

## 1. Visão Geral do Projeto

O projeto K-means 1D visa agrupar pontos de dados unidimensionais em K clusters. A Etapa 2 concentra-se em otimizar as etapas do algoritmo para execução em GPU, com um benchmark detalhado para avaliar o desempenho.

### Arquivos Utilizados

- dados.csv: Arquivo contendo os pontos de dados unidimensionais (temperaturas médias), derivado de city_temperature.csv após limpeza.
- centroides_iniciais.csv: Arquivo contendo os 16 centróides iniciais para o algoritmo.

## 2. Implementação CUDA (kmeans_1d_cuda.cu)

A implementação CUDA foi projetada para otimizar o Assignment Step (Etapa de Atribuição) na GPU, enquanto o Update Step (Etapa de Atualização) e a redução do SSE são realizados no Host (CPU).

### Componentes Principais:

- **__constant__ double d_C[16]*: Os 16 centróides são armazenados na memória constante da GPU. Esta memória é otimizada para leituras frequentes e rápidas por todos os *threads dentro de um warp.
- **assignment_kernel*: Este é o *kernel CUDA que executa o Assignment Step. Cada thread da GPU é responsável por um ponto de dado (X[i]), calculando a distância para todos os K centróides e atribuindo o ponto ao centróide mais próximo. Os resultados (assign[i] e sse_per_point[i]) são gravados na memória global da GPU.
- **update_step_1d (Host): Esta função é executada na CPU. Após a cópia dos resultados da atribuição da GPU para o Host, ela calcula os novos centróides com base nas atribuições atuais. Os novos centróides são então copiados de volta para a memória constante da GPU (cudaMemcpyToSymbol).
- **Função kmeans_1d (Host Driver): Gerencia o ciclo de iterações do K-means. Coordena o lançamento do assignment_kernel na GPU, a cópia de dados GPU-Host (D2H), a redução do SSE total no Host, a verificação de convergência e a atualização dos centróides no Host, seguido da cópia Host-GPU (H2D) dos novos centróides para a memória constante.
- **main**: Gerencia a alocação de memória (Host e Device), cópias iniciais (H2D) e mede o tempo total de execução usando cudaEvent_t para maior precisão.

## 3. Baseline Sequencial (kmeans_1d_baseline.c)

Para uma comparação de desempenho justa, foi utilizada uma versão sequencial do K-means 1D (originalmente um código OpenMP) executada com OMP_NUM_THREADS=1. O tempo de execução é medido usando omp_get_wtime() (wall-clock time), garantindo que a métrica de comparação com o CUDA seja consistente.

## 4. Benchmark e Resultados

Um script de shell automatizou a compilação e execução das versões sequencial e CUDA, coletando tempos de execução e calculando o Speedup.

### Processo do Benchmark:

1.  *Compilação do Baseline*: gcc -O2 -std=c99 -fopenmp kmeans_1d_baseline.c -o kmeans_baseline -lm
2.  *Execução do Baseline*: OMP_NUM_THREADS=1 ./kmeans_baseline dados.csv centroides_iniciais.csv 500 0.0001
    - Captura o tempo sequencial (T_seq).
3.  *Compilação CUDA*: nvcc -O2 -arch=sm_75 kmeans_1d_cuda.cu -o kmeans_cuda (otimizado para GPUs Tesla T4 no Colab).
4.  *Execução CUDA*: Testada com diferentes tamanhos de bloco (blockSize = 128, 256, 512, 1024).
    - Para cada blockSize, o kmeans_cuda é executado, e o tempo (T_cuda), SSE final e número de iterações são registrados.
5.  *Cálculo do Speedup*: Speedup = T_seq / T_cuda.
6.  *Saída*: Todos os resultados são salvos em resultados_cuda.csv e exibidos em formato tabular.

### Exemplo de Resultados Finais:

| Block_Size | Tempo(ms) | SSE_final                 | Iteracoes | Speedup |
|------------|-----------|---------------------------|-----------|---------|
| 128        | 3033.6    | 8842007.334724            | 160       | 2.6617  |
| 256        | 3042.7    | 8842007.334724            | 160       | 2.6537  |
| 512        | 3154.7    | 8842007.334724            | 160       | 2.5595  |
| 1024       | 3110.0    | 8842007.334724            | 160       | 2.5963  |

### Análise dos Resultados

Os resultados mostram que a implementação CUDA oferece um speedup significativo em comparação com a versão sequencial, com valores em torno de 2.5x a 2.6x. Pequenas variações no blockSize influenciam o desempenho, indicando a importância de otimizar este parâmetro para a GPU específica e a carga de trabalho. No exemplo acima, um blockSize de 128 ou 256 apresentou o melhor desempenho. O SSE final e o número de iterações são consistentes entre todas as execuções, demonstrando que a correção do algoritmo é mantida.