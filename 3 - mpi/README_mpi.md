# Projeto PCD: K-means 1D - Paralelização com MPI (Etapa 3)

Este repositório contém os artefatos da **Etapa 3** do projeto de Programação Concorrente e Distribuída (UNIFESP). O objetivo desta etapa foi implementar a paralelização distribuída do algoritmo K-means unidimensional utilizando a biblioteca **MPI (Message Passing Interface)**.

## 📋 Descrição do Projeto

O algoritmo K-means foi aplicado para agrupar dados de temperaturas globais (Dataset *City Temperature*). A implementação foca na divisão do domínio de dados entre múltiplos processos para acelerar a fase de *Assignment* (atribuição de pontos aos centróides mais próximos).

As principais características desta implementação são:
* **Linguagem:** C (Standard C99).
* **Biblioteca Paralela:** OpenMPI.
* **Estratégia:** Mestre-Escravo para I/O e Single-Program-Multiple-Data (SPMD) para o cálculo.
* **Comunicação:** * `MPI_Bcast`: Distribuição de parâmetros e centróides.
    * `MPI_Scatterv`: Distribuição balanceada dos dados (vetor de temperaturas).
    * `MPI_Allreduce`: Agregação de somas parciais e contagens para atualização dos centróides.

## 📂 Estrutura dos Arquivos

* `projeto_kmeans.ipynb`: Notebook principal contendo o código fonte (C), scripts de compilação e execução.
* `dados.csv`: Arquivo de entrada (gerado pelo notebook) contendo as temperaturas extraídas.
* `centroides_iniciais.csv`: Arquivo com os 16 centróides iniciais fixos (para reprodutibilidade).

## 🚀 Como Executar

Este projeto foi desenvolvido para rodar em ambiente Linux (ou Google Colab) com o compilador `mpicc` instalado.

### 1. Pré-requisitos
Instale o OpenMPI:
```bash
sudo apt-get update
sudo apt-get install -y openmpi-bin libopenmpi-dev
```

### 2. Compilação
Compile o código MPI gerado pelo notebook:
```bash
mpicc -O2 -std=c99 kmeans_1d_mpi.c -o kmeans_1d_mpi -lm
```

### 3. Execução
Para rodar com diferentes números de processadores (ex: 4):
```bash
mpirun --allow-run-as-root -n 4 ./kmeans_1d_mpi dados.csv centroides_iniciais.csv
```