# Caderno de Exercícios

## Estruturas de Dados Lineares e Gestão de Memória em C

---

## 1. 🎯 Introdução e Contextualização

O domínio de **ponteiros**, **alocação dinâmica** e da correta arquitetura de **Tipos Abstratos de Dados (TADs)** é um divisor de águas entre um programador iniciante e um engenheiro de computação.

Na prática:

- Eficiência não é acidental → é resultado de decisões conscientes
- Organização de dados impacta diretamente o desempenho
- Gestão de memória é crítica para estabilidade e segurança

Um código que funciona em pequena escala pode **falhar sob carga** se:

- houver uso incorreto do heap
- faltar encapsulamento
- não existir controle do ciclo de vida da memória

O objetivo deste caderno é fazer você pensar como **arquiteto de software**, não apenas como programador.

---

## 2. 📏 Critérios de Avaliação (Rigor Arquitetural)

Para que uma solução seja considerada correta:

### Segurança de Alocação

- Sempre verificar o retorno de `malloc`, `calloc` ou `realloc`
- `NULL` não tratado = **erro crítico**

### Gestão de Memória (Accountability)

- Todo `malloc` deve ter um `free`
- Vazamento de memória = falha grave

### Encapsulamento

- `main.c` deve usar apenas funções do `.h`
- Acesso direto à struct = **violação de projeto**

### Sem Variáveis Globais

- Comunicação entre módulos → via parâmetros e retornos

---

## 3. 🧠 Parte I — Fundamentos Teóricos (11 Questões)

### Conceito central

Separar:

- **O que fazer (interface)** → `.h`
- **Como fazer (implementação)** → `.c`

---

### Questões

#### 1. Passagem por valor vs referência

Explique por que funções como `empilha(int x, Pilha *p)` usam ponteiros.

O que aconteceria se a pilha fosse passada por valor?


---

#### 2. Stack vs Heap

Diferencie:

- Stack (automática)
- Heap (dinâmica)

Quem gerencia cada uma?

Por que retornar variável local é perigoso?


---

#### 3. Tipo Abstrato de Dado (TAD)

Defina formalmente um TAD.

Qual o papel do `.h`?


---

#### 4. LIFO e aplicações reais

Explique LIFO e aplique em:

- Navegadores (histórico)
- Editores (undo/redo)
- Chamadas de função (recursão)

---

#### 5. Ponteiros e vetores

Explique:


```
v[i] == *(v + i)
```

Como o compilador calcula o endereço?

---

#### 6. Complexidade (Pior Caso)

Qual o custo da busca em lista não ordenada?

Por que usamos pior caso?


---

#### 7. Encapsulamento na prática

Por que `main.c` não deve acessar `l->ultimo`?


---

#### 8. Alocação dinâmica vs estática

Compare:

- Flexibilidade
- Fragmentação

---

#### 9. Comparação de Pilhas

| Critério | Estática (Vetor) | Dinâmica (Encadeada) |
| --- | --- | --- |
| Overhead | Baixo | Alto |
| Cache | Alto | Baixo |
| Crescimento | Fixo | Dinâmico |
| Limite | Capacidade fixa (definida na criação) | RAM |

Explique cada linha da tabela e diga em que situação você escolheria cada implementação.

---

#### 10. Operador `->`

Explique por que:


```
*p.membro   // errado
(*p).membro // correto
p->membro   // correto
```

---

#### 11. Rastreamento de memória

Dado:


```
topo = -1
```

Simule:

```
push(10)
push(20)
pop()
push(30)
push(40)
```

Desenhe a tabela de memória

---

## 4. 💻 Parte II — Implementação Prática (15 Questões)

Aqui começa o nível **engenharia real**: ponteiros mal usados = bugs graves.

---

### Exercícios

#### 1. Struct + Ponteiro
- Crie `struct Ponto`
- Função que altera via ponteiro

---

#### 2. Alocação segura
- Criar vetor dinâmico
- Verificar erro
- Inicializar com zero

---

#### 3. Refatoração
- Remover globais:
    
    ```
    int pilha[MAX];
    int topo;
    ```
    
- Criar TAD

---

#### 4. Push com verificação
- Implementar `empilha`
- Tratar overflow

---

#### 5. Pop em lista encadeada
- Remover nó
- Liberar memória

---

#### 6. Inicialização segura
- `Lista* criaLista()`
- Retornar `NULL` se falhar

---

#### 7. Busca sequencial
- Implementar
- Justificar O(n)

---

#### 8. Inserção com deslocamento
- Lista estática
- Validar posição

---

#### 9. Liberação de lista
- Liberar todos os nós
- Evitar perda de referência

---

#### 10. realloc seguro
- Dobrar tamanho
- Usar ponteiro temporário

---

#### 11. Busca por posição
Implemente `buscaPos` nas duas versões e justifique as complexidades:

- Vetor → O(1)
- Lista → O(n)

---

#### 12. Inversão com pilha
- Usar estrutura auxiliar

---

#### 13. Lista circular
- Último aponta para primeiro

O `while (atual != NULL)` funciona?


---

#### 14. Reuso de funções (DRY)
- `removeValor` usando outras funções

Qual a complexidade total?


---

#### 15. Bug de precedência (prática)

Corrija o código abaixo e teste-o compilando:


```
*p.membro = 10;
```

Explique o erro

---

## 5. 📊 Análise de Complexidade

### Comparativo

| Operação | Vetor | Lista Encadeada |
| --- | --- | --- |
| Acesso | O(1) | O(n) |
| Inserção início | O(n) | O(1) |
| Inserção fim | O(1) | O(n)* |
| Localidade | Alta | Baixa |

\* Pode ser O(1) com ponteiro para o último.

---

## 6. 🧪 Questão Bônus (Projeto)

Desenvolva uma pilha com:

- push
- pop
- **findMax() → O(1)**

Dica:

Use uma **pilha auxiliar de máximos**

---

## 7. ✅ Checklist Final

Antes de entregar:

- [ ]  Todo `malloc` tem `free`
- [ ]  Verificou `NULL`
- [ ]  Estrutura validada antes de uso
- [ ]  Sem variáveis globais
- [ ]  Encapsulamento respeitado

---

## ⚠️ Regra de Ouro

> Variáveis globais são proibidas nas soluções deste caderno.
>
> Controle de memória e escopo é responsabilidade do programador.

---