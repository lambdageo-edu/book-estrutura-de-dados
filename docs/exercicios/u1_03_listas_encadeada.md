# Lista Encadeada

Usando como base essa implementação, crie as operações propostas a seguir:

```c
#include <stdio.h>
#include <stdlib.h>

typedef struct no {
    int valor;
    struct no* proximo;
} No;

typedef struct lista {
    No* inicio;
} Lista;

Lista* cria_lista() {
    Lista* self = malloc(sizeof(Lista));
    if (self == NULL) return NULL;
    self->inicio = NULL;
    return self;
}

No* cria_no(int valor, No* proximo) {
    No* self = malloc(sizeof(No));
    if (self == NULL) {
        fprintf(stderr, "Erro de alocação\n");
        exit(EXIT_FAILURE);
    }
    self->valor = valor;
    self->proximo = proximo;
    return self;
}

void inserir_inicio(Lista* lista, int valor) {
    No* novo_no = cria_no(valor, lista->inicio);
    lista->inicio = novo_no;
}

void inserir_final(Lista* lista, int valor) {
    No* novo_no = cria_no(valor, NULL);
    if (lista->inicio == NULL) {
        lista->inicio = novo_no;
    } else {
        No* iter = lista->inicio;
        while (iter->proximo != NULL) {
            iter = iter->proximo;
        }
        iter->proximo = novo_no;
    }
}

int tamanho(Lista* lista) {
    int count = 0;
    No* iter = lista->inicio;
    while (iter != NULL) {
        count++;
        iter = iter->proximo;
    }
    return count;
}

void imprime(Lista* lista) {
    No* iter = lista->inicio;
    while (iter != NULL) {
        printf("%d ", iter->valor);
        iter = iter->proximo;
    }
    printf("\n");
}

int elemento(Lista* lista, int pos) {
    No* iter = lista->inicio;
    int i = 0;
    while (iter != NULL && i < pos) {
        iter = iter->proximo;
        i++;
    }
    if (iter == NULL) {
        printf("Posição inválida!\n");
        return -1;
    }
    return iter->valor;
}

int main() {
    Lista* lista1 = cria_lista();

    inserir_final(lista1, 10);
    inserir_final(lista1, 20);
    inserir_final(lista1, 30);
    imprime(lista1);

    inserir_inicio(lista1, 5);
    imprime(lista1);

    printf("Tamanho: %d\n", tamanho(lista1));
    printf("Elemento na posição 2: %d\n", elemento(lista1, 2));

    return 0;
}

```

---

## 📚 Lista de Exercícios – Lista Encadeada Dinâmica

### Nível 1 – Operações básicas

1. **Função `vazia(Lista* l)`**
    - Retorna `1` se a lista estiver vazia, `0` caso contrário.
2. **Função `remover_inicio(Lista* l)`**
    - Remove o primeiro nó da lista.
    - Lembre-se de liberar a memória com `free()`.
3. **Função `remover_final(Lista* l)`**
    - Remove o último nó da lista.
    - Cuide dos casos com apenas um elemento.
4. **Função `buscar(Lista* l, int valor)`**
    - Retorna a posição do primeiro nó com o valor procurado, ou `-1` se não existir.

---

### Nível 2 – Inserções e remoções intermediárias

1. **Função `inserir_pos(Lista* l, int valor, int pos)`**
    - Insere um novo nó na posição indicada (0 = início).
    - Se `pos` for maior que o tamanho, insira no final.
2. **Função `remover_pos(Lista* l, int pos)`**
    - Remove o nó na posição indicada.
    - Cuide dos casos de lista vazia e posição inválida.
3. **Função `remover_valor(Lista* l, int valor)`**
    - Remove a **primeira ocorrência** de um valor na lista.
4. **Função `remover_todos(Lista* l, int valor)`**
    - Remove **todas as ocorrências** de um determinado valor.

---

### Nível 3 – Manipulações avançadas

1. **Função `inverter(Lista* l)`**
    - Inverta a ordem dos nós da lista (pode ser iterativa ou recursiva).
    - Exemplo: `10 → 20 → 30` → `30 → 20 → 10`.
2. **Função `copiar(Lista* origem)`**
    - Retorne uma nova lista com os mesmos valores da lista original.
3. **Função `concatenar(Lista* l1, Lista* l2)`**
    - Acrescente todos os nós de `l2` ao final de `l1`.
4. **Função `ordenar(Lista* l)`**
    - Ordene a lista em ordem crescente (use *bubble sort* ou *insertion sort* adaptado para listas encadeadas).

---

### Nível 4 – Desafios opcionais

1. **Função `maior_elemento(Lista* l)`**
    - Retorne o maior valor armazenado.
2. **Função `menor_elemento(Lista* l)`**
    - Retorne o menor valor armazenado.
3. **Função `iguais(Lista* l1, Lista* l2)`**
    - Retorna `1` se ambas têm o mesmo tamanho e os mesmos valores, senão `0`.
4. **Função `destruir_lista(Lista* l)`**
    - Libere a memória de **todos os nós** e também da estrutura `Lista`.
5. **Função `intercalar(Lista* l1, Lista* l2)`**
    - Crie uma nova lista alternando os elementos de `l1` e `l2` (um de cada, começando por `l1`; sobras vão para o final). Ex.: `[1,3,5]` e `[2,4]` → `[1,2,3,4,5]`.
    - Compare com o `intercalar` da lista estática, que combina listas ordenadas.

---

### Dica para os alunos

> Sempre desenhem o encadeamento dos nós no papel antes de codar — ajuda muito! Usem printf() e imprime(lista) após cada operação para conferir se a estrutura ainda está consistente. Cuidem para não perder referências aos nós (evita vazamento de memória).