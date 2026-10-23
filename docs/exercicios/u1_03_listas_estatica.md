# Lista Estática

Usando como base essa implementação, crie as operações propostas a seguir.

!!! note "Convenção desta lista"
    Aqui `ultimo` guarda a **quantidade** de elementos (0 = lista vazia), e não o índice do último elemento como no capítulo de Listas Estáticas (onde -1 = vazia). As duas convenções são comuns; observe como cada função muda conforme a escolha.

```c
#include <stdio.h>
#include <stdlib.h>

#define MAX 100

typedef struct lista {
   int itens[MAX];
   int ultimo;
} Lista;

Lista* cria_lista() {
    Lista* self = malloc(sizeof(Lista));
    if (self == NULL) return NULL;
    self->ultimo = 0;
    return self;
}

int tamanho(Lista* l) {
    return l->ultimo;
}

int vazia(Lista* l) {
    return l->ultimo == 0;
}

void inserir_final(Lista* l, int item) {
    if (l->ultimo < MAX) {
        l->itens[l->ultimo++] = item;
    }
}

void inserir(Lista *l, int pos, int item) {
    if (l->ultimo >= MAX || pos < 0) {
        return;                       /* lista cheia ou posição inválida */
    }
    if (pos > l->ultimo) {
        inserir_final(l, item);
        return;
    }
    for (int i = l->ultimo; i > pos; i--) {
        l->itens[i] = l->itens[i - 1];
    }
    l->itens[pos] = item;
    l->ultimo++;
}

void remove_ultimo(Lista* l) {
    if (!vazia(l)) l->ultimo--;
}

void remover(Lista* l, int pos) {
    if (pos < l->ultimo) {
        for (int i = pos; i < l->ultimo - 1; i++) {
            l->itens[i] = l->itens[i + 1];
        }
        l->ultimo--;
    }
}

void imprime(Lista *l) {
    for (int i = 0; i < l->ultimo; i++) {
        printf("%d ", l->itens[i]);
    }
    printf("\n");
}

int elemento(Lista* l, int pos) {
    if (pos < 0 || pos >= l->ultimo) return -1;   /* posição inválida */
    return l->itens[pos];
}

int main() {
    Lista* lista1 = cria_lista();
    inserir_final(lista1, 10);
    inserir_final(lista1, 20);
    inserir_final(lista1, 30);
    imprime(lista1);
    inserir(lista1, 1, 40);
    imprime(lista1);
    remove_ultimo(lista1);
    imprime(lista1);
    inserir_final(lista1, 90);
    imprime(lista1);
    remover(lista1, 2);
    imprime(lista1);

    return 0;
}

```

---

## 📚 Lista de Exercícios – Lista Estática

### Nível 1 – Operações básicas

1. **Função `buscar(Lista* l, int valor)`**
    - Percorra a lista e retorne o índice do valor procurado.
    - Se não existir, retorne `-1`.
    - Exemplo de uso:
        
        ```c
        // com a lista [10, 20, 30]:
        int pos = buscar(lista1, 20); // deve retornar 1
        
        ```
        
2. **Função `maior_elemento(Lista* l)`**
    - Retorne o maior valor armazenado na lista.
    - Se a lista estiver vazia, retorne `-1`.
3. **Função `menor_elemento(Lista* l)`**
    - Retorne o menor valor armazenado na lista.
    - Semelhante ao exercício anterior.
4. **Função `soma_elementos(Lista* l)`**
    - Calcule e retorne a soma de todos os elementos da lista.

---

### Nível 2 – Manipulação intermediária

1. **Função `inverter(Lista* l)`**
    - Inverta a ordem dos elementos *dentro do próprio vetor*.
    - Exemplo: `[10, 20, 30] → [30, 20, 10]`.
2. **Função `remover_valor(Lista* l, int valor)`**
    - Remova a **primeira ocorrência** de `valor` da lista.
    - Utilize a função `buscar` para achar o índice.
3. **Função `remover_todos(Lista* l, int valor)`**
    - Remova **todas as ocorrências** de um valor na lista.
    - Exemplo: `[1,2,2,3] → [1,3]`.
4. **Função `inserir_ordenado(Lista* l, int valor)`**
    - Insira o elemento mantendo a lista em **ordem crescente**.
    - Exemplo: `[10,20,40]` + `30` → `[10,20,30,40]`.

---

### Nível 3 – Operações avançadas

1. **Função `concatenar(Lista* l1, Lista* l2)`**
    - Insira todos os elementos de `l2` no final de `l1`.
2. **Função `iguais(Lista* l1, Lista* l2)`**
    - Verifique se duas listas possuem o mesmo tamanho e os mesmos valores.
3. **Função `copiar(Lista* origem, Lista* destino)`**
    - Copie todos os elementos de `origem` para `destino`.
4. **Função `ordenar(Lista* l)`**
    - Implemente um algoritmo simples de ordenação (*Bubble Sort* ou *Selection Sort*).

---

### Nível 4 – Desafios opcionais

1. **Função `intercalar(Lista* l1, Lista* l2, Lista* resultado)`**
    - Supondo `l1` e `l2` ordenadas, combine seus elementos em `resultado`, também ordenada.
    - Exemplo:
        
        ```
        l1 = [1,3,5]
        l2 = [2,4,6]
        resultado = [1,2,3,4,5,6]
        
        ```
        
2. **Função `remover_intervalo(Lista* l, int inicio, int fim)`**
    - Remova todos os elementos entre os índices `inicio` e `fim` (inclusive).
3. **Função `rotacionar(Lista* l, int k)`**
    - Desloque todos os elementos `k` posições à direita.
    - Exemplo: `[1,2,3,4]` e `k=1` → `[4,1,2,3]`.

---

### Dica para os alunos

> Ao finalizar cada função, testem com printf() e imprime(lista) para confirmar que o comportamento está correto.
>
> É interessante que façam também testes com listas vazias, listas cheias e posições inválidas.