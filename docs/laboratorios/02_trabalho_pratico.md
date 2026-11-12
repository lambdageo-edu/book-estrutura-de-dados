# Trabalho Prático: Algoritmos de Ordenação em C

**Formato:** grupos de até 3 alunos · **Linguagem:** C11 (`-std=c11`) · **Duração sugerida:** 4 semanas · **Prazo de entrega:** **[PREENCHER: data e hora limite]**

Este trabalho é a avaliação prática da disciplina e tem duas entregas indissociáveis: o **código** (repositório GitHub) e um **vídeo de defesa obrigatório**, no qual o grupo demonstra e explica o que implementou. O vídeo não é um extra — é a etapa final do trabalho e substitui a avaliação escrita da Terceira Unidade.

## Objetivos de Aprendizagem

1. Implementar **3 algoritmos básicos** e **3 algoritmos avançados** de ordenação em C.
2. Vivenciar o **controle de versão** com Git em equipe, com fluxo flexível e focado no histórico.
3. Aplicar noções básicas de **qualidade e medição**: testes, cobertura (`gcov`), análise estática (`cppcheck`) e profiling de desempenho (`gprof`).
4. Produzir **documentação técnica** e **análise empírica** dos resultados, confrontando as medições com a teoria de complexidade.
5. Desenvolver **transparência acadêmica** no uso de IA, validando e explicando todo o código entregue.
6. Defender oralmente as escolhas de implementação e os resultados obtidos.

## Organização do Grupo

Cada grupo define sua própria divisão de tarefas. Sugestão inicial (opcional):

- **Membro 1:** algoritmos básicos + testes iniciais
- **Membro 2:** algoritmos avançados + benchmark
- **Membro 3:** qualidade (`gcov`, `gprof`, `cppcheck`), `Makefile` e documentação

**Regra única:** todos devem conseguir compilar, executar e explicar qualquer parte do código — isso será cobrado diretamente no vídeo. O histórico Git deve refletir **contribuição ativa de todos os integrantes** (pelo menos alguns commits de cada um, com conteúdo relevante).

---

## Parte 1 — Implementação

### Algoritmos Obrigatórios

| Categoria | Algoritmos | Assinatura Padrão |
| --- | --- | --- |
| **Básicos** | Bubble Sort, Selection Sort, Insertion Sort | `void algoritmoSort(int v[], int n, Metricas *m);` |
| **Avançados** | Merge Sort, Quick Sort, Heap Sort | `void algoritmoSort(int v[], int n, Metricas *m);` |

Cada algoritmo deve estar em **arquivo `.c` separado**, com nomes de função `bubbleSort`, `selectionSort`, `insertionSort`, `mergeSort`, `quickSort` e `heapSort`.

### Contagem de Comparações e Trocas

Todos os algoritmos recebem um ponteiro para a estrutura abaixo, que o grupo coloca em `src/metricas.h`. O ponteiro `m` **nunca é `NULL`**: quem chama (testes, `main`, `bench`) sempre passa uma estrutura zerada.

```c
/* src/metricas.h */
#ifndef METRICAS_H
#define METRICAS_H

typedef struct {
    unsigned long comparacoes;  /* comparações entre dois valores do vetor */
    unsigned long trocas;       /* trocas de dois elementos, ou cópias/deslocamentos */
} Metricas;

#endif
```

- **Comparação:** apenas comparações entre valores do vetor (não conte `i < n` dos laços).
- **Trocas:** Bubble, Selection, Quick e Heap contam trocas de dois elementos; Insertion e Merge contam **movimentações** (deslocamentos e cópias de um elemento). Registre esse critério em `docs/analise.md`.

Exemplo de uso:

```c
Metricas m = {0, 0};
bubbleSort(v, n, &m);
printf("comparações=%lu trocas=%lu\n", m.comparacoes, m.trocas);
```

### Requisitos Específicos por Algoritmo

- **Merge Sort:** o vetor auxiliar deve ser alocado **dentro** da função (`malloc`), com verificação do retorno e `free` antes de retornar.
- **Quick Sort:** o pivô **não pode ser sempre o primeiro ou o último elemento**. Use o elemento do meio ou a mediana de três, e comente a escolha no código. Com pivô fixo e entrada já ordenada o algoritmo degenera para O(n²) e pode estourar a pilha nos tamanhos do benchmark.

### Ponto de Entrada (`src/main.c`)

O `main` deve aceitar argumentos de linha de comando, pois o profiling depende deles:

```text
./sort_main <n> [bubble|selection|insertion|merge|quick|heap]
```

- Gera um vetor aleatório de `n` elementos (semente fixa, `srand(42)`).
- Sem o segundo argumento, executa os seis algoritmos, cada um sobre uma **cópia** do mesmo vetor.
- Com o segundo argumento, executa apenas o algoritmo indicado.
- Imprime o tempo (usando `clock()` de `<time.h>`), as comparações e as trocas de cada execução.

### Fluxo Git/GitHub

Não há exigência de *branch protection*, revisão obrigatória ou bloqueio de `push` direto na `main`. O foco é **criar o hábito de versionar incrementalmente**.

Esperado no histórico:

- **Commits frequentes:** mínimo 1 a cada 2-3 dias de trabalho.
- **Mensagens descritivas**, seguindo o padrão `tipo: descrição` (ex.: `feat: adiciona heap sort`, `fix: corrige índice no partition do quicksort`, `docs: atualiza benchmark e uso de IA`). Evite mensagens como `update`, `fix`, `salva tudo`.
- **Participação visível:** `git log` deve mostrar commits de **todos** os integrantes (configure `git config user.name` e `user.email` em cada máquina).
- **Branches & PRs:** opcionais. Podem trabalhar direto na `main` ou criar branches por funcionalidade. Se usarem PRs, basta uma descrição breve.

!!! tip
    Rode `git pull` antes de começar e `git push` depois de testar uma nova funcionalidade. Isso evita conflitos e gera histórico natural.

### Compilação

- Compilação via `make` (modelo no [apêndice](#apendice-makefile-base)), com `-std=c11`.
- Zero erros de compilação. Warnings de `-Wall -Wextra` devem ser corrigidos (ou, se inevitáveis, justificados no `README`).

### Testes

- Arquivo `tests/test_basic.c` com **no mínimo 2 casos por algoritmo** (ex.: vetor aleatório, já ordenado, invertido), verificando se o vetor final está de fato ordenado.
- Cada teste deve imprimir `PASSOU` ou `FALHOU`, e o programa deve **terminar com código de saída diferente de zero** se algum teste falhar.
- Comando: `make test`.

### Benchmark e Análise Experimental

O vídeo cobra uma análise empírica, então ela precisa ser gerada de forma padronizada. Crie `src/bench.c` (com seu próprio `main`) que executa a matriz de experimentos abaixo e imprime **CSV** na saída padrão:

```text
algoritmo,tipo,n,tempo_ms,comparacoes,trocas
```

| Parâmetro | Especificação |
| --- | --- |
| **Tamanhos (`n`)** | 1.000 · 5.000 · 10.000 · 50.000 (todos os algoritmos). Os avançados também com 500.000. Não rode os básicos com 500.000. |
| **Tipos de entrada** | aleatório · crescente · decrescente |
| **Repetições** | 5 execuções por combinação; registre a **média** dos tempos |
| **Sementes** | `srand(42)` fixa, para reprodutibilidade |
| **Cronômetro** | `clock()` de `<time.h>`, medindo **apenas a ordenação** (não a geração nem a cópia do vetor) |
| **Compilação** | `-O2` (o benchmark mede desempenho real; só o `gprof` usa `-O0`) |

Comando: `make bench` (grava `docs/resultados.csv`). Em seguida, escreva `docs/analise.md` com:

1. Uma tabela-resumo (pode ser gerada do CSV) e, se desejarem, gráficos.
2. O critério adotado para contar "trocas" em cada algoritmo.
3. A comparação com a teoria: por que Bubble/Selection/Insertion crescem como O(n²) e Merge/Quick/Heap como O(n log n)? O que acontece com o Insertion Sort em entrada já ordenada? O que muda no Bubble Sort e no Quick Sort entre os três tipos de entrada?
4. Os resultados que **não** bateram com a teoria, com uma hipótese para a diferença.

### Análise Estática (`cppcheck`)

```bash
make cppcheck
```

O comando grava a saída bruta em `docs/cppcheck_bruto.txt`. Em `docs/cppcheck.md`, liste o que foi apontado, o que o grupo corrigiu e o que decidiu manter (com justificativa). O repositório entregue deve estar **sem erros** (`error`) apontados pelo `cppcheck`.

### Cobertura de Código (`gcov`)

O `gcov` mede quais linhas do programa foram de fato executadas pelos testes. O alvo `make coverage` do Makefile já faz todo o fluxo abaixo; os passos manuais servem para entender o que ele faz.

1. **Compile com suporte a cobertura:**

    ```bash
    gcc --coverage -g -O0 -c main.c
    gcc --coverage main.o -o main
    ```

    Compile e ligue em duas etapas: a partir do GCC 11, um único comando `gcc --coverage main.c -o main` nomeia os arquivos de dados como `main-main.gcda`, e o `gcov main.c` do passo 3 não os encontra. As flags equivalentes `-fprofile-arcs -ftest-coverage` fazem o GCC gerar os arquivos (`.gcno`) necessários para o `gcov`.

2. **Execute o programa ou a suíte de testes** (`./main` ou `make test`). Isso gera um arquivo `.gcda` com as estatísticas de execução.

3. **Gere o relatório:**

    ```bash
    gcov main.c
    ```

    O resultado é um arquivo `main.c.gcov` e um resumo no terminal, por exemplo, para um programa com 7 linhas executáveis:

    ```text
    File 'main.c'
    Lines executed:85.71% of 7
    Creating 'main.c.gcov'
    ```

4. **Interprete o resultado.** No arquivo `.gcov`, cada linha é prefixada pelo número de vezes que foi executada, por `-` se não é executável, ou por `#####` se é executável mas nunca foi executada:

    ```text
            -:    1:#include <stdio.h>
            1:    2:int main() {
            1:    3:    int x = 10;
            1:    4:    if (x > 5)
            1:    5:        printf("Maior\n");
            -:    6:    else
        #####:    7:        printf("Menor\n");
            1:    8:    return 0;
            1:    9:}
    ```

5. **Cobertura de branches (opcional).** Para ver quais desvios (`if`, `while`, ...) foram de fato tomados:

    ```bash
    gcov -b main.c
    ```

Para um relatório HTML mais legível, use [`gcovr`](https://github.com/gcovr/gcovr) ou `lcov`:

```bash
gcovr -r . --html --html-details -o coverage.html
```

!!! note "Critério de cobertura"
    A meta de **70%** refere-se à **cobertura de linhas** dos **seis arquivos de algoritmos** (`src/basicos/*.c` e `src/avancados/*.c`), medida com `make coverage` a partir dos testes. `main.c`, `bench.c` e os testes não entram na conta. Registre o percentual de cada arquivo em `docs/cobertura.md`.

### Profiling de Desempenho (`gprof`)

Enquanto o `gcov` responde "quais linhas foram executadas?", o `gprof` responde "onde o programa gastou mais tempo?".

1. **Compile com suporte a profiling** (flag `-pg`, sem otimização):

    ```bash
    gcc -pg -O0 -g bubble.c -o bubble
    ```

2. **Execute o programa** (`./bubble`). Isso gera o arquivo `gmon.out` com os dados coletados.

3. **Gere o relatório:**

    ```bash
    gprof bubble gmon.out > perfil.txt
    ```

    A saída inclui um **flat profile** (tempo gasto por função):

    ```text
    Flat profile:

    Each sample counts as 0.01 seconds.

     %   cumulative   self
    time   seconds   seconds   calls  ms/call  ms/call  name
    85.0      0.17     0.17        1   170.00   170.00  bubbleSort
    10.0      0.19     0.02        1    20.00    20.00  imprimirVetor
     5.0      0.20     0.01        1    10.00    10.00  main
    ```

    e um **call graph**, mostrando quem chamou quem, quantas vezes e quanto tempo cada chamada consumiu — útil para identificar gargalos que não aparecem no flat profile.

No projeto, use o alvo do Makefile, que já compila com `-O0` e executa o `main` com os parâmetros desejados:

```bash
make profile                          # n = 50000, todos os algoritmos
make profile N=2000000 ALG=quick      # só o Quick Sort, com n grande
```

!!! warning
    - **Não faça profiling com `-O2`.** O otimizador distorce as medições — sempre `-O0` (o alvo `make profile` já cuida disso).
    - O `gprof` amostra a cada **10 ms**. Se o programa roda mais rápido que isso, o *flat profile* sai vazio ou com zeros. Por isso o padrão é `n = 50000`; para analisar os avançados individualmente, aumente `N` e escolha o algoritmo com `ALG`.

Outras ferramentas relacionadas, caso queiram ir além: `valgrind --tool=massif` (uso de memória), `valgrind --tool=callgrind` (perfil detalhado de chamadas) e `perf` (profiling de sistema no Linux).

### Política de Uso de IA & Transparência

**Permitido e incentivado:** gerar trechos para estudo, pedir explicação de erros, revisar formatação, sugerir nomes de variáveis, usar como tutor.

**Não permitido:** entregar código completo sem entender, testar ou adaptar; ocultar prompts ou ferramentas utilizadas.

Crie `docs/uso-ia.md` seguindo este modelo:

```markdown
# Uso de Ferramentas de IA no Projeto

## Ferramentas Utilizadas
- [ ] ChatGPT
- [ ] GitHub Copilot
- [ ] Claude
- [ ] Gemini
- [ ] Outra: _______________

## Como Usamos (exemplos reais)
| Prompt / Solicitação | O que a IA gerou | O que adaptamos/validamos |
|----------------------|------------------|---------------------------|
| "Explique como o heapify restaura o max-heap" | Texto + pseudocódigo | Reescrevemos em C, ajustamos índices 2i+1 e adicionamos testes |
| "Corrija warning unused variable em merge.c" | Sugeriu remover var | Mantivemos, renomeamos e usamos no laço |

## Validação & Domínio
- [ ] Compilamos e rodamos todo código gerado ou sugerido
- [ ] Todos conseguem explicar o funcionamento de cada algoritmo em 2 minutos
- [ ] Não submetemos blocos inteiros sem revisão linha a linha

## Reflexão Final (3-5 linhas)
(Descreva onde a IA ajudou, onde atrapalhou, e o que o grupo aprendeu ao questionar as respostas.)
```

!!! note "Avaliação"
    O uso de IA **não penaliza**. Falta de transparência, código não testado ou grupo incapaz de explicar a entrega **penaliza**.

### Estrutura Obrigatória do Repositório

```text
ordenacao-<grupo>/
├── README.md                  # Instruções de build, membros, links
├── Makefile                   # Modelo fornecido
├── .gitignore
├── src/
│   ├── basicos/               # bubble.c, selection.c, insertion.c
│   ├── avancados/             # merge.c, quick.c, heap.c
│   ├── metricas.h             # struct Metricas
│   ├── main.c                 # Ponto de entrada (aceita n e algoritmo)
│   └── bench.c                # Benchmark que gera o CSV
├── tests/
│   └── test_basic.c           # Testes automatizados
└── docs/
    ├── resultados.csv         # Saída do make bench
    ├── analise.md             # Análise experimental x teoria
    ├── cobertura.md           # Relatório gcov + análise
    ├── gcov/                  # Saída do make coverage (resumo.txt e .gcov)
    ├── perfil_bruto.txt       # Saída do gprof (gerada pelo make profile)
    ├── perfil.md              # Interpretação do profiling
    ├── cppcheck_bruto.txt     # Saída do make cppcheck
    ├── cppcheck.md            # O que foi apontado, corrigido e mantido
    └── uso-ia.md              # Transparência de prompts e validação
```

Sugestão de `.gitignore` (binários e arquivos intermediários ficam de fora; os relatórios em `docs/` **devem** ser versionados):

```text
*.o
*.d
*.gcda
*.gcno
/*.gcov
gmon.out
/sort_main
/sort_profile
/test_basic
/bench_sort
```

### Cronograma Sugerido

| Semana | Foco | Meta Git |
| --- | --- | --- |
| **1** | Setup, `Makefile`, `metricas.h`, 2 básicos, 1 teste cada | Commits iniciais + estrutura |
| **2** | 3º básico, 2 avançados, testes rodando, esqueleto do `bench.c` | Histórico crescente (≥ 5 commits) |
| **3** | Completar avançados, rodar `make coverage`, `make cppcheck` e `make profile`; `make bench` completo | Commits de refino + relatórios brutos em `docs/` |
| **4** | Escrever `analise.md`, `cobertura.md`, `perfil.md`, `cppcheck.md`; gravar o vídeo; tag `v1.0` | Entrega + link no SIGAA |

---

## Parte 2 — Defesa em Vídeo (obrigatória)

O vídeo é a defesa oral do trabalho: demonstra que o grupo domina o código desenvolvido e as ferramentas utilizadas, e substitui a avaliação escrita da Terceira Unidade (**4,0 pontos**).

- **Formato:** gravação de **8 a 15 minutos**, publicada no YouTube (público ou não listado).
- **Envio:** o link do vídeo é enviado no SIGAA junto com a entrega do repositório.
- Não serão aceitas apresentações compostas apenas de slides — é obrigatório alternar para a tela do terminal e do editor, mostrando código e execução real.
- Os grupos devem ser exatamente os mesmos do trabalho prático, e todos os integrantes devem falar e demonstrar autoria do código.

### Estrutura Obrigatória do Vídeo

| # | Seção | Conteúdo | Tempo sugerido |
| --- | --- | --- | --- |
| 1 | **Apresentação da equipe** | Integrantes, turma e objetivos gerais | ~1 min |
| 2 | **Implementação dos algoritmos** | Organização do código-fonte (arquivos e estrutura) e explicação das partes relevantes | ~3 min |
| 3 | **Demonstração da execução** | Compilar e executar o programa ao vivo, mostrando entrada de dados, saída dos testes e coleta das métricas | ~2 min |
| 4 | **Análise experimental** | Discutir a tabela de tempos de execução, comparações e trocas, confrontando o desempenho medido com a teoria de complexidade | ~3 min |
| 5 | **Uso de ferramentas** | Demonstrar o `gcov` (cobertura obtida), o `gprof` (funções mais pesadas) e o `cppcheck` (correções feitas) | ~3 min |
| 6 | **Conclusões** | Qual algoritmo performou melhor, se as medições batem com a teoria e quais dificuldades a equipe encontrou | ~1 min |

---

## Avaliação

### Rubrica do Código (100 pts)

| Critério | O que será observado | Peso |
| --- | --- | --- |
| **Funcionalidade** | 6 algoritmos compilam, executam e ordenam corretamente, com contagem de comparações e trocas | 25 |
| **Histórico Git** | Commits regulares, mensagens claras, participação de todos, evolução visível | 20 |
| **Qualidade & Testes** | Testes passam, `gcov` ≥ 70% de linhas nos 6 arquivos de algoritmos, `cppcheck` sem erros | 20 |
| **Documentação & Análise** | `analise.md`, `cobertura.md`, `perfil.md`, `cppcheck.md`, `resultados.csv` e `README` funcional | 20 |
| **Colaboração & IA** | Divisão equilibrada, `docs/uso-ia.md` preenchido, código explicável | 15 |

!!! warning
    Repositórios com 1-2 commits no dia da entrega, mensagens vazias ou código que o grupo não consegue modificar serão penalizados na categoria Git/Colaboração.

### Rubrica do Vídeo (4,0 pts — substitui a avaliação escrita da Unidade 3)

| Critério | O que será observado | Pontos |
| --- | --- | --- |
| **Organização e Clareza** | Apresentação didática, áudio nítido, divisão do tempo e boa estrutura | 1.0 |
| **Demonstração do Código** | Explicação concisa da implementação e compilação/execução correta | 1.0 |
| **Análise de Resultados** | Discussão fundamentada nas medições empíricas comparando os algoritmos | 1.0 |
| **Domínio e Participação** | Todos os integrantes falam e demonstram autoria do código | 1.0 |

### Composição da Nota

!!! note "Nota final"
    - **Vídeo:** 4,0 pontos, na Terceira Unidade.
    - **Código:** a pontuação de 0 a 100 é convertida para **[PREENCHER: escala/peso conforme o plano de ensino]**.
    - **Individualização:** a nota é do grupo, mas pode ser **ajustada individualmente** quando o histórico Git, a participação no vídeo ou a arguição indicarem contribuição desigual.
    - **Atraso:** **[PREENCHER: política, ex.: −10% da nota por dia de atraso]**.

---

## Entrega do Projeto

1. **Repositório GitHub** atualizado (público, ou privado com acesso liberado ao professor: usuário **[PREENCHER: usuário GitHub do professor]**).
2. **Tag de versão** da entrega final:

    ```bash
    git tag -a v1.0 -m "Entrega final do trabalho de ordenação"
    git push origin main --tags
    ```

3. **Envio via SIGAA**, em um único comentário por grupo, contendo:
      - nomes completos dos integrantes;
      - link do repositório GitHub;
      - link do vídeo de apresentação no YouTube.

---

## Apêndice: Makefile Base {#apendice-makefile-base}

Copie este arquivo para a raiz do repositório. Ele inclui alvos para compilação, testes, benchmark, `gcov`, `gprof` e `cppcheck`.

```makefile
CC = gcc
WARN = -Wall -Wextra -std=c11
CFLAGS = $(WARN) -O2
DEPFLAGS = -MMD -MP
GCOV_FLAGS = --coverage -g -O0
PROFILE_FLAGS = -pg -g -O0

# Parâmetros do profiling: make profile N=50000 ALG=bubble
N ?= 50000
ALG ?=

ALG_SRCS = src/basicos/bubble.c src/basicos/selection.c src/basicos/insertion.c \
           src/avancados/merge.c src/avancados/quick.c src/avancados/heap.c
SRCS = src/main.c $(ALG_SRCS)
TEST_SRCS = tests/test_basic.c $(ALG_SRCS)
BENCH_SRCS = src/bench.c $(ALG_SRCS)

OBJS = $(SRCS:.c=.o)
TEST_OBJS = $(TEST_SRCS:.c=.o)
BENCH_OBJS = $(BENCH_SRCS:.c=.o)
ALL_OBJS = $(sort $(OBJS) $(TEST_OBJS) $(BENCH_OBJS))

TARGET = sort_main
TEST_TARGET = test_basic
BENCH_TARGET = bench_sort

all: $(TARGET)

$(TARGET): $(OBJS)
	$(CC) $(CFLAGS) -o $@ $^

$(TEST_TARGET): $(TEST_OBJS)
	$(CC) $(CFLAGS) -o $@ $^

$(BENCH_TARGET): $(BENCH_OBJS)
	$(CC) $(CFLAGS) -o $@ $^

%.o: %.c
	$(CC) $(CFLAGS) $(DEPFLAGS) -c $< -o $@

test: $(TEST_TARGET)
	./$(TEST_TARGET)

# Benchmark: gera a tabela de tempos, comparações e trocas em CSV
bench: $(BENCH_TARGET)
	./$(BENCH_TARGET) > docs/resultados.csv
	@echo "Resultados gravados em docs/resultados.csv"

# Cobertura: recompila tudo sem otimização e com instrumentação (CFLAGS substituído).
# Ao final, remove os objetos instrumentados para não contaminar o próximo 'make'.
coverage: CFLAGS = $(WARN) $(GCOV_FLAGS)
coverage: clean test
	@mkdir -p docs/gcov
	( gcov -o src/basicos src/basicos/*.c; gcov -o src/avancados src/avancados/*.c ) | tee docs/gcov/resumo.txt
	mv *.gcov docs/gcov/
	rm -f $(ALL_OBJS) $(ALL_OBJS:.o=.d) $(TEST_TARGET)
	find . \( -name '*.gcda' -o -name '*.gcno' \) -delete
	@echo "Relatórios .gcov e resumo em docs/gcov/"

# Profiling: compila direto das fontes, só com -O0 (sem o -O2 de CFLAGS)
profile:
	$(CC) $(WARN) $(PROFILE_FLAGS) -o sort_profile $(SRCS)
	./sort_profile $(N) $(ALG)
	gprof sort_profile gmon.out > docs/perfil_bruto.txt
	@echo "Profiling gerado em docs/perfil_bruto.txt (veja a seção 'Flat profile')"

# Análise estática
cppcheck:
	cppcheck --enable=warning,style,performance,portability --std=c11 \
	    --inline-suppr --suppress=missingIncludeSystem src tests 2> docs/cppcheck_bruto.txt
	@echo "Resultado em docs/cppcheck_bruto.txt"

clean:
	rm -f $(TARGET) $(TEST_TARGET) $(BENCH_TARGET) sort_profile $(ALL_OBJS) $(ALL_OBJS:.o=.d) *.gcov gmon.out
	find . \( -name '*.gcda' -o -name '*.gcno' \) -delete

.PHONY: all test bench coverage profile cppcheck clean

-include $(ALL_OBJS:.o=.d)
```

!!! warning "Makefile exige TAB"
    As linhas de comando das regras precisam começar com o caractere **TAB**, não com espaços. Se copiar o arquivo e aparecer `missing separator`, é isso.

## Dicas Rápidas

| Ferramenta | Dica |
| --- | --- |
| `gcov` | Se a cobertura ficar abaixo de 70%, adicione testes com `n=0`, `n=1`, vetores com duplicatas e invertidos. |
| `gprof` | O amostrador tem resolução de 10 ms: com `n` pequeno o relatório sai vazio. Aumente `N` no `make profile`. |
| `cppcheck` | Instale com `sudo apt install cppcheck`. Avisos de estilo podem ser mantidos se justificados em `docs/cppcheck.md`. |
| `make` | O Makefile acompanha as dependências dos `.h` (`-MMD`), mas se algo estranho acontecer após mudar `metricas.h`, rode `make clean`. |
| `git` | Uma mensagem como `fix: ajusta condição de parada no quicksort` vale mais que 10 commits `update`. |
| IA | Peça explicação, não código pronto. Valide linha a linha — a IA é copiloto, não piloto. |
| Entrega | Teste `make clean && make all && make test && make coverage && make bench && make profile && make cppcheck` antes de dar `push` (o `make bench` pode levar alguns minutos). |
