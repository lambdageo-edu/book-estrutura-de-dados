# Trabalho Prático: Algoritmos de Ordenação em C

**Formato:** grupos de até 3 alunos · **Linguagem:** C (padrão C99 ou C11) · **Duração sugerida:** 4 semanas

Este trabalho é a avaliação prática da disciplina e tem duas entregas indissociáveis: o **código** (repositório GitHub) e um **vídeo de defesa obrigatório**, no qual o grupo demonstra e explica o que implementou. O vídeo não é um extra — é a etapa final do trabalho e substitui a avaliação escrita da Terceira Unidade.

## Objetivos de Aprendizagem

1. Implementar **3 algoritmos básicos** e **3 algoritmos avançados** de ordenação em C.
2. Vivenciar o **controle de versão** com Git em equipe, com fluxo flexível e focado no histórico.
3. Aplicar noções básicas de **qualidade e medição**: testes, cobertura (`gcov`) e profiling de desempenho (`gprof`).
4. Produzir **documentação técnica** e **análise empírica** dos resultados.
5. Desenvolver **transparência acadêmica** no uso de IA, validando e explicando todo o código entregue.
6. Defender oralmente as escolhas de implementação e os resultados obtidos.

## Organização do Grupo

Cada grupo define sua própria divisão de tarefas. Sugestão inicial (opcional):

- **Membro 1:** algoritmos básicos + testes iniciais
- **Membro 2:** algoritmos avançados + benchmarks
- **Membro 3:** qualidade (`gcov`, `gprof`, `cppcheck`), `Makefile` e documentação

**Regra única:** todos devem conseguir compilar, executar e explicar qualquer parte do código — isso será cobrado diretamente no vídeo. O histórico Git deve refletir contribuição ativa de pelo menos 2 integrantes.

---

## Parte 1 — Implementação

### Algoritmos Obrigatórios

| Categoria | Algoritmos | Assinatura Padrão |
| --- | --- | --- |
| **Básicos** | Bubble Sort, Selection Sort, Insertion Sort | `void algoritmoSort(int v[], int n);` |
| **Avançados** | Merge Sort, Quick Sort, Heap Sort | `void algoritmoSort(int v[], int n);` |

Cada algoritmo deve estar em **arquivo `.c` separado**. O `main.c` pode conter chamadas, menu ou execução direta dos testes.

### Fluxo Git/GitHub

Não há exigência de *branch protection*, revisão obrigatória ou bloqueio de `push` direto na `main`. O foco é **criar o hábito de versionar incrementalmente**.

Esperado no histórico:

- **Commits frequentes:** mínimo 1 a cada 2-3 dias de trabalho.
- **Mensagens descritivas**, seguindo o padrão `tipo: descrição` (ex.: `feat: adiciona heap sort`, `fix: corrige índice no partition do quicksort`, `docs: atualiza benchmark e uso de IA`). Evite mensagens como `update`, `fix`, `salva tudo`.
- **Participação visível:** `git log` deve mostrar commits de pelo menos 2 emails/nomes diferentes.
- **Branches & PRs:** opcionais. Podem trabalhar direto na `main` ou criar branches por funcionalidade. Se usarem PRs, basta uma descrição breve.

!!! tip
    Rode `git pull` antes de começar e `git push` depois de testar uma nova funcionalidade. Isso evita conflitos e gera histórico natural.

### Compilação

- Compilação via `make` (modelo no [apêndice](#apendice-makefile-base)).
- Zero erros de compilação. Warnings críticos devem ser corrigidos.

### Testes

- Arquivo `tests/test_basic.c` com **no mínimo 2 casos por algoritmo** (ex.: vetor aleatório, já ordenado, invertido).
- Instrumente os algoritmos para contar **comparações e trocas** (por exemplo, com contadores passados por ponteiro): esses números entram na análise experimental do vídeo.
- Cada teste deve imprimir `PASSOU` ou `FALHOU`.
- Comando: `make test`.

### Cobertura de Código (`gcov`)

O `gcov` mede quais linhas do programa foram de fato executadas pelos testes.

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

    O resultado é um arquivo `main.c.gcov` e um resumo no terminal, por exemplo:

    ```text
    File 'main.c'
    Lines executed:85.71% of 14
    Creating 'main.c.gcov'
    ```

4. **Interprete o resultado.** No arquivo `.gcov`, cada linha é prefixada pelo número de vezes que foi executada, ou por `#####` se nunca foi executada:

    ```c
            1:int main() {
            1:    int x = 10;
            1:    if (x > 5)
            1:        printf("Maior\n");
        #####:    else
        #####:        printf("Menor\n");
            1:}
    ```

5. **Cobertura de branches (opcional).** Para ver quais desvios (`if`, `while`, ...) foram de fato tomados:

    ```bash
    gcov -b main.c
    ```

Para um relatório HTML mais legível, use [`gcovr`](https://github.com/gcovr/gcovr) ou `lcov`:

```bash
gcovr -r . --html --html-details -o coverage.html
```

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

!!! warning
    Não faça profiling com `-O2`. O otimizador distorce as medições — sempre use `-O0` (o alvo `make profile` do Makefile do apêndice já cuida disso).

Outras ferramentas relacionadas, caso queiram ir além: `valgrind --tool=massif` (uso de memória), `valgrind --tool=callgrind` (perfil detalhado de chamadas) e `perf` (profiling de sistema no Linux).

### Política de Uso de IA & Transparência

**Permitido e incentivado:** gerar trechos para estudo, pedir explicação de erros, revisar formatação, sugerir nomes de variáveis, usar como tutor.

**Não permitido:** entregar código completo sem entender, testar ou adaptar; ocultar prompts ou ferramentas utilizadas.

Crie `docs/uso-ia.md` seguindo este modelo:

```markdown
# Uso de Ferramentas de IA no Projeto

## Ferramentas Utilizadas
- [ ] ChatGPT / GPT-4o
- [ ] GitHub Copilot
- [ ] Claude / Gemini
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
ordenacao-trio/
├── README.md                  # Instruções de build, membros, links
├── Makefile                   # Modelo fornecido
├── src/
│   ├── basicos/                # bubble.c, selection.c, insertion.c
│   ├── avancados/               # merge.c, quick.c, heap.c
│   └── main.c                  # Ponto de entrada
├── tests/
│   └── test_basic.c           # Testes manuais
├── docs/
│   ├── cobertura.md            # Relatório gcov + análise
│   ├── perfil_bruto.txt        # Saída do gprof (gerada automaticamente)
│   ├── perfil.md                # Interpretação do profiling
│   └── uso-ia.md               # Transparência de prompts e validação
└── .gitignore
```

### Cronograma Sugerido

| Semana | Foco | Meta Git |
| --- | --- | --- |
| **1** | Setup, `Makefile`, 2 básicos, 1 teste cada | Commits iniciais + estrutura |
| **2** | Completar básicos, 2 avançados, testes rodando | Histórico crescente (≥ 5 commits) |
| **3** | Completar avançados, `gcov`, `gprof`, `cppcheck` | Commits de refino + `docs/` |
| **4** | Ajustes finais, gravação do vídeo, tag `v1.0` | Entrega + link no SIGAA |

---

## Parte 2 — Defesa em Vídeo (obrigatória)

O vídeo é a defesa oral do trabalho: demonstra que o grupo domina o código desenvolvido e as ferramentas utilizadas, e substitui a avaliação escrita da Terceira Unidade (**4,0 pontos**).

- **Formato:** gravação de **8 a 15 minutos**, publicada no YouTube (público ou não listado).
- **Envio:** o link do vídeo é enviado no SIGAA junto com a entrega do repositório.
- Não serão aceitas apresentações compostas apenas de slides — é obrigatório alternar para a tela do terminal e do editor, mostrando código e execução real.
- Os grupos devem ser exatamente os mesmos do trabalho prático, e todos os integrantes devem falar e demonstrar autoria do código.

### Estrutura Obrigatória do Vídeo

1. **Apresentação da equipe:** integrantes, turma e objetivos gerais.
2. **Implementação dos algoritmos:** organização do código-fonte (arquivos e estrutura) e explicação das partes relevantes.
3. **Demonstração da execução:** compilar e executar o programa ao vivo, mostrando entrada de dados, saída dos testes e coleta das métricas.
4. **Análise experimental:** discutir a tabela de tempos de execução, comparações e trocas, confrontando o desempenho medido com a teoria de complexidade.
5. **Uso de ferramentas:** demonstrar o funcionamento do `gcov` (cobertura obtida), `gprof` (funções mais pesadas) e `cppcheck` (correções feitas).
6. **Conclusões:** qual algoritmo performou melhor, se as medições batem com a teoria estudada e quais dificuldades a equipe encontrou.

---

## Avaliação

### Rubrica do Código (100 pts)

| Critério | O que será observado | Peso |
| --- | --- | --- |
| **Funcionalidade** | 6 algoritmos compilam, executam e ordenam corretamente | 25 |
| **Histórico Git** | Commits regulares, mensagens claras, participação do grupo, evolução visível | 20 |
| **Qualidade & Testes** | Testes passam, `gcov` ≥ 70% nas funções, `cppcheck` limpo de erros críticos | 20 |
| **Documentação & Análise** | `docs/cobertura.md`, `docs/perfil.md`, `README` funcional | 20 |
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

---

## Entrega do Projeto

1. **Repositório GitHub** atualizado (público, ou privado com acesso liberado ao professor).
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

Copie este arquivo para a raiz do repositório. Ele já inclui alvos para compilação, testes, `gcov` e `gprof`.

```makefile
CC = gcc
WARN = -Wall -Wextra -std=c11
CFLAGS = $(WARN) -O2
GCOV_FLAGS = --coverage -g -O0
PROFILE_FLAGS = -pg -g -O0

SRCS = src/main.c src/basicos/bubble.c src/basicos/selection.c src/basicos/insertion.c \
       src/avancados/merge.c src/avancados/quick.c src/avancados/heap.c
TEST_SRCS = tests/test_basic.c $(filter-out src/main.c,$(SRCS))
OBJS = $(SRCS:.c=.o)
TEST_OBJS = $(TEST_SRCS:.c=.o)

TARGET = sort_main
TEST_TARGET = test_basic

all: $(TARGET)

$(TARGET): $(OBJS)
	$(CC) $(CFLAGS) -o $@ $^

$(TEST_TARGET): $(TEST_OBJS)
	$(CC) $(CFLAGS) -o $@ $^

%.o: %.c
	$(CC) $(CFLAGS) -c $< -o $@

test: $(TEST_TARGET)
	./$(TEST_TARGET)

# Cobertura: recompila tudo sem otimização e com instrumentação (CFLAGS substituído)
coverage: CFLAGS = $(WARN) $(GCOV_FLAGS)
coverage: clean test
	gcov -o src/basicos src/basicos/*.c
	gcov -o src/avancados src/avancados/*.c
	@echo "Relatórios .gcov gerados na raiz do projeto"

# Profiling: só -O0 (sem o -O2 de CFLAGS) e só o main.c como ponto de entrada
profile: clean
	$(CC) $(WARN) $(PROFILE_FLAGS) -o sort_profile $(SRCS)
	./sort_profile
	gprof sort_profile gmon.out > docs/perfil_bruto.txt
	@echo "Profiling gerado em docs/perfil_bruto.txt"
	@echo "Abra o arquivo e analise a seção 'Flat Profile'"

clean:
	rm -f $(TARGET) $(TEST_TARGET) sort_profile $(OBJS) $(TEST_OBJS) *.gcov gmon.out
	find . -name '*.gcda' -o -name '*.gcno' | xargs rm -f

.PHONY: all test coverage profile clean
```

## Dicas Rápidas

| Ferramenta | Dica |
| --- | --- |
| `gcov` | Se a cobertura ficar abaixo de 70%, adicione testes com `n=0`, `n=1`, vetores com duplicatas e invertidos. |
| `gprof` | Não rode com `-O2` — o otimizador distorce medições. Use `make profile` (já aplica `-O0`). |
| `git` | Uma mensagem como `fix: ajusta condição de parada no quicksort` vale mais que 10 commits `update`. |
| IA | Peça explicação, não código pronto. Valide linha a linha — a IA é copiloto, não piloto. |
| Entrega | Teste `make clean && make all && make test && make coverage && make profile` antes de dar `push`. |
