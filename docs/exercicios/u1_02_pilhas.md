# Lista - Pilhas

---

## 1. O que será impresso na tela

Considerando a implementação do TAD Pilha baseada em ponteiros e alocação dinâmica vista em aula, analise o código abaixo:

```c
#include <stdio.h>

// Supondo que os TADs Pilha e Fila já estejam implementados como no livro:
// criaPilha(int), empilha(int, Pilha*), desempilha(Pilha*)  → devolve o valor
// criaFila(void), enfileira(Fila*, int), frente(Fila*), desenfileira(Fila*) → devolve 1/0

int main() {
    // Criação das estruturas dinâmicas
    Pilha *s = criaPilha(10);
    Fila *q = criaFila();
    int temp_val;

    empilha(85, s);
    empilha(34, s);

    temp_val = desempilha(s);
    enfileira(q, temp_val);

    temp_val = desempilha(s);
    enfileira(q, temp_val);

    empilha(92, s);

    temp_val = frente(q);
    desenfileira(q);
    empilha(temp_val, s);

    printf("%d\n", desempilha(s));
    printf("%d\n", desempilha(s));

    return 0;
}
```

**Pergunta:** Qual será a saída impressa no console ao final da execução?

---

## 2. Verificação de expressão balanceada

A **Pilha** é a estrutura natural para verificar se os parênteses de uma expressão estão balanceados: cada `(` é empilhado e cada `)` desempilha um `(` correspondente. Considere a implementação abaixo, que usa o TAD Pilha do capítulo 2.3:

```c
int balanceada(const char *expr) {
    Pilha *p = criaPilha(100);
    for (int i = 0; expr[i] != '\0'; i++) {
        if (expr[i] == '(') {
            empilha('(', p);
        } else {                      /* assume que o caractere é ')' */
            if (pilhaVazia(p)) { liberaPilha(p); return 0; }
            desempilha(p);
        }
    }
    int ok = pilhaVazia(p);
    liberaPilha(p);
    return ok;
}
```

Agora teste a implementação passando a seguinte expressão:

```
( 1 + 2 ) * (4 + (3 + 5) )
```

O algoritmo irá funcionar corretamente?
Caso contrário, corrija o algoritmo para analisar **apenas o balanceamento dos parênteses**, ignorando os números e operadores.

---

## 3. Simulação da avaliação de uma expressão pós-fixa

A avaliação de expressões em notação pós-fixa (polonesa reversa) é uma das principais aplicações práticas do conceito LIFO (*Last In, First Out*) de uma Pilha.

O algoritmo percorre a expressão da esquerda para a direita: cada operando é empilhado; cada operador desempilha dois valores (primeiro o da direita, depois o da esquerda), aplica a operação e empilha o resultado. Ao final, o resultado é o único valor na pilha.

Simule a avaliação da seguinte expressão:

```
a b + c * d /
```

Onde:

```
a = 2, b = 4, c = 3, d = 36
```

Preencha a tabela de rastreio mostrando o estado da pilha passo a passo:

| Expressão | Elemento | Ação (empilha / desempilha) | Pilha (Estado atual) |
| --- | --- | --- | --- |
|  |  |  |  |

---

## 4. Avaliação pós-fixa com números de múltiplos dígitos

Considere a implementação abaixo, que avalia expressões pós-fixas com operandos de **um dígito** (sem espaços), usando o TAD Pilha de inteiros:

```c
#include <ctype.h>

int avaliaPosfixa(const char *expr) {
    Pilha *p = criaPilha(100);
    for (int i = 0; expr[i] != '\0'; i++) {
        char c = expr[i];
        if (isdigit((unsigned char) c)) {
            empilha(c - '0', p);
        } else if (c == '+' || c == '-' || c == '*' || c == '/') {
            int b = desempilha(p);   /* operando da direita */
            int a = desempilha(p);   /* operando da esquerda */
            switch (c) {
                case '+': empilha(a + b, p); break;
                case '-': empilha(a - b, p); break;
                case '*': empilha(a * b, p); break;
                case '/': empilha(a / b, p); break;
            }
        }
    }
    int resultado = desempilha(p);
    liberaPilha(p);
    return resultado;
}
```

Modifique essa implementação para suportar:

- números com **vários dígitos**;
- **valores em ponto flutuante** (adapte a sua `struct pilha` e o vetor interno `valores` para armazenar `float` em vez de `int`).

Dica: com números de vários dígitos, os elementos precisam ser separados por espaço; `strtok` ou `strtod` ajudam a extrair cada um.

Teste com a expressão:

```
45.5 83 + 75.4 *
```