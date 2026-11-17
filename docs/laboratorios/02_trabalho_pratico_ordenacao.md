# Laboratório: Algoritmos de Ordenação em C — Passo a Passo

Este tutorial é o **caminho** para chegar ao Trabalho Prático de ordenação. A especificação (o que entregar, rubrica, prazos) está no final do capítulo, em [Trabalho Prático](#trabalho-pratico). Aqui você aprende **como construir** cada peça, em etapas pequenas, cada uma com um **checkpoint** (um comando e o resultado esperado) e um **commit sugerido**.

!!! info "Como usar este tutorial"
    - Nunca usou Git ou GitHub? Comece pela **Etapa 0**: ela explica conta, criação do repositório, commits e trabalho em dupla.
    - Siga as etapas **na ordem**. Só avance quando o checkpoint passar.
    - Faça **um commit ao fim de cada etapa**. É isso que cria o histórico que será avaliado.
    - A etapa 2 (Bubble Sort) é um exemplo **completo** do ciclo: código, teste, métricas. As demais etapas dão o contrato, dicas e os casos de teste; **o código é seu**.
    - Blocos recolhidos (`???`) contêm dicas. Tente primeiro sem abri-los.

| Etapa | Foco | Semana sugerida |
| --- | --- | --- |
| 0 | Git e GitHub: conta, repositório, commits, trabalho em dupla | antes da semana 1 |
| 1 | Setup do projeto: Makefile, `metricas.h`, esqueletos | 1 |
| 2 | Bubble Sort completo (código + teste + métricas) | 1 |
| 3 | Selection e Insertion Sort | 1–2 |
| 4 | Merge, Quick e Heap Sort | 2–3 |
| 5 | Ferramentas: `cppcheck`, `gcov`, `gprof` | 3 |
| 6 | Benchmark e análise experimental | 3–4 |
| 7 | Documentação, vídeo e entrega | 4 |

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

Os `(void)` evitam o warning de parâmetro não usado (`-Wextra`) enquanto a função está vazia. **Apague-os** quando implementar o 
