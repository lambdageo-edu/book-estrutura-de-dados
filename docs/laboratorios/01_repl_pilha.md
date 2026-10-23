# Laboratório: um REPL para uma máquina baseada em pilha

**Objetivo:** implementar um interpretador para uma máquina abstrata baseada em pilha.

**Contexto:** diversas linguagens de programação são compiladas para uma máquina abstrata baseada em pilha. Nessa máquina, os operandos e os resultados das operações são armazenados em uma pilha.

Nesta atividade vamos implementar um REPL (Read Eval Print Loop) similar ao que vocês utilizam ao interagir com o Python, porém interpretando comandos para uma máquina baseada em pilha, como explicado neste vídeo:

[Máquina baseada em pilha (YouTube)](https://www.youtube.com/watch?v=_j3jzoZaSbc)

O código-fonte base para o desenvolvimento está disponível em [profsergiocosta/lab-stackinterpreter](https://github.com/profsergiocosta/lab-stackinterpreter). Abaixo está o código que implementa o REPL. Ele já interage com o usuário, lê as entradas e passa cada linha para o interpretador:

```c
#include <stdio.h>

#include "interpret.h"

static void repl()
{
  char line[1024];
  for (;;)
  {
    printf("> ");

    if (!fgets(line, sizeof(line), stdin))
    {
      printf("\n");
      break;
    }

    interpret(line);
  }
}

int main () {

    repl();
    return 0;
}
```

O código base já traz o arquivo de cabeçalho do interpretador:

```c
#ifndef INTERPRET_H
#define INTERPRET_H

void interpret (const char *source) ;

#endif
```

E também já está disponível a implementação da extração do comando e do argumento passados pelo usuário:

```c
#include <string.h>
#include <stdlib.h>
#include <stdio.h>

#include "interpret.h"
#include "stack.h"

void interpret (const char *source) {

    char op[10] = "";
    char arg[10] = "";

    /* %9s limita a leitura ao tamanho dos vetores; o retorno diz quantos campos foram lidos */
    int lidos = sscanf (source, "%9s %9s", op, arg);
    if (lidos < 1) return;   /* linha vazia */
    printf("operação: %s\n", op);
    printf("argumento: %s\n",  arg);
}
```

Assim, quando o usuário digitar:

```text
> push 10
```

o comando será `push`, e o argumento a constante `10`. Você já pode compilar e experimentar.

Desenvolvam este laboratório em duas etapas: na primeira, apenas suporte a constantes; na segunda, suporte a variáveis.

## Etapa 1 — Constantes

Implementem um interpretador que funcione no formato REPL, aceitando comandos digitados pelo usuário, interpretando e executando as operações usando uma **pilha**.

### Comandos a implementar nesta etapa

1. `push <valor>` — empilha um número inteiro na pilha.
      - Exemplo: `push 20`
2. `add` — desempilha dois valores, soma, e empilha o resultado.
      - Exemplo: `push 5`, `push 3`, `add` → resultado na pilha: `8`
3. `sub` — desempilha dois valores e empilha `(penúltimo − topo)`. Exemplo: `push 9`, `push 4`, `sub` → `5`.
4. `mul` — desempilha dois valores, multiplica, e empilha o resultado.
5. `div` — desempilha dois valores e empilha `(penúltimo / topo)` (divisão inteira). Exemplo: `push 8`, `push 2`, `div` → `4`.
6. `print` — desempilha um valor e imprime imediatamente no console.
7. `exit` — encerra o REPL.

### Regras e observações

- A pilha deve ser mantida em memória enquanto o programa estiver rodando.
- O REPL deve continuar executando até que o usuário digite um comando de parada (como `exit`).
- Não é necessário, nesta etapa, trabalhar com variáveis — apenas valores inteiros.

### Exemplo de interação

```text
> push 10
> push 5
> add
> print
15
> push 8
> push 2
> div
> print
4
> exit
```

### Organização esperada do projeto

```text
/maquina-pilha
├── main.c            // Loop principal REPL
├── interpret.c       // Processamento dos comandos
├── interpret.h
├── stack.c           // Implementação da pilha
├── stack.h
└── README.md
```

### Sugestões

- Usem listas para implementar a pilha.
- Dividam as responsabilidades do código: uma parte para o REPL, outra para a lógica da pilha.
- Tratem erros simples, como tentar desempilhar de uma pilha vazia.

[Demonstração da Etapa 1 (YouTube)](https://www.youtube.com/watch?v=92Aqp95Dw64)

## Etapa 2 — Variáveis

Ampliem a funcionalidade do interpretador, implementando uma **memória de variáveis** usando uma **lista encadeada**. Esta etapa introduz o uso de variáveis nomeadas e operações entre elas.

**Conceitos praticados:**

- Implementação e uso de lista encadeada
- Armazenamento e atualização de variáveis
- Integração entre estruturas de dados: pilha e lista

### Novos comandos

`push <valor | nome_variável>`

- Se for um número inteiro: empilha normalmente.
- Se for o nome de uma variável: busca na lista encadeada pelo nome.
      - Se encontrada, empilha o valor.
      - Se não encontrada, exibe erro: `Variável não encontrada`.

```text
push a
```

`pop <nome_variável>`

- Desempilha um valor da pilha.
- Se a variável existir na lista, **atualiza o valor**.
- Se não existir, **cria um novo nó** com o nome e o valor.

```text
pop resultado
```

### Estrutura da lista (em `lista.h`)

```c
struct node {
    char key[15];
    int value;
    struct node* next;
};

struct list {
    struct node* first;
};
```

Funções sugeridas (em `lista.c`):

```c
void set_variable(struct list* l, const char* key, int value);
int get_variable(struct list* l, const char* key, int* found);
```

### Exemplo de código interpretado

O interpretador deve ser capaz de ler e executar comandos como:

```text
push 42
push 5
add
push 8
sub
pop a
push 56
push 8
add
pop b
push a
push b
add
push 6
add
print
```

Explicação resumida do que esse código faz:

- Empilha 42 e 5 → soma = 47
- Empilha 8 → subtrai: 47 - 8 = 39 → guarda em `a`
- Empilha 56 e 8 → soma = 64 → guarda em `b`
- Empilha `a` e `b` (valores 39 e 64) → soma = 103
- Empilha 6 → soma final = 109
- Imprime: `109`

### Estrutura esperada do projeto

```text
/maquina-pilha
├── main.c             // REPL principal
├── interpret.c        // Interpretador de comandos
├── interpret.h
├── stack.c            // Pilha
├── stack.h
├── lista.c            // Lista encadeada (variáveis)
├── lista.h
└── README.md
```

### Critérios de avaliação

- Implementação correta da lista encadeada
- Manipulação correta das variáveis com `push` e `pop`
- Integração com a pilha
- Código limpo e modular
- README claro e completo

[Demonstração da Etapa 2 (YouTube)](https://www.youtube.com/watch?v=WpFQuXFjyNQ)
