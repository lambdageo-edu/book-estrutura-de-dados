# Árvores: Conceitos e Binária

!!! note "Material de Apoio"
    Você pode acompanhar este capítulo utilizando os [Slides sobre Conceitos de Árvores](../slides/arvores_conceitos.pdf).

## 1. Introdução: O que são Estruturas de Dados?

---

> **Definição:** Organização de dados e operações (algoritmos) que podem ser aplicadas sobre esses dados como forma de apoio à solução de problemas. Estruturas de dados podem ser utilizadas para representar **Tipos Abstratos de Dados (TADs)** em alguma linguagem de programação.

### Exemplos de Estruturas de Dados:

| Lineares | Não-lineares |
| --- | --- |
| Pilhas | **Árvores** |
| Filas | Grafos |
| Listas encadeadas | Tabelas de dispersão |
| Vetores |  |

### Por que estudar estruturas não-lineares?

Enquanto estruturas lineares organizam dados em sequência (um após o outro), estruturas como **árvores** mantêm um **relacionamento hierárquico** entre os elementos, permitindo:

- Representar relações de parentesco, organização, dependência
- Acessar dados de forma mais eficiente em certos cenários
- Modelar problemas do mundo real de maneira mais natural

---

## 2. Árvores: Conceitos Fundamentais

### 2.1 Definição Recursiva

Uma **árvore enraizada** *T*, ou simplesmente árvore, é um conjunto finito de elementos denominados **nós** ou **vértices**, tal que:

```
• T = Ø, quando a árvore é dita vazia, ou

• T = {r} ∪ T₁ ∪ T₂ ∪ T₃ ∪ ... ∪ Tₙ, com n ≥ 0
```

**Nesta definição:**

- **r** é um nó especial chamado **raiz**
- Os demais nós formam conjuntos disjuntos *T₁, T₂, ..., Tₙ*, chamados de **subárvores** de *r*, cada qual uma árvore

> **Note a recursividade da definição:** uma árvore é definida em termos de outras árvores menores.

### 2.2 Exemplos de Árvores no Mundo Real

```
🌳 Árvore Genealógica          📚 Organização de um Livro
       Avós                           Capítulo 1
        │                             ├── Seção 1.1
    ┌───┴───┐                         ├── Seção 1.2
  Pais     Tios                      └── Seção 1.3
    │                                 └── Subseção 1.3.1
  ┌─┴─┐
Você  Irmão

🏢 Organograma Acadêmico        🌐 Estrutura HTML
   Diretor                      <html>
      │                          ├── <head>
   ┌──┴──┐                       └── <body>
Coord.  Coord.                      ├── <h1>
Acad.   Admin.                      ├── <p>
   │                                └── <div>
Professores                            ├── <img>
   │                                   └── <footer>
Alunos
```

### 2.3 Representações de Árvores

#### a) Representação Textual (aninhamento de chaves)

As sequências de chaves `{` e `}` representam as relações entre os nós; o rótulo de cada nó é inserido imediatamente à direita do `{` correspondente.

**Exemplos:**

```
Ta = {A}
Tb = {B, {A}}
Tc = {D, {E, {F}}, {G, {H, {I}}, {J, {K}, {L}}, {M}}}
```

#### b) Representação por Identação

```
Tc = {D, {E, {F}}, {G, {H, {I}}, {J, {K}, {L}}, {M}}}

D
├─ E
│  └─ F
└─ G
   ├─ H
   │  └─ I
   ├─ J
   │  ├─ K
   │  └─ L
   └─ M
```

#### c) Representação Gráfica (como grafo)

```mermaid
graph TD
    D((D)) --> E((E))
    D --> G((G))
    E --> F((F))
    G --> H((H))
    G --> J((J))
    G --> M((M))
    H --> I((I))
    J --> K((K))
    J --> L((L))
```

---

## 3. Terminologia de Árvores

Considere a árvore: `T = {A, {B, {D}, {E}}, {C, {F}}}`

![Árvore com raiz no nível 0, filhos no nível 1 e folhas no nível 2](images/arvores_conceitos/raiz_nivel.svg)

### 3.1 Relações Genealógicas

| Termo | Definição | Exemplo na árvore acima |
| --- | --- | --- |
| **Raiz** | Nó especial sem pai | `A` |
| **Filho** | Nó diretamente conectado abaixo de outro | `B` e `C` são filhos de `A` |
| **Pai** | Nó diretamente conectado acima de outro | `A` é pai de `B` e `C` |
| **Irmãos** | Nós que compartilham o mesmo pai | `B` e `C` são irmãos; `D` e `E` são irmãos |
| **Descendente** | Nó que está em alguma subárvore de outro | `D`, `E`, `F` são descendentes de `A` |
| **Ancestral** | Nó no caminho da raiz até um nó | `A` e `B` são ancestrais de `D` |

### 3.2 Grau, Folhas e Altura

![Árvore ilustrando o grau de cada nó](images/arvores_conceitos/grau_exemplo.svg)

- **Grau de um nó**: número de filhos que ele possui
- **Grau da árvore**: máximo entre os graus de seus nós
- **Folha (nó terminal)**: nó com grau zero (sem filhos)

### 3.3 Caminho e Comprimento

- **Caminho**: sequência de nós distintos *w₁, w₂, ..., wⱼ*, tal que existe sempre entre nós consecutivos a relação "é filho de" ou "é pai de"
- **Comprimento do caminho**: número de arestas (pares consecutivos) no caminho

**Exemplo:** Caminho de `A` até `F`: `A → C → F` (comprimento = 2)

### 3.4 Nível (Profundidade) e Altura

| Conceito | Definição | Exemplo |
| --- | --- | --- |
| **Nível de um nó** | Tamanho do caminho da raiz até esse nó | `A`: nível 0; `B,C`: nível 1; `D,E,F`: nível 2 |
| **Altura de um nó** | Tamanho do maior caminho desse nó até uma folha descendente | `D,E,F`: altura 0; `B,C`: altura 1; `A`: altura 2 |
| **Altura da árvore** | Altura da raiz | `altura(T) = 2` |

> **Dica:** Raiz tem nível 0; folhas têm altura 0.

---

## 4. Árvores Binárias: Definição Específica

### 4.1 Definição Formal

Uma **árvore binária** *T* é um conjunto finito de elementos, denominados nós ou vértices, tal que:

```
• T = Ø, quando a árvore é dita vazia, ou

• T = {r} ∪ T_esq ∪ T_dir
```

**Nesta definição:**

- **r** é um nó especial chamado **raiz**
- **T_esq** e **T_dir** são conjuntos disjuntos (podem ser vazios), chamados **subárvore à esquerda** e **subárvore à direita** de **r**, cada qual uma árvore binária

> **Diferença crucial:** Em árvores binárias, a ordem das subárvores importa! Subárvore esquerda ≠ subárvore direita.

### 4.2 Comparação: Árvore Geral vs. Árvore Binária

**Árvore Geral (n filhos):** ordem dos filhos não importa, grau arbitrário.

```mermaid
graph TD
    R1((•)) --> A1((•))
    R1 --> A2((•))
    R1 --> A3((•))
    A1 --> B1((•))
    A1 --> B2((•))
```

**Árvore Binária (máx. 2 filhos):** esquerda e direita são distintos, grau máximo = 2.

![Exemplo de árvore binária com no máximo dois filhos por nó](images/arvores_conceitos/comparacao_binaria.svg)

---

## 5. Implementação em Linguagem C

### 5.1 Definição do Tipo de Dado

```c
#include <stdio.h>
#include <stdlib.h>

// Definição do nó da árvore binária
typedef struct No {
    char valor;              // Informação armazenada (pode ser int, float, etc.)
    struct No* esquerda;     // Ponteiro para subárvore à esquerda
    struct No* direita;      // Ponteiro para subárvore à direita
} No;
```

**Explicação dos componentes:**

- `typedef struct No`: cria um novo tipo de dado chamado `No`
- `char valor`: campo de dados (neste exemplo, um caractere; pode ser adaptado)
- `struct No* esquerda/direita`: ponteiros recursivos para outros nós do mesmo tipo
- A estrutura é **autorreferenciada**: um `No` contém ponteiros para outros `No`

### 5.2 Função para Criar um Novo Nó

```c
// Função para criar um novo nó com valor informado
No* criarNo(char valor) {
    // Aloca memória dinamicamente na heap
    No* novo = (No*) malloc(sizeof(No));

    // Boa prática: verificar se a alocação foi bem-sucedida
    if (novo == NULL) {
        fprintf(stderr, "Erro: falha ao alocar memória para nó '%c'\n", valor);
        exit(EXIT_FAILURE);  // Encerra o programa com código de erro
    }

    // Inicializa os campos do nó
    novo->valor = valor;
    novo->esquerda = NULL;   // Inicialmente, sem filhos
    novo->direita = NULL;

    return novo;  // Retorna o ponteiro para o novo nó
}
```

> **Boas práticas demonstradas:**
> - Verificação de `malloc` para evitar *segmentation fault*
> - Inicialização explícita dos ponteiros com `NULL`
> - Mensagem de erro descritiva em `stderr`

### 5.3 Montando uma Árvore Manualmente

```c
int main() {
    // Passo 1: Criar todos os nós individualmente
    No* A = criarNo('A');
    No* B = criarNo('B');
    No* C = criarNo('C');
    No* D = criarNo('D');
    No* E = criarNo('E');
    No* F = criarNo('F');

    // Passo 2: Conectar os nós para formar a estrutura da árvore
    // Representação gráfica da árvore montada:
    //        A
    //       / \
    //      B   C
    //     / \   \
    //    D   E   F

    A->esquerda = B;    // B é filho à esquerda de A
    A->direita  = C;    // C é filho à direita de A
    B->esquerda = D;    // D é filho à esquerda de B
    B->direita  = E;    // E é filho à direita de B
    C->esquerda = NULL; // C não tem filho à esquerda (explícito)
    C->direita  = F;    // F é filho à direita de C

    // A árvore está pronta para ser percorrida ou manipulada
    // ... (próximas seções mostram como)

    return 0;
}
```

### 5.4 Função Auxiliar: Liberar Memória

```c
// Libera toda a memória alocada para a árvore (usando pós-ordem)
void liberarArvore(No* raiz) {
    if (raiz != NULL) {
        // Primeiro libera as subárvores (recursivamente)
        liberarArvore(raiz->esquerda);
        liberarArvore(raiz->direita);

        // Opcional: mensagem de debug (ANTES do free: depois dele, raiz não pode ser acessado)
        // printf("Liberando nó: %c\n", raiz->valor);

        // Depois libera o nó atual
        free(raiz);
    }
}
```

> **Importante:** Sempre chame `liberarArvore(raiz)` no final do `main()` para evitar *memory leaks*.

---

## 6. Tipos de Árvores Binárias

### 6.1 Árvore Estritamente Binária

> **Definição:** Cada nó tem grau **0 ou 2** (ou seja, todo nó tem exatamente 0 ou 2 filhos).

**a) Estritamente binária** — todos os nós têm 0 ou 2 filhos ✓

![Árvore estritamente binária: todo nó tem 0 ou 2 filhos](images/arvores_conceitos/estritamente_binaria_a.svg)

**b) NÃO estritamente binária** — nó com 1 filho, violando a regra ✗

![Árvore não estritamente binária: um nó tem apenas 1 filho](images/arvores_conceitos/estritamente_binaria_b.svg)

### 6.2 Árvore Binária Completa

> **Definição:** Árvore binária na qual todo nó que apresente alguma subárvore vazia está localizado no **último ou penúltimo nível** da árvore.

**a) Completa** — nós com subárvores vazias apenas nos últimos níveis ✓

![Árvore completa: subárvores vazias só nos últimos níveis](images/arvores_conceitos/completa_a.svg)

**b) NÃO completa** — nó com subárvore vazia em nível intermediário ✗

![Árvore não completa: subárvore vazia em nível intermediário](images/arvores_conceitos/completa_b.svg)

### 6.3 Árvore Binária Cheia (Perfeita)

> **Definição:** Todos os nós internos têm grau 2 e **todas as folhas estão no mesmo nível**.

!!! warning "Cuidado com a terminologia em inglês"
    O que chamamos aqui de árvore **cheia** corresponde, em inglês, a *perfect binary tree*. O termo *full binary tree* costuma designar a árvore **estritamente binária** (todo nó com 0 ou 2 filhos).

![Árvore binária cheia: todos os nós internos com 2 filhos, folhas no mesmo nível](images/arvores_conceitos/cheia_perfeita.svg)

✓ Todos os nós internos têm 2 filhos
✓ Todas as folhas estão no nível 2

### 6.4 Propriedades da Árvore Binária Cheia

Seja *k* o nível máximo (altura) da árvore:

| Propriedade | Fórmula |
| --- | --- |
| Número de nós no nível *i* | **2ⁱ** |
| Número total de nós | **2ᵏ⁺¹ − 1** |
| Altura, dado *n* nós | **log₂(n+1) − 1** |

**Exemplo numérico:**

- Árvore cheia de altura *k = 2*:
    - Nível 0: 2⁰ = 1 nó
    - Nível 1: 2¹ = 2 nós
    - Nível 2: 2² = 4 nós
    - Total: 1 + 2 + 4 = **7 nós** = 2³ − 1 ✓

---

## 7. Percursos em Árvores Binárias (Traversal)

### 7.1 Conceito de Percurso

> **Percorrer** uma árvore significa **visitar cada nó exatamente uma vez**. "Visitar" pode ser:
> - Imprimir o valor do nó
> - Somar valores numéricos
> - Contar nós
> - Buscar uma informação específica
> - Modificar o conteúdo do nó

Não existe um único percurso correto. A escolha depende da aplicação.

### 7.2 Os Três Percursos Básicos

| Percurso | Ordem de Visita | Aplicação Típica |
| --- | --- | --- |
| **Pré-ordem** | Raiz → Esquerda → Direita | Copiar árvore, expressão pré-fixa |
| **Em-ordem** | Esquerda → Raiz → Direita | Obter valores ordenados (em ABB) |
| **Pós-ordem** | Esquerda → Direita → Raiz | Liberar memória, expressão pós-fixa |

### 7.3 Implementação dos Percursos em C

Considere a árvore de exemplo:

![Árvore de exemplo para os percursos pré-ordem, em-ordem e pós-ordem](images/arvores_conceitos/percursos_exemplo.svg)

#### a) Pré-ordem (Pre-order)

```c
// Pré-ordem: Raiz, Esquerda, Direita
void preOrdem(No* raiz) {
    if (raiz != NULL) {
        printf("%c ", raiz->valor);        // 1. Visita a raiz
        preOrdem(raiz->esquerda);          // 2. Percorre subárvore esquerda
        preOrdem(raiz->direita);           // 3. Percorre subárvore direita
    }
}
```

**Execução passo a passo:**

```
preOrdem(A)
├─ imprime 'A'
├─ preOrdem(B)
│  ├─ imprime 'B'
│  ├─ preOrdem(D) → imprime 'D'
│  └─ preOrdem(E) → imprime 'E'
└─ preOrdem(C)
   ├─ imprime 'C'
   ├─ preOrdem(NULL) → nada
   └─ preOrdem(F) → imprime 'F'
```

**Saída:** `A B D E C F`

#### b) Em-ordem (In-order)

```c
// Em-ordem: Esquerda, Raiz, Direita
void emOrdem(No* raiz) {
    if (raiz != NULL) {
        emOrdem(raiz->esquerda);           // 1. Percorre subárvore esquerda
        printf("%c ", raiz->valor);        // 2. Visita a raiz
        emOrdem(raiz->direita);            // 3. Percorre subárvore direita
    }
}
```

**Saída para o mesmo exemplo:** `D B E A C F`

> **Observação:** Em Árvores Binárias de Busca (ABB), o percurso em-ordem retorna os elementos **ordenados crescentemente**.

#### c) Pós-ordem (Post-order)

```c
// Pós-ordem: Esquerda, Direita, Raiz
void posOrdem(No* raiz) {
    if (raiz != NULL) {
        posOrdem(raiz->esquerda);          // 1. Percorre subárvore esquerda
        posOrdem(raiz->direita);           // 2. Percorre subárvore direita
        printf("%c ", raiz->valor);        // 3. Visita a raiz
    }
}
```

**Saída para o mesmo exemplo:** `D E B F C A`

> **Aplicação:** Útil para liberar memória (deletar a árvore) ou avaliar expressões pós-fixas (notação polonesa reversa).

### 7.4 Código Completo de Exemplo

```c
#include <stdio.h>
#include <stdlib.h>

typedef struct No {
    char valor;
    struct No* esquerda;
    struct No* direita;
} No;

No* criarNo(char valor) {
    No* novo = (No*) malloc(sizeof(No));
    if (novo == NULL) {
        fprintf(stderr, "Erro de alocação\n");
        exit(EXIT_FAILURE);
    }
    novo->valor = valor;
    novo->esquerda = novo->direita = NULL;
    return novo;
}

void preOrdem(No* raiz) {
    if (raiz != NULL) {
        printf("%c ", raiz->valor);
        preOrdem(raiz->esquerda);
        preOrdem(raiz->direita);
    }
}

void emOrdem(No* raiz) {
    if (raiz != NULL) {
        emOrdem(raiz->esquerda);
        printf("%c ", raiz->valor);
        emOrdem(raiz->direita);
    }
}

void posOrdem(No* raiz) {
    if (raiz != NULL) {
        posOrdem(raiz->esquerda);
        posOrdem(raiz->direita);
        printf("%c ", raiz->valor);
    }
}

void liberarArvore(No* raiz) {
    if (raiz != NULL) {
        liberarArvore(raiz->esquerda);
        liberarArvore(raiz->direita);
        free(raiz);
    }
}

int main() {
    // Montagem da árvore
    No* A = criarNo('A');
    No* B = criarNo('B');
    No* C = criarNo('C');
    No* D = criarNo('D');
    No* E = criarNo('E');
    No* F = criarNo('F');

    A->esquerda = B; A->direita = C;
    B->esquerda = D; B->direita = E;
    C->direita = F;

    // Execução dos percursos
    printf("Pre-ordem:  ");
    preOrdem(A);  printf("\n");

    printf("Em-ordem:   ");
    emOrdem(A);  printf("\n");

    printf("Pos-ordem:  ");
    posOrdem(A); printf("\n");

    // Liberação de memória
    liberarArvore(A);

    return 0;
}
```

**Saída esperada:**

```
Pre-ordem:  A B D E C F
Em-ordem:   D B E A C F
Pos-ordem:  D E B F C A
```

---

## 8. Funções Utilitárias (Exercícios Guiados)

### 8.1 Contar o Número de Nós

```c
// Retorna o número total de nós na árvore
int contarNos(No* raiz) {
    if (raiz == NULL) {
        return 0;  // Caso base: árvore vazia
    }
    // Caso recursivo: 1 (raiz) + nós da esquerda + nós da direita
    return 1 + contarNos(raiz->esquerda) + contarNos(raiz->direita);
}
```

**Raciocínio:**

- Se a árvore é vazia → 0 nós
- Caso contrário → 1 (nó atual) + contagem das subárvores

### 8.2 Calcular a Altura da Árvore

```c
// Retorna a altura da árvore (folhas têm altura 0)
int altura(No* raiz) {
    if (raiz == NULL) {
        return -1;  // Definição: altura de árvore vazia = -1
    }
    // Altura = 1 + maior altura entre as subárvores
    int altEsq = altura(raiz->esquerda);
    int altDir = altura(raiz->direita);
    return 1 + (altEsq > altDir ? altEsq : altDir);
}
```

**Exemplo:** Para a árvore `A(B(D,E), C(F))`:

- `altura(D) = altura(E) = altura(F) = 0` (folhas)
- `altura(B) = 1 + max(0,0) = 1`
- `altura(C) = 1 + max(-1,0) = 1`
- `altura(A) = 1 + max(1,1) = 2` ✓

### 8.3 Espelhar a Árvore (Reflexão)

```c
// Inverte a árvore: subárvore esquerda ↔ direita
void espelhar(No* raiz) {
    if (raiz != NULL) {
        // Troca os ponteiros
        No* temp = raiz->esquerda;
        raiz->esquerda = raiz->direita;
        raiz->direita = temp;

        // Aplica recursivamente nas subárvores
        espelhar(raiz->esquerda);
        espelhar(raiz->direita);
    }
}
```

**Antes e depois:**

**Antes:**

![Árvore antes do espelhamento](images/arvores_conceitos/espelhamento_antes.svg)

**Depois (espelhada):**

![Árvore depois do espelhamento: filhos esquerdo e direito trocados em todos os níveis](images/arvores_conceitos/espelhamento_depois.svg)

---

## 9. Exercícios para Fixação

### Exercício 1 — Representação e Montagem

Dada a representação textual:

```
T = {X, {Y, {Z}, Ø}, {W, Ø, {V}}}
```

a) Desenhe a árvore correspondente em formato gráfico.

b) Escreva o código C para montá-la usando `criarNo()`.

c) Imprima os três percursos (pré, em, pós-ordem).

<details>
<summary>Gabarito parcial (clique para expandir)</summary>

```
a) Representação gráfica:
       X
      / \
     Y   W
    /     \
   Z       V

b) Montagem em C:
   No* X = criarNo('X');
   No* Y = criarNo('Y');
   No* Z = criarNo('Z');
   No* W = criarNo('W');
   No* V = criarNo('V');

   X->esquerda = Y; X->direita = W;
   Y->esquerda = Z; Y->direita = NULL;
   W->esquerda = NULL; W->direita = V;

c) Percursos:
   Pré-ordem:  X Y Z W V
   Em-ordem:   Z Y X W V
   Pós-ordem:  Z Y V W X
```

</details>

### Exercício 2 — Função de Contagem

Implemente e teste a função `contarNos()` apresentada na Seção 8.1. Verifique se ela retorna 6 para a árvore de exemplo `A(B(D,E), C(F))`.

### Exercício 3 — Cálculo de Altura

Implemente a função `altura()` e verifique:

- `altura(NULL) = -1`
- `altura(folha) = 0`
- `altura(A(B(D,E), C(F))) = 2`

### Exercício 4 — Árvore Cheia: Verificação

Escreva uma função que verifique se uma árvore binária é **cheia**:

```c
int ehCheia(No* raiz);
// Dica: uma árvore é cheia se:
// - for NULL, ou
// - suas duas subárvores forem cheias E tiverem a mesma altura
// (Alternativa: compare contarNos(raiz) com 2^(altura+1) - 1)
```

### Exercício 5 — Impressão Formatada

Crie uma função que imprima a árvore com identação para visualização hierárquica:

```
A
├─ B
│  ├─ D
│  └─ E
└─ C
   └─ F
```

<details>
<summary>Dica de implementação</summary>

O segredo é passar adiante um **prefixo** com as barras verticais dos níveis de cima: quem ainda tem irmão abaixo deixa `│  `; quem é o último deixa espaços.

```c
#include <stdio.h>

void imprimirRec(No* no, const char* prefixo, int ehUltimo) {
    printf("%s%s%c\n", prefixo, ehUltimo ? "└─ " : "├─ ", no->valor);

    char novoPrefixo[256];
    snprintf(novoPrefixo, sizeof novoPrefixo, "%s%s", prefixo, ehUltimo ? "   " : "│  ");

    if (no->esquerda != NULL)
        imprimirRec(no->esquerda, novoPrefixo, no->direita == NULL);
    if (no->direita != NULL)
        imprimirRec(no->direita, novoPrefixo, 1);
}

void imprimirArvore(No* raiz) {
    if (raiz == NULL) return;
    printf("%c\n", raiz->valor);
    if (raiz->esquerda != NULL)
        imprimirRec(raiz->esquerda, "", raiz->direita == NULL);
    if (raiz->direita != NULL)
        imprimirRec(raiz->direita, "", 1);
}
// Chamar com: imprimirArvore(raiz);
```

Repare que, quando um nó tem um único filho, o desenho não mostra se ele é esquerdo ou direito. Como extensão, imprima `(E)` ou `(D)` ao lado de cada nó.

</details>

---

## 10. Resumo do Capítulo

**Conceitos de árvores gerais**

- Definição recursiva: `T = {r} ∪ T₁ ∪ ... ∪ Tₙ`
- Representações: textual `{}`, identação, gráfica
- Terminologia: raiz, filho, pai, grau, folha, ancestral, nível, altura

**Árvores binárias: especificidades**

- Definição: `T = {r} ∪ T_esq ∪ T_dir` (ordem importa!)
- Implementação em C com `struct`, ponteiros e alocação dinâmica
- Tipos: estritamente binária, completa, cheia
- Propriedades matemáticas da árvore cheia

**Percursos (Traversal)**

- Pré-ordem: Raiz → Esq → Dir
- Em-ordem: Esq → Raiz → Dir
- Pós-ordem: Esq → Dir → Raiz
- Implementação recursiva em C

**Boas práticas de programação**

- Verificar retorno de `malloc`
- Inicializar ponteiros com `NULL`
- Liberar memória com `free()` em pós-ordem
- Usar recursão com caso base bem definido

**Funções utilitárias**

- `contarNos()`, `altura()`, `espelhar()`, `liberarArvore()`

---

## 11. Próximos Passos

**Tópicos abordados nos próximos capítulos:**

- Árvores Binárias de Busca (ABB): inserção, busca e remoção com ordenação
- Balanceamento de árvores (AVL)
- Para pesquisa complementar (fora do escopo deste livro): árvores Rubro-Negras, expressões aritméticas, índices de banco de dados (árvores B), compressão (Huffman)

**Leitura recomendada:**

- ZIVIANI, N. *Projeto de Algoritmos*. Capítulo 5: Pesquisa em Memória Primária (árvores de pesquisa).
- GOODRICH, M. T.; TAMASSIA, R.; MOUNT, D. *Data Structures and Algorithms in C++*. Capítulo 7: Trees.

**Prática sugerida:**

1. Compile e execute o código completo da Seção 7.4
2. Modifique o programa para aceitar entrada do usuário
3. Implemente pelo menos dois exercícios da Seção 9
4. Experimente representar expressões aritméticas como árvores binárias

---

> **Dica para provas:** Desenhe a árvore antes de codificar! Visualizar a estrutura ajuda a entender a recursão dos percursos e a lógica das operações.

---

*Material elaborado para a disciplina de Estrutura de Dados — Curso de Engenharia de Computação*

*Prof. Sérgio Souza Costa | Atualizado: 2026*