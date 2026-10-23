# Listas Encadeadas como TAD em C

**Disciplina:** Estrutura de Dados

**Tema:** Tipo Abstrato de Dados LISTA – Implementação Dinâmica (Encadeada) em C

**Pré-requisitos:** Ponteiros, `malloc`/`free`, TAD LISTA estática (capítulo anterior)

---

## 1. Revisão: O TAD LISTA e a Ideia de Independência de Implementação

### 1.1 O Que Vimos no Capítulo Anterior

No capítulo anterior, sobre **listas estáticas**, implementamos o TAD LISTA usando um vetor de tamanho fixo:

```c
/* Implementação estática (lista_estatica.c) */
struct lista {
    int valores[MAX];  /* Vetor pré-alocado */
    int ultimo;        /* Índice do último elemento */
};
```

**Características da versão estática:**

| Vantagem | Limitação |
| --- | --- |
| Acesso direto por posição: O(1) | Tamanho máximo fixo (`MAX`) |
| Simplicidade de implementação | Desperdício de memória se usar pouco |
| Localidade de referência (cache-friendly) | Inserção/remoção no meio: O(n) por deslocamento |

### 1.2 O Poder da Interface Única

A interface `lista.h` que definimos **não menciona** se a lista é estática ou dinâmica:

```c
/* lista.h – Interface (INALTERADA) */
typedef struct lista Lista;  /* Tipo opaco */

Lista* criaLista(void);
int insereFinal(Lista* l, int x);
int removePos(Lista* l, int pos);
/* ... demais operações ... */
```

> **Princípio fundamental:** O código cliente (`main.c`) que usa esta interface **não precisa mudar** quando trocamos a implementação interna. Isso é a essência de um TAD bem projetado.

---

## 2. Introdução Intuitiva à Complexidade de Algoritmos

### 2.1 Por Que Medir Complexidade?

Quando implementamos operações como `inserePos` ou `buscaValor`, queremos saber:

- **Quão rápido** a operação executa conforme a lista cresce?
- **Qual o custo** no pior cenário possível?
- **Vale a pena** usar esta estrutura para meu problema?

> **Objetivo:** Entender a **ordem de crescimento** do tempo de execução, sem fórmulas complexas.

### 2.2 Classes de Complexidade

Da mais lenta para a mais rápida em crescimento, as classes que aparecerão com frequência são:

| Classe | Nome | Exemplo neste livro |
| --- | --- | --- |
| O(1) | Constante | Acesso por posição em vetor |
| O(log n) | Logarítmica | Busca binária em vetor ordenado (Capítulo 5) |
| O(n) | Linear | Inserção no meio de uma lista estática |
| O(n²) | Quadrática | Ordenações básicas (Capítulo 9) |
| O(2ⁿ) | Exponencial | Testar todos os subconjuntos de um conjunto |

O Capítulo 8 apresenta um gráfico com as curvas de crescimento reais dessas classes.

### 2.3 Melhor Caso vs. Pior Caso – Exemplo Prático

**Operação: `buscaValor(l, x)` na lista estática**

```c
int buscaValor(const Lista* l, int x) {
    for (int i = 0; i <= l->ultimo; i++) {  /* Percorre elemento por elemento */
        if (l->valores[i] == x) {
            return i;  /* Encontrou! */
        }
    }
    return -1;  /* Não encontrou */
}
```

| Cenário | Descrição | Comparações | Complexidade |
| --- | --- | --- | --- |
| **Melhor caso** | `x` está na primeira posição (`i=0`) | 1 | **O(1)** |
| **Pior caso** | `x` não está na lista ou está no final | `n` (tamanho da lista) | **O(n)** |
| **Caso médio** | `x` está distribuído uniformemente | `n/2` | **O(n)** |

> **Convenção:** Quando dizemos que uma operação é **O(n)**, estamos nos referindo ao **pior caso**, a menos que especificado o contrário.

### 2.4 Tabela de Complexidade – Lista Estática

| Operação | Complexidade | Por quê? |
| --- | --- | --- |
| `criaLista`, `vazia`, `cheia` | **O(1)** | Acesso direto a variáveis |
| `buscaPos`, `insereFinal` | **O(1)** | Acesso direto por índice |
| `buscaValor`, `removeValor` | **O(n)** | Pode precisar percorrer toda a lista |
| `inserePos`, `removePos` (meio) | **O(n)** | Precisa deslocar até `n` elementos |
| `conta` | **O(1)** | Calcula `ultimo + 1` |
| `imprime` | **O(n)** | Percorre todos os elementos |

---

## 3. O Problema do Espaço Fixo e Como Bibliotecas Reais Lidam

### 3.1 O Limite de `MAX`

Na implementação estática, definimos:

```c
#define MAX 100  /* E se precisarmos de 101 elementos? */
```

**Cenários problemáticos:**

1. **Subutilização:** `MAX = 10000`, mas usamos apenas 10 elementos → desperdício de ~99% da memória
2. **Estouro:** `MAX = 100`, mas precisamos inserir o 101º elemento → falha silenciosa ou erro

### 3.2 Estratégias de Bibliotecas Reais

| Estratégia | Como funciona | Exemplo |
| --- | --- | --- |
| **Vetor dinâmico** (`std::vector` em C++, `ArrayList` em Java) | Começa com capacidade pequena; quando enche, aloca um vetor maior (ex: 2×), copia os elementos e libera o antigo | Crescimento amortizado O(1) para inserção no final |
| **Lista encadeada** (abordagem deste capítulo) | Cada elemento é alocado sob demanda; não há limite pré-definido | Inserção/remoção O(1) se já tivermos a posição |
| **Híbrido** (`std::deque`) | Combina blocos de vetor com encadeamento | Acesso por posição O(1), crescimento flexível |

> **Insight:** Não existe "melhor estrutura" universal. A escolha depende do padrão de uso: muitas inserções no meio? Lista encadeada pode ser melhor. Muitos acessos por posição? Vetor é mais eficiente.

---

## 4. Lista Encadeada: Conceito e Estrutura

### 4.1 Definição Recursiva

> "Uma lista encadeada é definida recursivamente como:
> - **Caso base:** uma lista vazia (`NULL`), ou
> - **Caso recursivo:** um nó contendo um valor e um ponteiro para outra lista encadeada."

```
Lista = ∅  (vazia)
     OU
Lista = [valor] → Lista  (nó apontando para o restante)
```

### 4.2 Estrutura do Nó

```c
/* Nó da lista encadeada – estrutura interna, não exposta */
typedef struct no {
    int valor;           /* Dado armazenado */
    struct no* prox;     /* Ponteiro para o próximo nó */
} No;
```

**Representação visual:**

```
Lista com 3 elementos: [10] → [20] → [30] → NULL
                        ↑
                      inicio

Cada caixa é um No*:
┌─────────────┐
│ valor: 10   │
│ prox: ───┐  │
└─────────┼──┘
          ▼
      ┌─────────────┐
      │ valor: 20   │
      │ prox: ───┐  │
      └─────────┼──┘
                ▼
            ┌─────────────┐
            │ valor: 30   │
            │ prox: NULL  │  ← fim da lista
            └─────────────┘
```

### 4.3 Estrutura do TAD (Encapsulamento)

```c
/* lista.c – Implementação encadeada */
#include <stdio.h>
#include <stdlib.h>
#include "lista.h"

/* Definição do nó (privada) */
typedef struct no {
    int valor;
    struct no* prox;
} No;

/* Definição da lista (privada) */
struct lista {
    No* inicio;   /* Ponteiro para o primeiro nó */
    int tamanho;  /* Contador de elementos (opcional, mas útil) */
};
```

> **Nota:** Mantivemos o campo `tamanho` para que operações como `conta()` sejam O(1), assim como na versão estática.

---

## 5. Implementação das Operações – Versão Encadeada

### 5.1 Criação e Destruição

```c
Lista* criaLista(void) {
    Lista* l = malloc(sizeof(Lista));
    if (l == NULL) return NULL;
    l->inicio = NULL;   /* Lista vazia */
    l->tamanho = 0;
    return l;
}

void destroiLista(Lista* l) {
    if (l == NULL) return;

    /* Percorre a lista liberando cada nó */
    No* atual = l->inicio;
    while (atual != NULL) {
        No* proximo = atual->prox;  /* Salva próximo antes de liberar */
        free(atual);
        atual = proximo;
    }

    free(l);  /* Libera a struct da lista */
}
```

### 5.2 Consultas Básicas

```c
int conta(const Lista* l) {
    return (l == NULL) ? 0 : l->tamanho;  /* O(1) graças ao contador */
}

int vazia(const Lista* l) {
    return (l == NULL) || (l->inicio == NULL);
}

/* Nota: 'cheia' não faz sentido para lista encadeada (alocação sob demanda) */
int cheia(const Lista* l) {
    return 0;  /* Sempre retorna falso – teoricamente nunca "enche" */
}

void imprime(const Lista* l) {
    if (vazia(l)) {
        printf("Lista vazia.\n");
        return;
    }

    printf("Lista [%d elementos]: ", l->tamanho);
    No* atual = l->inicio;
    while (atual != NULL) {
        printf("%d ", atual->valor);
        atual = atual->prox;
    }
    printf("\n");
}
```

### 5.3 Inserção no Final – Passo a Passo Visual

**Objetivo:** `insereFinal(l, 30)` em uma lista `[10] → [20] → NULL`

```
Estado inicial:
[10] → [20] → NULL
 ↑
inicio

Passo 1: Criar novo nó com valor 30
[novo: 30 | prox: ?]

Passo 2: Percorrer até o último nó (20)
[10] → [20] → NULL
              ↑
            atual

Passo 3: Fazer último apontar para novo
[10] → [20] → [30] → NULL

Passo 4: Atualizar contador
tamanho: 2 → 3
```

**Implementação:**

```c
int insereFinal(Lista* l, int x) {
    if (l == NULL) return 0;

    /* Criar novo nó */
    No* novo = malloc(sizeof(No));
    if (novo == NULL) return 0;  /* Falha na alocação */
    novo->valor = x;
    novo->prox = NULL;  /* Será o último */

    /* Caso especial: lista vazia */
    if (vazia(l)) {
        l->inicio = novo;
    } else {
        /* Percorrer até o último nó */
        No* atual = l->inicio;
        while (atual->prox != NULL) {
            atual = atual->prox;
        }
        atual->prox = novo;
    }

    l->tamanho++;
    return 1;
}
```

> **Complexidade:** `insereFinal` é **O(n)** na lista encadeada simples, pois precisa percorrer até o final. Veremos depois como otimizar para O(1) com um ponteiro para o fim.

### 5.4 Inserção por Posição – Caso Geral

```c
int inserePos(Lista* l, int x, int pos) {
    if (l == NULL) return 0;
    if (pos < 0 || pos > l->tamanho) return 0;  /* Posição válida: 0..tamanho */

    /* Criar novo nó */
    No* novo = malloc(sizeof(No));
    if (novo == NULL) return 0;
    novo->valor = x;

    /* Caso especial: inserir no início */
    if (pos == 0) {
        novo->prox = l->inicio;
        l->inicio = novo;
    } else {
        /* Percorrer até o nó anterior à posição */
        No* anterior = l->inicio;
        for (int i = 0; i < pos - 1; i++) {
            anterior = anterior->prox;
        }
        /* Inserir novo entre anterior e anterior->prox */
        novo->prox = anterior->prox;
        anterior->prox = novo;
    }

    l->tamanho++;
    return 1;
}
```

**Visualização: `inserePos(l, 15, 1)` em `[10] → [20] → [30]`**

```
Estado inicial:
[10] → [20] → [30] → NULL
 ↑
inicio (pos 0)

Passo 1: Criar nó [15]
[novo: 15 | prox: ?]

Passo 2: Percorrer até posição pos-1 = 0 (nó 10)
[10] → [20] → [30]
 ↑
anterior

Passo 3: novo->prox = anterior->prox (15 aponta para 20)
[15] → [20] → [30]

Passo 4: anterior->prox = novo (10 aponta para 15)
[10] → [15] → [20] → [30] → NULL ✓
```

### 5.5 Remoção por Posição

```c
int removePos(Lista* l, int pos) {
    if (l == NULL || vazia(l)) return 0;
    if (pos < 0 || pos >= l->tamanho) return 0;

    No* removido;

    /* Caso especial: remover o primeiro */
    if (pos == 0) {
        removido = l->inicio;
        l->inicio = removido->prox;
    } else {
        /* Percorrer até o nó anterior */
        No* anterior = l->inicio;
        for (int i = 0; i < pos - 1; i++) {
            anterior = anterior->prox;
        }
        removido = anterior->prox;
        anterior->prox = removido->prox;  /* "Pula" o nó removido */
    }

    free(removido);  /* Libera memória */
    l->tamanho--;
    return 1;
}
```

### 5.6 Buscas e Remoção por Valor

```c
int buscaPos(const Lista* l, int pos) {
    if (l == NULL || pos < 0 || pos >= l->tamanho) {
        return -1;
    }

    No* atual = l->inicio;
    for (int i = 0; i < pos; i++) {
        atual = atual->prox;
    }
    return atual->valor;
}

int buscaValor(const Lista* l, int x) {
    if (l == NULL) return -1;

    No* atual = l->inicio;
    int pos = 0;
    while (atual != NULL) {
        if (atual->valor == x) {
            return pos;  /* Encontrou */
        }
        atual = atual->prox;
        pos++;
    }
    return -1;  /* Não encontrou */
}

/* Remove primeira ocorrência – reusa buscaValor e removePos */
int removeValor(Lista* l, int x) {
    int pos = buscaValor(l, x);
    if (pos == -1) return 0;
    return removePos(l, pos);
}
```

---

## 6. Tabela Comparativa: Estática vs. Encadeada

| Operação | Estática (vetor) | Encadeada (nós) | Observação |
| --- | --- | --- | --- |
| `criaLista` | O(1) | O(1) | Ambas alocam apenas a struct |
| `insereFinal` | **O(1)** | O(n)* | Encadeada precisa percorrer; pode ser O(1) com ponteiro para fim |
| `inserePos(0)` | O(n) | **O(1)** | Encadeada não precisa deslocar, só ajustar ponteiros |
| `inserePos(meio)` | O(n) | O(n) | Ambas precisam percorrer até a posição |
| `removePos` | O(n) | O(n) | Estática: deslocar; Encadeada: percorrer + ajustar ponteiros |
| `buscaPos` | **O(1)** | O(n) | Vetor permite acesso direto; encadeada requer percurso |
| `buscaValor` | O(n) | O(n) | Ambas podem precisar verificar todos os elementos |
| `imprime` | O(n) | O(n) | Percorrer todos os elementos |
| Uso de memória | Fixo (`MAX * sizeof(int)`) | Dinâmico (`n * (sizeof(int)+ponteiro)`) | Encadeada tem overhead de ponteiros |
| Localidade de referência | **Alta** (vetor contíguo) | Baixa (nós espalhados na heap) | Vetor é mais cache-friendly |

> \* Pode ser otimizado para O(1) adicionando um campo `No* fim` na struct `lista`.

---

## 7. O Poder do TAD: Mesmo Cliente, Duas Implementações

### 7.1 Estrutura de Diretórios

```
projeto_lista/
├── lista.h              /* Interface única (comum a ambas) */
├── lista_estatica.c     /* Implementação com vetor */
├── lista_encadeada.c    /* Implementação com nós */
├── main.c               /* Cliente – NÃO muda! */
├── Makefile             /* Escolhe qual implementação compilar */
└── testes/
    └── main_testes.c    /* Testes automatizados */
```

### 7.2 Makefile para Alternar Implementações

```makefile
# Makefile
CC = gcc
CFLAGS = -Wall -Wextra -std=c11

# Escolha a implementação: "estatica" ou "encadeada"
IMPLEMENTACAO = encadeada

# Compila a implementação escolhida (lista_estatica.o ou lista_encadeada.o)
programa: main.o lista_$(IMPLEMENTACAO).o
	$(CC) $(CFLAGS) main.o lista_$(IMPLEMENTACAO).o -o programa

main.o: main.c lista.h
	$(CC) $(CFLAGS) -c main.c

lista_%.o: lista_%.c lista.h
	$(CC) $(CFLAGS) -c $< -o $@

clean:
	rm -f *.o programa

# 'programa' é sempre religado, para que trocar IMPLEMENTACAO tenha efeito
.PHONY: programa clean

# Exemplo de uso:
# make IMPLEMENTACAO=estatica programa   # Compila com vetor
# make IMPLEMENTACAO=encadeada programa  # Compila com nós
```

### 7.3 Cliente Exemplo (`main.c`) – Funciona com Ambas

```c
/* main.c – Usa o TAD LISTA sem saber da implementação */
#include <stdio.h>
#include "lista.h"

int main(void) {
    Lista* l = criaLista();
    if (l == NULL) {
        fprintf(stderr, "Falha ao criar lista\n");
        return 1;
    }

    /* Cenário de teste – idêntico para ambas implementações */
    insereFinal(l, 10);
    insereFinal(l, 20);
    insereFinal(l, 30);
    inserePos(l, 15, 1);      /* Inserir 15 na posição 1 */

    printf("Lista após inserções:\n");
    imprime(l);               /* Esperado: 10 15 20 30 */

    printf("Busca valor 20: posição %d\n", buscaValor(l, 20));

    removeValor(l, 15);
    removePos(l, 1);
    imprime(l);               /* Esperado: 10 30 */

    destroiLista(l);
    return 0;
}
```

> **Verificação:** Se `lista.h` não mudar, `main.c` **não precisa ser recompilado** quando trocamos de `lista_estatica.c` para `lista_encadeada.c`. Basta recompilar apenas o arquivo de implementação e relinkar.

---

## 8. Folha de Referência Rápida – Lista Encadeada

```
┌─────────────────────────────────────┐
│ LISTA ENCADEADA EM C – CHEAT SHEET  │
├─────────────────────────────────────┤
│ ESTRUTURA INTERNA                   │
│ • struct no { int valor; No* prox; }│
│ • struct lista { No* inicio; int tamanho; }│
│ • vazia ⇔ inicio == NULL           │
│ • "cheia" ⇔ sempre false (aloca sob demanda)│
├─────────────────────────────────────┤
│ OPERAÇÕES – COMPLEXIDADE            │
│ • Acesso por posição: O(n) ⚠       │
│ • Inserção no início: O(1) ✓       │
│ • Inserção no final: O(n) ⚠*       │
│ • Inserção/remoção no meio: O(n) ⚠ │
│ • Busca por valor: O(n) ⚠          │
│ * Pode ser O(1) com ponteiro para fim│
├─────────────────────────────────────┤
│ PADRÕES COMUNS                      │
│ • Sempre verificar malloc == NULL  │
│ • Salvar próximo antes de free()   │
│ • Atualizar ponteiros na ordem correta│
│ • Manter contador 'tamanho' para O(1) em conta()│
├─────────────────────────────────────┤
│ ERROS FREQUENTES                   │
│ □ Esquecer de atualizar l->inicio ao remover o primeiro│
│ □ Perder referência ao liberar nó (vazamento de memória)│
│ □ Acessar atual->prox sem verificar atual != NULL│
│ □ Não incrementar/decrementar 'tamanho' após modificar│
└─────────────────────────────────────┘
```

---

## 9. Guia de Estudo e Prática

### Exercícios Recomendados

- [ ]  Implementar `insereFinal` com otimização O(1) adicionando `No* fim` na struct
- [ ]  Criar um teste que insere 1000 elementos e mede tempo (estática vs. encadeada)
- [ ]  Desenhar o estado dos ponteiros após:
    
    ```
    criaLista() → insereFinal(5) → inserePos(3, 0) → removePos(1)
    ```
    

### Para Aprofundar

1. **Por que `buscaPos` é O(n) na encadeada e O(1) na estática?**
    
    → Vetores permitem acesso direto por índice (`valores[i]`); listas exigem percurso sequencial de ponteiros.
    
2. **Como implementar uma lista duplamente encadeada?**
    
    → Adicionar campo `No* ant` no nó; permite percurso nos dois sentidos e remoção O(1) se já tiver o nó.
    
3. **O que é "localidade de referência" e por que importa?**
    
    → Elementos contíguos na memória (vetor) são carregados juntos no cache da CPU, acelerando acesso sequencial.
    

### Desafio Opcional

Implemente `inverte(Lista* l)` para lista encadeada **sem usar memória auxiliar** (apenas ajustando ponteiros). Dica: use três ponteiros (`anterior`, `atual`, `proximo`) e inverta as setas.

<details>
<summary>Clique para ver uma solução</summary>

```c
void inverte(Lista* l) {
    No* anterior = NULL;
    No* atual = l->inicio;

    while (atual != NULL) {
        No* proximo = atual->prox;  /* Salva próximo */
        atual->prox = anterior;     /* Inverte seta */
        anterior = atual;           /* Avança anterior */
        atual = proximo;            /* Avança atual */
    }

    l->inicio = anterior;  /* Novo início é o antigo fim */
}
```

</details>

### Debug Challenge

O código abaixo tem um bug de vazamento de memória. Encontre e corrija:

```c
int removePos(Lista* l, int pos) {
    /* ... validações ... */
    if (pos == 0) {
        l->inicio = l->inicio->prox;  /* Bug: não libera o nó antigo! */
    }
    /* ... resto do código ... */
}
```

<details>
<summary>Clique para ver a resposta</summary>

É necessário salvar o nó a ser removido em uma variável temporária, atualizar os ponteiros, e só então chamar `free()` no nó removido:

```c
if (pos == 0) {
    No* removido = l->inicio;
    l->inicio = removido->prox;
    free(removido);  /* Libera memória */
}
```

</details>

---

## 10. Continuando o Estudo

### 10.1 Otimizações Possíveis

| Otimização | Como funciona | Ganho |
| --- | --- | --- |
| **Ponteiro para fim** | Adicionar `No* fim` na struct `lista` | `insereFinal` torna-se O(1) |
| **Lista circular** | Último nó aponta para o primeiro | Facilita certas operações de fila |
| **Lista duplamente encadeada** | Cada nó tem `prox` e `ant` | Remoção O(1) se já tiver o nó; percurso reverso |

### 10.2 Quando Usar Cada Implementação?

```
✅ Use lista ESTÁTICA (vetor) quando:
   • O tamanho máximo é conhecido e razoável
   • Muitos acessos por posição (buscaPos)
   • Performance de cache é crítica

✅ Use lista ENCADEADA quando:
   • O tamanho é imprevisível ou muito variável
   • Muitas inserções/remoções no início ou meio
   • Memória é limitada e quer evitar desperdício
```

### 10.3 Próximo Capítulo: Filas

Agora que dominamos duas implementações do TAD LISTA, vamos aplicar esse conhecimento ao TAD FILA, também em versão estática (vetor circular) e encadeada. Fica como exercício fazer o mesmo com a PILHA: uma pilha encadeada é uma lista em que inserções e remoções acontecem sempre no início.

---

## 🔗 Referências e Leitura Complementar

1. **Linked List vs Array – GeeksforGeeks**: Comparação prática de desempenho
2. **Big O Cheat Sheet**: Visualização intuitiva de complexidades
3. **Celes, W. et al. – Introdução a Estruturas de Dados**: Capítulos sobre listas encadeadas em C
4. **MIT OpenCourseWare – Data Structures**: Aulas sobre complexidade e escolha de estruturas
5. [**Visualgo.net/list**](http://visualgo.net/list): Animações interativas de operações em listas estáticas e encadeadas

---

> **Nota:** Para fixar os conceitos, implemente as operações você mesmo, execute os testes sugeridos e compare o comportamento das versões estática e encadeada. A prática com ponteiros é essencial para dominar listas encadeadas.
