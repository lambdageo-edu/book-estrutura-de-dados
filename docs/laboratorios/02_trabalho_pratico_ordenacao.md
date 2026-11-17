# Laboratório: Algoritmos de Ordenação em C — Passo a Passo

Este tutorial é o **caminho** para chegar ao Trabalho Prático de ordenação. A especificação (o que entregar, rubrica e forma de envio) está no final do capítulo, em [Trabalho Prático](#trabalho-pratico). Aqui você aprende **como construir** cada peça, em etapas pequenas, cada uma com um **checkpoint** (um comando e o resultado esperado) e um **commit sugerido**.

!!! info "Como usar este tutorial"
    - Nunca usou Git ou GitHub? Comece pela **Etapa 0**: ela explica conta, criação do repositório, commits e trabalho em dupla.
    - Siga as etapas **na ordem**. Só avance quando o checkpoint passar.
    - Faça **um commit ao fim de cada etapa**. É isso que cria o histórico que será avaliado.
    - A etapa 2 (Bubble Sort) é um exemplo **completo** do ciclo: código, teste, métricas. As demais etapas dão o contrato, dicas e os casos de teste; **o código é seu**.
    - Blocos recolhidos (`???`) contêm dicas. Tente primeiro sem abri-los.

!!! info "Sobre o prazo de 2 semanas"
    O prazo sugerido é de **2 semanas**, mas isso **não quer dizer 2 semanas de trabalho todos os dias**. Ninguém tem tempo livre diariamente, e tudo bem. O prazo existe para a dupla **se organizar**.

    Este **não é um trabalho para fazer de uma vez** (nem em uma hora, nem em uma noite). Ele tem partes que levam tempo: implementar e testar seis algoritmos, rodar o benchmark, escrever a análise, ensaiar e gravar o vídeo. Combinem **quando** cada um consegue trabalhar, dividam as tarefas e **comecem cedo**, em sessões curtas, para não deixar tudo para o último dia.

As etapas deste tutorial são:

| Etapa | Foco |
| --- | --- |
| 0 | Git e GitHub: conta, repositório, commits, trabalho em dupla |
| 1 | Setup do projeto: Makefile, `metricas.h`, esqueletos |
| 2 | Bubble Sort completo (código + teste + métricas) |
| 3 | Selection e Insertion Sort |
| 4 | Merge, Quick e Heap Sort |
| 5 | Ferramentas: `cppcheck`, `gcov`, `gprof` |
| 6 | Benchmark e análise experimental |
| 7 | Documentação, vídeo e entrega |

---

## Etapa 0 — Git e GitHub: o essencial

Seu histórico no Git **faz parte da nota** (20 pontos da rubrica de código, mais a definição de quem apresenta o vídeo). Por isso esta etapa vem antes de qualquer código. Se você já usa Git no dia a dia, leia só as seções 0.6 a 0.8 e faça o **Checkpoint 0**.

!!! info "Sistema operacional"
    O laboratório usa `gcc`, `make`, `gcov` e `gprof`, que funcionam nativamente em Linux. No Windows, use o **WSL** (Ubuntu); no macOS, o `gcc` do Xcode/Homebrew serve, mas o `gprof` pode se comportar diferente. Todos os comandos deste tutorial assumem um terminal Linux.

### 0.1 Os conceitos em 1 minuto

- **Git** guarda o histórico do seu projeto no seu computador. Cada ponto salvo do histórico é um **commit**: uma "fotografia" dos arquivos, com autor, data e uma mensagem.
- **GitHub** é um servidor onde fica uma cópia do repositório na internet (o **remoto**, chamado `origin`). É por ele que a dupla troca código e que o professor corrige.
- Um arquivo passa por **três áreas** até virar histórico:

| Área | O que é | Como entra | Como ver |
| --- | --- | --- | --- |
| **Diretório de trabalho** | Os arquivos que você edita | Você edita no editor | `git status` |
| **Área de preparação** (*staging*) | O que irá no próximo commit | `git add arquivo` | `git status`, `git diff --staged` |
| **Histórico** (commits) | Fotografias salvas | `git commit -m "..."` | `git log` |

O **remoto** só recebe o histórico quando você faz `git push`, e só traz o histórico dos outros quando você faz `git pull`.

### 0.2 Instalar o Git

```bash
sudo apt install git        # Ubuntu / WSL
git --version               # deve imprimir algo como: git version 2.43.0
```

### 0.3 Conta no GitHub e autenticação

1. Crie uma conta em [github.com](https://github.com) (se ainda não tiver). Use um e-mail que você consulta e **lembre-se de qual**: ele será importante no passo 0.4.
2. O GitHub não aceita mais a **senha da conta** no terminal. Escolha **uma** das duas formas de autenticar:

=== "Opção A: chave SSH (recomendada)"

    Você configura uma vez e nunca mais digita senha.

    ```bash
    ssh-keygen -t ed25519 -C "seu-email@exemplo.com"   # aperte Enter em todas as perguntas
    cat ~/.ssh/id_ed25519.pub                          # copie a saída inteira
    ```

    No GitHub: foto de perfil → **Settings** → **SSH and GPG keys** → **New SSH key** → cole a chave e salve. Teste:

    ```bash
    ssh -T git@github.com
    ```

    Resposta esperada: `Hi <seu-usuario>! You've successfully authenticated, but GitHub does not provide shell access.` (isso é sucesso, apesar do "does not provide").

=== "Opção B: HTTPS com token"

    No GitHub: **Settings** → **Developer settings** → **Personal access tokens** → gere um token com permissão de escrita em repositórios (`repo`). No `git push`, quando o terminal pedir a senha, **cole o token** no lugar dela. Para não digitar sempre, instale o GitHub CLI e rode `gh auth login`.

!!! note
    Os nomes dos menus do GitHub mudam de vez em quando. Se não achar um item, procure por "SSH keys" ou "personal access token" na ajuda do GitHub.

!!! tip "Usou `===` e viu texto estranho?"
    As abas acima (`=== "..."`) exigem a extensão `pymdownx.tabbed` no `mkdocs.yml`. Sem ela, o conteúdo aparece em sequência, o que também funciona.

### 0.4 Identidade: nome e e-mail (cada integrante, uma vez)

O Git marca cada commit com um nome e um e-mail. O GitHub **só associa o commit à sua conta se o e-mail for o mesmo da conta**:

```bash
git config --global user.name  "Seu Nome Completo"
git config --global user.email "mesmo-email-da-conta-github@exemplo.com"
git config --global init.defaultBranch main
git config --global pull.rebase false      # evita um erro chato no git pull (veja 0.8)
git config --global --list                 # confira o resultado
```

!!! warning "E-mail errado = commits que \"não existem\""
    Se o e-mail do `git config` não bater com o da conta, seus commits aparecem sem o seu perfil, e na contagem de participação você parece não ter contribuído. Se preferir não expor seu e-mail, o GitHub oferece um e-mail privado (`...@users.noreply.github.com`) em **Settings → Emails**; use esse no `git config`. **Configure antes do primeiro commit**: corrigir depois é trabalhoso.

Se mexer em dois computadores (casa e laboratório), repita 0.3 e 0.4 em cada um. No computador de laboratório compartilhado, use `git config` **sem** `--global` dentro da pasta do projeto, para não deixar sua identidade para o próximo usuário.

### 0.5 Criar o repositório e colocar a dupla para trabalhar

**Só um** integrante faz isto:

1. No GitHub: botão **New repository** (ou o `+` no canto superior).
2. Nome: `ordenacao-<dupla>` (por exemplo, `ordenacao-ana-beto`).
3. Visibilidade: **Public**, ou **Private** (nesse caso, é obrigatório dar acesso ao professor, veja o passo 5).
4. Marque **Add a README file**. Isso já cria o repositório com o primeiro commit e a branch `main`. Deixe `.gitignore` e licença em "None" (você usará o `.gitignore` da Etapa 1).
5. Depois de criado: **Settings** → **Collaborators** → **Add people**. Adicione o colega **e**, se o repositório for privado, o usuário do professor (**[PREENCHER: usuário GitHub do professor]**).
6. O colega recebe um convite (por e-mail e em github.com/notifications) e precisa **aceitá-lo**. Sem isso, o `git push` dele será recusado.

**Cada um** (inclusive quem criou) clona o repositório para o seu computador. Na página do repositório, o botão verde **Code** mostra a URL; use a aba **SSH** se escolheu a Opção A, ou **HTTPS** se escolheu a B:

```bash
git clone git@github.com:<usuario>/ordenacao-<dupla>.git     # SSH
# ou: git clone https://github.com/<usuario>/ordenacao-<dupla>.git
cd ordenacao-<dupla>
git log --oneline          # mostra o commit inicial criado pelo GitHub
```

!!! danger "Não crie o repositório duas vezes"
    Um erro comum é cada integrante criar o **seu** repositório e depois tentar juntá-los. Só um cria; o outro **clona**.

### 0.6 O ciclo de trabalho (repita sempre)

```bash
git pull                       # 1. traz o que o colega fez (SEMPRE antes de começar)
# ... edite, compile, teste ...
git status                     # 2. o que mudou?
git diff                       # 3. (opcional) ver as linhas alteradas
git add src/basicos/bubble.c   # 4. escolhe o que vai no commit
git commit -m "feat: implementa bubble sort com contagem de métricas"   # 5. salva no histórico local
git push                       # 6. envia para o GitHub
```

O `git status` é seu melhor amigo. Exemplo de saída:

```text
On branch main
Changes not staged for commit:
        modified:   src/basicos/bubble.c

Untracked files:
        tests/test_basic.c
```

- **modified** = arquivo que o Git já conhece e que você alterou.
- **Untracked** = arquivo novo que o Git ainda não acompanha: precisa de `git add`.
- Se aparecer `.o`, `sort_main` ou `gmon.out` na lista, o `.gitignore` da Etapa 1 não está funcionando ou ainda não foi criado.

Outros comandos úteis:

```bash
git log --oneline              # histórico resumido (q para sair)
git log --oneline --graph      # mostra também ramificações e junções
git shortlog -sn               # quantos commits por autor (é o que a avaliação olha!)
git show                       # o que mudou no último commit
```

!!! warning "Dois erros muito comuns"
    - **`git commit` sem `-m`** abre um editor de texto (geralmente o `vim`). Para sair salvando: `Esc`, depois `:wq` e Enter. Para desistir: `Esc`, depois `:q!` e Enter.
    - **Esquecer o `git add`**: o `git commit` diz `no changes added to commit` e mostra os arquivos em vermelho. Faça o `add` e tente de novo.

### 0.7 Boas mensagens de commit

O padrão exigido é `tipo: descrição`, em uma linha, dizendo **o que o commit faz**:

| Tipo | Quando usar | Exemplo |
| --- | --- | --- |
| `feat` | Funcionalidade nova | `feat: adiciona heap sort` |
| `fix` | Correção de erro | `fix: corrige índice no partition do quicksort` |
| `test` | Testes | `test: adiciona casos n=0 e n=1` |
| `docs` | Documentação e relatórios | `docs: atualiza benchmark e uso de IA` |
| `refactor` | Reorganizar sem mudar o comportamento | `refactor: extrai função troca` |
| `style` | Formatação, sem mudar lógica | `style: corrige warnings do -Wextra` |
| `chore` | Makefile, `.gitignore`, manutenção | `chore: adiciona alvo profile ao Makefile` |

**Ruins:** `update`, `fix`, `salva tudo`, `ajustes`, `asdf`. **Boas:** descrevem a mudança e permitem entender o projeto só lendo o `git log`.

Regras práticas:

- **Um commit = uma ideia.** Implementou o Selection Sort e consertou o Makefile? São dois commits.
- **Commit pequeno e frequente** vale mais que um commit gigante na véspera. O mínimo exigido é um a cada 2-3 dias, mas o ritmo ideal é **um por etapa deste tutorial** (cada etapa traz a mensagem sugerida).
- **Só commite o que compila e passa em `make test`.** Código quebrado no `main` atrapalha o colega.
- **Nunca commite binários** (`.o`, executáveis, `gmon.out`). Eles ficam fora pelo `.gitignore`.

### 0.8 Trabalhando em dupla sem brigar com o Git

**A regra de ouro:** `git pull` **antes** de começar e `git push` **depois** de testar. Quanto menos tempo seu código fica só no seu computador, menos conflitos.

**Divida o trabalho por arquivo.** Como cada algoritmo fica em um `.c` separado, uma divisão natural é, por exemplo:

| Integrante A | Integrante B |
| --- | --- |
| Bubble, Selection, Merge | Insertion, Quick, Heap |

Os arquivos compartilhados (`tests/test_basic.c`, `Makefile`, `src/bench.c`) são onde os conflitos acontecem: **commite e envie mudanças pequenas neles, com frequência**.

!!! warning "Divisão equilibrada importa"
    Se só um integrante concentrar os commits relevantes de código, o outro grava o vídeo inteiro sozinho. Combinem a divisão desde o início e cada um commite com **a própria identidade**, no próprio computador.

#### Situação 1: o `push` é recusado

Você tenta enviar, mas o colega já enviou algo antes:

```text
 ! [rejected]        main -> main (fetch first)
error: failed to push some refs to 'github.com:...'
hint: Updates were rejected because the remote contains work that you do not
hint: have locally. ...
```

Isso **não é erro seu**: o remoto tem commits que você ainda não tem. A solução é trazê-los e tentar de novo:

```bash
git pull
git push
```

Se o `pull` reclamar `You have divergent branches and need to specify how to reconcile them`, você não fez o `git config --global pull.rebase false` do passo 0.4. Faça-o e rode o `git pull` de novo.

#### Situação 2: conflito de merge

Acontece quando os dois editaram **o mesmo trecho** (por exemplo, os dois acrescentaram uma linha no `main` do `test_basic.c`, no mesmo ponto). O `git pull` avisa:

```text
Auto-merging tests/test_basic.c
CONFLICT (content): Merge conflict in tests/test_basic.c
Automatic merge failed; fix conflicts and then commit the result.
```

Não entre em pânico: o Git pausou e está pedindo sua ajuda. Passo a passo:

1. `git status` lista o arquivo em **both modified** (ambos modificaram).
2. Abra o arquivo e procure `<<<<<<<`. Você verá:

    ```text
    <<<<<<< HEAD
        testaAlgoritmo("selection", selectionSort);
    =======
        testaAlgoritmo("insertion", insertionSort);
    >>>>>>> e48fb78...
    ```

    Entre `<<<<<<<` e `=======` está a **sua** versão; entre `=======` e `>>>>>>>`, a **do colega**.
3. Decida o resultado final e **apague as três linhas de marcação** (`<<<<<<<`, `=======`, `>>>>>>>`). Aqui, o certo é manter **as duas** chamadas:

    ```c
        testaAlgoritmo("selection", selectionSort);
        testaAlgoritmo("insertion", insertionSort);
    ```

4. **Compile e teste** (`make test`) para garantir que o resultado funciona.
5. Conclua a junção e envie:

    ```bash
    git add tests/test_basic.c
    git commit -m "merge: integra testes de selection e insertion"
    git push
    ```

Se quiser desistir e voltar ao estado anterior ao `pull`: `git merge --abort`.

Depois da junção, o `git log --oneline --graph` mostra o desvio e o encontro das duas linhas de trabalho:

```text
*   9cdf59e merge: integra testes de selection e insertion
|\
| * e48fb78 feat: implementa insertion sort
* | 6012f38 feat: implementa selection sort
|/
* c0e807e chore: estrutura inicial
```

!!! tip "Como evitar a maioria dos conflitos"
    `git pull` ao sentar para trabalhar; commits pequenos; `git push` logo após `make test` passar; avisar o colega por mensagem quando for mexer em arquivo compartilhado (Makefile, `test_basic.c`).

### 0.9 Desfazendo coisas com segurança

| Situação | Comando |
| --- | --- |
| Joguei fora minhas edições de um arquivo (**irreversível**) | `git restore arquivo.c` |
| Fiz `git add` de um arquivo por engano | `git restore --staged arquivo.c` |
| Commitei um `.o` ou `gmon.out` sem querer | `git rm --cached arquivo.o` e adicione o padrão ao `.gitignore`; depois commite |
| Um commit já enviado estava errado | `git revert <hash>` (cria um novo commit que desfaz o anterior) |

!!! danger "Evite estes comandos"
    `git push --force` e `git reset --hard` podem **apagar o trabalho do colega** ou o seu. Se alguém sugerir um deles para "resolver" um erro, peça ajuda ao professor antes.

### 0.10 Erros frequentes

| Mensagem | O que significa | O que fazer |
| --- | --- | --- |
| `Author identity unknown` / `Please tell me who you are` | Falta `user.name`/`user.email` | Passo 0.4 |
| `Permission denied (publickey)` | A chave SSH não está no GitHub | Passo 0.3, opção A |
| `Authentication failed` / pede senha e recusa | Usou a senha da conta em vez do token | Passo 0.3, opção B |
| `Permission to ... denied to <usuario>` / `403` | Convite de colaborador não aceito, ou repositório de outra pessoa | Aceite o convite (0.5) |
| `nothing to commit, working tree clean` | Não há mudanças a salvar | Normal: você já commitou tudo |
| `no changes added to commit` | Esqueceu o `git add` | `git add arquivo` e commite |
| `rejected ... (fetch first)` | O remoto tem commits que você não tem | `git pull`, depois `git push` |
| `fatal: not a git repository` | Está fora da pasta do projeto | `cd ordenacao-<dupla>` |

### Checkpoint 0

Faça este teste **com o colega**, antes de qualquer código. Ele valida tudo: convite, identidade, autenticação, `pull` e `push`.

1. **Integrante A**: edite o `README.md` incluindo os nomes completos da dupla e rode:

    ```bash
    git add README.md
    git commit -m "docs: adiciona integrantes ao README"
    git push
    ```

2. **Integrante B**: `git pull`, abra o `README.md` e confira que a edição do A chegou. Acrescente uma linha com o link do repositório e:

    ```bash
    git add README.md
    git commit -m "docs: adiciona link do repositório ao README"
    git push
    ```

3. **Integrante A**: `git pull` e confira a linha do B.
4. **Os dois** rodam:

    ```bash
    git shortlog -sn
    ```

    O resultado deve listar os **dois nomes**, cada um com pelo menos 1 commit:

    ```text
         2  Seu Nome
         1  Nome do Colega
    ```

    Quem criou o repositório pode aparecer com **um commit a mais** (o commit inicial gerado pelo próprio GitHub, que usa o nome do perfil). Isso é normal. Mas se o **nome ou e-mail de alguém** não aparecer, ou aparecer com um nome inesperado, revise o passo 0.4 **agora**, antes de commitar código.

!!! note "Esses commits de README não contam como contribuição relevante"
    Servem só para validar o ambiente. Para a avaliação, o que conta são commits de **código** (algoritmos, testes, benchmark). Por isso, combinem desde já quem implementa o quê.

---

## Etapa 1 — Setup

**Objetivo:** ter um repositório com a estrutura obrigatória, onde `make` compila e `./sort_main` roda, mesmo que os algoritmos ainda não façam nada.

### 1.1 Repositório clonado

Se você fez a **Etapa 0**, o repositório já está criado, clonado e com a identidade configurada nos dois computadores. Confirme, dentro da pasta do projeto:

```bash
git status                 # deve dizer: nothing to commit, working tree clean
git log --oneline          # deve listar os commits do Checkpoint 0
git config user.email      # deve ser o e-mail da sua conta GitHub
```

Se algum desses comandos falhar, volte à Etapa 0 antes de seguir.

### 1.2 Estrutura de diretórios

```bash
mkdir -p src/basicos src/avancados tests docs/gcov
touch docs/.gitkeep
```

O Git não versiona diretórios vazios; o `.gitkeep` garante que `docs/` exista para os alvos `make bench`, `make cppcheck` e `make profile`, que gravam arquivos lá dentro.

### 1.3 Makefile e `.gitignore`

Copie o Makefile do [apêndice](#apendice-makefile-base) para a raiz (lembre: as linhas de comando começam com **TAB**). Copie também o `.gitignore` sugerido na especificação.

### 1.4 `src/metricas.h`

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

### 1.5 `src/algoritmos.h` (protótipos)

O `main`, os testes e o benchmark precisam conhecer as seis funções. Em vez de repetir os protótipos em cada arquivo, centralize-os em um cabeçalho:

```c
/* src/algoritmos.h */
#ifndef ALGORITMOS_H
#define ALGORITMOS_H

#include "metricas.h"

void bubbleSort(int v[], int n, Metricas *m);
void selectionSort(int v[], int n, Metricas *m);
void insertionSort(int v[], int n, Metricas *m);
void mergeSort(int v[], int n, Metricas *m);
void quickSort(int v[], int n, Metricas *m);
void heapSort(int v[], int n, Metricas *m);

#endif
```

!!! note
    Este arquivo é um complemento à estrutura obrigatória. Pode ser incluído sem problema.

### 1.6 Esqueletos (*stubs*) dos seis algoritmos

O Makefile compila os seis arquivos de uma vez. Para que `make` funcione desde já, crie **esqueletos vazios** com este script (rode uma vez, na raiz do repositório):

```bash
for par in basicos/bubble:bubbleSort basicos/selection:selectionSort \
           basicos/insertion:insertionSort avancados/merge:mergeSort \
           avancados/quick:quickSort avancados/heap:heapSort; do
  arq=${par%%:*}; fn=${par##*:}
  cat > src/$arq.c <<EOF
#include "../algoritmos.h"

void $fn(int v[], int n, Metricas *m) {
    (void)v; (void)n; (void)m;  /* TODO */
}
EOF
done
```

Os `(void)` evitam o warning de parâmetro não usado (`-Wextra`) enquanto a função está vazia. **Apague-os** quando implementar o algoritmo.

### 1.7 `src/main.c` (completo)

O `main` é infraestrutura, não é o foco do trabalho: use esta versão e estude-a, pois você vai precisar explicá-la no vídeo.

```c
/* src/main.c */
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <time.h>
#include "algoritmos.h"

typedef struct {
    const char *nome;
    void (*fn)(int[], int, Metricas *);
} Algoritmo;

static const Algoritmo ALGS[] = {
    {"bubble",    bubbleSort},
    {"selection", selectionSort},
    {"insertion", insertionSort},
    {"merge",     mergeSort},
    {"quick",     quickSort},
    {"heap",      heapSort},
};
#define NUM_ALGS (sizeof ALGS / sizeof ALGS[0])

int main(int argc, char *argv[]) {
    if (argc < 2) {
        fprintf(stderr, "uso: %s <n> [bubble|selection|insertion|merge|quick|heap]\n", argv[0]);
        return 1;
    }
    int n = atoi(argv[1]);
    if (n <= 0) {
        fprintf(stderr, "n deve ser um inteiro positivo\n");
        return 1;
    }
    const char *alvo = (argc >= 3) ? argv[2] : NULL;

    int *orig = malloc((size_t)n * sizeof(int));
    int *v    = malloc((size_t)n * sizeof(int));
    if (orig == NULL || v == NULL) {
        fprintf(stderr, "falta de memória\n");
        free(orig);
        free(v);
        return 1;
    }

    srand(42);                         /* semente fixa: execuções reprodutíveis */
    for (int i = 0; i < n; i++)
        orig[i] = rand() % 1000000;

    int encontrou = 0;
    for (size_t k = 0; k < NUM_ALGS; k++) {
        if (alvo != NULL && strcmp(alvo, ALGS[k].nome) != 0)
            continue;
        encontrou = 1;

        memcpy(v, orig, (size_t)n * sizeof(int));   /* cada algoritmo ordena uma cópia */
        Metricas m = {0, 0};

        clock_t ini = clock();                       /* cronometra só a ordenação */
        ALGS[k].fn(v, n, &m);
        clock_t fim = clock();

        double ms = 1000.0 * (double)(fim - ini) / CLOCKS_PER_SEC;
        printf("%-10s n=%d  tempo=%.3f ms  comparações=%lu  trocas=%lu\n",
               ALGS[k].nome, n, ms, m.comparacoes, m.trocas);
    }

    if (!encontrou)
        fprintf(stderr, "algoritmo desconhecido: %s\n", alvo);

    free(orig);
    free(v);
    return encontrou ? 0 : 1;
}
```

### Checkpoint 1

```bash
make
./sort_main 10
```

Saída esperada (os tempos podem variar; as métricas são zero porque os algoritmos ainda estão vazios):

```text
bubble     n=10  tempo=0.001 ms  comparações=0  trocas=0
selection  n=10  tempo=0.000 ms  comparações=0  trocas=0
insertion  n=10  tempo=0.000 ms  comparações=0  trocas=0
merge      n=10  tempo=0.000 ms  comparações=0  trocas=0
quick      n=10  tempo=0.000 ms  comparações=0  trocas=0
heap       n=10  tempo=0.000 ms  comparações=0  trocas=0
```

Verifique também: `./sort_main 10 quick` imprime só uma linha, e `./sort_main 10 xyz` informa algoritmo desconhecido.

!!! success "Commit sugerido"
    ```bash
    git add .
    git commit -m "feat: estrutura inicial, Makefile, metricas.h e esqueletos dos algoritmos"
    git push
    ```

---

## Etapa 2 — Bubble Sort completo (o ciclo inteiro)

**Objetivo:** implementar um algoritmo, testá-lo e entender as métricas. Tudo o que você fizer aqui se repete nas outras etapas.

### 2.1 A ideia

Percorra o vetor comparando vizinhos; se estiverem fora de ordem, troque. Após a 1ª passada, o maior elemento está no final; após a 2ª, o segundo maior está na penúltima posição, e assim por diante. Por isso cada passada pode parar um pouco antes.

Se uma passada inteira **não trocar nada**, o vetor já está ordenado e podemos parar. Essa otimização muda o comportamento em entrada ordenada, e você vai discutir isso na análise.

### 2.2 Código

```c
/* src/basicos/bubble.c */
#include "../algoritmos.h"

void bubbleSort(int v[], int n, Metricas *m) {
    for (int i = 0; i < n - 1; i++) {
        int trocou = 0;
        /* a cada passada, os i últimos elementos já estão no lugar */
        for (int j = 0; j < n - 1 - i; j++) {
            m->comparacoes++;                 /* compara dois valores do vetor */
            if (v[j] > v[j + 1]) {
                int aux = v[j];
                v[j] = v[j + 1];
                v[j + 1] = aux;
                m->trocas++;                  /* troca de dois elementos */
                trocou = 1;
            }
        }
        if (!trocou)                          /* passada sem trocas: já ordenado */
            break;
    }
}
```

Observe o que **é** e o que **não é** contado: só a comparação `v[j] > v[j+1]` entre valores do vetor. O `j < n - 1 - i` é controle de laço e **não** conta.

### 2.3 Teste automatizado

Crie `tests/test_basic.c`. Ele já traz a infraestrutura que servirá para os seis algoritmos:

```c
/* tests/test_basic.c */
#include <stdio.h>
#include <stdlib.h>
#include "../src/algoritmos.h"

typedef void (*SortFn)(int[], int, Metricas *);

static int falhas = 0;

static int estaOrdenado(const int v[], int n) {
    for (int i = 1; i < n; i++)
        if (v[i - 1] > v[i]) return 0;
    return 1;
}

static long soma(const int v[], int n) {
    long s = 0;
    for (int i = 0; i < n; i++) s += v[i];
    return s;
}

/* Ordena uma cópia de 'orig' e confere: (1) está ordenado e (2) não perdeu elementos */
static void verifica(const char *alg, const char *caso, SortFn fn, const int orig[], int n) {
    int *v = malloc((size_t)(n > 0 ? n : 1) * sizeof(int));
    if (v == NULL) { fprintf(stderr, "falta de memória\n"); exit(2); }
    for (int i = 0; i < n; i++) v[i] = orig[i];

    long antes = soma(v, n);
    Metricas m = {0, 0};
    fn(v, n, &m);

    int ok = estaOrdenado(v, n) && soma(v, n) == antes;
    printf("%-7s  %-10s %s\n", ok ? "PASSOU" : "FALHOU", alg, caso);
    if (!ok) falhas++;
    free(v);
}

static void testaAlgoritmo(const char *nome, SortFn fn) {
    int aleatorio[]   = {5, 1, 4, 2, 8};
    int ordenado[]    = {1, 2, 3, 4, 5, 6};
    int invertido[]   = {6, 5, 4, 3, 2, 1};
    int duplicatas[]  = {3, 1, 3, 2, 1, 3, 2};
    int negativos[]   = {0, -5, 7, -1, 3};
    int unico[]       = {42};
    int vazio[1]      = {0};

    verifica(nome, "aleatório",  fn, aleatorio,  5);
    verifica(nome, "ordenado",   fn, ordenado,   6);
    verifica(nome, "invertido",  fn, invertido,  6);
    verifica(nome, "duplicatas", fn, duplicatas, 7);
    verifica(nome, "negativos",  fn, negativos,  5);
    verifica(nome, "n=1",        fn, unico,      1);
    verifica(nome, "n=0",        fn, vazio,      0);
}

/* Confere as métricas do Bubble no vetor {5,1,4,2,8}: 9 comparações e 4 trocas */
static void testaMetricasBubble(void) {
    int v[] = {5, 1, 4, 2, 8};
    Metricas m = {0, 0};
    bubbleSort(v, 5, &m);
    int ok = (m.comparacoes == 9 && m.trocas == 4);
    printf("%-7s  %-10s métricas {5,1,4,2,8}\n", ok ? "PASSOU" : "FALHOU", "bubble");
    if (!ok) {
        printf("         esperado comp=9 trocas=4, obtido comp=%lu trocas=%lu\n",
               m.comparacoes, m.trocas);
        falhas++;
    }
}

int main(void) {
    testaAlgoritmo("bubble", bubbleSort);
    testaMetricasBubble();
    /* TODO: adicione aqui as chamadas dos demais algoritmos, à medida que os implementar */

    if (falhas > 0) {
        printf("\n%d teste(s) falharam\n", falhas);
        return 1;               /* código de saída != 0: o make test também falha */
    }
    printf("\nTodos os testes passaram\n");
    return 0;
}
```

!!! tip "Por que conferir a soma?"
    Um algoritmo com bug pode devolver um vetor "ordenado" que **perdeu ou duplicou elementos** (por exemplo, `{1,1,1,1,1}`). A soma é um teste barato que pega esse tipo de erro. Um teste forte confere ordem **e** preservação dos elementos.

### 2.4 Entendendo as métricas à mão

Vamos simular `{5, 1, 4, 2, 8}`:

| Passada | Comparações | Trocas | Vetor ao fim |
| --- | --- | --- | --- |
| 1 | 4 | 3 | `1 4 2 5 8` |
| 2 | 3 | 1 | `1 2 4 5 8` |
| 3 | 2 | 0 | `1 2 4 5 8` → sem trocas, **para** |
| **Total** | **9** | **4** | |

O total de **4 trocas** é exatamente o número de **inversões** do vetor: pares fora de ordem, `(5,1) (5,4) (5,2) (4,2)`. Cada troca de vizinhos desfaz **uma** inversão. Guarde essa ideia: ela explica por que o Bubble faz tantas trocas em entrada invertida, e vai reaparecer no Insertion Sort.

### Checkpoint 2

```bash
make test
```

```text
PASSOU   bubble      aleatório
PASSOU   bubble      ordenado
PASSOU   bubble      invertido
PASSOU   bubble      duplicatas
PASSOU   bubble      negativos
PASSOU   bubble      n=1
PASSOU   bubble      n=0
PASSOU   bubble      métricas {5,1,4,2,8}

Todos os testes passaram
```

```bash
make                        # o make test NÃO religa o sort_main: sem este passo, as métricas saem zeradas
./sort_main 5000 bubble
```

!!! warning "Métricas zeradas?"
    Se `./sort_main` mostrar `comparações=0`, você esqueceu o `make`. O `make test` recompila os objetos, mas só religa o `test_basic`; o `sort_main` continua sendo o binário antigo, com o esqueleto vazio. Vale para todas as etapas: **depois de mudar um algoritmo, rode `make` antes de usar `./sort_main`**.

Você deve ver milhões de comparações (na ordem de n²/2 ≈ 12,5 milhões). Teste também `./sort_main 10000 bubble`: o número de comparações deve ficar **cerca de 4 vezes maior**. Anote isso; é a sua primeira evidência de O(n²).

!!! example "Experimento rápido (2 minutos)"
    Dobre o `n` (2000 → 4000 → 8000) e anote o tempo. Se o algoritmo é O(n²), dobrar `n` deve multiplicar o tempo por aproximadamente quanto?

!!! success "Commit sugerido"
    ```bash
    git add src/basicos/bubble.c tests/test_basic.c
    git commit -m "feat: implementa bubble sort com contagem de métricas e testes"
    git push
    ```

---

## Etapa 3 — Selection e Insertion Sort

**Objetivo:** repetir o ciclo da Etapa 2 sozinho(a): implementar, testar, conferir métricas. Cada algoritmo em seu `.c`, **substituindo o esqueleto**.

### 3.1 Selection Sort

**Contrato:** `void selectionSort(int v[], int n, Metricas *m);` em `src/basicos/selection.c`.

**Ideia:** para cada posição `i`, encontre o **menor** elemento do trecho `v[i..n-1]` e coloque-o em `i`.

**O que contar:**

- *Comparação:* cada `v[j] < v[menor]` (entre valores do vetor).
- *Troca:* cada troca de dois elementos. Troque **só se** `menor != i`; trocar um elemento com ele mesmo é trabalho inútil e infla a métrica.

??? tip "Dica 1 — estrutura"
    Dois laços aninhados. O externo escolhe a posição `i` (de `0` a `n-2`). O interno percorre `j` de `i+1` até `n-1` guardando o **índice** do menor visto até agora. Só depois de terminar o laço interno ocorre a troca.

??? tip "Dica 2 — o que esperar das métricas"
    O número de comparações **não depende da entrada**: é sempre `n(n-1)/2`. Já o número de trocas é **no máximo `n-1`**. Isso torna o Selection atraente quando trocar é caro.

**Testes:** acrescente `testaAlgoritmo("selection", selectionSort);` ao `main` de teste, e um teste de métricas.

!!! success "Checkpoint 3a — métricas para conferir"
    Para `{5, 1, 4, 2, 8}`, com a regra "troca só se `menor != i`", o resultado esperado é **10 comparações e 2 trocas**. Adapte `testaMetricasBubble` para um teste análogo do Selection.

### 3.2 Insertion Sort

**Contrato:** `void insertionSort(int v[], int n, Metricas *m);` em `src/basicos/insertion.c`.

**Ideia:** mantenha o prefixo `v[0..i-1]` ordenado. Pegue `v[i]` (a "chave"), **desloque** para a direita os elementos maiores que ela e insira a chave na lacuna.

**O que contar:**

- *Comparação:* cada comparação entre a chave e um elemento do vetor.
- *Movimentação:* Insertion e Merge contam **movimentações** (deslocamentos/cópias), não trocas de dois elementos. Escolha um critério e **registre-o** em `docs/analise.md`. Recomendação para começar: **contar cada deslocamento** (`v[j+1] = v[j]`) como 1 movimentação.

??? tip "Dica 1 — cuidado com a condição do laço"
    Uma forma comum é `while (j >= 0 && v[j] > chave)`. O problema: a comparação `v[j] > chave` **só acontece quando `j >= 0`**, e é preciso contá-la **toda vez que ela é avaliada**, inclusive na que falha e encerra o laço. Pense em como contar sem perder essa última comparação (por exemplo, reescrevendo o laço com `break`).

??? tip "Dica 2 — relação com Bubble"
    Contando só deslocamentos, o número de movimentações é igual ao número de **inversões** do vetor, o mesmo valor que as trocas do Bubble. Use isso para validar seu código.

!!! success "Checkpoint 3b — métricas para conferir"
    Para `{5, 1, 4, 2, 8}`, contando apenas deslocamentos: **7 comparações e 4 movimentações**. Se você contar também a cópia final da chave, o número de movimentações muda: **documente o critério** que usou.

!!! question "Pergunta para a análise"
    Quantas comparações o Insertion Sort faz em um vetor **já ordenado** de tamanho `n`? E o Bubble com a otimização de parada antecipada? Confira com `./sort_main` (você precisará adaptar a entrada) ou no `bench`.

!!! success "Commit sugerido (um por algoritmo)"
    ```bash
    git commit -m "feat: implementa selection sort com testes"
    git commit -m "feat: implementa insertion sort com testes"
    ```

---

## Etapa 4 — Merge, Quick e Heap Sort

**Objetivo:** os três algoritmos O(n log n). São os mais fáceis de errar nos índices; por isso **teste cada um assim que escrevê-lo**, antes de passar ao próximo.

!!! warning "Antes de começar"
    Rode `make test` a cada algoritmo novo. Um bug de índice em Merge/Quick/Heap costuma aparecer como `Segmentation fault`. Se acontecer, compile com `-g -O0` e use `gdb` ou `valgrind ./test_basic`.

### 4.1 Merge Sort

**Contrato:** `void mergeSort(int v[], int n, Metricas *m);` em `src/avancados/merge.c`.

**Requisitos obrigatórios:**

- O vetor auxiliar é alocado **dentro** da função (`malloc`), com **verificação do retorno** e `free` antes de retornar.
- Cada chamada da função pública cria o auxiliar **uma única vez**; a recursão usa uma função auxiliar `static`.

**Estrutura sugerida:**

```text
mergeSort(v, n, m):            // função pública
    aux = malloc(n * sizeof(int))
    se aux == NULL → trate o erro e retorne
    ordena(v, aux, 0, n-1, m)  // função static, recursiva
    free(aux)

ordena(v, aux, esq, dir, m):
    se esq >= dir → retorna
    meio = esq + (dir - esq) / 2
    ordena(v, aux, esq, meio, m)
    ordena(v, aux, meio+1, dir, m)
    intercala(v, aux, esq, meio, dir, m)
```

??? tip "Dica 1 — intercalação"
    Mantenha três índices: `i` na metade esquerda, `j` na direita e `k` no destino. Compare `v[i]` com `v[j]` (**isto é uma comparação**), copie o menor para `aux[k]` e avance. Depois, copie o resto da metade que sobrou.

??? tip "Dica 2 — o que contar como movimentação"
    Um critério simples e defensável: contar **cada elemento copiado de volta para `v`** (ou cada escrita em `aux`, mas **escolha um só** e documente). O importante é ser consistente.

??? tip "Dica 3 — estabilidade"
    Ao empatar (`v[i] == v[j]`), qual lado você copia primeiro? Escolher o da esquerda torna o Merge Sort **estável**. Isso é assunto do vídeo.

### 4.2 Quick Sort

**Contrato:** `void quickSort(int v[], int n, Metricas *m);` em `src/avancados/quick.c`.

**Requisito obrigatório:** o pivô **não pode ser sempre o primeiro ou o último elemento**. Use o **elemento do meio** ou a **mediana de três**, e **comente a escolha no código**.

!!! danger "Por que isso importa?"
    Com pivô fixo (primeiro/último) em entrada **já ordenada**, cada partição separa apenas 1 elemento: o algoritmo degenera para O(n²) e a recursão fica com profundidade `n`, podendo **estourar a pilha** nos tamanhos do benchmark. Você vai observar a diferença entre as três entradas (aleatória, crescente, decrescente) na análise.

**Estrutura sugerida:** função pública que chama uma `static void ordena(int v[], int baixo, int alto, Metricas *m)`.

??? tip "Dica 1 — partição"
    Duas abordagens clássicas: **Lomuto** (mais simples, um índice) e **Hoare** (dois índices que se aproximam, menos trocas). Escolha uma e domine-a; no vídeo você explicará por quê.

??? tip "Dica 2 — pivô do meio com Lomuto"
    Troque o elemento do meio com o último (`v[alto]`) **antes** de particionar, e então use o esquema normal. Lembre de contar essa troca em `m->trocas`.

??? tip "Dica 3 — duplicatas"
    Teste `{3,3,3,3,3}` e vetores com muitos valores repetidos. Se seu laço de partição usar `<=` ou `<` de forma inadequada, pode entrar em laço infinito ou degenerar.

### 4.3 Heap Sort

**Contrato:** `void heapSort(int v[], int n, Metricas *m);` em `src/avancados/heap.c`.

**Ideia:** (1) transforme o vetor em um **max-heap**; (2) repetidamente troque a raiz (maior) com o último elemento do heap, reduza o tamanho do heap e **restaure** a propriedade com uma operação de descida (*sift-down* / *heapify*).

**Índices (vetor começando em 0):**

| Nó `i` | Índice |
| --- | --- |
| Filho esquerdo | `2*i + 1` |
| Filho direito | `2*i + 2` |
| Pai | `(i - 1) / 2` |
| Último nó com filho | `n/2 - 1` |

??? tip "Dica 1 — construção do heap"
    Chame o *sift-down* para `i` de `n/2 - 1` até `0` (de trás para frente).

??? tip "Dica 2 — sift-down"
    Compare o nó com seus filhos, descubra o **maior** dos três e, se não for o próprio nó, troque e continue descendo. Cuidado para **não ler** `2*i+2` se ele for `>= tamanho do heap`.

??? tip "Dica 3 — contagem"
    Cada comparação entre dois valores do vetor conta (filho esquerdo vs. direito; maior filho vs. pai). Cada troca de dois elementos conta, tanto na descida quanto na extração da raiz.

### 4.4 Testes que todo algoritmo deve passar

Em `testaAlgoritmo` já existem os casos básicos. Acrescente estes, que costumam revelar bugs:

| Caso | Por quê |
| --- | --- |
| Vetor com `n = 2` (ordenado e invertido) | Menor caso com recursão/partição de fato |
| Todos iguais (`{7,7,7,7}`) | Pega laço infinito no Quick e instabilidade na partição |
| Vetor grande aleatório (ex.: 1000 elementos com `srand(1)`) | Pega erros que só aparecem em recursão profunda |
| Vetor grande **ordenado** e **invertido** (ex.: 10000) | Pega degeneração do Quick e estouro de pilha |

### Checkpoint 4

```bash
make test
```

Todos os seis algoritmos aparecendo com `PASSOU` em todos os casos, e:

```bash
./sort_main 50000
```

deve terminar em cerca de 10 segundos, quase todos gastos pelo Bubble (~8 s com `n = 50000`). Os avançados levam poucos milissegundos. Compare os tempos: os três básicos devem ser **ordens de grandeza** mais lentos que os três avançados. Se o Quick ou o Merge estiver tão lento quanto o Bubble, há um problema (pivô fixo? alocação dentro da recursão?).

!!! success "Commits sugeridos"
    ```bash
    git commit -m "feat: adiciona merge sort com vetor auxiliar alocado dinamicamente"
    git commit -m "feat: adiciona quick sort com pivô do meio"
    git commit -m "feat: adiciona heap sort"
    git commit -m "test: adiciona casos de borda (n=0, n=1, duplicatas, grandes)"
    ```

---

## Etapa 5 — Ferramentas de qualidade e medição

**Objetivo:** usar `cppcheck`, `gcov` e `gprof`, e saber **interpretar** a saída de cada um. O alvo do Makefile faz o trabalho; sua tarefa é entender e documentar.

### 5.1 `cppcheck` (análise estática)

Instale e rode:

```bash
sudo apt install cppcheck      # uma vez
make cppcheck
cat docs/cppcheck_bruto.txt
```

Se o arquivo estiver **vazio**, o `cppcheck` não encontrou nada. Caso contrário, cada linha tem o formato `arquivo:linha:coluna: severidade: mensagem [id]`.

Em `docs/cppcheck.md`, faça uma tabela com três colunas: **o que foi apontado**, **o que você corrigiu**, **o que manteve (e por quê)**.

!!! warning
    O repositório entregue **não pode ter** apontamentos de severidade `error`. Avisos de `style` podem ser mantidos **se justificados**.

### 5.2 `gcov` (cobertura de linhas)

O `gcov` responde: **quais linhas foram executadas pelos testes?**

```bash
make coverage
cat docs/gcov/resumo.txt
```

Você verá algo como:

```text
File 'src/basicos/bubble.c'
Lines executed:100.00% of 11
Creating 'bubble.c.gcov'
```

(os números exatos dependem do seu código). Ao final de cada grupo (`basicos` e `avancados`) o `resumo.txt` também traz uma linha de **total do grupo**, sem nome de arquivo; ela não substitui o percentual de cada arquivo que você deve registrar. Abra um arquivo `.gcov` em `docs/gcov/`:

```text
        5:   10:            if (v[j] > v[j + 1]) {
        3:   11:                int aux = v[j];
    #####:   12:                ...
```

- Número à esquerda = quantas vezes a linha executou.
- `-` = linha não executável (comentário, declaração).
- `#####` = **executável, mas nunca executada**. É o que você procura.

**Meta:** **≥ 70% de cobertura de linhas** nos seis arquivos de algoritmos (`src/basicos/*.c` e `src/avancados/*.c`). `main.c`, `bench.c` e os testes não entram na conta.

!!! tip "Se a cobertura ficar baixa"
    Procure as linhas `#####` e pergunte: **que entrada faria essa linha executar?** Geralmente é `n=0`, `n=1`, duplicatas, vetor invertido ou o caminho de erro do `malloc` (esse último pode ser aceitável de ficar sem cobertura; justifique em `docs/cobertura.md`).

Registre o percentual **de cada arquivo** em `docs/cobertura.md` e comente as linhas `#####` que restarem.

!!! info "Por que o Makefile compila em duas etapas?"
    A partir do GCC 11, compilar e ligar em **um único comando** com `--coverage` nomeia os arquivos de dados de um jeito que o `gcov` não encontra. O alvo `make coverage` já cuida disso, compilando cada `.c` em `.o` primeiro e ligando depois.

### 5.3 `gprof` (onde o tempo é gasto?)

```bash
make profile                          # n = 50000, todos os algoritmos
make profile N=2000000 ALG=quick      # só o Quick Sort, com n grande
head -30 docs/perfil_bruto.txt
```

No **flat profile**, a coluna `% time` mostra qual função consome mais tempo. Com todos os algoritmos executados sobre `n = 50000`, os três básicos devem aparecer no topo, e os avançados com tempo próximo de zero.

```text
Flat profile:

Each sample counts as 0.01 seconds.
  %   cumulative   self              self     total
 time   seconds   seconds    calls  ms/call  ms/call  name
 ...
```

!!! warning "Duas armadilhas clássicas"
    1. **Não faça profiling com `-O2`.** O compilador reorganiza e embute funções, e o resultado distorce a realidade. O alvo `make profile` já usa `-O0`.
    2. O `gprof` amostra a cada **10 ms**. Algoritmos rápidos com `n` pequeno saem com **zeros**. Aumente `N` e use `ALG=` para analisar os avançados individualmente.

!!! info "Por que `calls = 1` na função recursiva?"
    Em funções recursivas, como o `ordena` do Quick Sort, a coluna `calls` do `gprof` mostra só as chamadas **vindas de fora** da recursão (aqui, 1). Para ver as chamadas recursivas, olhe o **call graph**, que as lista à parte (`1+N`). Já funções chamadas dentro do laço, como `troca`, aparecem com milhões de chamadas.

Em `docs/perfil.md`, responda: quais funções dominam o tempo? Isso bate com a teoria? Por que o `gprof` não "enxerga" bem o Quick Sort com `n = 50000`?

!!! success "Commits sugeridos"
    ```bash
    git commit -m "docs: adiciona saída bruta do cppcheck e análise"
    git commit -m "docs: adiciona relatório gcov e análise de cobertura"
    git commit -m "docs: adiciona perfil gprof e interpretação"
    ```

---

## Etapa 6 — Benchmark e análise experimental

**Objetivo:** gerar um CSV padronizado e **explicar** o que ele mostra, comparando com a teoria.

### 6.1 O que o `bench.c` precisa fazer

Execute a matriz de experimentos abaixo e imprima **CSV** na saída padrão, com este cabeçalho:

```text
algoritmo,tipo,n,tempo_ms,comparacoes,trocas
```

| Parâmetro | Especificação |
| --- | --- |
| Tamanhos (`n`) | 1.000 · 5.000 · 10.000 · 50.000 (todos); os avançados também com 500.000 |
| Tipos de entrada | aleatório · crescente · decrescente |
| Repetições | 5 por combinação; registre a **média** dos tempos |
| Semente | `srand(42)` fixa |
| Cronômetro | `clock()` medindo **apenas a ordenação** |
| Compilação | `-O2` |

### 6.2 Esqueleto

```c
/* src/bench.c */
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <time.h>
#include "algoritmos.h"

typedef void (*SortFn)(int[], int, Metricas *);

typedef struct {
    const char *nome;
    SortFn fn;
    int avancado;            /* 1 se também roda com n = 500000 */
} Algoritmo;

static void geraEntrada(int v[], int n, const char *tipo) {
    /* TODO: preencha v conforme o tipo: "aleatorio", "crescente" ou "decrescente" */
    (void)v; (void)n; (void)tipo;
}

int main(void) {
    const Algoritmo algs[] = {
        {"bubble", bubbleSort, 0},
        /* TODO: os demais cinco algoritmos */
    };
    const char *tipos[] = {"aleatorio", "crescente", "decrescente"};
    const int tamanhos[] = {1000, 5000, 10000, 50000, 500000};
    const int REPS = 5;

    printf("algoritmo,tipo,n,tempo_ms,comparacoes,trocas\n");

    /* TODO: para cada algoritmo, tipo e tamanho (pulando 500000 nos básicos):
     *   1. gere a entrada original UMA vez;
     *   2. repita REPS vezes: copie a entrada, zere as métricas, cronometre SÓ a ordenação;
     *   3. some os tempos; no fim, imprima a MÉDIA e as métricas (de uma execução);
     */
    (void)algs; (void)tipos; (void)tamanhos; (void)REPS;
    return 0;
}
```

??? tip "Dica 1 — o que cronometrar"
    O relógio (`clock()`) deve cercar **só** a chamada do algoritmo. A **cópia** do vetor original e a **geração** da entrada ficam **fora**.

??? tip "Dica 2 — as métricas variam entre as 5 repetições?"
    Para a mesma entrada, comparações e trocas são **determinísticas**, então basta registrar uma execução. Só o **tempo** varia, daí a média.

??? tip "Dica 3 — crescente e decrescente"
    São triviais de gerar com um laço (`v[i] = i` e `v[i] = n - i`). O aleatório usa `rand()` com a semente fixa, para ser reprodutível.

### 6.3 Rodando

```bash
make bench          # grava docs/resultados.csv (cerca de 3 minutos, quase todos em Bubble e Selection com n = 50000)
head docs/resultados.csv
```

### Checkpoint 6

- O CSV tem o cabeçalho certo e **nenhuma linha** de Bubble/Selection/Insertion com `n=500000`.
- Total de linhas: 3 tipos × (3 básicos × 4 tamanhos + 3 avançados × 5 tamanhos) = **81 linhas** de dados, mais o cabeçalho.
- `wc -l docs/resultados.csv` deve dar **82**.

### 6.4 Escrevendo `docs/analise.md`

Estrutura obrigatória:

1. **Tabela-resumo** (pode ser gerada do CSV) e, se quiserem, gráficos.
2. **O critério que vocês adotaram para contar "trocas"** em cada algoritmo.
3. **Comparação com a teoria.** Perguntas-guia:
    - Por que Bubble/Selection/Insertion crescem como O(n²) e Merge/Quick/Heap como O(n log n)? Mostre isso nos números: quando `n` passa de 10.000 para 50.000 (5×), por quanto o tempo é multiplicado?
    - O que acontece com o **Insertion Sort em entrada já ordenada**?
    - O que muda no **Bubble Sort** e no **Quick Sort** entre os três tipos de entrada?
    - O Selection Sort faz quantas trocas, comparado ao Bubble? E o tempo?

    !!! tip "Uma surpresa a esperar"
        Com pivô do meio, o Quick Sort em entrada **crescente ou decrescente** tende a ficar **mais rápido** que em entrada aleatória (por volta de 4 a 5 vezes mais rápido em `n = 50000`), e o Heap Sort também melhora um pouco. Não é um erro: o pivô do meio é ótimo nesses casos e o padrão previsível ajuda o processador. Explique isso em vez de esperar que "ordenado" seja sempre o pior caso.

4. **Resultados que NÃO bateram com a teoria**, com uma hipótese. Exemplos de coisas a investigar: um algoritmo O(n log n) mais lento que outro com a mesma complexidade (constantes, cache, alocação); o Quick e o Heap com tempos tão diferentes apesar de ambos serem O(n log n) (compare as comparações e as trocas de cada um no CSV); o tempo do Quick em entrada ordenada (depende do pivô); o ponto em que o Insertion deixa de ser competitivo (nos tamanhos do benchmark ele já perde do Merge com `n = 1.000`: por que um algoritmo O(n²) ainda é usado como parte de ordenações reais?).

!!! tip "Como calcular a razão entre tempos"
    Para checar crescimento, divida o tempo de `n = 50000` pelo de `n = 10000`. Em O(n²) espere algo perto de 25; em O(n log n), algo entre 5 e 6 (os 5× de `n` vezes a razão dos logaritmos, cerca de 1,18). Desvios são justamente o que você deve explicar no item 4: por exemplo, em um teste deste roteiro o Bubble com entrada aleatória ficou perto de **33×**, bem acima dos 25× esperados. Por quê? Levante hipóteses (padrão de desvios do `if`, cache) e, se puder, teste uma delas.

!!! success "Commits sugeridos"
    ```bash
    git commit -m "feat: adiciona benchmark que gera CSV"
    git commit -m "docs: adiciona resultados.csv e análise experimental"
    ```

---

## Etapa 7 — Documentação final, vídeo e entrega

### 7.1 `docs/uso-ia.md`

Preencha **ao longo do trabalho**, não só no fim. Registre prompts reais, o que a IA gerou e **o que vocês adaptaram ou descobriram estar errado**. Use o modelo da especificação. O uso de IA **não penaliza**; a falta de transparência e a incapacidade de explicar o código, sim.

### 7.2 `README.md`

Deve conter: nomes dos integrantes, como compilar (`make`), como testar (`make test`), como rodar (`./sort_main <n> [algoritmo]`) e links para a análise em `docs/`.

### 7.3 Teste de ponta a ponta

Antes do `push` final, num diretório limpo:

```bash
make clean && make all && make test && make coverage && make bench && make profile && make cppcheck
```

Tudo precisa terminar sem erro. Confira que **não há warnings** de `-Wall -Wextra` na compilação.

### 7.4 Roteiro do vídeo (até 20 min)

Antes de gravar, **ensaiem**: cada integrante precisa ser capaz de compilar, executar e explicar qualquer parte.

| # | Seção | O que mostrar | Tempo |
| --- | --- | --- | --- |
| 1 | Apresentação | Integrantes, turma, objetivo | ~1 min |
| 2 | Implementação | Estrutura de arquivos; um trecho relevante de cada algoritmo | ~3 min |
| 3 | Execução ao vivo | `make`, `make test`, `./sort_main` | ~2 min |
| 4 | Análise experimental | Tabela e comparação com a teoria | ~3 min |
| 5 | Ferramentas | `gcov`, `gprof`, `cppcheck` | ~3 min |
| 6 | Conclusões | Melhor algoritmo, o que bateu e não bateu com a teoria, dificuldades | ~1 min |

!!! warning "Slides sozinhos não bastam"
    É obrigatório alternar para o terminal e o editor, mostrando código e execução real.

### 7.5 Conferir o histórico, criar a tag e entregar

**Antes de criar a tag**, confira o que o professor vai ver:

```bash
git status                   # deve estar limpo (nada a commitar)
git pull && git push         # local e GitHub sincronizados
git shortlog -sn             # os dois nomes, com commits de código
git log --oneline            # mensagens no padrão tipo: descrição
```

Abra também a página do repositório no GitHub (aba **Commits**) e confirme: os arquivos de `docs/` estão lá (relatórios **versionados**), **não há** `.o`/executáveis/`gmon.out`, e o professor tem acesso se o repositório for privado.

Uma **tag** marca um commit como "a versão entregue". É a fotografia oficial da entrega, que não muda mesmo que alguém commite depois:

```bash
git tag -a v1.0 -m "Entrega final do trabalho de ordenação"
git push origin main --tags          # envia os commits E a tag
git tag                              # lista as tags locais (deve mostrar v1.0)
```

No GitHub, a tag aparece em **Releases/Tags** na página do repositório. Se precisar corrigir algo **depois** da tag, commite normalmente e, com autorização do professor, crie uma nova tag (`v1.1`), em vez de apagar a anterior.

No SIGAA, em **um único comentário por dupla**: nomes completos, link do repositório e link do vídeo no YouTube.

### Checklist final

- [ ] Seis algoritmos, cada um em seu `.c`, com os nomes de função exigidos
- [ ] Merge com `malloc` dentro da função, com verificação e `free`
- [ ] Quick com pivô do meio ou mediana de três, **comentado no código**
- [ ] `src/main.c` aceita `<n> [algoritmo]` e usa `srand(42)`
- [ ] `make test` passa e retorna código ≠ 0 quando há falha
- [ ] `gcov` ≥ 70% de linhas nos seis arquivos; `cppcheck` sem `error`
- [ ] `docs/` com todos os arquivos obrigatórios, **versionados**
- [ ] Commits regulares, com mensagens no padrão `tipo: descrição`
- [ ] `docs/uso-ia.md` preenchido
- [ ] Vídeo publicado e link enviado no SIGAA junto com o do repositório
- [ ] Tag `v1.0` criada e enviada

---

## Trabalho Prático

Esta seção reúne **o que será exigido e avaliado**. O "como fazer" de cada item está nas Etapas 0 a 7 acima; aqui ficam os requisitos, a estrutura obrigatória, a rubrica e a entrega.

**Formato:** em dupla · **Linguagem:** C11 (`-std=c11`) · **Prazo sugerido:** 2 semanas, tempo para a dupla se organizar, e não 2 semanas de trabalho diário. O trabalho não é feito em uma única sessão; comece cedo.

O trabalho é a avaliação prática da disciplina e tem **duas entregas indissociáveis**: o **código** (repositório GitHub) e um **vídeo de defesa obrigatório**, no qual a dupla demonstra e explica o que implementou. O vídeo substitui a avaliação escrita da Terceira Unidade.

### Objetivos de aprendizagem

1. Implementar **3 algoritmos básicos** e **3 avançados** de ordenação em C.
2. Praticar **controle de versão** com Git em equipe, com histórico incremental.
3. Aplicar **qualidade e medição**: testes, cobertura (`gcov`), análise estática (`cppcheck`) e profiling (`gprof`).
4. Produzir **documentação técnica** e **análise empírica**, confrontando as medições com a teoria de complexidade.
5. Praticar **transparência acadêmica** no uso de IA, validando e explicando todo o código entregue.
6. Defender oralmente as escolhas de implementação e os resultados.

### Organização da dupla

Existem **duas formas aceitas** de organizar o desenvolvimento e a entrega:

1. **Formato ideal (equilibrado):** ambos contribuem com commits frequentes e relevantes de código, e **ambos** participam do vídeo, dividindo a explicação.
2. **Formato assimétrico:** se um integrante tiver participação mínima nos commits (por exemplo, apenas um ou dois commits triviais, como editar o `README.md`), **esse integrante assume integralmente a apresentação do vídeo**, explicando sozinho todo o código, as ferramentas e os resultados.

**Regra única:** em qualquer formato, **ambos** devem conseguir compilar, executar e explicar qualquer parte do código. O vídeo verifica o domínio real sobre o que foi entregue.

### Requisitos de implementação

| Categoria | Algoritmos | Assinatura |
| --- | --- | --- |
| **Básicos** | Bubble Sort, Selection Sort, Insertion Sort | `void algoritmoSort(int v[], int n, Metricas *m);` |
| **Avançados** | Merge Sort, Quick Sort, Heap Sort | `void algoritmoSort(int v[], int n, Metricas *m);` |

- Cada algoritmo em **arquivo `.c` separado**, com as funções `bubbleSort`, `selectionSort`, `insertionSort`, `mergeSort`, `quickSort` e `heapSort`.
- A estrutura `Metricas` fica em `src/metricas.h` (veja a Etapa 1.4). O ponteiro `m` **nunca é `NULL`**: quem chama sempre passa uma estrutura zerada.
- **Comparação:** conta apenas comparações entre valores do vetor (não conte o `i < n` dos laços).
- **Trocas:** Bubble, Selection, Quick e Heap contam trocas de dois elementos; Insertion e Merge contam **movimentações** (deslocamentos e cópias). O critério adotado deve ser registrado em `docs/analise.md`.
- **Merge Sort:** vetor auxiliar alocado **dentro** da função (`malloc`), com verificação do retorno e `free` antes de retornar.
- **Quick Sort:** o pivô **não pode ser sempre o primeiro ou o último elemento**. Use o elemento do meio ou a mediana de três, e **comente a escolha no código**.
- **`src/main.c`:** aceita `./sort_main <n> [bubble|selection|insertion|merge|quick|heap]`, gera vetor aleatório com `srand(42)`, e imprime tempo (`clock()`), comparações e trocas. Sem o 2º argumento, executa os seis algoritmos, cada um sobre uma **cópia** do mesmo vetor.
- **Compilação:** via `make`, com `-std=c11`. Zero erros; warnings de `-Wall -Wextra` devem ser corrigidos (ou justificados no `README`).

### Requisitos de teste, qualidade e medição

| Item | Exigência |
| --- | --- |
| **Testes** | `tests/test_basic.c` com **no mínimo 2 casos por algoritmo** (ex.: aleatório, ordenado, invertido). Cada teste imprime `PASSOU` ou `FALHOU`, e o programa **termina com código ≠ 0** se algum falhar. Comando: `make test`. |
| **Cobertura** | **≥ 70% de linhas** nos **seis arquivos de algoritmos** (`src/basicos/*.c` e `src/avancados/*.c`), medida com `make coverage`. `main.c`, `bench.c` e os testes não entram na conta. Percentual de cada arquivo em `docs/cobertura.md`. |
| **Análise estática** | `make cppcheck` grava `docs/cppcheck_bruto.txt`. Em `docs/cppcheck.md`: o que foi apontado, corrigido e mantido (com justificativa). **Sem apontamentos `error`.** |
| **Profiling** | `make profile` grava `docs/perfil_bruto.txt` (compilado com `-O0`). Interpretação em `docs/perfil.md`. |
| **Benchmark** | `make bench` grava `docs/resultados.csv` (formato e parâmetros na Etapa 6). Análise em `docs/analise.md`. |

**Parâmetros do benchmark:** tamanhos 1.000 · 5.000 · 10.000 · 50.000 (todos); os avançados também com 500.000 (**não** rode os básicos com 500.000) · entradas aleatória, crescente e decrescente · 5 repetições por combinação, registrando a **média** dos tempos · `srand(42)` · `clock()` medindo **só a ordenação** · compilação com `-O2`.

### Fluxo Git/GitHub

Não há exigência de *branch protection*, revisão obrigatória ou bloqueio de `push` direto na `main`. O foco é o **hábito de versionar incrementalmente** (veja a Etapa 0):

- **Commits frequentes:** mínimo 1 a cada 2-3 dias de trabalho.
- **Mensagens descritivas** no padrão `tipo: descrição`. Evite `update`, `fix`, `salva tudo`.
- **Participação visível:** o `git log` define o formato de defesa da dupla. Cada integrante deve configurar `user.name` e `user.email` na própria máquina.
- **Branches e PRs:** opcionais.

### Política de uso de IA e transparência

**Permitido e incentivado:** gerar trechos para estudo, pedir explicação de erros, revisar formatação, sugerir nomes de variáveis, usar como tutor.

**Não permitido:** entregar código completo sem entender, testar ou adaptar; ocultar prompts ou ferramentas utilizadas.

Crie `docs/uso-ia.md` neste modelo:

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
(Descreva onde a IA ajudou, onde atrapalhou, e o que a dupla aprendeu ao questionar as respostas.)
```

!!! note "Avaliação"
    O uso de IA **não penaliza**. Falta de transparência, código não testado ou dupla incapaz de explicar a entrega **penaliza**.

### Estrutura obrigatória do repositório

```text
ordenacao-<dupla>/
├── README.md                  # Instruções de build, membros, links
├── Makefile                   # Modelo no apêndice
├── .gitignore
├── src/
│   ├── basicos/               # bubble.c, selection.c, insertion.c
│   ├── avancados/             # merge.c, quick.c, heap.c
│   ├── metricas.h             # struct Metricas
│   ├── algoritmos.h           # protótipos (complemento sugerido neste tutorial)
│   ├── main.c                 # Ponto de entrada (aceita n e algoritmo)
│   └── bench.c                # Benchmark que gera o CSV
├── tests/
│   └── test_basic.c           # Testes automatizados
└── docs/
    ├── resultados.csv         # Saída do make bench
    ├── analise.md             # Análise experimental x teoria
    ├── cobertura.md           # Relatório gcov + análise
    ├── gcov/                  # Saída do make coverage (resumo.txt e .gcov)
    ├── perfil_bruto.txt       # Saída do gprof (make profile)
    ├── perfil.md              # Interpretação do profiling
    ├── cppcheck_bruto.txt     # Saída do make cppcheck
    ├── cppcheck.md            # O que foi apontado, corrigido e mantido
    └── uso-ia.md              # Transparência de prompts e validação
```

`.gitignore` sugerido (binários e intermediários ficam de fora; os relatórios em `docs/` **devem** ser versionados):

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

### Defesa em vídeo (obrigatória)

- **Formato:** gravação de **até 20 minutos**, publicada no YouTube (público ou não listado). O link vai no SIGAA junto com a entrega do repositório.
- **Não são aceitas apresentações só com slides:** é obrigatório alternar para o terminal e o editor, mostrando código e execução real.
- **Apresentadores:** se o histórico de commits for equilibrado, **os dois** apresentam; se um integrante tiver participação mínima ou superficial, **ele apresenta o vídeo completo sozinho**, demonstrando domínio de todo o código do colega.
- **Estrutura e tempos:** veja a tabela da Etapa 7.4.

### Avaliação

**Rubrica do código (100 pts):**

| Critério | O que será observado | Peso |
| --- | --- | --- |
| **Funcionalidade** | 6 algoritmos compilam, executam e ordenam corretamente, com contagem de comparações e trocas | 25 |
| **Histórico Git** | Commits regulares, mensagens claras, evolução visível (e definição do formato de defesa da dupla) | 20 |
| **Qualidade e testes** | Testes passam, `gcov` ≥ 70% de linhas nos 6 arquivos, `cppcheck` sem erros | 20 |
| **Documentação e análise** | `analise.md`, `cobertura.md`, `perfil.md`, `cppcheck.md`, `resultados.csv` e `README` funcional | 20 |
| **Colaboração e IA** | Divisão equilibrada (ou compensação na apresentação), `docs/uso-ia.md` preenchido, código explicável | 15 |

!!! warning
    Repositórios com 1-2 commits no dia da entrega, mensagens vazias ou código que a dupla não consegue modificar serão penalizados em Git/Colaboração.

**Rubrica do vídeo (4,0 pts, substitui a avaliação escrita da Unidade 3):**

| Critério | O que será observado | Pontos |
| --- | --- | --- |
| **Organização e clareza** | Apresentação didática, áudio nítido, divisão do tempo e boa estrutura | 1,0 |
| **Demonstração do código** | Explicação concisa da implementação e compilação/execução correta | 1,0 |
| **Análise de resultados** | Discussão fundamentada nas medições, comparando os algoritmos | 1,0 |
| **Domínio e participação** | Apresentação conforme o histórico Git, demonstrando domínio total do código | 1,0 |

!!! note "Composição da nota final"
    - **Vídeo:** 4,0 pontos, na Terceira Unidade.
    - **Código:** a pontuação de 0 a 100 é convertida para **[PREENCHER: escala/peso conforme o plano de ensino]**.
    - **Individualização:** a nota é da dupla, mas pode ser **ajustada individualmente** quando o histórico Git, a participação no vídeo ou a arguição indicarem contribuição desigual.
    - **Atraso:** **[PREENCHER: política, ex.: −10% da nota por dia de atraso]**.

### Entrega do projeto

1. **Repositório GitHub** atualizado (público, ou privado com acesso liberado ao professor: usuário **[PREENCHER: usuário GitHub do professor]**).
2. **Tag `v1.0`** na versão final (comandos na Etapa 7.5).
3. **Envio via SIGAA**, em **um único comentário por dupla**, com: nomes completos dos integrantes, link do repositório e link do vídeo no YouTube.

!!! tip "Teste final antes do `push`"
    `make clean && make all && make test && make coverage && make bench && make profile && make cppcheck` (o `make bench` leva cerca de 3 minutos).

## Apêndice: Makefile Base {#apendice-makefile-base}

Copie o arquivo abaixo para a **raiz** do repositório, com o nome `Makefile`. Ele inclui os alvos de compilação, testes, benchmark, `gcov`, `gprof` e `cppcheck`.

!!! warning "Makefile exige TAB"
    As linhas de comando das regras precisam começar com o caractere **TAB**, não com espaços. Se ao copiar o arquivo aparecer `missing separator`, é isso. Em editores como VS Code, confira se a indentação das linhas de comando não foi convertida em espaços (canto inferior direito: "Tab Size" / "Spaces").

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

### O que cada alvo faz

| Comando | O que faz | Gera / altera | Observações |
| --- | --- | --- | --- |
| `make` ou `make all` | Compila o programa principal | `sort_main` | Use **depois de alterar um algoritmo** e antes de rodar `./sort_main` |
| `make test` | Compila e roda os testes | `test_basic` | Retorna código ≠ 0 se algum teste falhar. **Não** atualiza o `sort_main` |
| `make bench` | Compila e roda o benchmark | `docs/resultados.csv` | Leva cerca de 3 minutos; usa `-O2` |
| `make coverage` | Recompila com `--coverage -O0`, roda os testes e gera os relatórios do `gcov` | `docs/gcov/` (`resumo.txt` e `.gcov`) | **Executa `clean` antes** e remove os objetos no final; rode `make` de novo depois |
| `make profile` | Compila com `-pg -O0`, roda e gera o perfil do `gprof` | `docs/perfil_bruto.txt` | Aceita `N=` e `ALG=`; veja abaixo |
| `make cppcheck` | Roda a análise estática | `docs/cppcheck_bruto.txt` | Arquivo **vazio** = nenhum apontamento |
| `make clean` | Apaga binários, objetos e arquivos do `gcov`/`gprof` | (remove) | Use quando algo estranho acontecer após mudar um `.h` |

**Parâmetros do `make profile`:**

```bash
make profile                          # n = 50000, todos os algoritmos
make profile N=200000 ALG=merge       # só o Merge Sort, com n maior
```

**Como o Makefile funciona (para explicar no vídeo):**

- `CFLAGS = -Wall -Wextra -std=c11 -O2` vale para tudo, **exceto** `coverage` (que o substitui por `--coverage -g -O0`) e `profile` (que usa `-pg -g -O0`). Por isso o `gprof` e o `gcov` não usam `-O2`.
- `DEPFLAGS = -MMD -MP` faz o compilador gerar arquivos `.d` com as dependências de cada `.o` nos `.h`. Assim, se você alterar `metricas.h`, o `make` recompila o que for preciso. A linha `-include $(ALL_OBJS:.o=.d)` no final lê esses arquivos.
- A regra `%.o: %.c` ensina o `make` a compilar **qualquer** `.c` em `.o` (`$<` é o arquivo de entrada, `$@` é o de saída).
- `main.c`, `test_basic.c` e `bench.c` compartilham os **mesmos seis arquivos de algoritmos** (`ALG_SRCS`) e cada um gera o seu executável.
- `.PHONY` declara alvos que são **ações**, não arquivos (`test`, `clean`...), para que o `make` os execute sempre.

!!! tip "`make: Nothing to be done` ou `make: 'sort_main' is up to date`?"
    O `make` só recompila o que mudou. Se você acabou de editar um arquivo e ele diz que nada há a fazer, confira se **salvou** o arquivo e se o `Makefile` lista esse `.c` em `ALG_SRCS`.
