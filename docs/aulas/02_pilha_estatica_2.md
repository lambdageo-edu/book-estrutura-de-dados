# Pilha Estática II: Funções, Ponteiros e Structs

Este capítulo dá continuidade à implementação de pilha em C, evoluindo a versão "nua" do capítulo anterior — baseada em variáveis globais — para uma versão organizada em torno de funções, `struct` e ponteiros. Ao final, teremos um TAD de pilha propriamente dito: com interface clara e com todo acesso ao estado interno feito por funções.

Pré-requisitos: o conceito de pilha (LIFO), a implementação "nua" com variáveis globais vista no capítulo anterior, e sintaxe básica de C (variáveis, laços, condicionais).

Ao final deste capítulo, você deve ser capaz de: compreender o comportamento de funções em C (passagem por valor e como simular passagem por referência com ponteiros); usar `struct` para agrupar dados e criar um tipo composto (`Pilha`); entender por que ponteiros são necessários para modificar estruturas dentro de funções; refatorar o código para respeitar o conceito de TAD (encapsulamento lógico); e reconhecer as limitações da alocação estática, preparando o terreno para a alocação dinâmica do próximo capítulo.

## 1. Funções em C

Uma função em C é um bloco de código que recebe zero ou mais parâmetros, executa uma tarefa bem definida e pode retornar um único valor — uma subrotina especializada que divide um programa grande em pedaços menores e reutilizáveis. Sua sintaxe básica é:

```c
tipo_retorno nome_da_funcao(tipo1 param1, tipo2 param2, ...) {
    // comandos
    return valor; // Opcional se void
}
```

`tipo_retorno` pode ser `int`, `float`, `char`, `void`, entre outros; o nome da função deve ser descritivo (`empilha`, `calculaMedia`); e `return` só aparece se o tipo de retorno não for `void`. Um exemplo simples:

```c
#include <stdio.h>

int dobro(int x) {
    return 2 * x;
}

int main() {
    int n = 5;
    int resultado = dobro(n);
    printf("O dobro de %d é %d\n", n, resultado);
    return 0;
}
```

Aqui, `n` é copiado para o parâmetro `x`; `x` é uma variável local da função, e alterar `x` não altera `n`. Quando uma função não precisa retornar valor — apenas imprimir, inicializar ou modificar algum estado —, seu tipo de retorno é `void`:

```c
void imprimirMensagem() {
    printf("Olá de uma função void!\n");
}
```

Isso leva a um ponto central: em C, os parâmetros de função são **passados por valor** (por cópia). O exemplo clássico do erro que essa regra causa é uma tentativa de trocar dois valores:

```c
void trocaErrada(int a, int b) {
    int temp = a;
    a = b;
    b = temp;
    // a e b são cópias; os originais não mudam
}

int main() {
    int x = 10, y = 20;
    trocaErrada(x, y);
    printf("x=%d y=%d\n", x, y); // Saída: x=10 y=20
    return 0;
}
```

A troca não tem efeito nenhum sobre `x` e `y` no `main`, porque `a` e `b` são cópias locais. A conclusão é direta: se uma função precisa modificar o valor de uma variável original, copiar o valor não basta — é preciso passar a referência (o endereço) da variável.

## 2. Revisitando o TAD Pilha

O TAD "pilha" já foi apresentado como uma coleção de dados junto de um conjunto de operações bem definidas, que esconde a implementação. As operações típicas de uma pilha são: empilhar (`push`), desempilhar (`pop`), ver o topo sem remover (`top`/`peek`), testar se está vazia (`isEmpty`) e testar se está cheia (`isFull`).

C não tem um tipo `Pilha` nativo, mas oferece as ferramentas para construí-lo: tipos primitivos (`int`, `float`) e a possibilidade de compor tipos novos com `struct`. As operações, por sua vez, são naturalmente representadas por funções. Assim, o TAD de pilha que vamos construir tem duas partes: `struct pilha`, o tipo concreto que guarda os dados, e um conjunto de funções — `empilha()`, `desempilha()`, `pilhaVazia()` e outras — que definem o comportamento.

## 3. Criando o Tipo `Pilha` com `struct`

A definição da struct agrupa o vetor de valores e o índice do topo em um único tipo:

```c
#define MAX 100

// Definição do tipo Pilha
typedef struct pilha {
    int valores[MAX];
    int topo;
} Pilha;
```

`int valores[MAX]` armazena os elementos da pilha em um vetor estático; `int topo` guarda o índice do topo; e `typedef struct pilha { ... } Pilha;` cria um novo tipo `Pilha`, que passa a poder ser usado como `Pilha p1;` — assim como `int` ou `float`, só que composto por um vetor e um inteiro.

Nada, porém, impede ainda que o código cliente acesse os membros da struct diretamente:

```c
#include <stdio.h>

#define MAX 100

typedef struct pilha {
    int valores[MAX];
    int topo;
} Pilha;

int main() {
    Pilha p;
    p.topo = -1; // Inicializa a pilha vazia

    // Empilha 10 (Acesso direto)
    if (p.topo < MAX - 1) {
        p.topo++;
        p.valores[p.topo] = 10;
    }

    // Desempilha (Acesso direto)
    if (p.topo != -1) {
        int valor = p.valores[p.topo];
        printf("Desempilhado: %d\n", valor);
        p.topo--;
    }

    return 0;
}
```

O operador `.` acessa os campos da struct (`p.valores`, `p.topo`) — é essencialmente o mesmo código da pilha "nua" do capítulo anterior, apenas organizado dentro de um tipo `Pilha`. O problema persiste: o `main` ainda mexe diretamente no estado interno, violando o encapsulamento que um TAD deveria garantir.

## 4. Funções para a Pilha

### 4.1 Uma Tentativa (Errada) Sem Ponteiros

Antes de corrigir o problema, vale ver o erro conceitual que aparece ao tentar resolvê-lo apenas com funções, sem ponteiros:

```c
void empilha(int x, Pilha p) { // Recebe uma CÓPIA da struct
    if (p.topo < MAX - 1) {
        p.topo++;
        p.valores[p.topo] = x;
    } else {
        printf("Pilha cheia!\n");
    }
}

int main() {
    Pilha p;
    p.topo = -1;

    empilha(10, p); // Passando a struct por valor

    // Mesmo após empilha, p não foi alterado
    if (p.topo != -1) {
        printf("Valor no topo: %d\n", p.valores[p.topo]);
    } else {
        // É este ramo que executa: topo continua -1
        printf("Pilha vazia!\n");
    }

    return 0;
}
```

Como `Pilha p` é passado por valor, `empilha` recebe uma cópia inteira de `p`. A modificação de `p.topo` e `p.valores` ocorre apenas nessa cópia — ao voltar para o `main`, `p.topo` continua `-1` e o elemento empilhado simplesmente não aparece. É exatamente o mesmo problema da troca de valores da seção 1, agora em uma struct.

### 4.2 Ponteiros: Passando a Referência, Não a Cópia

Para que `empilha` consiga de fato alterar a pilha do chamador, a função precisa receber não uma cópia, mas uma referência para a pilha original — e em C, referência é feita com ponteiros:

```c
int x = 10;
int *ptr = &x;   // ptr guarda o endereço de x

// & → operador de endereço ("endereçar")
// * → operador de acesso ao valor apontado ("desreferenciar")
```

Uma forma simples de lembrar: `&` significa "me dê o endereço"; `*` significa "me dê o valor que está nesse endereço". Com isso, a função `empilha` pode ser reescrita para receber um ponteiro para `Pilha`:

```c
void empilha(int x, Pilha *p) { // Recebe o ENDEREÇO da pilha
    if (p->topo < MAX - 1) {    // Usa operador seta (->)
        p->topo++;
        p->valores[p->topo] = x;
    } else {
        printf("Pilha cheia!\n");
    }
}
```

Aqui `p` é um ponteiro para `Pilha`, e `p->topo` é equivalente a `(*p).topo`: primeiro desreferencia `p`, depois acessa o membro. Agora a função modifica o `Pilha` original apontado por `p`, não uma cópia.

A mesma técnica corrige a troca de valores da seção 1. Rigorosamente, C continua passando tudo por valor: o que é copiado agora é o *endereço*, e por meio dele a função alcança as variáveis originais.

```c
void troca(int *a, int *b) {
    int temp = *a;
    *a = *b;
    *b = temp;
}

int main() {
    int x = 10, y = 20;
    troca(&x, &y);
    printf("x=%d y=%d\n", x, y); // Saída: x=20 y=10
    return 0;
}
```

### 4.3 Código Completo do TAD

Juntando as peças, chegamos a uma interface de pilha completa, em que todo acesso ao estado interno passa por funções:

```c
#include <stdio.h>

#define MAX 100

typedef struct pilha {
    int valores[MAX];
    int topo;
} Pilha;

// Interface do TAD
void inicializaPilha(Pilha *p);
int pilhaVazia(Pilha *p);
int pilhaCheia(Pilha *p);
void empilha(int x, Pilha *p);
int desempilha(Pilha *p);
int topoPilha(Pilha *p);

// Implementação
void inicializaPilha(Pilha *p) {
    p->topo = -1;
}

int pilhaVazia(Pilha *p) {
    return p->topo == -1;
}

int pilhaCheia(Pilha *p) {
    return p->topo == MAX - 1;
}

void empilha(int x, Pilha *p) {
    if (pilhaCheia(p)) {
        printf("Pilha cheia!\n");
        return;
    }
    p->topo++;
    p->valores[p->topo] = x;
}

int desempilha(Pilha *p) {
    if (pilhaVazia(p)) {
        printf("Pilha vazia!\n");
        return -1; // Valor de erro
    }
    int valor = p->valores[p->topo];
    p->topo--;
    return valor;
}

int topoPilha(Pilha *p) {
    if (pilhaVazia(p)) {
        printf("Pilha vazia!\n");
        return -1;
    }
    return p->valores[p->topo];
}

// Uso do TAD
int main() {
    Pilha p;
    inicializaPilha(&p); // Passa o endereço

    empilha(10, &p);
    empilha(20, &p);
    empilha(30, &p);

    printf("Topo: %d\n", topoPilha(&p));
    printf("Desempilhado: %d\n", desempilha(&p));

    empilha(40, &p);

    while (!pilhaVazia(&p)) {
        printf("Desempilhando: %d\n", desempilha(&p));
    }

    return 0;
}
```

Note que o `main` agora usa apenas as funções da interface — o estado interno (`valores`, `topo`) só é acessado por meio de ponteiro, dentro das próprias funções do TAD. Isso já se aproxima de um TAD "limpo": quem usa a pilha só precisa conhecer o tipo `Pilha` e o conjunto de funções disponíveis. Note, porém, que a proteção ainda é apenas uma convenção: como a `struct` está visível no mesmo arquivo, nada impede o `main` de escrever `p.topo = 1000`. O encapsulamento de fato, com arquivos `.h`/`.c` separados e tipo opaco, é apresentado no capítulo de Listas Estáticas.

Observe também que `desempilha` e `topoPilha` usam `-1` como valor de erro, o que é ambíguo se `-1` for um valor legítimo na pilha. Por isso o `main` sempre testa `pilhaVazia` antes de desempilhar.

## 5. O Limite do Vetor Estático

Mesmo com funções, ponteiros e passagem por referência resolvidos, resta uma limitação: o vetor `valores[MAX]` continua com tamanho fixo, definido em tempo de compilação. Não é possível criar uma pilha de tamanho arbitrário — 20, 30, 1000 elementos — com esse modelo, porque `MAX` é uma constante fixa no código-fonte. Se quisermos flexibilidade real, `valores` precisa deixar de ser um vetor estático e passar a ser alocado dinamicamente.

Esse é o assunto do próximo capítulo: substituir `int valores[MAX];` por `int *valores;` dentro de `struct pilha`, alocar `p->valores` com `malloc` de acordo com a capacidade desejada, e introduzir funções como `criaPilha(int capacidade)` e `liberaPilha(Pilha *p)` para gerenciar essa memória — tornando a pilha verdadeiramente flexível e reutilizável para qualquer tamanho.
