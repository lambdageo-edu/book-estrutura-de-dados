# Pilha III: Alocação Dinâmica

Este capítulo mostra como modelar o TAD de pilha em C como se fosse um objeto referenciado por um ponteiro — semelhante ao que acontece em Java ou Python — e evolui a implementação para uma pilha completamente dinâmica, com tamanho configurável em tempo de execução. O caminho passa por: tratar a pilha sempre como ponteiro (`Pilha *p`); entender a diferença entre `stack` e `heap`; usar `malloc` e `free` na prática (e conhecer `calloc` e `realloc`); explorar a relação entre vetores e ponteiros (`v[i]` é equivalente a `*(v + i)`); e, por fim, alocar tanto a `struct` quanto o vetor interno dinamicamente, a partir de um parâmetro `size`.

## 1. Pilha como Ponteiro: a Analogia com Java e Python

Em C, `Pilha *p` é um ponteiro para a estrutura, não a estrutura em si — e isso é semelhante ao que acontece em linguagens como Java e Python. Quando se escreve `new Stack()` em Java ou `pilha = Pilha()` em Python, o objeto é criado no heap, e a variável é apenas um ponteiro ou referência para esse objeto. Trabalhar com `Pilha *p` em C é, portanto, tratar a pilha como referência, do mesmo jeito que essas linguagens fazem por baixo dos panos: `Pilha *p` é o equivalente conceitual ao "objeto no heap, acessado por referência".

Isso também levanta uma questão sobre o tamanho do vetor interno da pilha: até aqui, `MAX` é fixo em tempo de compilação. Mas uma pilha é, no fundo, um container — e faz sentido que ela possa ter tamanhos diferentes em situações diferentes. É isso que este capítulo resolve.

## 2. Alocação Dinâmica: `Stack` vs `Heap`

A memória de um programa em execução se divide, de forma simplificada, em duas áreas:

| Área | Descrição | Gerenciamento |
| --- | --- | --- |
| **`stack`** (pilha de execução) | Espaço automático para variáveis locais | Cria e destrói automaticamente |
| **`heap`** (área dinâmica) | Espaço para alocação explícita | Programador aloca e libera manualmente |

A `stack` guarda variáveis locais de funções; o espaço é criado e destruído automaticamente a cada chamada e retorno, e seu tamanho é limitado. Já a `heap` é usada para alocação explícita, via `malloc`, `calloc` ou `realloc`; o programador é responsável por liberar essa memória com `free`, e seu tamanho é limitado apenas pela memória disponível no sistema. Uma forma simples de lembrar: `stack` é memória "automática"; `heap` é memória "manual".

As três funções centrais para trabalhar com a heap são:

```c
#include <stdlib.h>

void *malloc(size_t tamanho_em_bytes);   // Aloca N bytes brutos (não inicializa)
void *calloc(size_t n, size_t size);     // Aloca n elementos de size bytes (inicializa em 0)
void free(void *ptr);                    // Libera memória alocada
```

A regra de ouro é simples: toda vez que se usa `malloc` ou `calloc` com sucesso, mais cedo ou mais tarde é preciso usar `free` sobre o mesmo ponteiro.

### 2.1 Alocando um Único Inteiro

```c
#include <stdio.h>
#include <stdlib.h>

int main() {
    int *p = malloc(sizeof(int));

    if (p != NULL) {
        *p = 42;
        printf("Valor alocado: %d\n", *p);
        free(p);  // Libera a memória
    } else {
        printf("Falha na alocação!\n");
    }

    return 0;
}
```

`sizeof(int)` dá o tamanho em bytes de um `int` na arquitetura (geralmente 4 bytes). O próprio `p` é um ponteiro que vive na `stack`, mas o `int` que ele aponta mora na `heap` — `*p` desreferencia o ponteiro para acessar esse valor. É essencial sempre verificar se `malloc` retornou `NULL`, o que indica falha na alocação. Visualmente, o resultado é:

```
Stack:  [ p ] → contém endereço 0x1000
Heap:   [ 0x1000 ] → contém valor 42
```

### 2.2 Alocando um Vetor de Inteiros

```c
#include <stdio.h>
#include <stdlib.h>

int main() {
    int n = 100;
    int *v = malloc(n * sizeof(int));

    if (v != NULL) {
        // Preenche o vetor
        for (int i = 0; i < n; i++) {
            v[i] = i * 2;
        }

        // Acessa alguns valores
        printf("v[0] = %d\n", v[0]);
        printf("v[50] = %d\n", v[50]);

        free(v);  // Libera o vetor
    } else {
        printf("Falha na alocação!\n");
    }

    return 0;
}
```

Aqui `v` é um ponteiro para o primeiro elemento do vetor; `v + i` é o endereço do elemento de índice `i`; e `*(v + i)` é o valor armazenado nesse endereço. Para qualquer vetor `v`, vale a equivalência

```
v[i] ≡ *(v + i)
```

o que mostra o quanto vetor e ponteiro são próximos em C — as duas notações abaixo produzem exatamente o mesmo resultado:

```c
printf("v[0] = %d\n", v[0]);      // Sintaxe de vetor
printf("*(v+0) = %d\n", *(v+0));  // Sintaxe de ponteiro (mesmo resultado)

printf("v[1] = %d\n", v[1]);
printf("*(v+1) = %d\n", *(v+1));  // Mesmo resultado
```

## 3. A Struct `Pilha` Alocada na Heap

O próximo passo é aplicar essas ideias à struct `Pilha` definida no capítulo anterior — começando pela própria `struct`, ainda com um vetor estático interno:

```c
#define MAX 100

typedef struct pilha {
    int valores[MAX];  // Ainda estático
    int topo;
} Pilha;
```

Se a pilha agora é alocada com `malloc` — `Pilha *self = malloc(sizeof(Pilha))` —, o acesso aos seus membros precisa passar pelo ponteiro:

```c
Pilha *self = malloc(sizeof(Pilha));
if (self == NULL) {
    return NULL;
}

(*self).topo = -1;   // Funciona, mas é verboso
self->topo = -1;     // Equivalente, mais simples
```

Em `(*self).topo`, primeiro `self` é desreferenciado, e só depois o membro `topo` é acessado. Note que `*self.topo` não seria equivalente: o operador `.` tem precedência maior que `*`, então a expressão seria interpretada como `*(self.topo)` — um erro, já que `self` é um ponteiro e não tem membro `topo`; o `.` só pode ser aplicado depois de `self` ser desreferenciado. A regra geral é: para acessar um membro de uma struct por meio de um ponteiro, usa-se `(*p).membro` ou `p->membro`.

De fato, `p->membro` é apenas uma sintaxe abreviada para `(*p).membro`:

```c
(*self).topo = -1;   // Forma longa
self->topo = -1;     // Forma curta
```

O operador `->` é tão natural em C que a forma longa praticamente não aparece em código real.

Juntando as peças, a `struct` já mora na heap, ainda que o vetor interno permaneça estático:

```c
#include <stdio.h>
#include <stdlib.h>

#define MAX 100

typedef struct pilha {
    int valores[MAX];
    int topo;
} Pilha;

Pilha *criaPilha() {
    Pilha *self = malloc(sizeof(Pilha));
    if (self == NULL) {
        return NULL;
    }
    self->topo = -1;
    return self;
}

void empilha(int x, Pilha *self) {
    if (self->topo == MAX - 1) {
        printf("Pilha cheia!\n");
        return;
    }
    self->topo++;
    self->valores[self->topo] = x;
}

int desempilha(Pilha *self) {
    if (self->topo == -1) {
        printf("Pilha vazia!\n");
        return -1;
    }
    int valor = self->valores[self->topo];
    self->topo--;
    return valor;
}

int pilhaVazia(Pilha *self) {
    return self->topo == -1;
}

int main() {
    Pilha *p = criaPilha();
    if (p == NULL) {
        printf("Falha ao criar pilha!\n");
        return 1;
    }

    empilha(10, p);
    empilha(20, p);
    empilha(30, p);

    while (!pilhaVazia(p)) {
        printf("Desempilhando: %d\n", desempilha(p));
    }

    free(p);  // Libera a struct (vetor ainda estático)

    return 0;
}
```

O `main` trabalha apenas com ponteiros — `pilhaVazia(p)`, `empilha(..., p)`, `desempilha(p)` —, e o estado interno da pilha (`valores`, `topo`) é manipulado só por funções, usando `p->...`. Falta apenas um passo: o vetor `valores` ainda é de tamanho fixo (`MAX`).

## 4. Vetor Dinâmico: Tamanho Definido em Tempo de Execução

Se a `struct` já mora na heap, e `malloc` aceita qualquer tamanho como parâmetro, não há motivo para o vetor `valores` continuar fixo — basta receber o tamanho da pilha como parâmetro e alocar o vetor dinamicamente também:

```c
typedef struct pilha {
    int *valores;   // Agora é um ponteiro
    int topo;
    int tamanho;    // Tamanho máximo da pilha
} Pilha;
```

`valores` passa a ser um vetor alocado dinamicamente, e o novo campo `tamanho` guarda o tamanho máximo que essa pilha específica pode ter.

A função `criaPilha` passa a receber um parâmetro `size` e realiza quatro passos: aloca a própria `struct Pilha` com `malloc(sizeof(Pilha))`; inicializa `topo = -1` e `tamanho = size`; aloca o vetor de inteiros com `size * sizeof(int)`; e retorna o ponteiro `self` para essa pilha, ou `NULL` se algo falhar.

```c
Pilha *criaPilha(int size) {
    // 1. Aloca a struct Pilha
    Pilha *self = malloc(sizeof(Pilha));
    if (self == NULL) {
        return NULL;
    }

    // 2. Inicializa topo e tamanho
    self->topo = -1;
    self->tamanho = size;

    // 3. Aloca o vetor de inteiros
    self->valores = malloc(size * sizeof(int));
    if (self->valores == NULL) {
        free(self);   // Se malloc do vetor falha, libera a struct
        return NULL;
    }

    return self;
}
```

Com isso, tanto a estrutura quanto o vetor interno são alocados na heap, e o tamanho da pilha passa a ser definido em tempo de execução — não mais fixado no código-fonte. As demais funções do TAD só precisam trocar a constante `MAX` pelo campo `self->tamanho`:

```c
int pilhaCheia(Pilha *self) {
    return self->topo == self->tamanho - 1;
}

int pilhaVazia(Pilha *self) {
    return self->topo == -1;
}

void empilha(int x, Pilha *self) {
    if (pilhaCheia(self)) {
        printf("Pilha cheia!\n");
        return;
    }
    self->topo++;
    self->valores[self->topo] = x;
}

int desempilha(Pilha *self) {
    if (pilhaVazia(self)) {
        printf("Pilha vazia!\n");
        return -1;
    }
    int valor = self->valores[self->topo];
    self->topo--;
    return valor;
}

int topoPilha(Pilha *self) {
    if (pilhaVazia(self)) {
        printf("Pilha vazia!\n");
        return -1;
    }
    return self->valores[self->topo];
}
```

### 4.1 Liberando a Pilha Corretamente

Com dois blocos alocados — a struct e o vetor interno — a liberação também precisa de dois passos, na ordem certa:

```c
void liberaPilha(Pilha *self) {
    if (self != NULL) {
        free(self->valores);   // Primeiro libera o vetor interno
        free(self);            // Depois libera a struct
    }
}
```

A regra é liberar primeiro o recurso mais "interno" (`valores`), depois o que o contém (`self`). Inverter a ordem tem consequências sérias:

```c
// ERRADO
free(self);            // Perde o endereço de self->valores
free(self->valores);   // Comportamento indefinido!

// CORRETO
free(self->valores);   // Libera o vetor primeiro
free(self);            // Depois libera a struct
```

O erro mais comum, porém, é mais sutil: esquecer de chamar `free(self->valores)` e liberar apenas `free(self)`. Nesse caso a `struct` desaparece, mas o bloco de memória do vetor fica "órfão" — inacessível, porém ainda ocupado —, um vazamento de memória (*memory leak*) clássico. Ferramentas como o `valgrind` ajudam a detectar esse tipo de falha na prática.

Visualmente, uma pilha criada com `criaPilha(50)` ocupa dois blocos distintos na heap:

```
Heap:
┌─────────────────┐
│  Pilha *self    │ → 0x1000 (struct)
│  topo = -1      │
│  tamanho = 50   │
│  valores ───────┼──→ 0x2000 (vetor de 50 ints)
└─────────────────┘
```

— o que deixa claro por que são necessárias duas chamadas a `free`, uma para cada bloco.

### 4.2 Exemplo de Uso

```c
int main() {
    Pilha *p1 = criaPilha(10);   // Pilha com 10 elementos
    if (p1 == NULL) {
        printf("Falha ao criar pilha pequena!\n");
        return 1;
    }

    Pilha *p2 = criaPilha(1000); // Pilha grande
    if (p2 == NULL) {
        printf("Falha ao criar pilha grande!\n");
        liberaPilha(p1);
        return 1;
    }

    // Empilha alguns valores na primeira
    for (int i = 0; i < 10; i++) {
        empilha(i * 10, p1);
    }

    printf("Topo de p1: %d\n", topoPilha(p1));

    while (!pilhaVazia(p1)) {
        printf("Desempilhando: %d\n", desempilha(p1));
    }

    // Libera ambas as pilhas
    liberaPilha(p1);
    liberaPilha(p2);

    return 0;
}
```

`p1` e `p2` são duas pilhas completamente independentes, com tamanhos diferentes definidos em tempo de execução — exatamente o que a versão "nua" do primeiro capítulo era incapaz de oferecer.

## Síntese

Neste capítulo, a pilha deixou de ser uma coleção de variáveis globais e passou a ser tratada sempre como ponteiro (`Pilha *p`), no mesmo espírito de uma referência a objeto em Java ou Python. A `struct` e, em seguida, o vetor interno passaram a ser alocados na heap com `malloc`, com o tamanho definido em tempo de execução por um parâmetro `size` — e cada `malloc` bem-sucedido tem sua contraparte em `free`, na ordem correta.

Uma pergunta natural fica em aberto: e se a pilha preencher todo o vetor e ainda precisar crescer? A função `realloc` permite aumentar `self->valores` em tempo de execução, criando uma pilha que "cresce sozinha" quando enche — um refinamento que fica como estudo adicional. O próximo capítulo aplica esses mesmos conceitos — `struct`, ponteiros, `malloc`/`free` — ao TAD **lista**: primeiro com vetor e compilação separada (`.h`/`.c`), depois como **lista encadeada**, em que cada elemento é alocado individualmente na heap.
