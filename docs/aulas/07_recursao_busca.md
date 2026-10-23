# Recursão e Algoritmos de Busca

## Recursão e Fundamentos de Busca

### Roteiro

1. Introdução à Recursão
2. As Três Leis da Recursão
3. Cálculo de Fatorial
4. Iteração vs Recursão (Soma de Array)
5. Introdução aos Algoritmos de Busca (Busca Sequencial)

### 1. Introdução à Recursão

**Recursão** é um método para resolver problemas que envolve quebrar o problema em subproblemas cada vez menores até atingir um problema simples o bastante, que possa ser resolvido trivialmente. Em geral, a recursão envolve uma função que chama a si mesma.

**Exemplos de uso na Computação:**

- Quick-sort (ordenação rápida)
- Ordenação por intercalação (Merge sort)
- Muitas estruturas de dados são recursivas por natureza, como listas encadeadas e árvores.

### 2. As Três Leis da Recursão

Todos os algoritmos recursivos devem obedecer a três leis importantes (formulação de Miller e Ranum, *Problem Solving with Algorithms and Data Structures*):

1. **Um algoritmo recursivo deve possuir um caso base (base case).**
2. **Um algoritmo recursivo deve modificar o seu estado e se aproximar do caso base.**
3. **Um algoritmo recursivo deve chamar a si mesmo, recursivamente.**

**Exemplos de casos base (ou triviais):**

- Qual o fatorial de 0? *(É 1)*
- Quanto é um dado X multiplicado por 0? *(É 0)*

### 3. Cálculo de Fatorial

A função fatorial de um inteiro não negativo pode ser definida como:

$$
n! = \begin{cases} 
1 & \text{se } n = 0 \\ 
n \times (n-1)! & \text{se } n > 0 
\end{cases}
$$

#### Implementação em C

```c
#include <stdio.h>

// Função recursiva para calcular o fatorial
int fat(int n) {
    // 1. Caso base
    if (n == 0) {
        return 1;
    }
    // 2 e 3. Modifica o estado (n-1) e chama a si mesma
    else {
        return n * fat(n - 1);
    }
}

int main() {
    printf("Fatorial de 4: %d\n", fat(4)); // Imprime 24
    return 0;
}
```

O computador resolve isso utilizando a **Pilha de Execução (Call Stack)**, onde cada chamada fica pausada aguardando o retorno da próxima até chegar ao caso base (`fat(0)`), quando então começa a desempilhar multiplicando os resultados de volta.

### 4. Iteração para Recursão (Soma de Array)

Na versão iterativa, um laço acumula a soma:

```c
int soma_array_iter(int numeros[], int tamanho) {
    int soma = 0;
    for (int i = 0; i < tamanho; i++) {
        soma += numeros[i];
    }
    return soma;
}
```

Qual é a regra para somar os elementos de um array recursivamente?
**"A soma de uma lista de números é o primeiro elemento mais a soma do restante da lista."**

```c
#include <stdio.h>

// Solução Recursiva usando ponteiros em C
int soma_array_rec(int numeros[], int tamanho) {
    if (tamanho == 0) { // Caso base: vetor vazio soma 0
        return 0;
    }
    else {
        // Retorna o 1º elemento + a soma do resto (avança o ponteiro e diminui tamanho)
        return numeros[0] + soma_array_rec(numeros + 1, tamanho - 1);
    }
}
```

### 5. Introdução aos Algoritmos de Busca

Por que busca é importante na computação? Pense no Google, uma das maiores empresas do mundo, criada fundamentalmente a partir de um poderoso algoritmo de busca!

**Busca** é o processo algorítmico de encontrar um item específico numa coleção de itens. Geralmente devolvemos Verdadeiro/Falso (`true`/`false`) ou a posição do elemento.

#### A Busca Sequencial

Quando itens são armazenados em um array, eles têm uma relação linear. Na busca sequencial, começamos pelo primeiro item e visitamos elemento por elemento até achar o alvo ou chegar ao fim do array.

```c
#include <stdio.h>
#include <stdbool.h> // Para usar true/false em C

bool busca_sequencial(int lista[], int tamanho, int item) {
    for (int pos = 0; pos < tamanho; pos++) {
        if (lista[pos] == item) {
            return true; // Encontrou!
        }
    }
    return false; // Percorreu tudo e não encontrou
}

int main() {
    int lista_teste[] = {1, 2, 32, 8, 17, 19, 42, 13, 0};
    int tamanho = 9;

    printf("Encontrou o 3? %d\n", busca_sequencial(lista_teste, tamanho, 3));  // 0 (False)
    printf("Encontrou o 13? %d\n", busca_sequencial(lista_teste, tamanho, 13)); // 1 (True)
    return 0;
}
```

## Busca Binária e Análise de Desempenho

### Roteiro

1. O Problema da Busca Sequencial
2. Busca Binária: O Conceito
3. Busca Binária: Implementação Iterativa
4. Busca Binária: Implementação Recursiva
5. Análise de Complexidade (Busca Binária vs Sequencial)

### 1. O Problema da Busca Sequencial

Na busca sequencial, se o item que queremos for o último do array (ou se ele nem estiver lá), teremos que verificar **todos** os `N` elementos.
Contudo, se os nossos dados estiverem **ordenados** (ex: ordem alfabética ou crescente), não precisamos olhar um por um. Podemos ser muito mais eficientes.

### 2. Busca Binária: O Conceito

Pense em procurar um nome numa lista telefônica ou dicionário. Você não lê da página 1 até a 500. Você abre no meio!

1. Olha primeiro o item do meio.
2. Se esse elemento é o que estamos buscando, a procura terminou.
3. Se não for:
    - Se o alvo for **maior** que o meio, sabemos que ele necessariamente está na **metade superior**.
    - Se o alvo for **menor** que o meio, sabemos que ele necessariamente está na **metade inferior**.

### 3. Busca Binária: Implementação Iterativa

Nesta implementação em C, usamos duas variáveis de controle (`primeiro` e `ultimo`) para representar os "limites" de onde estamos procurando no array.

```c
#include <stdio.h>
#include <stdbool.h>

bool busca_binaria_iterativa(int lista[], int tamanho, int item) {
    int primeiro = 0;
    int ultimo = tamanho - 1;

    while (primeiro <= ultimo) {
        // Encontra o meio exato do intervalo atual
        int meio = (primeiro + ultimo) / 2;

        if (lista[meio] == item) {
            return true; // Encontrou!
        }
        else {
            if (item < lista[meio]) {
                // Descarta a metade superior
                ultimo = meio - 1;
            } else {
                // Descarta a metade inferior
                primeiro = meio + 1;
            }
        }
    }
    return false; // Primeiro cruzou o ultimo, o item não existe.
}

int main() {
    // A LISTA DEVE ESTAR ORDENADA
    int lista_teste[] = {0, 1, 2, 8, 13, 17, 19, 32, 42};

    printf("Busca Binaria 3: %d\n", busca_binaria_iterativa(lista_teste, 9, 3));  // 0
    printf("Busca Binaria 13: %d\n", busca_binaria_iterativa(lista_teste, 9, 13)); // 1
    return 0;
}
```

### 4. Busca Binária: Implementação Recursiva

Podemos unir os dois temas deste capítulo: recursão e busca! A busca binária é um algoritmo clássico de recursão.
A cada chamada, o "estado modificado" é a redução do intervalo de busca (`primeiro` a `ultimo`).

```c
#include <stdio.h>
#include <stdbool.h>

// Aqui recebemos os índices 'primeiro' e 'ultimo' em vez do tamanho total
bool busca_binaria_recursiva(int lista[], int primeiro, int ultimo, int item) {
    // 1. Caso base: A lista "esvaziou" (os limites se cruzaram)
    if (primeiro > ultimo) {
        return false;
    }

    int meio = (primeiro + ultimo) / 2;

    // 2. Caso base de sucesso
    if (lista[meio] == item) {
        return true;
    }

    // 3. Chamada Recursiva: aproximando do caso base reduzindo o problema pela metade
    if (item < lista[meio]) {
        // Busca na metade inferior
        return busca_binaria_recursiva(lista, primeiro, meio - 1, item);
    } else {
        // Busca na metade superior
        return busca_binaria_recursiva(lista, meio + 1, ultimo, item);
    }
}

int main() {
    int lista[] = {0, 1, 2, 8, 13, 17, 19, 32, 42};
    // Passamos índice inicial (0) e final (tamanho-1 = 8)
    printf("Busca Recursiva 13: %d\n", busca_binaria_recursiva(lista, 0, 8, 13));
    return 0;
}
```

*Nota sobre C:* Em linguagens como Python, poderíamos criar "fatias" da lista (`lista[:meio]`). Em C, é muito mais eficiente e seguro passar os índices `primeiro` e `ultimo` para indicar qual parte do array original estamos analisando.

### 5. Análise de Desempenho (Sequencial vs Binária)

Para analisar a busca binária, precisamos ter em mente que cada comparação elimina cerca de metade dos itens restantes a serem considerados.

| Comparações | Itens restantes aproximados |
| --- | --- |
| 1 | n/2 |
| 2 | n/4 |
| 3 | n/8 |
| i | $n/2^i$ |

Dado essa característica, o número máximo de comparações para encontrar (ou não) um elemento é $\lfloor \log_2 n \rfloor + 1$, ou seja, da ordem de $\log_2 n$.

**Exemplo Prático (1.000.000 de registros):**

- **Busca Sequencial:** No pior cenário, você fará **1.000.000** de comparações.
- **Busca Binária:** No pior cenário, $\log_2(1.000.000)$ resulta em aproximadamente **20** comparações. A diferença de performance é brutal em grandes volumes de dados!

### Próximos Passos

1. Modifique as funções de busca para retornar o **índice** do elemento encontrado em vez de Verdadeiro/Falso. Retorne `-1` caso não encontre.
2. Teste o que acontece se você passar um array não-ordenado para a Busca Binária (ele vai falhar sem dar erro de compilação!).