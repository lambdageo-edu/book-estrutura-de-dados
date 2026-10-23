# Listas Estáticas como TAD em C

**Disciplina:** Estrutura de Dados

**Tema:** Tipo Abstrato de Dados LISTA – Implementação Estática em C

**Pré-requisitos:** Variáveis, structs, ponteiros e funções em C

---

## 1. Revisão: Tipos de Dados e Tipos Abstratos de Dados (TAD)

### 1.1 Tipos de Dados em C

Um **tipo de dado** define:

- O conjunto de valores que uma variável pode armazenar
- As operações válidas sobre esses valores

**Exemplos em C:**

| Categoria | Exemplos |
| --- | --- |
| Tipos simples | `int`, `float`, `double`, `char`, `bool` |
| Tipos estruturados | `struct`, `union`, `enum` |
| Tipos derivados | arrays, ponteiros |

```c
// Exemplo: tipo estruturado
struct Ponto {
    int x;
    int y;
};
// Valores: qualquer par (x, y) de inteiros
// Operações: acesso a campos, atribuição, passar como parâmetro
```

### 1.2 Tipo Abstrato de Dados (TAD) – Definição Formal

Um Tipo Abstrato de Dados (TAD, em inglês *ADT*) é um modelo matemático definido por seu comportamento (semântica) do ponto de vista de quem o usa: especifica os valores possíveis e as operações sobre esses valores, independentemente de como são implementados.

Em notação matemática, um TAD pode ser visto como uma tupla **(V, O)**, onde:

- **V** = conjunto de valores possíveis
- **O** = conjunto de operações válidas sobre V

### 1.3 Definição Intuitiva

> "Um TAD é um 'container' de dados que especifica **o que** pode ser armazenado e **quais operações** podem ser realizadas, escondendo **como** tudo isso é implementado na memória."

**Ideias-chave:**

| Conceito | Significado |
| --- | --- |
| **Abstração** | O usuário conhece a interface (assinaturas e comportamento), não os detalhes internos |
| **Encapsulamento** | O estado interno (vetor, índices, ponteiros) é protegido; acesso apenas via operações válidas |
| **Reusabilidade** | O mesmo TAD pode ser usado em múltiplos programas sem reescrever a estrutura |

---

## 2. Pilha como Exemplo de TAD

### 2.1 Relembrando a Pilha

A **Pilha** é tradicionalmente usada na literatura para ilustrar o conceito de TAD, pois possui uma interface simples e comportamento bem definido.

**TAD PILHA de inteiros (abstrato):**

- **Valores:** Todas as sequências finitas de inteiros `(x₁, x₂, ..., xₙ)`
- **Operações típicas:**
    - `criaPilha()`: cria pilha vazia
    - `empilha(x, p)`: insere `x` no topo
    - `desempilha(p)`: remove e retorna valor do topo
    - `pilhaVazia(p)`: verifica se está vazia
    - `topo(p)`: observa o topo sem remover

**Política de acesso:** **LIFO** (*Last In, First Out*) – o último elemento inserido é o primeiro a ser removido.

### 2.2 A Pilha na Literatura

Em materiais acadêmicos sobre estruturas de dados, a pilha aparece como exemplo clássico de como separar "o que faz" (contrato das operações) do "como faz" (implementação com vetor vs. lista encadeada).


### 2.3 Implementação Anterior (Tudo em Um Arquivo)

No capítulo sobre pilha, implementamos algo assim:

```c
// pilha_tudo_em_um.c
#include <stdio.h>
#include <stdlib.h>

#define MAX 100

typedef struct pilha {
    int valores[MAX];
    int topo;
} Pilha;

Pilha* criaPilha() { /* ... */ }
void empilha(int x, Pilha* p) { /* ... */ }
int desempilha(Pilha* p) { /* ... */ }
int pilhaVazia(Pilha* p) { /* ... */ }

int main() {
    Pilha* p = criaPilha();
    empilha(10, p);
    // ... uso da pilha ...
    free(p);
    return 0;
}
```

**Limitação:** Interface e implementação misturadas. Para reutilizar em outro projeto, precisaríamos copiar todo o código.

---

## 3. TADs e Classes em Orientação a Objetos

### 3.1 Relação Conceitual

Em linguagens OO (Java, C++, C#), uma **classe** é a forma natural de implementar um TAD:

| TAD (conceito) | Classe em OO (implementação) |
| --- | --- |
| Conjunto de valores | Atributos (privados) |
| Operações | Métodos (públicos) |
| Encapsulamento | Modificadores `private`/`public` |
| Interface | Assinaturas dos métodos públicos |

**Exemplo: TAD PILHA ↔ Classe `Stack`**

```java
// Java - Classe como implementação de TAD
public class Stack {
    private int[] valores;  // estado interno (privado)
    private int topo;

    public Stack() { /* criaPilha */ }
    public void push(int x) { /* empilha */ }
    public int pop() { /* desempilha */ }
    public boolean isEmpty() { /* pilhaVazia */ }
    // Cliente usa apenas métodos públicos, sem ver internals
}
```

### 3.2 E em C, Como Implementar TAD?

C não possui classes, mas podemos simular TADs usando:

1. **`struct`** para representar o estado interno
2. **Funções** que recebem ponteiros para a `struct` como operações
3. **Arquivos separados**:
    - `.h` (header): expõe apenas a interface (protótipos)
    - `.c` (implementação): contém o código real, escondido do cliente

Em C, portanto, um TAD costuma ocupar dois arquivos: um `.h` com a interface e um `.c` com a implementação.

---

## 4. Conceito Formal de LISTA

### 4.1 Lista Linear – Definição Matemática

> "Uma lista linear é uma sequência finita de zero ou mais elementos `x₁, x₂, ..., xₙ`, onde cada `xᵢ` é de um tipo definido e `n ≥ 0` é o tamanho da lista. Se `n ≥ 1`, então `x₁` é o primeiro elemento, `xₙ` é o último, e para `1 < i < n`, `xᵢ₋₁` precede `xᵢ` e `xᵢ₊₁` sucede `xᵢ`."

**Propriedades fundamentais:**

- Elementos têm **posição relativa** (ordem importa)
- Acesso permitido a **qualquer posição** (diferente da pilha)
- Tamanho pode variar dinamicamente (em implementações adequadas)

### 4.2 Analogias com o Mundo Real

| Analogia | Mapeamento para Lista | Operações Típicas |
| --- | --- | --- |
| **Lista de compras** | Itens em ordem de adição | Inserir no final, remover quando comprado |
| **Playlist de músicas** | Faixas numeradas (0, 1, 2...) | Inserir em posição específica, pular para posição X |
| **Fila do banco com senha** | Pessoas com número de ordem | Buscar posição da sua senha, remover quando chamado |
| **Histórico de navegação** | Páginas visitadas em ordem | Acessar página 3 do histórico, remover página específica |

### 4.3 Representações Possíveis para o Mesmo TAD

O **mesmo TAD LISTA** pode ser implementado de formas diferentes:

| Representação | Características | Quando usar |
| --- | --- | --- |
| **Estática (vetor)** | Elementos em posições contíguas; tamanho fixo definido em compilação | Quando o tamanho máximo é conhecido e não muda muito |
| **Dinâmica (lista encadeada)** | Elementos em nós com ponteiros; alocação sob demanda | Quando o tamanho é imprevisível ou muda frequentemente |
| **Vetor dinâmico** | Array que cresce/reduz com `realloc` | Compromisso entre acesso rápido e flexibilidade |

!!! warning "Importante"
    Neste capítulo, focaremos na **implementação estática com vetor**. No próximo capítulo, implementaremos o **mesmo TAD** com lista encadeada, mantendo a mesma interface. Isso demonstra o poder da abstração: o código cliente não precisa mudar quando a implementação interna muda.

---

## 5. Interface do TAD LISTA de Inteiros

### 5.1 Operações Definidas (Versão Simplificada)

Para esta primeira versão, trabalhamos com:

- **Tipo de elemento:** `int` (apenas valores, sem chave-valor)
- **Lista não ordenada:** elementos mantêm ordem de inserção

| Categoria | Operação | Descrição | Retorno |
| --- | --- | --- | --- |
| **Criação** | `criaLista()` | Cria lista vazia | `Lista*` ou `NULL` se falhar |
|  | `destroiLista(l)` | Libera memória da lista | `void` |
| **Consultas** | `conta(l)` | Retorna número de elementos | `int` |
|  | `vazia(l)` | Verifica se lista está vazia | `1` se vazia, `0` caso contrário |
|  | `cheia(l)` | Verifica se lista atingiu capacidade máxima | `1` se cheia, `0` caso contrário |
|  | `imprime(l)` | Exibe elementos na saída padrão | `void` |
| **Inserções** | `insereFinal(l, x)` | Adiciona `x` ao final da lista | `1` se sucesso, `0` se falha |
|  | `inserePos(l, x, pos)` | Adiciona `x` na posição `pos` (0 ≤ pos ≤ tamanho) | `1` se sucesso, `0` se falha |
| **Remoções** | `removePos(l, pos)` | Remove elemento da posição `pos` | `1` se removeu, `0` se falha |
|  | `removeValor(l, x)` | Remove primeira ocorrência do valor `x` | `1` se removeu, `0` se não encontrado |
| **Buscas** | `buscaPos(l, pos)` | Retorna valor na posição `pos` | Valor ou `-1` se posição inválida |
|  | `buscaValor(l, x)` | Retorna posição da primeira ocorrência de `x` | Posição ou `-1` se não encontrado |

### 5.2 Arquivo de Interface: `lista.h`

```c
/* lista.h – Interface do TAD LISTA estática de inteiros */
#ifndef LISTA_H
#define LISTA_H

/* Tipo opaco: o cliente só vê o ponteiro, não a struct interna */
typedef struct lista Lista;

/* ========== CRIAÇÃO/DESTRUIÇÃO ========== */
Lista* criaLista(void);
void destroiLista(Lista* l);

/* ========== CONSULTAS BÁSICAS ========== */
int conta(const Lista* l);
int vazia(const Lista* l);
int cheia(const Lista* l);
void imprime(const Lista* l);

/* ========== INSERÇÕES ========== */
int insereFinal(Lista* l, int x);
int inserePos(Lista* l, int x, int pos);

/* ========== REMOÇÕES ========== */
int removePos(Lista* l, int pos);
int removeValor(Lista* l, int x);

/* ========== BUSCAS ========== */
int buscaPos(const Lista* l, int pos);
int buscaValor(const Lista* l, int x);

#endif
```

**Por que usar `typedef struct lista Lista;` sem expor os campos?**

- **Encapsulamento forte**: o cliente não pode acessar `l->valores` ou `l->ultimo` diretamente
- **Independência de implementação**: se mudarmos a struct interna, o código cliente não precisa ser alterado
- **Contrato claro**: o cliente só interage via funções declaradas

---

## 6. Compilação Separada em C

### 6.1 Estrutura de Arquivos do Projeto

```
projeto_lista/
├── lista.h      /* Interface pública – o cliente #include este arquivo */
├── lista.c      /* Implementação privada – NÃO é incluído pelo cliente */
├── main.c       /* Código cliente – usa apenas a interface */
└── Makefile     /* Opcional: automatiza compilação */
```

### 6.2 Como Funciona a Compilação Separada

1. **Pré-processamento**: `#include "lista.h"` copia o conteúdo do header para `main.c`
2. **Compilação individual**:
    
    ```bash
    gcc -c lista.c -o lista.o    # Compila implementação para objeto
    gcc -c main.c -o main.o      # Compila cliente para objeto
    ```
    
3. **Linkagem**: Junta os objetos para criar o executável
    
    ```bash
    gcc lista.o main.o -o programa
    ```
    

### 6.3 Vantagens da Abordagem Modular

| Vantagem | Explicação |
| --- | --- |
| **Reusabilidade** | `lista.h` e `lista.c` podem ser usados em múltiplos projetos |
| **Manutenção** | Alterações na implementação (`lista.c`) não exigem recompilar `main.c`, apenas relinkar |
| **Encapsulamento** | Cliente não vê detalhes internos; erros de implementação ficam isolados |
| **Colaboração** | Uma pessoa pode desenvolver a interface, outra a implementação, outra o cliente |

O `.h` funciona como um contrato entre as duas partes: quem desenvolve o TAD e quem o utiliza.

---

## 7. Implementação Estática: `lista.c`

### 7.1 Estrutura Interna (Escondida do Cliente)

```c
/* lista.c – Implementação estática do TAD LISTA */
#include <stdio.h>
#include <stdlib.h>
#include "lista.h"

#define MAX 100  /* Limite estático – detalhe de implementação, fora do .h */

/* Definição concreta – NÃO exposta no .h */
struct lista {
    int valores[MAX];  /* Vetor estático */
    int ultimo;        /* Índice do último elemento: -1 = lista vazia */
};
```

**Convenções de estado:**

- `ultimo == -1` → lista vazia (0 elementos)
- `ultimo == n - 1` → lista com `n` elementos
- `ultimo == MAX - 1` → lista cheia

### 7.2 Criação e Destruição

```c
Lista* criaLista(void) {
    Lista* l = malloc(sizeof(Lista));
    if (l == NULL) return NULL;  /* Falha na alocação */
    l->ultimo = -1;               /* Inicializa como vazia */
    return l;
}

void destroiLista(Lista* l) {
    if (l != NULL) {
        free(l);  /* Libera memória alocada */
    }
}
```

### 7.3 Operações de Consulta

```c
int conta(const Lista* l) {
    return l->ultimo + 1;  /* Se ultimo=-1 → 0; se ultimo=2 → 3 elementos */
}

int vazia(const Lista* l) {
    return l->ultimo == -1;
}

int cheia(const Lista* l) {
    return l->ultimo == MAX - 1;
}

void imprime(const Lista* l) {
    if (vazia(l)) {
        printf("Lista vazia.\n");
        return;
    }
    printf("Lista [%d elementos]: ", conta(l));
    for (int i = 0; i <= l->ultimo; i++) {
        printf("%d ", l->valores[i]);
    }
    printf("\n");
}
```

### 7.4 Inserções

```c
int insereFinal(Lista* l, int x) {
    if (l == NULL || cheia(l)) return 0;
    l->ultimo++;
    l->valores[l->ultimo] = x;
    return 1;
}

int inserePos(Lista* l, int x, int pos) {
    /* Validações */
    if (l == NULL || cheia(l)) return 0;
    if (pos < 0 || pos > l->ultimo + 1) return 0;

    /* Caso especial: inserir no final → delega */
    if (pos == l->ultimo + 1) {
        return insereFinal(l, x);
    }

    /* Caso geral: deslocar elementos para a direita */
    for (int i = l->ultimo; i >= pos; i--) {
        l->valores[i + 1] = l->valores[i];
    }

    l->valores[pos] = x;
    l->ultimo++;
    return 1;
}
```

**Visualização de `inserePos(l, 15, 1)` em `[10, 20, 30]`:**

```
Estado inicial: [10, 20, 30]  (ultimo = 2)

Passo 1: i=2 → valores[3] = valores[2] → [10, 20, 30, 30]
Passo 2: i=1 → valores[2] = valores[1] → [10, 20, 20, 30]
Passo 3: i=0 → PARA (i < pos)
Passo 4: valores[1] = 15 → [10, 15, 20, 30]
Passo 5: ultimo++ → ultimo = 3

Resultado: [10, 15, 20, 30] ✓
```

### 7.5 Remoções

```c
int removePos(Lista* l, int pos) {
    if (l == NULL || vazia(l)) return 0;
    if (pos < 0 || pos > l->ultimo) return 0;

    /* Deslocar elementos para a esquerda */
    for (int i = pos; i < l->ultimo; i++) {
        l->valores[i] = l->valores[i + 1];
    }

    l->ultimo--;
    return 1;
}

int removeValor(Lista* l, int x) {
    int pos = buscaValor(l, x);  /* Reusa busca! */
    if (pos == -1) return 0;
    return removePos(l, pos);    /* Reusa remoção por posição! */
}
```

### 7.6 Buscas

```c
int buscaPos(const Lista* l, int pos) {
    if (l == NULL || pos < 0 || pos > l->ultimo) {
        return -1;  /* Posição inválida */
    }
    return l->valores[pos];
}

int buscaValor(const Lista* l, int x) {
    if (l == NULL) return -1;

    for (int i = 0; i <= l->ultimo; i++) {
        if (l->valores[i] == x) {
            return i;  /* Encontrou */
        }
    }
    return -1;  /* Não encontrou */
}
```

---

## 8. Código Cliente Exemplo: `main.c`

```c
/* main.c – Usa o TAD LISTA sem conhecer a implementação */
#include <stdio.h>
#include "lista.h"

int main(void) {
    /* 1. Criar lista */
    Lista* l = criaLista();
    if (l == NULL) {
        fprintf(stderr, "Falha ao criar lista\n");
        return 1;
    }

    /* 2. Inserções básicas */
    insereFinal(l, 10);
    insereFinal(l, 20);
    insereFinal(l, 30);
    imprime(l);  /* Saída: Lista [3 elementos]: 10 20 30 */

    /* 3. Inserção por posição */
    inserePos(l, 15, 1);
    imprime(l);  /* Saída: Lista [4 elementos]: 10 15 20 30 */

    /* 4. Consultas */
    printf("Tamanho: %d\n", conta(l));                    /* 4 */
    printf("Valor na pos 2: %d\n", buscaPos(l, 2));       /* 20 */
    printf("Pos do valor 30: %d\n", buscaValor(l, 30));   /* 3 */

    /* 5. Remoções */
    removeValor(l, 15);   /* Remove primeira ocorrência de 15 */
    imprime(l);           /* Lista [3 elementos]: 10 20 30 */

    removePos(l, 1);      /* Remove elemento na posição 1 (20) */
    imprime(l);           /* Lista [2 elementos]: 10 30 */

    /* 6. Cleanup */
    destroiLista(l);
    return 0;
}
```

✅ **Verificação importante:** `main.c` **nunca** acessa `l->valores` ou `l->ultimo`. Todo acesso é feito via funções da interface. Isso é **encapsulamento em ação**.

---

## 9. Folha de Referência Rápida

```
┌─────────────────────────────────────┐
│ LISTA ESTÁTICA EM C – CHEAT SHEET   │
├─────────────────────────────────────┤
│ REPRESENTAÇÃO INTERNA               │
│ • struct { int valores[MAX];        │
│            int ultimo; }            │
│ • vazia ⇔ ultimo == -1              │
│ • cheia ⇔ ultimo == MAX-1           │
│ • n elementos ⇔ ultimo == n-1       │
├─────────────────────────────────────┤
│ OPERAÇÕES – ASSINATURAS             │
│ Criação:  Lista* criaLista()        │
│           void destroiLista(Lista*) │
│ Consultas: int conta(), vazia()...  │
│ Inserções: insereFinal(), inserePos()│
│ Remoções:  removePos(), removeValor()│
│ Buscas:    buscaPos(), buscaValor() │
├─────────────────────────────────────┤
│ COMPLEXIDADE (para reflexão)       │
│ • Acesso por posição: O(1) ✓       │
│ • Inserção/remoção no fim: O(1) ✓  │
│ • Inserção/remoção no meio: O(n) ⚠ │
│ • Busca por valor: O(n) ⚠          │
├─────────────────────────────────────┤
│ ERROS COMUNS – CHECKLIST           │
│ □ Validar ponteiro NULL antes de usar│
│ □ Checar limites (inserePos aceita │
│   pos = ultimo+1; demais, ≤ ultimo) │
│ □ Atualizar 'ultimo' após modificar│
│ □ Não acessar valores[i] sem validar│
│ □ Lembrar: ultimo = -1 → vazia!    │
└─────────────────────────────────────┘
```

---

## 10. Guia de Estudo e Prática

### Exercícios Recomendados

- [ ]  Reimplementar `inserePos` sem consultar o código
- [ ]  Criar um teste que cause "posição inválida" e verificar que a função retorna `0`
- [ ]  Desenhar o estado da memória após esta sequência:
    
    ```
    insereFinal(5) → inserePos(3, 0) → removePos(1) → buscaValor(5)
    ```
    

### Para Aprofundar

1. **Por que `removeValor` chama `buscaValor` e `removePos`?**
    
    → Princípio **DRY** (*Don't Repeat Yourself*): reutilizar código testado reduz bugs.
    
2. **Como adaptar o TAD para `float`? E para `struct Pessoa`?**
    
    → Substituir `int` pelo tipo desejado nos parâmetros e no vetor interno.
    
3. **O que mudaria se a lista fosse ORDENADA?**
    
    → `insereFinal` não faria sentido; `inserePos` teria que encontrar a posição correta para manter a ordem.
    

### Desafio Opcional

Implemente `inverte(Lista* l)` que inverte a ordem dos elementos **sem usar vetor auxiliar**. Dica: use dois índices (`inicio`, `fim`) e troque os valores.

<details>
<summary>Clique para ver uma solução</summary>

```c
void inverte(Lista* l) {
    int inicio = 0;
    int fim = l->ultimo;
    while (inicio < fim) {
        /* Trocar valores[inicio] com valores[fim] */
        int temp = l->valores[inicio];
        l->valores[inicio] = l->valores[fim];
        l->valores[fim] = temp;

        inicio++;
        fim--;
    }
}
```

</details>

### Debug Challenge

O código abaixo tem um bug sutil. Encontre e corrija:

```c
int removePos(Lista* l, int pos) {
    for (int i = pos; i < l->ultimo; i++) {
        l->valores[i] = l->valores[i + 1];
    }
    /* O que falta aqui? */
    return 1;
}
```

<details>
<summary>Clique para ver a resposta</summary>

Falta atualizar `l->ultimo--` após o deslocamento. Sem isso, a lista reporta um elemento a mais do que realmente possui.
</details>

---

## 11. Continuando o Estudo: Listas Dinâmicas

### Pergunta para Reflexão

```c
/* lista.h (INALTERADO) */
typedef struct lista Lista;  /* Tipo opaco */

/*
 * Nesta implementação usamos:
 *   struct lista { int valores[MAX]; int ultimo; }
 *
 * Desafio: Como reescreveríamos a struct para remover
 * o limite MAX e alocar memória sob demanda,
 * mantendo a MESMA interface lista.h?
 *
 * Dica: pense em "nós" com ponteiros...
 */
```

### Comparativo: Estática vs. Dinâmica

| Característica | Lista Estática (Atual) | Lista Encadeada (Próximo Capítulo) |
| --- | --- | --- |
| Alocação | Tempo de compilação (`MAX`) | Tempo de execução (`malloc`) |
| Crescimento | Fixo | Ilimitado (até memória acabar) |
| Acesso por posição | O(1) – direto no vetor | O(n) – percorrer ponteiros |
| Inserção/remoção no meio | O(n) – deslocar elementos | O(1) – ajustar ponteiros (se já tiver a posição) |
| Uso de memória | Pode desperdiçar espaço | Usa apenas o necessário + overhead de ponteiros |

!!! tip "Mensagem Final"
    *"Neste capítulo, dominamos a lista estática. No próximo capítulo, vamos libertar a lista do limite MAX – e descobrir por que, às vezes, pagar o custo de ponteiros vale a pena."*

---

## 🔗 Referências e Leitura Complementar

1. **Abstract Data Types – Wikipedia**: Definição formal e exemplos
2. **Array Implementation of List ADT – e-PG Pathshala**: Implementação detalhada com exemplos
3. **COMP2521 – Abstract Data Types (UNSW)**: Slides com exemplos de Stack, Queue, Set em C
4. **Celes, W. et al. – Introdução a Estruturas de Dados**: Capítulos sobre TADs e compilação separada em C
5. **GeeksforGeeks – Implementation of Stack Using Array in C**: Exemplo prático de ADT estático

---

!!! note "Nota"
    Em caso de dúvidas, revise os exemplos de código, execute os testes sugeridos e consulte as referências. A prática com código é essencial para fixar os conceitos de TAD e encapsulamento.
