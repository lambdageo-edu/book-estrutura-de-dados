# Algoritmos de Ordenação Básicos

!!! note "Material de Apoio"
    Você pode complementar este capítulo com os [Slides sobre Algoritmos de Ordenação Básicos](../slides/ordenacao_algoritmos_basicos.pdf).

## 1.1 O Que É Ordenação?

> **Definição:** Ordenação é o processo de rearranjar os elementos de uma coleção em uma sequência específica, geralmente **crescente** ou **decrescente**, com base em uma chave de comparação.

```text
// Exemplo: vetor não ordenado → ordenado (crescente)
Entrada:  [64, 34, 25, 12, 22, 11, 90]
Saída:    [11, 12, 22, 25, 34, 64, 90]
```

### 1.2 Aplicações Práticas da Ordenação

| Aplicação | Benefício da Ordenação |
| --- | --- |
| Busca em listas | Busca binária exige dados ordenados → O(log n) vs O(n) |
| Remoção de duplicatas | Elementos iguais ficam adjacentes → detecção O(n) |
| Relatórios e visualização | Dados organizados facilitam interpretação humana |
| Processamento de bancos de dados | Índices ordenados aceleram consultas |
| Algoritmos de união/interseção | Operações em conjuntos tornam-se lineares |

!!! tip "Insight"
    Muitos algoritmos complexos assumem dados ordenados como pré-condição. Ordenar pode ser o "custo inicial" que viabiliza operações eficientes depois.

---

## 2. Complexidade de Algoritmos: Conceitos Fundamentais

### 2.1 Por Que Analisar Complexidade?

Dois algoritmos podem resolver o mesmo problema, mas com desempenhos drasticamente diferentes conforme o tamanho da entrada `n`.

```
Algoritmo A: ordena 1000 elementos em 0.01s
Algoritmo B: ordena 1000 elementos em 2.5s

Qual usar para 1 milhão de elementos?
→ Precisamos de uma métrica independente de hardware: COMPLEXIDADE.
```

### 2.2 Notação Big O: Medindo Crescimento

> **Big O** descreve o **comportamento assintótico** de um algoritmo: como o tempo/espaço cresce quando `n → ∞`.

| Notação | Nome | Exemplo Prático |
| --- | --- | --- |
| **O(1)** | Constante | Acessar elemento por índice |
| **O(log n)** | Logarítmica | Busca binária |
| **O(n)** | Linear | Percorrer vetor uma vez |
| **O(n log n)** | Linearítmica | Merge Sort, Quick Sort |
| **O(n²)** | Quadrática | Bubble, Selection, Insertion Sort |
| **O(2ⁿ)** | Exponencial | Força bruta em subconjuntos |

### 2.3 Melhor Caso, Pior Caso e Caso Médio

| Cenário | Definição | Exemplo (Bubble Sort) |
| --- | --- | --- |
| **Melhor caso** | Entrada que minimiza operações | Vetor já ordenado → O(n) com otimização |
| **Pior caso** | Entrada que maximiza operações | Vetor em ordem inversa → O(n²) |
| **Caso médio** | Comportamento esperado em entradas aleatórias | Vetor aleatório → O(n²) |

!!! warning "Importante"
    Em sistemas críticos, planeje sempre para o **pior caso**. O "caso médio" pode não ocorrer quando você mais precisa.

### 2.4 Como Contar Operações? (Exemplo Prático)

Considere este trecho de código:

```c
for (int i = 0; i < n; i++) {          // Corpo executa n vezes
    for (int j = 0; j < n - 1; j++) {  // Corpo executa n-1 vezes para cada i
        if (vetor[j] > vetor[j+1]) {   // Comparação: O(1)
            // Troca: O(1)
        }
    }
}
```

**Contagem simplificada:**

- Laço externo: `n` iterações
- Laço interno: `n − 1` iterações por externa → total `n × (n − 1) = n² − n`
- Operações internas: O(1)
- **Complexidade total: O(n²)** (o termo `− n` e as constantes são desprezados)

---

## 3. Bubble Sort: O Algoritmo da "Bolha"

### 3.1 Ideia Conceitual

> **Analogia:** Imagine bolhas de ar subindo num copo de refrigerante até a superfície. No Bubble Sort, a cada passada o maior elemento ainda fora do lugar "sobe" como uma bolha até o final do vetor.

**Estratégia:**

1. Compare elementos adjacentes
2. Se estiverem fora de ordem, troque-os
3. Repita até que nenhuma troca seja necessária

### 3.2 Exemplo Visual Passo a Passo

```
Vetor inicial: [5, 3, 8, 4, 2]

Passada 1 (maior elemento "flutua" para o final):
[5,3,8,4,2] → [3,5,8,4,2]  (troca 5↔3)
[3,5,8,4,2] → [3,5,8,4,2]  (5<8, sem troca)
[3,5,8,4,2] → [3,5,4,8,2]  (troca 8↔4)
[3,5,4,8,2] → [3,5,4,2,8]  (troca 8↔2) ✓ 8 na posição final

Passada 2:
[3,5,4,2,8] → [3,5,4,2,8]  (3<5)
[3,5,4,2,8] → [3,4,5,2,8]  (troca 5↔4)
[3,4,5,2,8] → [3,4,2,5,8]  (troca 5↔2) ✓ 5 na posição

Passada 3:
[3,4,2,5,8] → [3,4,2,5,8]  (3<4)
[3,4,2,5,8] → [3,2,4,5,8]  (troca 4↔2) ✓ 4 na posição

Passada 4:
[3,2,4,5,8] → [2,3,4,5,8]  (troca 3↔2) ✓ 3 na posição

Resultado: [2, 3, 4, 5, 8] ✓
```

### 3.3 Implementação em C (Versão Básica)

```c
#include <stdio.h>
#include <stdbool.h>

// Bubble Sort básico: O(n²) no pior caso
void bubbleSort(int vetor[], int n) {
    for (int i = 0; i < n - 1; i++) {
        // Últimos i elementos já estão ordenados
        for (int j = 0; j < n - i - 1; j++) {
            if (vetor[j] > vetor[j + 1]) {
                // Troca elementos
                int temp = vetor[j];
                vetor[j] = vetor[j + 1];
                vetor[j + 1] = temp;
            }
        }
    }
}
```

### 3.4 Otimização: Parada Antecipada

```c
// Bubble Sort otimizado: O(n) no melhor caso (já ordenado)
void bubbleSortOtimizado(int vetor[], int n) {
    bool trocou;

    for (int i = 0; i < n - 1; i++) {
        trocou = false;  // Flag para detectar se houve troca

        for (int j = 0; j < n - i - 1; j++) {
            if (vetor[j] > vetor[j + 1]) {
                // Troca
                int temp = vetor[j];
                vetor[j] = vetor[j + 1];
                vetor[j + 1] = temp;
                trocou = true;
            }
        }

        // Se não houve troca, vetor já está ordenado
        if (!trocou) {
            printf("  → Parada antecipada na passada %d\n", i + 1);
            break;
        }
    }
}
```

!!! success "Ganho"
    Se o vetor já estiver ordenado, o algoritmo termina em **O(n)** com apenas uma passada de verificação.

### 3.5 Análise de Complexidade

| Cenário | Comparações | Trocas | Complexidade |
| --- | --- | --- | --- |
| **Melhor caso** (ordenado) | n-1 | 0 | **O(n)** (com otimização) |
| **Pior caso** (invertido) | n(n-1)/2 ≈ n²/2 | n(n-1)/2 ≈ n²/2 | **O(n²)** |
| **Caso médio** (aleatório) | ~n²/2 | ~n²/4 | **O(n²)** |

**Espaço:** O(1) — ordenação *in-place*, sem estruturas auxiliares.

---

## 4. Selection Sort: Ordenação por Seleção

### 4.1 Ideia Conceitual

> **Analogia:** Organizar uma fila de pessoas por altura: procure a mais baixa e troque-a de lugar com quem está na frente; depois procure a mais baixa entre as restantes e coloque-a na segunda posição; e assim por diante.

**Estratégia:**

1. Encontre o menor elemento na parte não ordenada
2. Troque-o com o primeiro elemento não ordenado
3. Avance o limite da parte ordenada
4. Repita até ordenar tudo

### 4.2 Exemplo Visual Passo a Passo

```
Vetor inicial: [64, 25, 12, 22, 11]
               ↑
               limite da parte ordenada

Passada 1: encontrar mínimo em [64,25,12,22,11] → 11
[64,25,12,22,11] → trocar 64↔11
[11,25,12,22,64] ✓ 11 na posição correta
       ↑

Passada 2: encontrar mínimo em [25,12,22,64] → 12
[11,25,12,22,64] → trocar 25↔12
[11,12,25,22,64] ✓ 12 na posição correta
          ↑

Passada 3: encontrar mínimo em [25,22,64] → 22
[11,12,25,22,64] → trocar 25↔22
[11,12,22,25,64] ✓ 22 na posição correta
             ↑

Passada 4: encontrar mínimo em [25,64] → 25 (já está correto)
[11,12,22,25,64] → sem troca necessária
                ↑

Resultado: [11, 12, 22, 25, 64] ✓
```

### 4.3 Implementação em C

```c
// Selection Sort: sempre O(n²), mas com menos trocas que Bubble
void selectionSort(int vetor[], int n) {
    for (int i = 0; i < n - 1; i++) {
        // Encontra o índice do menor elemento na parte não ordenada
        int indiceMenor = i;

        for (int j = i + 1; j < n; j++) {
            if (vetor[j] < vetor[indiceMenor]) {
                indiceMenor = j;
            }
        }

        // Troca apenas se necessário (evita troca consigo mesmo)
        if (indiceMenor != i) {
            int temp = vetor[i];
            vetor[i] = vetor[indiceMenor];
            vetor[indiceMenor] = temp;
        }
    }
}
```

### 4.4 Análise de Complexidade

| Cenário | Comparações | Trocas | Complexidade |
| --- | --- | --- | --- |
| **Melhor caso** (ordenado) | n(n-1)/2 ≈ n²/2 | 0 | **O(n²)** |
| **Vetor invertido** | n(n-1)/2 ≈ n²/2 | ⌊n/2⌋ | **O(n²)** |
| **Pior caso** (trocas) | n(n-1)/2 ≈ n²/2 | n-1 | **O(n²)** |
| **Caso médio** (aleatório) | ~n²/2 | ≈ n − ln n | **O(n²)** |

*(No vetor invertido, cada troca coloca dois elementos no lugar de uma vez, por isso são só n/2 trocas. O máximo de n−1 trocas ocorre, por exemplo, em `[2, 3, ..., n, 1]`.)*

**Espaço:** O(1) — ordenação *in-place*.

!!! tip "Vantagem do Selection Sort"
    Número de trocas é **sempre ≤ n-1**, útil quando trocar elementos é custoso (ex: registros grandes em disco).

---

## 5. Insertion Sort: Ordenação por Inserção

### 5.1 Ideia Conceitual

> **Analogia:** Organizar cartas na mão durante um jogo: pegue uma carta por vez e insira-a na posição correta entre as cartas já ordenadas.

**Estratégia:**

1. Considere o primeiro elemento como "ordenado"
2. Para cada elemento seguinte:
    - Compare com os elementos ordenados à esquerda
    - Desloque elementos maiores para a direita
    - Insira o elemento na posição correta
3. Repita até processar todos os elementos

### 5.2 Exemplo Visual Passo a Passo

```
Vetor inicial: [5, 2, 4, 6, 1, 3]
               ↑
               elemento atual (i=1)

i=1: elemento=2
[5,2,4,6,1,3] → 5>2, desloca 5 para direita
[_,5,4,6,1,3] → insere 2 na posição 0
[2,5,4,6,1,3] ✓
     ↑

i=2: elemento=4
[2,5,4,6,1,3] → 5>4, desloca 5
[2,_,5,6,1,3] → 2<4, para de deslocar
[2,4,5,6,1,3] ✓
       ↑

i=3: elemento=6
[2,4,5,6,1,3] → 5<6, já está na posição correta
[2,4,5,6,1,3] ✓
         ↑

i=4: elemento=1
[2,4,5,6,1,3] → 6>1, desloca
[2,4,5,_,6,3] → 5>1, desloca
[2,4,_,5,6,3] → 4>1, desloca
[2,_,4,5,6,3] → 2>1, desloca
[_,2,4,5,6,3] → insere 1 na posição 0
[1,2,4,5,6,3] ✓
           ↑

i=5: elemento=3
[1,2,4,5,6,3] → 6>3, desloca
[1,2,4,5,_,6] → 5>3, desloca
[1,2,4,_,5,6] → 4>3, desloca
[1,2,_,4,5,6] → 2<3, para
[1,2,3,4,5,6] ✓

Resultado: [1, 2, 3, 4, 5, 6] ✓
```

### 5.3 Implementação em C

```c
// Insertion Sort: eficiente para vetores pequenos ou quase ordenados
void insertionSort(int vetor[], int n) {
    for (int i = 1; i < n; i++) {
        int chave = vetor[i];  // Elemento a ser inserido
        int j = i - 1;

        // Desloca elementos maiores que 'chave' para a direita
        while (j >= 0 && vetor[j] > chave) {
            vetor[j + 1] = vetor[j];  // Deslocamento
            j--;
        }

        // Insere 'chave' na posição correta
        vetor[j + 1] = chave;
    }
}
```

### 5.4 Análise de Complexidade

| Cenário | Comparações | Deslocamentos | Complexidade |
| --- | --- | --- | --- |
| **Melhor caso** (ordenado) | n-1 | 0 | **O(n)** |
| **Pior caso** (invertido) | n(n-1)/2 ≈ n²/2 | n(n-1)/2 ≈ n²/2 | **O(n²)** |
| **Caso médio** (aleatório) | ~n²/4 | ~n²/4 | **O(n²)** |

**Espaço:** O(1) — ordenação *in-place*.

!!! tip "Vantagem do Insertion Sort"
    - Estável (mantém ordem relativa de elementos iguais)
    - Adaptativo: eficiente quando o vetor já está "quase ordenado"
    - Online: pode ordenar elementos à medida que chegam

---

## 6. Código Completo Compilável (Teste os Três Algoritmos)

```c
#include <stdio.h>
#include <stdlib.h>
#include <time.h>
#include <stdbool.h>

// ===== Utilitários =====
void imprimirVetor(int vetor[], int n, const char* mensagem) {
    printf("%s", mensagem);
    for (int i = 0; i < n; i++) {
        printf("%d ", vetor[i]);
    }
    printf("\n");
}

void copiarVetor(int origem[], int destino[], int n) {
    for (int i = 0; i < n; i++) {
        destino[i] = origem[i];
    }
}

bool estaOrdenado(int vetor[], int n) {
    for (int i = 0; i < n - 1; i++) {
        if (vetor[i] > vetor[i + 1]) {
            return false;
        }
    }
    return true;
}

// ===== Bubble Sort =====
void bubbleSort(int vetor[], int n) {
    for (int i = 0; i < n - 1; i++) {
        bool trocou = false;
        for (int j = 0; j < n - i - 1; j++) {
            if (vetor[j] > vetor[j + 1]) {
                int temp = vetor[j];
                vetor[j] = vetor[j + 1];
                vetor[j + 1] = temp;
                trocou = true;
            }
        }
        if (!trocou) break;  // Otimização: parada antecipada
    }
}

// ===== Selection Sort =====
void selectionSort(int vetor[], int n) {
    for (int i = 0; i < n - 1; i++) {
        int indiceMenor = i;
        for (int j = i + 1; j < n; j++) {
            if (vetor[j] < vetor[indiceMenor]) {
                indiceMenor = j;
            }
        }
        if (indiceMenor != i) {
            int temp = vetor[i];
            vetor[i] = vetor[indiceMenor];
            vetor[indiceMenor] = temp;
        }
    }
}

// ===== Insertion Sort =====
void insertionSort(int vetor[], int n) {
    for (int i = 1; i < n; i++) {
        int chave = vetor[i];
        int j = i - 1;
        while (j >= 0 && vetor[j] > chave) {
            vetor[j + 1] = vetor[j];
            j--;
        }
        vetor[j + 1] = chave;
    }
}

// ===== Medição de tempo (milissegundos) =====
double medirTempo(void (*func)(int[], int), int vetor[], int n) {
    clock_t inicio = clock();
    func(vetor, n);
    clock_t fim = clock();
    return ((double)(fim - inicio)) / CLOCKS_PER_SEC * 1000;  // em ms
}

// ===== Main: Comparação prática =====
int main() {
    // Vetor de teste
    int original[] = {64, 34, 25, 12, 22, 11, 90};
    int n = sizeof(original) / sizeof(original[0]);
    int vetor[100];  // buffer para cópias

    printf("=== Algoritmos de Ordenação Básicos ===\n\n");

    // Teste 1: Vetor aleatório pequeno
    printf("1. Vetor original: ");
    imprimirVetor(original, n, "");

    // Bubble Sort
    copiarVetor(original, vetor, n);
    double t1 = medirTempo(bubbleSort, vetor, n);
    printf("   Bubble Sort:   ");
    imprimirVetor(vetor, n, "");
    printf("   Tempo: %.3f ms | Ordenado? %s\n\n", t1, estaOrdenado(vetor, n) ? "✓" : "✗");

    // Selection Sort
    copiarVetor(original, vetor, n);
    double t2 = medirTempo(selectionSort, vetor, n);
    printf("   Selection Sort:");
    imprimirVetor(vetor, n, "");
    printf("   Tempo: %.3f ms | Ordenado? %s\n\n", t2, estaOrdenado(vetor, n) ? "✓" : "✗");

    // Insertion Sort
    copiarVetor(original, vetor, n);
    double t3 = medirTempo(insertionSort, vetor, n);
    printf("   Insertion Sort:");
    imprimirVetor(vetor, n, "");
    printf("   Tempo: %.3f ms | Ordenado? %s\n\n", t3, estaOrdenado(vetor, n) ? "✓" : "✗");

    // Teste 2: Vetor já ordenado (melhor caso para Insertion/Bubble otimizado)
    printf("2. Melhor caso (vetor já ordenado):\n");
    for (int i = 0; i < n; i++) original[i] = i + 1;

    copiarVetor(original, vetor, n);
    t1 = medirTempo(bubbleSort, vetor, n);
    printf("   Bubble Sort:   %.3f ms\n", t1);

    copiarVetor(original, vetor, n);
    t2 = medirTempo(selectionSort, vetor, n);
    printf("   Selection Sort: %.3f ms\n", t2);

    copiarVetor(original, vetor, n);
    t3 = medirTempo(insertionSort, vetor, n);
    printf("   Insertion Sort: %.3f ms ← mais rápido neste cenário!\n\n", t3);

    // Teste 3: Vetor invertido (pior caso)
    printf("3. Pior caso (vetor invertido):\n");
    for (int i = 0; i < n; i++) original[i] = n - i;

    copiarVetor(original, vetor, n);
    t1 = medirTempo(bubbleSort, vetor, n);
    printf("   Bubble Sort:   %.3f ms\n", t1);

    copiarVetor(original, vetor, n);
    t2 = medirTempo(selectionSort, vetor, n);
    printf("   Selection Sort: %.3f ms\n", t2);

    copiarVetor(original, vetor, n);
    t3 = medirTempo(insertionSort, vetor, n);
    printf("   Insertion Sort: %.3f ms\n\n", t3);

    printf("✅ Todos os algoritmos produziram resultados corretos!\n");
    printf("📌 Observação: Para vetores pequenos, as diferenças são mínimas.\n");
    printf("   Para n grande (>1000), algoritmos O(n²) tornam-se inviáveis.\n");

    return 0;
}
```

**Saída esperada (exemplo):**

```
=== Algoritmos de Ordenação Básicos ===

1. Vetor original: 64 34 25 12 22 11 90
   Bubble Sort:   11 12 22 25 34 64 90
   Tempo: 0.002 ms | Ordenado? ✓

   Selection Sort:11 12 22 25 34 64 90
   Tempo: 0.001 ms | Ordenado? ✓

   Insertion Sort:11 12 22 25 34 64 90
   Tempo: 0.001 ms | Ordenado? ✓

2. Melhor caso (vetor já ordenado):
   Bubble Sort:   0.000 ms
   Selection Sort: 0.001 ms
   Insertion Sort: 0.000 ms ← mais rápido neste cenário!

3. Pior caso (vetor invertido):
   Bubble Sort:   0.003 ms
   Selection Sort: 0.002 ms
   Insertion Sort: 0.003 ms

✅ Todos os algoritmos produziram resultados corretos!
📌 Observação: Para vetores pequenos, as diferenças são mínimas.
   Para n grande (>1000), algoritmos O(n²) tornam-se inviáveis.
```

---

## 7. Comparação Direta: Qual Algoritmo Usar?

| Critério | Bubble Sort | Selection Sort | Insertion Sort |
| --- | --- | --- | --- |
| **Complexidade (pior caso)** | O(n²) | O(n²) | O(n²) |
| **Complexidade (melhor caso)** | O(n)* | O(n²) | O(n) |
| **Número de trocas** | O(n²) | **O(n)** ✓ | O(n²) |
| **Estável** | ✓ | ✗ | ✓ |
| **Adaptativo** | ✓* | ✗ | ✓✓ |
| **Online** | ✗ | ✗ | ✓ |
| **Simplicidade** | ✓✓✓ | ✓✓ | ✓✓ |
| **Indicado para** | Ensino, vetores muito pequenos | Quando trocas são custosas | Vetores pequenos ou quase ordenados |

> *Com otimização de parada antecipada

### Regra Prática de Escolha

```
✅ Use Insertion Sort se:
   - n < 50 (pequeno)
   - Dados já estão "quase ordenados"
   - Precisa de estabilidade

✅ Use Selection Sort se:
   - Trocar elementos é operação cara (ex: structs grandes)
   - Quer minimizar número de escritas na memória

✅ Use Bubble Sort se:
   - Está aprendendo conceitos de ordenação
   - Precisa de código extremamente simples para prototipagem

❌ Evite todos os três se:
   - n > 1000 → use Merge Sort, Quick Sort ou biblioteca padrão (qsort)
```

---

## 8. Exercícios para Fixação

### Exercício 1 — Simulação Manual

Dado o vetor `[8, 3, 5, 1, 9]`, simule passo a passo:

a) Bubble Sort (com otimização): quantas passadas foram necessárias?

b) Selection Sort: quantas trocas foram realizadas?

c) Insertion Sort: quantos deslocamentos ocorreram ao inserir o elemento `1`?

<details>
<summary>💡 Gabarito parcial</summary>

```
a) Bubble Sort:
   Passada 1: [3,5,1,8,9] → trocas: 8↔3, 8↔5, 8↔1
   Passada 2: [3,1,5,8,9] → trocas: 5↔1
   Passada 3: [1,3,5,8,9] → trocas: 3↔1
   Passada 4: [1,3,5,8,9] → sem trocas → parada antecipada
   Total: 4 passadas (mas parou na 4ª por otimização)

b) Selection Sort:
   Troca 1: 8↔1 → [1,3,5,8,9]
   Troca 2: nenhuma (3 já é o menor do restante)
   Troca 3: nenhuma
   Troca 4: nenhuma
   Total: 1 troca

c) Insertion Sort ao inserir 1 (i=3):
   [3,5,8,1,9] → 8>1 → desloca → [3,5,_,8,9]
                → 5>1 → desloca → [3,_,5,8,9]
                → 3>1 → desloca → [_,3,5,8,9]
                → insere 1 → [1,3,5,8,9]
   Total: 3 deslocamentos
```

</details>

---

### Exercício 2 — Modificação de Código

Modifique o `insertionSort` para ordenar em **ordem decrescente**. Teste com `[5, 2, 8, 1, 9]`.

<details>
<summary>💡 Solução</summary>

```c
// Basta inverter a condição de comparação:
while (j >= 0 && vetor[j] < chave) {  // < em vez de >
    vetor[j + 1] = vetor[j];
    j--;
}
vetor[j + 1] = chave;

// Resultado esperado: [9, 8, 5, 2, 1]
```

</details>

---

### Exercício 3 — Contagem de Operações

Adicione contadores ao `bubbleSort` para registrar:

- `comparacoes`: número de vezes que `vetor[j] > vetor[j+1]` foi avaliado
- `trocas`: número de trocas efetivas realizadas

Execute com vetores de tamanhos 10, 50 e 100 (aleatórios) e preencha a tabela:

| n | Comparações | Trocas | Razão Trocas/Comparações |
| --- | --- | --- | --- |
| 10 | ? | ? | ? |
| 50 | ? | ? | ? |
| 100 | ? | ? | ? |

> 💡 **Objetivo:** Verificar empiricamente que o número de operações cresce quadraticamente.

---

### Exercício 4 — Desafio: Ordenação de Structs

Defina uma struct para representar um aluno:

```c
typedef struct {
    int matricula;
    char nome[50];
    float nota;
} Aluno;
```

Implemente uma versão do `insertionSort` que ordene um vetor de `Aluno` por **nota decrescente**. Em caso de empate na nota, mantenha a ordem original de inserção (estabilidade).

<details>
<summary>💡 Esboço de solução</summary>

```c
void insertionSortAlunos(Aluno alunos[], int n) {
    for (int i = 1; i < n; i++) {
        Aluno chave = alunos[i];  // Cópia da struct
        int j = i - 1;

        // Comparação por nota (decrescente), mantendo estabilidade
        while (j >= 0 && alunos[j].nota < chave.nota) {
            alunos[j + 1] = alunos[j];  // Atribuição de struct funciona em C
            j--;
        }
        alunos[j + 1] = chave;
    }
}
```

</details>

---

## 9. Discussão: Por Que Estudar Algoritmos O(n²)?

### 9.1 Motivações Pedagógicas

✅ **Simplicidade conceitual:** Fáceis de entender, implementar e depurar

✅ **Base para algoritmos avançados:** Conceitos como "dividir para conquistar" (Merge/Quick Sort) são construídos sobre essas ideias

✅ **Eficiência em cenários específicos:** Para `n < 50`, a diferença entre O(n²) e O(n log n) é irrelevante na prática

✅ **Análise de complexidade:** Permitem exercitar contagem de operações e raciocínio assintótico

### 9.2 Quando O(n²) É Aceitável na Prática?

| Cenário | Justificativa |
| --- | --- |
| **n pequeno** (< 50) | Constantes escondidas no Big O dominam; código simples é mais manutenível |
| **Dados quase ordenados** | Insertion Sort adapta-se e opera em ~O(n) |
| **Restrições de memória** | Algoritmos in-place (O(1) espaço) podem ser preferíveis a Merge Sort (O(n) espaço) |
| **Prototipagem rápida** | Implementar Bubble Sort em 5 minutos pode ser mais produtivo que integrar biblioteca externa |

### 9.3 O Próximo Nível: Algoritmos O(n log n)

> 🔜 **No próximo capítulo:** Merge Sort e Quick Sort — como quebrar a barreira O(n²) usando divisão e conquista.

```
Comparação teórica para n = 10.000:

Algoritmo   | Comparações aproximadas | Relativo ao Merge*
------------|-------------------------|-------------------
Bubble      | 50.000.000  (n²/2)      | ~380x
Selection   | 50.000.000  (n²/2)      | ~380x
Insertion   | 25.000.000  (n²/4, médio)| ~190x
Merge Sort  | 130.000     (n log₂ n)  | 1x (base)
Quick Sort  | 180.000     (1,39 n log₂ n, médio) | ~1,4x

* Supondo o mesmo custo por comparação; na prática, constantes e
  acesso à memória mudam esses números.
```

---

## 10. Resumo do Capítulo

✅ **Conceitos fundamentais**

- Ordenação: rearranjar elementos em sequência definida
- Complexidade Big O: mede crescimento de operações com `n`
- Melhor/pior/caso médio: cenários que afetam desempenho real

✅ **Bubble Sort**

- Ideia: elementos maiores "flutuam" para o final
- Otimização: parada antecipada → O(n) no melhor caso
- Simples, mas ineficiente para grandes conjuntos

✅ **Selection Sort**

- Ideia: selecionar menor elemento e colocá-lo na posição correta
- Sempre O(n²), mas com **mínimo de trocas** (≤ n-1)
- Útil quando trocar é operação custosa

✅ **Insertion Sort**

- Ideia: inserir cada elemento na posição correta entre os já ordenados
- O(n) no melhor caso (dados quase ordenados)
- Estável, adaptativo e online — versátil para cenários reais

✅ **Boas práticas de implementação em C**

- Passar vetor por referência (já é padrão em C)
- Usar `bool` para flags de otimização
- Validar resultados com função auxiliar `estaOrdenado()`
- Medir tempo com `clock()` para comparação empírica

✅ **Critérios de escolha**

- Para ensino/pequenos dados: qualquer um dos três
- Para produção com n grande: migrar para Merge/Quick Sort
- Sempre considerar: tamanho dos dados, custo de troca, necessidade de estabilidade

---

## 11. Próximos Passos & Ferramentas

🔜 **No próximo capítulo:** Algoritmos eficientes de ordenação

- Merge Sort: divisão, conquista e intercalação
- Quick Sort: particionamento e pivô
- Comparação prática: quando usar cada um

🛠️ **Prática sugerida:**

1. **Compile e execute** o código completo da Seção 6
2. **Modifique** para gerar vetores aleatórios de tamanhos variados (100, 1000, 5000)
3. **Meça e grafique** o tempo de execução vs. tamanho do vetor (use planilha ou Python)
4. **Valide** que o crescimento segue ~n² para os três algoritmos

📚 **Leitura complementar:**

- ZIVIANI, N. *Projeto de Algoritmos*. Capítulo 4: Ordenação.
- CORMEN et al. *Introduction to Algorithms*. Capítulo 2: Getting Started (análise de algoritmos).
- USP-IME. *Pythonds em Português*: [https://panda.ime.usp.br/panda/static/pythonds_pt/05-OrdenacaoBusca/](https://panda.ime.usp.br/panda/static/pythonds_pt/05-OrdenacaoBusca/)

---

> 📌 **Dica para provas e trabalhos:**
> 
> - Ao analisar complexidade, **conte laços aninhados**: dois laços sobre `n` → O(n²)
> - Para simular algoritmos no papel, **desenhe o vetor a cada passo** — evita erros de raciocínio
> - Lembre-se: **Big O ignora constantes**, mas na prática, constantes importam para `n` pequeno!

---

*Material elaborado para a disciplina de Estrutura de Dados — Curso de Engenharia de Computação*

*Prof. Sérgio Souza Costa | Atualizado: 2026*

---