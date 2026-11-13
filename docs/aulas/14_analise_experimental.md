# Análise Experimental e Ferramentas de Qualidade (gcov, gprof e cppcheck)

**Disciplina:** Estrutura de Dados

A teoria de complexidade nos diz **como** o custo de um algoritmo cresce. Neste capítulo aprendemos a **verificar isso na prática**: medindo tempo, contando operações e usando três ferramentas de qualidade para C: `gcov` (o que os testes cobrem), `gprof` (onde o tempo é gasto) e `cppcheck` (erros que o compilador não aponta).

!!! info "Pré-requisitos"
    - **Teoria:** a notação Big-O, os casos (melhor, médio e pior) e o teste de dobrar `n` estão na Seção 2 do capítulo *Algoritmos de Ordenação Básicos*. Aqui não repetimos a teoria, apenas a usamos.
    - **Algoritmos:** os exemplos e o trabalho prático usam os seis algoritmos de ordenação (básicos e eficientes). Estude antes os dois capítulos de ordenação.

!!! info "Conexão com o trabalho prático"
    Tudo o que aparece aqui é cobrado no **Trabalho Prático de Algoritmos de Ordenação**: a análise experimental (Seção 2), a cobertura (Seção 3), o profiling (Seção 4) e a análise estática (Seção 5). A Seção 8 mostra, ferramenta por ferramenta, onde cada conceito entra na entrega.

---

## 1. Da Teoria à Medição

Cada ferramenta deste capítulo responde a uma pergunta diferente sobre o seu programa:

| Pergunta | Ferramenta | Como observa o programa |
| --- | --- | --- |
| O custo cresce como a teoria prevê? | `clock()` e contagem de operações | Executando |
| Os meus testes exercitam o código? | `gcov` | Executando |
| Onde o programa gasta mais tempo? | `gprof` | Executando |
| Há erros que o compilador não aponta? | `cppcheck` | **Sem executar** (lê o código-fonte) |

Um fluxo de trabalho razoável é: **(1)** conferir se o programa está correto e se os testes o exercitam (`gcov`); **(2)** procurar erros latentes (`cppcheck`); **(3)** medir o crescimento do custo e comparar com a teoria; **(4)** se algo destoar da teoria, usar o `gprof` para descobrir **onde** o tempo está sendo gasto.

---

## 2. Medição Empírica: Tempo e Contagem de Operações

A teoria prevê **como** o custo cresce; a medição confirma (ou contradiz) a previsão no seu computador. Há duas medidas complementares:

- **Tempo de execução**, com `clock()` da biblioteca `<time.h>`. Depende da máquina, do compilador e da carga do sistema.
- **Contagem de operações** (comparações e trocas), feita pelo próprio código. É **exata e reprodutível**, independente da máquina.

### 2.1 Exemplo: Bubble Sort com Três Tipos de Entrada

No trabalho prático, todos os algoritmos recebem um ponteiro para uma estrutura `Metricas`, que acumula os contadores. Este exemplo aplica a ideia ao Bubble Sort **simples** (sem parada antecipada):

```c
#include <stdio.h>
#include <stdlib.h>
#include <time.h>

typedef struct {
    unsigned long comparacoes;
    unsigned long trocas;
} Metricas;

void bubbleSort(int v[], int n, Metricas *m) {
    for (int i = 0; i < n - 1; i++) {
        for (int j = 0; j < n - i - 1; j++) {
            m->comparacoes++;
            if (v[j] > v[j+1]) {
                int temp = v[j];
                v[j] = v[j+1];
                v[j+1] = temp;
                m->trocas++;
            }
        }
    }
}

// tipo: 0 = aleatório, 1 = crescente, 2 = decrescente
void gerarVetor(int v[], int n, int tipo) {
    srand(42);
    for (int i = 0; i < n; i++) {
        if (tipo == 0)      v[i] = rand();
        else if (tipo == 1) v[i] = i;
        else                v[i] = n - i;
    }
}

int main(void) {
    const int n = 5000;
    const char *nomes[] = {"aleatorio", "crescente", "decrescente"};
    int *v = malloc(n * sizeof(int));
    if (v == NULL) return 1;

    for (int tipo = 0; tipo < 3; tipo++) {
        gerarVetor(v, n, tipo);
        Metricas m = {0, 0};

        clock_t ini = clock();
        bubbleSort(v, n, &m);
        clock_t fim = clock();

        double ms = 1000.0 * (fim - ini) / CLOCKS_PER_SEC;
        printf("%-12s tempo=%7.1f ms  comparacoes=%lu  trocas=%lu\n",
               nomes[tipo], ms, m.comparacoes, m.trocas);
    }
    free(v);
    return 0;
}
```

Saída em uma máquina de teste (compilado com `gcc -O2`; os tempos variam, as contagens não):

```text
aleatorio    tempo=   61.5 ms  comparacoes=12497500  trocas=6170685
crescente    tempo=   10.2 ms  comparacoes=12497500  trocas=0
decrescente  tempo=   88.6 ms  comparacoes=12497500  trocas=12497500
```

Como interpretar:

- As **comparações** são idênticas nos três casos (`5000 · 4999 / 2 = 12.497.500`): o Bubble Sort simples não "enxerga" que o vetor já está ordenado.
- As **trocas** variam de 0 (crescente) a `n(n-1)/2` (decrescente); no vetor aleatório, cerca de metade.
- O **tempo** acompanha as trocas: o caso crescente é o mais rápido porque nenhuma troca é feita.

!!! note "Qual versão do Bubble Sort você está medindo?"
    Este exemplo usa a versão **simples**, que sempre faz `n(n-1)/2` comparações. A versão **otimizada** (com parada antecipada) faria apenas `n-1` comparações no vetor crescente. As duas estão analisadas na Seção 3.5 do capítulo de ordenação básica. Ao apresentar resultados, **diga sempre qual versão foi medida**.

### 2.2 Boas Práticas de Medição

| Prática | Motivo |
| --- | --- |
| Medir **só a ordenação** (não a geração do vetor nem a cópia) | O cronômetro deve capturar apenas o que está sendo comparado |
| Usar semente fixa (`srand(42)`) | Resultados reproduzíveis |
| Repetir e tirar a **média** | Reduz o ruído do sistema |
| Variar `n` (por exemplo 1.000, 5.000, 10.000, 50.000) | Permite ver a **curva de crescimento**, não só um ponto |
| Testar entrada aleatória, crescente e decrescente | Expõe melhor e pior caso |
| Compilar o **benchmark** com `-O2` | Mede o desempenho real do programa |

!!! note "Quais flags usar em cada ferramenta"
    | Atividade | Flags | Motivo |
    | --- | --- | --- |
    | Testes e benchmark | `-O2` | Desempenho real |
    | `gcov` | `-O0 --coverage` | Correspondência direta com as linhas do código |
    | `gprof` | `-O0 -g -pg` | O *inline* do otimizador esconde funções |
    | `cppcheck` | (não compila) | Analisa o código-fonte |

### 2.3 O Teste de Dobrar `n`

Para descobrir a ordem de crescimento a partir das medições, meça o tempo para `n` e para `2n` e calcule a **razão**:

```text
razão = T(2n) / T(n)
```

| Razão observada | Crescimento sugerido |
| --- | --- |
| ≈ 1 | O(1) ou O(log n) |
| ≈ 2 | O(n) |
| um pouco acima de 2 (cerca de 2,1 a 2,3) | O(n log n) |
| ≈ 4 | O(n²) |

Se `T(n) = k · nᵃ`, então a razão vale `2ᵃ`, e o expoente estimado é `a = log₂(razão)`.

**Exemplo (valores hipotéticos).** Suponha que um algoritmo levou 60 ms com `n = 5.000` e 240 ms com `n = 10.000`. A razão é `240 / 60 = 4`, e `a = log₂(4) = 2`: o comportamento é quadrático, como previsto para o Bubble Sort.

### 2.4 Comparando com a Teoria

Ao analisar os resultados no trabalho prático, vá além de dizer "o Quick Sort foi mais rápido". Pergunte:

- Quando `n` dobra, o tempo dos algoritmos O(n²) **quadruplica**? E o dos O(n log n) pouco mais que dobra?
- Os resultados **contradizem** a teoria em algum ponto? Por quê? (Constantes ocultas, efeitos de cache, entrada pequena, parada antecipada...)
- A contagem de comparações e trocas explica a diferença de tempo entre os algoritmos?
- Para o Quick Sort: a entrada (ordenada, invertida) cai no pior caso? Isso depende de **como o pivô é escolhido**.

---

## 3. Cobertura de Código com `gcov`

O `gcov` é uma ferramenta integrada ao compilador GCC que analisa quais linhas do código-fonte foram de fato executadas durante um teste e quantas vezes cada uma rodou. Isso é chamado de **Cobertura de Código** (*Code Coverage*).

### 3.1 Como funciona o `gcov`?

Ao compilar o programa com flags especiais, o GCC insere instruções adicionais que gravam a contagem de execuções.

- **`.gcno`**: arquivo criado na compilação. Contém informações para mapear o binário de volta ao código-fonte.
- **`.gcda`**: arquivo criado ao executar o binário. Contém os dados de execução coletados em tempo de execução.

### 3.2 Passo a Passo de Uso

1. **Compilação com suporte a cobertura.** Usamos a flag `--coverage` (ou `-fprofile-arcs -ftest-coverage`) junto com `-O0`, que desativa as otimizações do compilador e mantém a correspondência direta de linhas. Compile e ligue em **duas etapas**:

    ```bash
    gcc -O0 -g --coverage -c ordenacao.c
    gcc --coverage ordenacao.o -o programa
    ```

    !!! note "GCC 11 ou superior"
        Compilar e ligar num único comando (`gcc --coverage ordenacao.c -o programa`) gera os arquivos com o nome `programa-ordenacao.gcda` e `.gcno`, e `gcov ordenacao.c` não os encontra. Nesse caso, use `gcov programa-ordenacao.c` ou prefira as duas etapas acima.

2. **Executar o programa** para gerar os dados de contagem:

    ```bash
    ./programa
    ```

    Isso cria o arquivo `ordenacao.gcda` no diretório.

3. **Gerar o relatório `gcov`**, apontando para o arquivo fonte:

    ```bash
    gcov ordenacao.c
    ```

    O resumo no terminal informa o percentual de linhas executadas:

    ```text
    File 'ordenacao.c'
    Lines executed:84.91% of 53
    Creating 'ordenacao.c.gcov'
    ```

4. **Ler o relatório.** O arquivo `ordenacao.c.gcov` traz, à esquerda de cada linha, a contagem de execuções:

    - `10`: a linha foi executada 10 vezes.
    - `#####`: linha executável que **nunca** foi executada.
    - `-`: linha sem código executável (comentários, chaves, declarações).
    - `19998*`: a linha rodou, mas **parte dela não** (por exemplo, o `return 0;` escrito na mesma linha do `if`). O `*` aparece nas versões mais recentes do `gcov`.

    Trecho real do exemplo da Seção 7:

    ```text
            1:   43:    int *v1 = malloc(n * sizeof(int));
            1:   44:    int *v2 = malloc(n * sizeof(int));
            1:   45:    if (v1 == NULL || v2 == NULL) {
        #####:   46:        fprintf(stderr, "Erro: falha ao alocar memória\n");
        #####:   47:        free(v1);
        #####:   48:        free(v2);
        #####:   49:        return 1;
            -:   50:    }
    ```

    O bloco de tratamento de erro do `malloc` nunca rodou, porque a alocação não falhou nesse teste. É um exemplo típico de código defensivo que fica sem cobertura.

### 3.3 Cobertura de Desvios (*branches*)

A cobertura de linhas diz se uma linha rodou. A cobertura de **desvios** diz se cada decisão (`if`, `while`, `&&`, `||`) tomou **os dois caminhos**. Para ativá-la, use `-b`:

```bash
gcov -b ordenacao.c
```

```text
File 'ordenacao.c'
Lines executed:84.91% of 53
Branches executed:100.00% of 36
Taken at least once:83.33% of 36
Calls executed:84.62% of 13
Creating 'ordenacao.c.gcov'
```

- **Branches executed:** percentual de desvios que chegaram a ser avaliados.
- **Taken at least once:** percentual de desvios em que **cada direção** foi tomada ao menos uma vez. É a medida mais exigente: um `if` que sempre foi verdadeiro conta como "executado", mas não como "tomado nos dois sentidos".

### 3.4 Projetos com Vários Arquivos

No trabalho prático, cada algoritmo fica em um `.c` separado, dentro de `src/basicos/` e `src/avancados/`. Quando os objetos ficam em outro diretório, use `-o` para dizer onde estão os arquivos `.gcno` e `.gcda`:

```bash
gcov -o src/basicos src/basicos/*.c
gcov -o src/avancados src/avancados/*.c
```

O Makefile do trabalho já faz tudo isso com `make coverage`, e grava os relatórios em `docs/gcov/`. A meta é **70% de cobertura de linhas** nos seis arquivos de algoritmos.

!!! warning "Cobertura não é correção"
    Um programa pode ter 100% de cobertura e ainda estar errado, porque a cobertura só mede **o que rodou**, não **se o resultado está certo**. Por isso o exemplo da Seção 7 inclui a função `estaOrdenado()`: sem verificar o resultado, o teste só prova que o código não travou. Se a sua cobertura ficar abaixo da meta, acrescente testes com `n = 0`, `n = 1`, vetores com elementos repetidos, ordenados e invertidos.

---

## 4. Profiling de Desempenho com `gprof`

O `gprof` é um **profiler** do GNU. Ele mede quanto tempo a CPU gasta em cada função e quantas vezes cada função foi chamada. Enquanto o `gcov` responde "quais linhas foram executadas?", o `gprof` responde "**onde o programa gastou mais tempo?**".

### 4.1 Como funciona o `gprof`?

O `gprof` combina duas técnicas:

- **Amostragem:** a cada **10 ms** o programa é interrompido e o `gprof` registra em qual função a CPU está. Esse é o tempo de cada função.
- **Instrumentação:** o compilador insere uma chamada no início de cada função (flag `-pg`) para contar quem chamou quem e quantas vezes.

Ao final da execução, os dados são gravados no arquivo `gmon.out`.

### 4.2 Passo a Passo de Uso

1. **Compilação com suporte a profile**, usando a flag `-pg`:

    ```bash
    gcc -O0 -g -pg -o programa ordenacao.c
    ```

    !!! warning "Use `-O0`"
        Com `-O2` ou `-O3` o compilador pode embutir (*inline*) funções inteiras dentro de quem as chama, e elas somem do relatório. Para o profiling, use sempre `-O0`, lembrando que os tempos absolutos serão maiores que os de produção.

2. **Executar o programa** normalmente, o que gera o arquivo `gmon.out`:

    ```bash
    ./programa
    ```

3. **Gerar o relatório formatado**, passando o executável e o arquivo gerado:

    ```bash
    gprof programa gmon.out > relatorio.txt
    ```

### 4.3 Interpretando o Relatório do `gprof`

O relatório contém duas tabelas principais.

#### A. Flat Profile (Perfil Plano)

Mostra o tempo gasto em cada função, ordenado por consumo. Saída real do programa da Seção 7, com `n = 10000`:

```text
Flat profile:

Each sample counts as 0.01 seconds.
  %   cumulative   self              self     total
 time   seconds   seconds    calls  ms/call  ms/call  name
100.00      0.12     0.12        1   120.00   120.00  bubbleSort
  0.00      0.12     0.00        2     0.00     0.00  estaOrdenado
  0.00      0.12     0.00        1     0.00     0.00  quickSort
```

- **`% time`:** porcentagem do tempo total gasta nesta função.
- **`self seconds`:** tempo gasto exclusivamente nesta função.
- **`calls`:** número de chamadas.
- **`self ms/call`:** tempo médio por chamada.

O Quick Sort leva menos que o intervalo de amostragem de 0,01 s e aparece com 0%. Os tempos absolutos variam de uma máquina para outra.

#### B. Call Graph (Grafo de Chamadas)

Mostra qual função chamou qual outra, e quantas vezes. Trecho do mesmo exemplo:

```text
index % time    self  children    called     name
                0.00    0.00       2/2           main [2]
[3]      0.0    0.00    0.00       2         estaOrdenado [3]
-----------------------------------------------
                               11928             quickSort [4]
                0.00    0.00       1/1           main [2]
[4]      0.0    0.00    0.00       1+11928   quickSort [4]
                               11928             quickSort [4]
```

O valor `1+11928` do `quickSort` indica **1 chamada externa** (vinda do `main`) mais **11928 chamadas recursivas**. O call graph é útil para identificar gargalos que não aparecem no flat profile.

### 4.4 A Resolução de 10 ms

Como o `gprof` amostra a cada 10 ms, programas muito rápidos **não geram dados suficientes**. Rodando o mesmo exemplo com `n = 1000`, o relatório sai assim:

```text
Flat profile:

Each sample counts as 0.01 seconds.
 no time accumulated
```

Por isso, no trabalho prático o profiling usa `n = 50000` por padrão. Para analisar os algoritmos avançados (que são rápidos), aumente `n` e escolha um algoritmo por vez:

```bash
make profile                          # n = 50000, todos os algoritmos
make profile N=2000000 ALG=quick      # só o Quick Sort, com n grande
```

### 4.5 Visualização Gráfica (Opcional)

O `gprof2dot` converte o relatório em um grafo, desenhado pelo `dot` (pacote Graphviz):

```bash
pip install gprof2dot
sudo apt install graphviz
gprof programa gmon.out | gprof2dot | dot -Tpng -o grafo.png
```

---

## 5. Análise Estática com `cppcheck`

Enquanto o `gcov` e o `gprof` observam o programa **em execução**, o `cppcheck` lê o **código-fonte sem executá-lo** e procura erros que o compilador costuma deixar passar: vazamentos de memória, acessos fora dos limites, variáveis não usadas, ponteiros nulos.

### 5.1 Instalação e Uso

```bash
sudo apt install cppcheck
cppcheck --enable=warning,style,performance,portability --std=c11 \
         --inline-suppr --suppress=missingIncludeSystem src tests
```

O `cppcheck` escreve os resultados na saída de erro (`stderr`). Para guardá-los em arquivo, redirecione com `2> arquivo.txt` (é o que faz o `make cppcheck` do trabalho).

### 5.2 Exemplo

Considere este arquivo com dois problemas:

```c
#include <stdlib.h>

void exemplo(void) {
    int v[5];
    int *p = malloc(10 * sizeof(int));
    v[5] = 1;
}
```

Saída real do `cppcheck`:

```text
bug.c:6:6: error: Array 'v[5]' accessed at index 5, which is out of bounds. [arrayIndexOutOfBounds]
    v[5] = 1;
     ^
bug.c:7:1: error: Memory leak: p [memleak]
}
^
```

O vetor `v` tem posições de 0 a 4, então `v[5]` está fora dos limites; e a memória alocada para `p` nunca é liberada. O `cppcheck` também emite avisos de estilo (como variáveis atribuídas e nunca lidas), omitidos acima.

### 5.3 Classificação das Mensagens

| Tipo | Significado | O que fazer |
| --- | --- | --- |
| `error` | Erro real no código (acesso inválido, vazamento) | **Corrigir sempre** |
| `warning` | Comportamento perigoso ou suspeito | Corrigir, ou justificar por escrito |
| `style` | Sugestão de estilo (variável não usada, `const`) | Pode manter, se justificado |
| `performance` | Possível ganho de desempenho | Avaliar caso a caso |
| `portability` | Pode se comportar diferente em outras plataformas | Avaliar caso a caso |

!!! tip "O `malloc` sem verificação"
    Ao analisar uma versão anterior do exemplo da Seção 7, **sem** o teste `if (v1 == NULL || v2 == NULL)`, o `cppcheck` apontou vários avisos `nullPointerOutOfMemory` (uso de ponteiro que pode ser nulo se a alocação falhar). É a mesma boa prática dos capítulos anteriores: **sempre verifique o retorno do `malloc`**.

---

## 6. Escolhendo a Ferramenta Certa

| Situação | Ferramenta |
| --- | --- |
| "Não sei se meus testes passam por todos os caminhos do código." | `gcov` |
| "O programa está mais lento do que a teoria previa. Onde está o gargalo?" | `gprof` |
| "Quero achar vazamento de memória ou acesso fora dos limites sem rodar nada." | `cppcheck` |
| "Quero saber se o tempo dobra ou quadruplica quando `n` dobra." | `clock()` e o teste de dobrar `n` (Seção 2.3) |
| "Quantas comparações e trocas o algoritmo fez, independente da máquina?" | Contadores de operações (`Metricas`) |

!!! note "Limites das ferramentas"
    O `cppcheck` analisa o código sem rodar e pode errar em ambos os sentidos (deixar passar um erro ou apontar algo que não é um problema). O `gcov` e o `gprof` só enxergam o que **foi executado** com a entrada usada. Nenhuma das três prova que o programa está correto: para isso são necessários testes que verifiquem os resultados.

---

## 7. Visualizando em Conjunto: Um Exemplo Completo

Temos um arquivo `ordenacao.c` com o **Bubble Sort** (versão simples) e o **Quick Sort**. Queremos comparar a performance, verificar se os resultados estão corretos, ver o que os testes cobrem e procurar erros no código.

### Código Exemplo (`ordenacao.c`)

```c
#include <stdio.h>
#include <stdlib.h>

void bubbleSort(int v[], int n) {
    for (int i = 0; i < n - 1; i++) {
        for (int j = 0; j < n - i - 1; j++) {
            if (v[j] > v[j+1]) {
                int temp = v[j];
                v[j] = v[j+1];
                v[j+1] = temp;
            }
        }
    }
}

void quickSort(int v[], int n) {
    if (n <= 1) return;
    int pivo = v[n / 2];
    int i = 0, j = n - 1;
    while (i <= j) {
        while (v[i] < pivo) i++;
        while (v[j] > pivo) j--;
        if (i <= j) {
            int temp = v[i];
            v[i] = v[j];
            v[j] = temp;
            i++; j--;
        }
    }
    if (j > 0) quickSort(v, j + 1);
    if (i < n) quickSort(v + i, n - i);
}

int estaOrdenado(const int v[], int n) {
    for (int i = 0; i < n - 1; i++) {
        if (v[i] > v[i+1]) return 0;
    }
    return 1;
}

int main(int argc, char *argv[]) {
    int n = (argc > 1) ? atoi(argv[1]) : 10000;
    int *v1 = malloc(n * sizeof(int));
    int *v2 = malloc(n * sizeof(int));
    if (v1 == NULL || v2 == NULL) {
        fprintf(stderr, "Erro: falha ao alocar memória\n");
        free(v1);
        free(v2);
        return 1;
    }

    // Gerar dados aleatórios (mesmos valores nos dois vetores)
    srand(42);
    for (int i = 0; i < n; i++) {
        int val = rand() % 100000;
        v1[i] = val;
        v2[i] = val;
    }

    printf("Ordenando com Bubble Sort (n=%d)...\n", n);
    bubbleSort(v1, n);

    printf("Ordenando com Quick Sort (n=%d)...\n", n);
    quickSort(v2, n);

    if (!estaOrdenado(v1, n) || !estaOrdenado(v2, n)) {
        fprintf(stderr, "ERRO: vetor não ficou ordenado!\n");
        free(v1);
        free(v2);
        return 1;
    }
    printf("OK: os dois vetores estão ordenados.\n");

    free(v1);
    free(v2);
    return 0;
}
```

O programa aceita o tamanho do vetor como argumento (`./programa 50000`) e usa `n = 10000` por padrão.

### Script de Análise Completa (`analise.sh`)

Você pode usar um script para compilar e analisar tudo de uma vez:

```bash
#!/bin/bash
set -e

echo "=== 1. COMPILANDO PARA COBERTURA (GCOV) ==="
gcc -O0 -g --coverage -c ordenacao.c
gcc --coverage ordenacao.o -o prog_cov

echo "=== 2. EXECUTANDO ==="
./prog_cov

echo "=== 3. RELATÓRIO GCOV (linhas e branches) ==="
gcov -b ordenacao.c
echo "--- Linhas nunca executadas ---"
grep -n '#####' ordenacao.c.gcov || true

echo "=== 4. PROFILE (GPROF) ==="
gcc -O0 -g -pg ordenacao.c -o prog_prof
./prog_prof
gprof prog_prof gmon.out > relatorio_gprof.txt
echo "--- Flat profile ---"
head -n 10 relatorio_gprof.txt
echo "--- Call graph ---"
sed -n '/Call graph/,/This table describes/p' relatorio_gprof.txt

echo "=== 5. ANÁLISE ESTÁTICA (CPPCHECK) ==="
cppcheck --enable=warning,style,performance,portability --std=c11 ordenacao.c
```

### O Que Observar nos Resultados

| Ferramenta | O que o exemplo mostra |
| --- | --- |
| `gcov` | 84,91% das linhas. As oito linhas `#####` são os dois blocos de tratamento de erro (falha do `malloc` e vetor não ordenado), que nunca rodam em uma execução normal. |
| `gcov -b` | Só 83,33% dos desvios foram tomados nos dois sentidos: os `if` de erro nunca foram verdadeiros. |
| `gprof` | O `bubbleSort` concentra praticamente todo o tempo, enquanto o `quickSort` fica abaixo da resolução de 10 ms. |
| `cppcheck` | Resta apenas uma sugestão de estilo (`argv` poderia ser `const`), que pode ser mantida com justificativa. |

---

## 8. Da Aula ao Trabalho Prático

| Conceito desta aula | Entregável do trabalho | Comando |
| --- | --- | --- |
| Tempo (`clock()`), comparações e trocas, comparação com a teoria (Seção 2) | `docs/resultados.csv` e `docs/analise.md` | `make bench` |
| Cobertura de linhas ≥ 70% nos 6 algoritmos (Seção 3) | `docs/cobertura.md` e `docs/gcov/` | `make coverage` |
| Profiling com `n` grande (Seção 4) | `docs/perfil_bruto.txt` e `docs/perfil.md` | `make profile` |
| Análise estática (Seção 5) | `docs/cppcheck_bruto.txt` e `docs/cppcheck.md` | `make cppcheck` |

No **vídeo de defesa**, o grupo precisa mostrar o `gcov` (cobertura obtida), o `gprof` (funções mais pesadas), o `cppcheck` (correções feitas) e confrontar a tabela de medições com a teoria (capítulo de ordenação básica, Seção 2, e tabela comparativa do capítulo de ordenação eficiente).

---

## 9. Exercícios para Fixação

### Exercício 1 — Contagem no Bubble Sort

Para um vetor de `n = 5` elementos, quantas comparações o Bubble Sort simples (sem parada antecipada) faz em um vetor crescente? E em um decrescente? Quantas trocas faz em cada caso?

<details>
<summary>Gabarito</summary>

```text
Comparações: n(n-1)/2 = 5·4/2 = 10 nos dois casos.
Trocas: 0 no vetor crescente; 10 no decrescente (todo par está invertido).
```

</details>

### Exercício 2 — Entrada Já Ordenada

Quantas comparações o Insertion Sort faz em um vetor já ordenado de `n = 1000` elementos? Qual o seu custo nesse caso, em notação Big-O?

<details>
<summary>Gabarito</summary>

```text
n - 1 = 999 comparações (cada elemento é comparado uma vez com o anterior
e já está na posição certa). Custo: O(n), o melhor caso do algoritmo.
```

</details>

### Exercício 3 — Relatório Vazio

Você rodou o `gprof` com `n = 1000` e o relatório saiu com "no time accumulated". Isso significa que o programa não gastou tempo? O que fazer?

<details>
<summary>Gabarito</summary>

```text
Não. O programa gastou menos que o intervalo de amostragem do gprof (10 ms),
então nenhuma amostra caiu nas funções. A solução é aumentar o n (ou repetir
a execução), como em: make profile N=50000.
```

</details>

### Exercício 4 — Linhas `#####`

No relatório do `gcov` do exemplo, as linhas do bloco `if (!estaOrdenado(...))` aparecem como `#####`. Isso é um problema? Como conseguir cobri-las?

<details>
<summary>Gabarito</summary>

```text
Não é um defeito: o bloco só roda quando a ordenação falha, e o programa
está correto. Mas mostra que o caminho de erro nunca foi exercitado.
Para cobri-lo, um teste precisa provocar a condição (por exemplo, chamando
a verificação com um vetor desordenado de propósito, em um teste separado).
Lembre: 100% de cobertura não prova que o programa está correto.
```

</details>

### Exercício 5 — Identificando a Complexidade pelas Medições

Um algoritmo de ordenação foi medido com entrada aleatória e produziu estes tempos:

| `n` | Tempo |
| --- | --- |
| 10.000 | 20 ms |
| 20.000 | 82 ms |
| 40.000 | 330 ms |

Calcule as razões `T(2n) / T(n)`. Qual é a ordem de crescimento provável?

<details>
<summary>Gabarito</summary>

```text
Razões: 82/20 = 4,1  e  330/82 ≈ 4,0.
Dobrar n multiplica o tempo por cerca de 4, logo o expoente é
a = log₂(4) = 2 → crescimento quadrático, O(n²).
Compatível com Bubble, Selection ou Insertion Sort em entrada aleatória.
```

</details>

### Exercício 6 — Qual Ferramenta Usar?

Para cada situação, indique a ferramenta mais adequada (`gcov`, `gprof`, `cppcheck` ou medição com `clock()`):

a) Descobrir qual função do programa consome mais tempo de CPU.

b) Saber se algum `if` de tratamento de erro nunca foi executado nos seus testes.

c) Encontrar um `malloc` sem o `free` correspondente sem executar o programa.

d) Verificar se o tempo do Merge Sort cresce como `n log n`.

<details>
<summary>Gabarito</summary>

```text
a) gprof (flat profile)
b) gcov (linhas marcadas com #####; gcov -b para os desvios)
c) cppcheck (aviso "Memory leak")
d) Medição com clock() para vários n e comparação das razões T(2n)/T(n),
   que devem ficar um pouco acima de 2.
```

</details>

---

## 10. Resumo de Comandos Úteis

```text
┌──────────────────────────────────────────────────────────────┐
│ COMMAND CHEATSHEET - COBERTURA, PERFIL E ANÁLISE ESTÁTICA    │
├──────────────────────────────────────────────────────────────┤
│ GCOV (COBERTURA DE CÓDIGO)                                   │
│ 1. gcc -O0 -g --coverage -c fonte.c                          │
│    gcc --coverage fonte.o -o bin                             │
│ 2. ./bin (roda os testes)                                    │
│ 3. gcov fonte.c      (linhas; gera fonte.c.gcov)             │
│    gcov -b fonte.c   (inclui cobertura de desvios)           │
│    gcov -o dir dir/*.c   (objetos em outro diretório)        │
│                                                              │
│ GPROF (ANÁLISE DE TEMPO DE CPU)                              │
│ 1. gcc -O0 -g -pg -o bin fonte.c                             │
│ 2. ./bin (gera gmon.out; use n grande: amostragem de 10 ms)  │
│ 3. gprof bin gmon.out > relatorio.txt                        │
│                                                              │
│ CPPCHECK (ANÁLISE ESTÁTICA)                                  │
│ cppcheck --enable=warning,style,performance,portability \    │
│          --std=c11 src tests 2> relatorio.txt                │
│                                                              │
│ BENCHMARK (TEMPO E CONTAGEM)                                 │
│ gcc -O2 -o bench bench.c   (tempo real, sem -pg/--coverage)  │
│ razão = T(2n)/T(n):  ~2 → O(n)   ~4 → O(n²)                  │
│                                                              │
│ FERRAMENTAS VISUAIS COMPLEMENTARES                           │
│ • gcovr / lcov + genhtml (relatórios HTML do gcov)           │
│ • gprof bin gmon.out | gprof2dot | dot -Tpng -o grafo.png    │
└──────────────────────────────────────────────────────────────┘
```

---

## 11. Resumo do Capítulo

✅ **Medição**

- Tempo (`clock()`) depende da máquina; contagem de operações é exata e reprodutível
- Varie `n`, repita e use semente fixa; compile o benchmark com `-O2`
- Teste de dobrar `n`: razão ≈ 2 → linear, ≈ 4 → quadrático, pouco acima de 2 → O(n log n)

✅ **`gcov`**

- Mostra quais linhas e desvios foram executados (`#####` = nunca executada)
- Compile com `-O0 --coverage`; a meta do trabalho é 70% de linhas
- Cobertura não é correção: verifique sempre os resultados

✅ **`gprof`**

- Mostra onde a CPU gasta tempo (flat profile) e quem chama quem (call graph)
- Compile com `-O0 -g -pg`; amostragem de 10 ms exige `n` grande

✅ **`cppcheck`**

- Análise estática: acha vazamentos, acessos fora dos limites e ponteiros nulos sem executar
- Corrija sempre os `error`; justifique por escrito o que mantiver

✅ **Prática**

- Sempre diga qual versão do algoritmo foi medida
- Confronte as medições com a teoria e explique as divergências (constantes, cache, `n` pequeno, escolha do pivô)

---

*Material elaborado para a disciplina de Estrutura de Dados — Curso de Engenharia de Computação*

*Prof. Sérgio Souza Costa | Atualizado: 2026*
