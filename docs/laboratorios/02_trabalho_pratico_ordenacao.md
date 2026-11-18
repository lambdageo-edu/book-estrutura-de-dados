# Laboratório: Algoritmos de Ordenação em C — Passo a Passo

Este tutorial é o **caminho** para chegar ao Trabalho Prático de ordenação. A especificação (o que entregar, rubrica e forma de envio) está no final do capítulo, em [Trabalho Prático](#trabalho-pratico). Aqui você aprende **como construir** cada peça, em etapas pequenas, cada uma com um **checkpoint** (um comando e o resultado esperado) e um **commit sugerido**.

!!! warning "Ambiente obrigatório: Linux (nativo, WSL ou VirtualBox)"
    Este tutorial e todo o fluxo de compilação, testes e análise foram elaborados para rodar em **Linux**. 
    
    Tentar executar este laboratório diretamente no **Windows** exigirá diversas adaptações manuais nos comandos e no `Makefile`, além de **algumas ferramentas simplesmente não funcionarem** (como o `gprof`, o `AddressSanitizer` e scripts de terminal), obrigando você a buscar substitutos por conta própria — ou impedindo que realize parte dos experimentos exigidos.

    **Não tem Linux instalado?** Você tem duas alternativas gratuitas e práticas:
    
    - **WSL (Windows Subsystem for Linux) — Recomendado:** mais leve, rápido e integrado ao Windows e ao VS Code (veja o passo a passo de instalação na **Etapa 0.2**).
    - **Máquina Virtual (VirtualBox):** instale uma distribuição como **Ubuntu** ou **Linux Mint** para ter um ambiente completo e isolado.

    **Por que aprender isso agora?** A grande maioria dos servidores, aplicações em nuvem e sistemas web roda sobre **Linux**. Dominar o terminal e as ferramentas desse ecossistema não é apenas um requisito desta disciplina, mas um investimento essencial para a sua formação e carreira na computação.


!!! info "Como usar este tutorial"
    - Nunca usou Git ou GitHub? Comece pela **Etapa 0**: ela explica conta, criação do repositório, commits e trabalho em dupla.
    - Siga as etapas **na ordem**. Só avance quando o checkpoint passar.
    - Faça **um commit ao fim de cada etapa**. É isso que cria o histórico que será avaliado.
    - A etapa 2 (Bubble Sort) é um exemplo **completo** do ciclo: código, teste, métricas. As demais etapas dão o contrato, dicas e os casos de teste; **o código é seu**.
    - Blocos recolhidos (`???`) contêm dicas. Tente primeiro sem abri-los.

!!! info "Sobre o prazo de 2 semanas"
    O prazo sugerido é de **2 semanas**, mas isso **não quer dizer 2 semanas de trabalho todos os dias**. Ninguém tem tempo livre diariamente, e tudo bem. O prazo existe para a dupla **se organizar**.

    Este **não é um trabalho para fazer de uma vez** (nem em uma hora, nem em uma noite). Ele tem partes que levam tempo: implementar e testar seis algoritmos, rodar o benchmark, escrever a análise, ensaiar e gravar o vídeo. Combinem **quando** cada um consegue trabalhar, dividam as tarefas e **comecem cedo**, em sessões curtas, para não deixar tudo para o último dia.

!!! info "O repositório é a sua vitrine"
    O repositório deste trabalho é **público**, de propósito: ele serve como **vitrine** do que você sabe fazer. Professores, colegas e, no futuro, recrutadores e orientadores podem abri-lo e ver seu código, seu histórico de commits, sua documentação e sua análise. Por isso, capriche desde o começo: mensagens de commit claras, um `README` que explique o projeto e documentação bem escrita. Dicas na Etapa 0.5 e na Etapa 7.

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

!!! info "Terminal e ferramentas"
    Todos os comandos deste tutorial assumem um terminal Linux (nativo, via **WSL** no Windows ou via máquina virtual). No macOS, o `gcc` do Xcode/Homebrew funciona para compilar, mas ferramentas como o `gprof` podem se comportar de forma diferente.

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

### 0.2 Instalar as ferramentas

Instale de uma vez o Git, o compilador e as ferramentas que o laboratório usa (`build-essential` traz `gcc`, `make`, `gcov` e `gprof`; o `cppcheck` é a análise estática da Etapa 5):

```bash
sudo apt update
sudo apt install git build-essential cppcheck
```

Confira que tudo foi instalado: cada comando abaixo deve imprimir uma versão. Se algum disser `command not found`, repita a instalação.

```bash
git --version
gcc --version
make --version
gprof --version
gcov --version
cppcheck --version
```

!!! tip "Windows: como ter um terminal Linux (WSL)"
    No PowerShell (como administrador), rode `wsl --install` e reinicie o computador se for pedido. Depois abra o aplicativo **Ubuntu** e use os comandos acima.

    **Clone o repositório dentro do Linux** (por exemplo, em `~/`), e não em pastas do Windows como `/mnt/c/...`: é mais rápido e evita problemas de permissão e de fim de linha. No VS Code, a extensão **WSL** permite abrir a pasta do Linux (rode `code .` dentro dela).

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
    Se o e-mail do `git config` não bater com o da conta, seus commits aparecem sem o seu perfil, e na contagem de participação você parece não ter contribuído. **Atenção:** como o repositório é público, o e-mail usado nos commits fica **visível para qualquer pessoa** que olhar o histórico. Se não quiser expor seu e-mail pessoal, use o e-mail privado que o GitHub oferece (`...@users.noreply.github.com`), em **Settings → Emails** (marque "Keep my email addresses private"); coloque esse e-mail no `git config`, e os commits continuam associados à sua conta. **Configure antes do primeiro commit**: corrigir depois é trabalhoso.

Se mexer em dois computadores (casa e laboratório), repita 0.3 e 0.4 em cada um. No computador de laboratório compartilhado, use `git config` **sem** `--global` dentro da pasta do projeto, para não deixar sua identidade para o próximo usuário.

### 0.5 Criar o repositório e colocar a dupla para trabalhar

**Só um** integrante faz isto:

1. No GitHub: botão **New repository** (ou o `+` no canto superior).
2. Nome: `ordenacao-<dupla>` (por exemplo, `ordenacao-ana-beto`).
3. Visibilidade: **Public**. O repositório é público de propósito, pois é a **vitrine** do seu trabalho (veja o quadro abaixo).
4. Marque **Add a README file**. Isso já cria o repositório com o primeiro commit e a branch `main`. Deixe `.gitignore` e licença em "None" (você usará o `.gitignore` da Etapa 1).
5. Depois de criado: **Settings** → **Collaborators** → **Add people**. Adicione o colega. Como o repositório é público, o professor e qualquer pessoa conseguem ver o código sem precisar de convite.
6. O colega recebe um convite (por e-mail e em github.com/notifications) e precisa **aceitá-lo**. Sem isso, o `git push` dele será recusado.

**Cada um** (inclusive quem criou) clona o repositório para o seu computador. Na página do repositório, o botão verde **Code** mostra a URL; use a aba **SSH** se escolheu a Opção A, ou **HTTPS** se escolheu a B:

```bash
git clone git@github.com:<usuario>/ordenacao-<dupla>.git     # SSH
# ou: git clone https://github.com/<usuario>/ordenacao-<dupla>.git
cd ordenacao-<dupla>
git log --oneline          # mostra o commit inicial criado pelo GitHub
```

!!! tip "Seu repositório é uma vitrine"
    Por ser público, ele aparece no perfil de quem o criou e pode ser visto por qualquer pessoa. Alguns cuidados:

    - **Capriche no que aparece:** commits com mensagens descritivas, `README` claro, `docs/` organizado. É o primeiro contato de quem abre o projeto.
    - **Nunca commite dados pessoais ou segredos:** senhas, tokens de acesso, chaves privadas, documentos pessoais. Tudo o que entra no histórico do Git fica visível, mesmo que o arquivo seja apagado depois.
    - **Mostre o projeto:** no seu perfil do GitHub, você pode **fixar** (*pin*) este repositório para que ele apareça no topo.

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

!!! info "Arquivos que os dois vão editar neste laboratório"
    Alguns arquivos mudam em várias etapas, e é neles que os conflitos acontecem:

    | Arquivo | Quando muda | Como evitar conflito |
    | --- | --- | --- |
    | `Makefile` | Etapas 2, 5 e 6 | Um de vocês faz as alterações da etapa, comita e dá `push` na hora; o outro faz `git pull` antes de mexer no arquivo. |
    | `tests/test_basic.c` | A cada algoritmo implementado | Cada um acrescenta as chamadas dos **seus** algoritmos usando os auxiliares que já existem (`testaAlgoritmo`, `testaMetricas`); **não criem funções novas com o mesmo nome**. Se der conflito no `main`, mantenha as duas versões, como no exemplo abaixo. |
    | `src/bench.c` | Etapa 6 | Um escreve e o outro revisa e testa. |

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
mkdir -p src/basicos src/avancados tests docs
touch docs/.gitkeep
```

O Git não versiona diretórios vazios; o `.gitkeep` garante que `docs/` exista para os alvos `make bench`, `make cppcheck` e `make profile`, que você vai criar nas Etapas 5 e 6 e que gravam arquivos lá dentro.

### 1.3 Makefile inicial e `.gitignore`

O `make` compila o projeto com um único comando. Ele lê um arquivo chamado `Makefile`, que descreve **o que construir** e **como**. Você **não vai copiar um Makefile pronto**: vai construí-lo aos poucos, acrescentando cada alvo na etapa em que ele passa a ser necessário. Agora, só o essencial: **compilar o programa** e **limpar** o que foi gerado.

Crie o arquivo `Makefile` (sem extensão) na raiz do repositório:

```make
CC = gcc
WARN = -Wall -Wextra -std=c11
CFLAGS = $(WARN) -O2
DEPFLAGS = -MMD -MP

ALG_SRCS = src/basicos/bubble.c src/basicos/selection.c src/basicos/insertion.c \
           src/avancados/merge.c src/avancados/quick.c src/avancados/heap.c
SRCS = src/main.c $(ALG_SRCS)

OBJS = $(SRCS:.c=.o)
ALL_OBJS = $(sort $(OBJS))

TARGET = sort_main

all: $(TARGET)

$(TARGET): $(OBJS)
	$(CC) $(CFLAGS) -o $@ $^

%.o: %.c
	$(CC) $(CFLAGS) $(DEPFLAGS) -c $< -o $@

clean:
	rm -f $(TARGET) $(ALL_OBJS) $(ALL_OBJS:.o=.d)

.PHONY: all clean

-include $(ALL_OBJS:.o=.d)
```

!!! warning "Makefile exige TAB"
    As linhas de comando de cada regra (as que ficam logo abaixo de `alvo: dependências`) precisam começar com o caractere **TAB**, não com espaços. Se o `make` responder `missing separator`, é isso. No VS Code, confira no canto inferior direito se a indentação está em "Tab", e não em "Spaces".

**O que cada parte faz** (você vai precisar explicar isto no vídeo):

| Trecho | Significado |
| --- | --- |
| `CC`, `WARN`, `CFLAGS` | **Variáveis**: o compilador, os avisos (`-Wall -Wextra`, padrão C11) e a otimização `-O2`. Usa-se `$(CFLAGS)` para ler o valor. |
| `ALG_SRCS`, `SRCS` | Listas de arquivos `.c`. A barra invertida `\` no fim da linha continua a lista na linha seguinte. |
| `OBJS = $(SRCS:.c=.o)` | Troca `.c` por `.o` em cada item: a lista dos arquivos objeto (código já compilado, mas ainda não ligado). |
| `ALL_OBJS` | Todos os objetos do projeto. Por enquanto são só os de `OBJS`; vai crescer nas próximas etapas. |
| `all: $(TARGET)` | O alvo **padrão**: é o que acontece quando você digita só `make`. |
| `$(TARGET): $(OBJS)` | Lê-se: "para construir `sort_main`, preciso de todos os `.o`; depois, rodo o comando de ligação". `$@` é o nome do alvo e `$^` é a lista de dependências. |
| `%.o: %.c` | **Regra geral**: ensina o `make` a compilar *qualquer* `.c` em `.o`. `$<` é o `.c` de entrada, e `-c` significa "compile, mas não ligue". |
| `DEPFLAGS = -MMD -MP` e `-include ...` | O compilador gera um arquivo `.d` por `.c`, listando os `.h` de que ele depende. Assim, se você alterar `metricas.h`, o `make` recompila o que for preciso. |
| `clean` | Apaga os arquivos gerados (executável, `.o`, `.d`). |
| `.PHONY` | Avisa que `all` e `clean` são **ações**, e não arquivos. |

Agora o `.gitignore`, na raiz. Ele diz ao Git quais arquivos **não** versionar (binários e arquivos intermediários), para que não entrem nos seus commits:

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

A barra no começo (`/sort_main`) restringe o padrão à raiz do projeto. Os relatórios em `docs/` **devem** ser versionados, então não estão na lista.

Você ainda não consegue rodar `make`, porque os fontes ainda não existem. Isso acontece nas próximas seções, e o primeiro teste do Makefile é o Checkpoint 1.

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

O `#ifndef`/`#define`/`#endif` (*header guard*) impede que o arquivo seja incluído duas vezes na mesma compilação. Como a struct é usada pelos algoritmos:

```c
Metricas m = {0, 0};      /* quem chama: zera os dois contadores */
bubbleSort(v, n, &m);     /* passa o ENDEREÇO de m (o & é essencial) */

/* dentro de bubbleSort, m é um PONTEIRO: */
m->comparacoes++;         /* a seta acessa um campo através do ponteiro */
```

Passar o endereço permite que a função **altere** a struct de quem a chamou; é assim que os contadores voltam para o `main`, os testes e o benchmark.

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

O Makefile compila os seis arquivos de uma vez. Para que `make` funcione desde já, crie **esqueletos vazios** com este script (no terminal rode uma vez, na raiz do repositório):

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

**O que observar no `main.c`** (você vai explicar isto no vídeo):

- **`ALGS[]`** é um vetor de structs com o nome e um **ponteiro para função** (`void (*fn)(int[], int, Metricas *)`). Como os seis algoritmos têm a mesma assinatura, o mesmo laço chama qualquer um deles com `ALGS[k].fn(v, n, &m)`.
- **`srand(42)`** com `rand() % 1000000` gera um vetor aleatório **reproduzível**.
- **`orig` e `v`:** o original é guardado e cada algoritmo ordena uma **cópia** (`memcpy`), para todos receberem exatamente os mesmos dados.
- **`clock()`** é chamado antes e depois **só da ordenação**; `CLOCKS_PER_SEC` converte para milissegundos. Mede tempo de CPU, com resolução limitada: tempos muito pequenos aparecem como `0.000`.
- **`argc` e `argv`** trazem os argumentos de linha de comando (`atoi` converte `<n>`; `strcmp` filtra o algoritmo).
- **`malloc` seguido de verificação e `free`:** todo `malloc` é checado, e tudo o que foi alocado é liberado.

!!! note "Os números podem diferir em outros sistemas"
    O `rand()` depende da biblioteca do sistema. Nos exemplos deste tutorial (Linux), as contagens com `n` grande podem diferir um pouco no macOS ou em outro sistema; a ordem de grandeza e as conclusões são as mesmas. Já as métricas dos vetores pequenos e fixos (como `{5,1,4,2,8}`) não dependem do sistema.

### Checkpoint 1

```bash
make
./sort_main 10
```

!!! warning "missing separator"
    As linhas de comando das regras precisam começar com o caractere **TAB**, não com espaços. Se ao copiar o arquivo aparecer `missing separator`, é isso. Uma opção, apague o espaço e substituiu por tab.

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

!!! tip "Apareceu um erro?"
    O [Apêndice B](#apendice-b-erros-comuns-e-depuracao) lista as mensagens mais comuns do `make` e do `gcc`, com a causa e a solução de cada uma.

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

/* Confere as comparações e trocas/movimentações esperadas para um vetor conhecido */
static void testaMetricas(const char *alg, const char *caso, SortFn fn,
                          const int orig[], int n, unsigned long comp, unsigned long mov) {
    int *v = malloc((size_t)n * sizeof(int));
    if (v == NULL) { fprintf(stderr, "falta de memória\n"); exit(2); }
    for (int i = 0; i < n; i++) v[i] = orig[i];

    Metricas m = {0, 0};
    fn(v, n, &m);

    int ok = (m.comparacoes == comp && m.trocas == mov);
    printf("%-7s  %-10s métricas %s\n", ok ? "PASSOU" : "FALHOU", alg, caso);
    if (!ok) {
        printf("         esperado comp=%lu mov=%lu, obtido comp=%lu mov=%lu\n",
               comp, mov, m.comparacoes, m.trocas);
        falhas++;
    }
    free(v);
}

int main(void) {
    int base[]     = {5, 1, 4, 2, 8};
    int ordenado[] = {1, 2, 3, 4, 5};

    testaAlgoritmo("bubble", bubbleSort);
    testaMetricas("bubble", "{5,1,4,2,8}", bubbleSort, base, 5, 9, 4);
    testaMetricas("bubble", "ordenado", bubbleSort, ordenado, 5, 4, 0);
    /* TODO: adicione aqui as chamadas dos demais algoritmos, à medida que os implementar */

    if (falhas > 0) {
        printf("\n%d teste(s) falharam\n", falhas);
        return 1;               /* código de saída != 0: o make test também falha */
    }
    printf("\nTodos os testes passaram\n");
    return 0;
}
```

!!! tip "Para que serve o `testaMetricas`?"
    As métricas fazem parte do enunciado, então vale testá-las. A função confere as comparações e trocas para um vetor **conhecido**: com `{5,1,4,2,8}` o Bubble faz 9 comparações e 4 trocas (simulação na seção 2.5) e, com o vetor **já ordenado** `{1,2,3,4,5}`, faz só 4 comparações e **nenhuma troca**, pois a primeira passada não troca nada e o algoritmo para. Nas Etapas 3 e 4 você reutiliza a mesma função para os outros algoritmos.

!!! tip "Por que conferir a soma?"
    Um algoritmo com bug pode devolver um vetor "ordenado" que **perdeu ou duplicou elementos** (por exemplo, `{1,1,1,1,1}`). A soma é um teste barato que pega esse tipo de erro. Um teste forte confere ordem **e** preservação dos elementos.

### 2.4 Acrescentar o alvo `test` ao Makefile

!!! info "O Makefile é um arquivo compartilhado"
    Um de vocês faz estas alterações, comita e dá `push`; o outro faz `git pull` antes de mexer no Makefile (veja a Etapa 0.8).

Para o `make test` funcionar, o Makefile precisa saber compilar o `tests/test_basic.c` junto com os algoritmos. Faça estas alterações (cada bloco diz onde entra):

**Logo abaixo da linha `SRCS = ...`**:

```make
TEST_SRCS = tests/test_basic.c $(ALG_SRCS)
```

**Logo abaixo de `OBJS = ...`** (e **troque** a linha `ALL_OBJS` pela nova versão):

```make
TEST_OBJS = $(TEST_SRCS:.c=.o)
ALL_OBJS = $(sort $(OBJS) $(TEST_OBJS))
```

**Logo abaixo de `TARGET = sort_main`**:

```make
TEST_TARGET = test_basic
```

**Logo abaixo da regra `$(TARGET): $(OBJS)`**, a regra que liga o executável de testes:

```make
$(TEST_TARGET): $(TEST_OBJS)
	$(CC) $(CFLAGS) -o $@ $^
```

**Antes do alvo `clean`**, o novo alvo `test`:

```make
test: $(TEST_TARGET)
	./$(TEST_TARGET)
```

**Substitua** o `clean` e a linha `.PHONY` (mantenha a linha `-include` no final):

```make
clean:
	rm -f $(TARGET) $(TEST_TARGET) $(ALL_OBJS) $(ALL_OBJS:.o=.d)

.PHONY: all test clean
```

O alvo `test` depende de `$(TEST_TARGET)`: o `make` primeiro constrói o executável de testes e só depois roda `./test_basic`. Se algum teste falhar, o programa devolve código ≠ 0 e o próprio `make` acusa erro.

Note que o `sort_main` e o `test_basic` **compartilham os mesmos `.o`** dos algoritmos. Por isso, depois de alterar um algoritmo, o `make test` recompila o `.o`, mas **não religa o `sort_main`**: é preciso rodar `make` também (veja o aviso no Checkpoint 2).

??? note "Makefile completo ao fim da Etapa 2 (para comparar com o seu)"
    ```make
    CC = gcc
    WARN = -Wall -Wextra -std=c11
    CFLAGS = $(WARN) -O2
    DEPFLAGS = -MMD -MP

    ALG_SRCS = src/basicos/bubble.c src/basicos/selection.c src/basicos/insertion.c \
               src/avancados/merge.c src/avancados/quick.c src/avancados/heap.c
    SRCS = src/main.c $(ALG_SRCS)
    TEST_SRCS = tests/test_basic.c $(ALG_SRCS)

    OBJS = $(SRCS:.c=.o)
    TEST_OBJS = $(TEST_SRCS:.c=.o)
    ALL_OBJS = $(sort $(OBJS) $(TEST_OBJS))

    TARGET = sort_main
    TEST_TARGET = test_basic

    all: $(TARGET)

    $(TARGET): $(OBJS)
    	$(CC) $(CFLAGS) -o $@ $^

    $(TEST_TARGET): $(TEST_OBJS)
    	$(CC) $(CFLAGS) -o $@ $^

    %.o: %.c
    	$(CC) $(CFLAGS) $(DEPFLAGS) -c $< -o $@

    test: $(TEST_TARGET)
    	./$(TEST_TARGET)

    clean:
    	rm -f $(TARGET) $(TEST_TARGET) $(ALL_OBJS) $(ALL_OBJS:.o=.d)

    .PHONY: all test clean

    -include $(ALL_OBJS:.o=.d)
    ```

### 2.5 Entendendo as métricas à mão

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
PASSOU   bubble     aleatório
PASSOU   bubble     ordenado
PASSOU   bubble     invertido
PASSOU   bubble     duplicatas
PASSOU   bubble     negativos
PASSOU   bubble     n=1
PASSOU   bubble     n=0
PASSOU   bubble     métricas {5,1,4,2,8}
PASSOU   bubble     métricas ordenado

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
    git add Makefile src/basicos/bubble.c tests/test_basic.c
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

**Testes:** no `main` do `test_basic.c`, acrescente `testaAlgoritmo("selection", selectionSort);` e a verificação das métricas com o auxiliar que você já tem, por exemplo `testaMetricas("selection", "{5,1,4,2,8}", selectionSort, base, 5, 10, 2);` (os vetores `base` e `ordenado` já estão declarados no `main`).

!!! success "Checkpoint 3a — métricas para conferir"
    Para `{5, 1, 4, 2, 8}`, com a regra "troca só se `menor != i`", o resultado esperado é **10 comparações e 2 trocas**. Para o vetor já ordenado `{1,2,3,4,5}`: **10 comparações e 0 trocas** (o número de comparações não depende da entrada).

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
    Para `{5, 1, 4, 2, 8}`, contando apenas deslocamentos: **7 comparações e 4 movimentações**. Para o vetor já ordenado `{1,2,3,4,5}`: **4 comparações e 0 movimentações**. Se você contar também a cópia final da chave, o número de movimentações muda: **documente o critério** que usou.

!!! question "Pergunta para a análise"
    Quantas comparações o Insertion Sort faz em um vetor **já ordenado** de tamanho `n`? E o Bubble com a otimização de parada antecipada? Os testes com `{1,2,3,4,5}` (`n = 5`) já mostram o resultado; generalize para `n` e guarde a resposta para o `analise.md`. Para tamanhos grandes, o benchmark da Etapa 6 confirma.

### Checkpoint 3

```bash
make test
```

Todas as linhas dos dois algoritmos com `PASSOU`, incluindo estas:

```text
PASSOU   selection  métricas {5,1,4,2,8}
PASSOU   selection  métricas ordenado
PASSOU   insertion  métricas {5,1,4,2,8}
PASSOU   insertion  métricas ordenado
```

!!! success "Commit sugerido (um por algoritmo)"
    ```bash
    git commit -m "feat: implementa selection sort com testes"
    git commit -m "feat: implementa insertion sort com testes"
    ```

---

## Etapa 4 — Merge, Quick e Heap Sort

**Objetivo:** os três algoritmos O(n log n). São os mais fáceis de errar nos índices; por isso **teste cada um assim que escrevê-lo**, antes de passar ao próximo.

!!! warning "Antes de começar"
    Rode `make test` a cada algoritmo novo. Um bug de índice em Merge/Quick/Heap costuma aparecer como `Segmentation fault`. Se acontecer, **não adivinhe**: o [Apêndice B](#apendice-b-erros-comuns-e-depuracao) mostra como o AddressSanitizer aponta a linha exata do erro, sem instalar nada.

### 4.1 Merge Sort

**Contrato:** `void mergeSort(int v[], int n, Metricas *m);` em `src/avancados/merge.c`.

**Requisitos obrigatórios:**

- O vetor auxiliar é alocado **dentro** da função (`malloc`), com **verificação do retorno** e `free` antes de retornar.
- Cada chamada da função pública cria o auxiliar **uma única vez**; a recursão usa uma função auxiliar `static`.
- Trate `n < 2` **antes** de alocar (não há nada a ordenar): `malloc(0)` pode devolver `NULL` em algumas plataformas, e isso não significa falta de memória.

**Estrutura sugerida:**

```text
mergeSort(v, n, m):            // função pública
    se n < 2 → retorna         // nada a ordenar (e evita malloc(0))
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
| Vetores grandes (10.000 elementos): aleatório com duplicatas, ordenado e invertido | Pegam erros que só aparecem com muitos elementos ou recursão profunda, e a degeneração do Quick em entrada ordenada |

**1.** Dentro de `testaAlgoritmo`, depois do caso `n=0`, acrescente os casos pequenos e a chamada dos grandes:

```c
    int dois_ord[] = {1, 2}, dois_inv[] = {2, 1}, iguais[] = {7, 7, 7, 7};
    verifica(nome, "n=2 ord",    fn, dois_ord,   2);
    verifica(nome, "n=2 inv",    fn, dois_inv,   2);
    verifica(nome, "iguais",     fn, iguais,     4);

    testaGrandes(nome, fn);
```

**2.** Crie a função `testaGrandes` **antes** de `testaAlgoritmo` (ela usa o `verifica`):

```c
/* Vetores grandes: pegam erros que só aparecem com muitos elementos ou recursão profunda */
static void testaGrandes(const char *nome, SortFn fn) {
    int n = 10000;
    int *g = malloc((size_t)n * sizeof(int));
    if (g == NULL) { fprintf(stderr, "falta de memória\n"); exit(2); }

    srand(1);
    for (int i = 0; i < n; i++) g[i] = rand() % 5000;      /* com duplicatas */
    verifica(nome, "grande aleat", fn, g, n);

    for (int i = 0; i < n; i++) g[i] = i;
    verifica(nome, "grande ord",   fn, g, n);

    for (int i = 0; i < n; i++) g[i] = n - i;
    verifica(nome, "grande inv",   fn, g, n);

    free(g);
}
```

**3.** No `main`, acrescente `testaAlgoritmo("merge", mergeSort);` (e o mesmo para `quick` e `heap`) à medida que implementar cada um. Com os seis algoritmos, o `make test` leva cerca de 1 segundo.

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

**Objetivo:** usar `cppcheck`, `gcov` e `gprof`, e saber interpretar a saída de cada um. Primeiro você estende o Makefile com **três alvos novos** (`cppcheck`, `coverage` e `profile`); depois usa cada um. A tarefa final é entender e documentar os resultados.

### 5.0 Acrescentar os alvos ao Makefile

!!! info "O Makefile é um arquivo compartilhado"
    Um de vocês faz estas alterações, comita e dá `push`; o outro faz `git pull` antes de mexer no Makefile (veja a Etapa 0.8).

**Logo abaixo da linha `DEPFLAGS = ...`**, os parâmetros de compilação das duas ferramentas. Ambos desligam a otimização (`-O0`), porque o otimizador reorganiza o código e distorce as medições:

```make
GCOV_FLAGS = --coverage -g -O0
PROFILE_FLAGS = -pg -g -O0
```

**Depois dessas linhas**, os parâmetros do profiling. O `?=` significa "use este valor **a menos que** ele tenha sido passado na linha de comando", o que permite `make profile N=200000 ALG=merge`:

```make
# Parâmetros do profiling: make profile N=50000 ALG=bubble
N ?= 50000
ALG ?=
```

**Antes do alvo `clean`**, os três alvos novos:

```make
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
```

O que cada um faz:

- **`coverage`**: a linha `coverage: CFLAGS = ...` troca o valor de `CFLAGS` **só para este alvo** (sem `-O2`, com instrumentação). Ele depende de `clean` e de `test`: primeiro apaga tudo para recompilar **tudo instrumentado**, depois roda os testes, depois o `gcov` lê os dados e gera os relatórios, e por fim os objetos instrumentados são apagados para não contaminar o próximo `make`.
- **`profile`**: compila **direto das fontes** com `-pg -O0`, roda o programa com `N` e `ALG` e gera o relatório do `gprof`.
- **`cppcheck`**: roda a análise estática sobre `src` e `tests` e grava a saída (os avisos vão para a saída de erro, por isso o `2>`).

**Substitua** o `clean` e a linha `.PHONY` (mantenha o `-include` no final):

```make
clean:
	rm -f $(TARGET) $(TEST_TARGET) sort_profile $(ALL_OBJS) $(ALL_OBJS:.o=.d) *.gcov gmon.out
	find . \( -name '*.gcda' -o -name '*.gcno' \) -delete

.PHONY: all test coverage profile cppcheck clean
```

!!! warning "O `make coverage` apaga os objetos"
    Como ele executa `clean` antes e remove os objetos no final, **depois dele o `sort_main` não existe mais**. Rode `make` de novo se quiser usá-lo.

??? note "Makefile completo ao fim da Etapa 5 (para comparar com o seu)"
    ```make
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

    OBJS = $(SRCS:.c=.o)
    TEST_OBJS = $(TEST_SRCS:.c=.o)
    ALL_OBJS = $(sort $(OBJS) $(TEST_OBJS))

    TARGET = sort_main
    TEST_TARGET = test_basic

    all: $(TARGET)

    $(TARGET): $(OBJS)
    	$(CC) $(CFLAGS) -o $@ $^

    $(TEST_TARGET): $(TEST_OBJS)
    	$(CC) $(CFLAGS) -o $@ $^

    %.o: %.c
    	$(CC) $(CFLAGS) $(DEPFLAGS) -c $< -o $@

    test: $(TEST_TARGET)
    	./$(TEST_TARGET)

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
    	rm -f $(TARGET) $(TEST_TARGET) sort_profile $(ALL_OBJS) $(ALL_OBJS:.o=.d) *.gcov gmon.out
    	find . \( -name '*.gcda' -o -name '*.gcno' \) -delete

    .PHONY: all test coverage profile cppcheck clean

    -include $(ALL_OBJS:.o=.d)
    ```

### 5.1 `cppcheck` (análise estática)

O `cppcheck` foi instalado na Etapa 0.2. Rode:

```bash
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

**Meta:** **≥ 70% de cobertura de linhas no conjunto** dos seis arquivos de algoritmos (`src/basicos/*.c` e `src/avancados/*.c`): o total de linhas executadas dividido pelo total de linhas executáveis, somando os seis arquivos. `main.c`, `bench.c` e os testes não entram na conta.

O `resumo.txt` mostra o percentual **de cada arquivo**. Para obter o **total** dos seis, rode:

```bash
awk '/^File/{f=$2} /^Lines executed/ && f!=""{p=$2; sub(/.*:/,"",p); sub("%","",p); n=$4; ex+=p*n/100; tot+=n; f=""} END{printf "TOTAL: %.2f%% (%d de %d linhas)\n", 100*ex/tot, ex+0.5, tot}' docs/gcov/resumo.txt
```

```text
TOTAL: 100.00% (95 de 95 linhas)
```

(Os números dependem do seu código. Se os testes cobrirem pouco, o total cai: por exemplo, sem testar o Quick e o Heap, o mesmo comando dá cerca de 63%, abaixo da meta.)

!!! tip "Se a cobertura ficar baixa"
    Procure as linhas `#####` e pergunte: **que entrada faria essa linha executar?** Geralmente é `n=0`, `n=1`, duplicatas, vetor invertido ou o caminho de erro do `malloc` (esse último pode ser aceitável de ficar sem cobertura; justifique em `docs/cobertura.md`).

Registre em `docs/cobertura.md` o percentual **de cada arquivo** e o **total**, e comente as linhas `#####` que restarem.

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

### Checkpoint 5

```bash
make cppcheck && wc -c docs/cppcheck_bruto.txt   # 0 bytes = nenhum apontamento
make coverage && ls docs/gcov                    # seis arquivos .gcov e o resumo.txt
make profile && head -1 docs/perfil_bruto.txt    # deve imprimir: Flat profile:
```

```text
bubble.c.gcov  heap.c.gcov  insertion.c.gcov  merge.c.gcov  quick.c.gcov  resumo.txt  selection.c.gcov
```

Rode também o comando `awk` da seção 5.2: o **TOTAL** precisa ser **≥ 70%**. Lembre que o `make coverage` apaga o `sort_main`: rode `make` de novo antes de usá-lo.

!!! success "Commits sugeridos"
    ```bash
    git commit -m "chore: adiciona alvos cppcheck, coverage e profile ao Makefile"
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

### 6.3 Acrescentar o alvo `bench` ao Makefile

!!! info "O Makefile é um arquivo compartilhado"
    Um de vocês faz estas alterações, comita e dá `push`; o outro faz `git pull` antes de mexer no Makefile (veja a Etapa 0.8).

Este é o último trecho do Makefile. Faça as alterações:

**Logo abaixo de `TEST_SRCS = ...`**:

```make
BENCH_SRCS = src/bench.c $(ALG_SRCS)
```

**Logo abaixo de `TEST_OBJS = ...`** (e **troque** a linha `ALL_OBJS`):

```make
BENCH_OBJS = $(BENCH_SRCS:.c=.o)
ALL_OBJS = $(sort $(OBJS) $(TEST_OBJS) $(BENCH_OBJS))
```

**Logo abaixo de `TEST_TARGET = test_basic`**:

```make
BENCH_TARGET = bench_sort
```

**Logo abaixo da regra `$(TEST_TARGET): ...`**:

```make
$(BENCH_TARGET): $(BENCH_OBJS)
	$(CC) $(CFLAGS) -o $@ $^
```

**Logo abaixo do alvo `test`**, o alvo `bench`. Ele redireciona (`>`) a saída do programa, que é o CSV, para `docs/resultados.csv`:

```make
# Benchmark: gera a tabela de tempos, comparações e trocas em CSV
bench: $(BENCH_TARGET)
	./$(BENCH_TARGET) > docs/resultados.csv
	@echo "Resultados gravados em docs/resultados.csv"
```

**Substitua** o `clean` e a linha `.PHONY` (mantenha o `-include` no final):

```make
clean:
	rm -f $(TARGET) $(TEST_TARGET) $(BENCH_TARGET) sort_profile $(ALL_OBJS) $(ALL_OBJS:.o=.d) *.gcov gmon.out
	find . \( -name '*.gcda' -o -name '*.gcno' \) -delete

.PHONY: all test bench coverage profile cppcheck clean
```

Pronto: seu Makefile agora é **igual ao do [apêndice](#apendice-makefile-base)**. Se algo não funcionar, compare o seu com o do apêndice, linha a linha.

### 6.4 Rodando

```bash
make bench          # grava docs/resultados.csv (cerca de 3 minutos, quase todos em Bubble e Selection com n = 50000)
head docs/resultados.csv
```

### Checkpoint 6

- O CSV tem o cabeçalho certo e **nenhuma linha** de Bubble/Selection/Insertion com `n=500000`.
- Total de linhas: 3 tipos × (3 básicos × 4 tamanhos + 3 avançados × 5 tamanhos) = **81 linhas** de dados, mais o cabeçalho.
- `wc -l docs/resultados.csv` deve dar **82**.

### 6.5 Escrevendo `docs/analise.md`

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
    git commit -m "feat: adiciona benchmark que gera CSV e alvo bench no Makefile"
    git commit -m "docs: adiciona resultados.csv e análise experimental"
    ```

---

## Etapa 7 — Documentação final, vídeo e entrega

### 7.1 `docs/uso-ia.md`

Preencha **ao longo do trabalho**, não só no fim. Registre prompts reais, o que a IA gerou e **o que vocês adaptaram ou descobriram estar errado**. Use o [modelo da especificação](#politica-de-uso-de-ia-e-transparencia). O uso de IA **não penaliza**; a falta de transparência e a incapacidade de explicar o código, sim.

### 7.2 `README.md` e modelos dos relatórios

O `README.md` é a **primeira coisa** que um visitante vê no repositório: ele funciona como a capa da vitrine. Deve conter uma breve descrição do projeto, os nomes dos integrantes, como compilar (`make`), como testar (`make test`), como rodar (`./sort_main <n> [algoritmo]`) e links para a análise em `docs/`. Um esqueleto:

```markdown
# Ordenação em C — <nome da dupla>

**Integrantes:** Nome Completo 1, Nome Completo 2 · **Turma:** ...

## Sobre o projeto
Implementação e análise experimental de seis algoritmos de ordenação (Bubble, Selection, Insertion, Merge, Quick e Heap) em C, com testes automatizados, cobertura de código, análise estática e profiling.

## Como compilar e executar
    make                      # compila ./sort_main
    ./sort_main 50000         # os 6 algoritmos com n = 50000
    ./sort_main 50000 quick   # só o Quick Sort

## Testes e ferramentas
`make test` · `make coverage` · `make bench` · `make profile` · `make cppcheck`

## Documentação
- [Análise experimental](docs/analise.md) · [Cobertura](docs/cobertura.md)
- [Profiling](docs/perfil.md) · [cppcheck](docs/cppcheck.md) · [Uso de IA](docs/uso-ia.md)
- Vídeo: <link do YouTube>

## Warnings
(Se algum warning de -Wall -Wextra não pôde ser corrigido, justifique aqui.)
```

??? note "Esqueletos dos relatórios `cobertura.md`, `perfil.md` e `cppcheck.md`"
    **`docs/cobertura.md`**

    | Arquivo | Cobertura de linhas | Linhas `#####` (não executadas) e motivo |
    | --- | --- | --- |
    | `src/basicos/bubble.c` | 100% | nenhuma |
    | ... | ... | ... |
    | **Total (6 arquivos)** | **xx%** (meta ≥ 70%) | |

    **`docs/perfil.md`**: comando usado (valores de `N` e `ALG`); as funções que mais consomem tempo (copie o *flat profile*); interpretação: isso bate com a teoria? por que os avançados quase não aparecem com `n = 50000`?

    **`docs/cppcheck.md`**

    | O que foi apontado | O que corrigimos | O que mantivemos (e por quê) |
    | --- | --- | --- |
    | (ou: "nada foi apontado") | | |

### 7.3 Teste de ponta a ponta (num clone novo)

Antes do `push` final, **teste num clone novo** do repositório. Isso garante que nada depende de arquivos que só existem no seu computador (e que `docs/` e o `.gitignore` estão corretos):

```bash
cd /tmp
git clone <url-do-seu-repositório> teste-final
cd teste-final
make clean && make all && make test && make coverage && make bench && make profile && make cppcheck
```

O `make bench` leva cerca de 3 minutos. Tudo precisa terminar **sem erro e sem warnings** de `-Wall -Wextra`. No clone, o `git status` vai mostrar os relatórios de `docs/` como modificados, porque foram regerados: é normal. **Não commite a partir desse clone**; apague-o depois (`rm -rf /tmp/teste-final`).

Se algo falhar só no clone, é sinal de arquivo que não foi versionado: rode `git status` no seu repositório original e procure arquivos novos (*untracked*).

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

!!! tip "Como gravar e publicar o vídeo"
    - **Gravação:** use qualquer ferramenta que grave a tela e o microfone, como o OBS Studio, ou a gravação de uma reunião (Meet, Teams, Zoom) aberta só por vocês, compartilhando a tela.
    - **Antes de gravar:** aumente a fonte do terminal e do editor (o texto pequeno é ilegível em vídeo), feche abas e notificações, e faça um teste de 30 segundos para conferir o áudio.
    - **Publicação:** no YouTube, **Criar → Enviar vídeo**, e em visibilidade escolha **Não listado** (ou Público). Abra o link numa janela anônima para confirmar que ele abre sem login, e confira que a duração é de **até 20 minutos**.

### 7.5 Conferir o histórico, criar a tag e entregar

**Antes de criar a tag**, confira o que o professor vai ver:

```bash
git status                   # deve estar limpo (nada a commitar)
git pull && git push         # local e GitHub sincronizados
git shortlog -sn             # os dois nomes, com commits de código
git log --oneline            # mensagens no padrão tipo: descrição
```

Abra também a página do repositório no GitHub, de preferência numa **janela anônima** (sem login), que é como qualquer visitante o verá, e confirme que o repositório abre. Na aba **Commits**, confirme: os arquivos de `docs/` estão lá (relatórios **versionados**), **não há** `.o`/executáveis/`gmon.out`.

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
- [ ] `gcov` ≥ 70% de linhas no **total** dos seis arquivos; `cppcheck` sem `error`
- [ ] `docs/` com todos os arquivos obrigatórios, **versionados**
- [ ] Commits regulares, com mensagens no padrão `tipo: descrição`
- [ ] `docs/uso-ia.md` preenchido
- [ ] Repositório **público**, com `README` claro (é a sua vitrine) e sem dados pessoais ou segredos
- [ ] Vídeo publicado e link enviado no SIGAA junto com o do repositório
- [ ] Tag `v1.0` criada e enviada

---

## Trabalho Prático

Esta seção reúne **o que será exigido e avaliado**. O "como fazer" de cada item está nas Etapas 0 a 7 acima; aqui ficam os requisitos, a estrutura obrigatória, a rubrica e a entrega.

**Formato:** em dupla · **Linguagem:** C11 (`-std=c11`) · **Prazo sugerido:** 2 semanas, tempo para a dupla se organizar, e não 2 semanas de trabalho diário. O trabalho não é feito em uma única sessão; comece cedo.

O trabalho é a avaliação prática da disciplina e tem **duas entregas indissociáveis**: o **código** (repositório GitHub **público**) e um **vídeo de defesa obrigatório**, no qual a dupla demonstra e explica o que implementou. O vídeo substitui a avaliação escrita da Terceira Unidade.

### Objetivos de aprendizagem

1. Implementar **3 algoritmos básicos** e **3 avançados** de ordenação em C.
2. Praticar **controle de versão** com Git em equipe, com histórico incremental.
3. Aplicar **qualidade e medição**: testes, cobertura (`gcov`), análise estática (`cppcheck`) e profiling (`gprof`).
4. Produzir **documentação técnica** e **análise empírica**, confrontando as medições com a teoria de complexidade.
5. Praticar **transparência acadêmica** no uso de IA, validando e explicando todo o código entregue.
6. Defender oralmente as escolhas de implementação e os resultados.
7. Construir um projeto **público** que sirva de **vitrine** do seu trabalho (portfólio), com histórico e documentação que outras pessoas possam ler.

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
| **Cobertura** | **≥ 70% de linhas no conjunto** dos **seis arquivos de algoritmos** (`src/basicos/*.c` e `src/avancados/*.c`), ou seja, linhas executadas ÷ linhas executáveis somando os seis, medida com `make coverage`. `main.c`, `bench.c` e os testes não entram na conta. Percentual de cada arquivo em `docs/cobertura.md`. |
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
├── Makefile                   # Construído ao longo das etapas (versão final no apêndice)
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
| **Qualidade e testes** | Testes passam, `gcov` ≥ 70% de linhas no total dos 6 arquivos, `cppcheck` sem erros | 20 |
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

1. **Repositório GitHub** atualizado e **público**. Ele é a vitrine do seu trabalho: capriche no `README` e na documentação.
2. **Tag `v1.0`** na versão final (comandos na Etapa 7.5).
3. **Envio via SIGAA**, em **um único comentário por dupla**, com: nomes completos dos integrantes, link do repositório e link do vídeo no YouTube.

!!! tip "Teste final antes do `push`"
    `make clean && make all && make test && make coverage && make bench && make profile && make cppcheck` (o `make bench` leva cerca de 3 minutos).

## Apêndice A: Makefile Base {#apendice-makefile-base}

Este é o Makefile **final**: o mesmo que você terá ao concluir a Etapa 6, depois de construí-lo aos poucos nas Etapas 1, 2, 5 e 6. Serve de **referência**: se algo der errado no seu, compare com ele. Ele inclui os alvos de compilação, testes, benchmark, `gcov`, `gprof` e `cppcheck`.

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

## Apêndice B: Erros comuns e depuração {#apendice-b-erros-comuns-e-depuracao}

### Mensagens frequentes

| Mensagem | Causa provável | O que fazer |
| --- | --- | --- |
| `Makefile:30: *** missing separator.  Stop.` | Uma linha de comando começa com espaços em vez de **TAB** | Troque a indentação por TAB (o número é a linha com problema) |
| `undefined reference to 'heapSort'` seguido de `ld returned 1 exit status` | O nome da função difere do protótipo (atenção às maiúsculas: `heapsort` ≠ `heapSort`), o arquivo não está em `ALG_SRCS`, ou o esqueleto não tem a função | Confira o nome **exato** em `algoritmos.h` e no `.c`, e a lista `ALG_SRCS` |
| `error: unknown type name 'Metricas'` | Falta `#include "../algoritmos.h"` no `.c` (ou o caminho está errado) | Inclua o cabeçalho na primeira linha do arquivo |
| `No rule to make target 'src/bench.o', needed by 'bench_sort'` | O arquivo `src/bench.c` não existe (ainda) ou tem outro nome | Crie o arquivo, no caminho e com o nome corretos |
| `warning: unused variable 'x' [-Wunused-variable]` | Variável declarada e nunca usada | Remova ou use a variável: os warnings devem ser corrigidos |
| `make: 'sort_main' is up to date.` | Nada mudou desde a última compilação | Confira se **salvou** o arquivo |
| `./sort_main: No such file or directory` | Ainda não compilou, ou o `make coverage` apagou o executável | Rode `make` |
| `comparações=0 trocas=0` no `sort_main` depois de implementar | O `make test` não religa o `sort_main` | Rode `make` antes de `./sort_main` |
| `make: *** [Makefile:..: test] Error 1` | Algum teste falhou | Procure a primeira linha `FALHOU` acima da mensagem |
| `Segmentation fault (core dumped)` | Acesso inválido à memória: índice fora do vetor, recursão sem fim, ponteiro nulo | Veja a seção seguinte |

### Descobrindo a linha de um erro de memória

Compilando com o **AddressSanitizer**, o GCC insere verificações que apontam a linha exata de um acesso inválido. Não precisa instalar nada além do `gcc`:

```bash
gcc -std=c11 -g -O0 -fsanitize=address,undefined -o test_asan \
    tests/test_basic.c src/basicos/*.c src/avancados/*.c
./test_asan
```

Em um exemplo real, trocar `j < n - 1 - i` por `j <= n - 1 - i` no Bubble Sort (lê uma posição **depois** do fim do vetor) produz algo assim (resumido):

```text
==218==ERROR: AddressSanitizer: heap-buffer-overflow on address 0x503000000054 ...
READ of size 4 at 0x503000000054 thread T0
    #0 0x... in bubbleSort src/basicos/bubble.c:10
    #1 0x... in verifica tests/test_basic.c:30
    #2 0x... in testaAlgoritmo tests/test_basic.c:47
```

Como ler: `heap-buffer-overflow` diz **o tipo** do erro (acesso além do fim de uma área alocada); `READ` diz que foi uma **leitura**; a linha `#0` mostra **onde** (`bubble.c:10`); as linhas seguintes mostram **quem chamou**. Vá até a linha indicada e examine os índices.

Depois de usar, apague o executável (`rm test_asan`, e `rm test_dbg` se usar o `gdb`/`valgrind`) para não o commitar.

O AddressSanitizer detecta também o **estouro de pilha** causado por recursão sem fim (`ERROR: AddressSanitizer: stack-overflow`), apontando a função e a linha.

### `gdb` e `valgrind` (opcionais)

O AddressSanitizer costuma bastar. As duas ferramentas abaixo são alternativas; instale-as, se quiser, com `sudo apt install gdb valgrind`. Compile **sem `-O2`** e com `-g`, para que apareçam nomes de funções e números de linha:

```bash
gcc -std=c11 -g -O0 -o test_dbg tests/test_basic.c src/basicos/*.c src/avancados/*.c
```

| Ferramenta | Quando usar | Limitação |
| --- | --- | --- |
| **`gdb`** | O programa **trava** (`Segmentation fault`) e você quer ver **onde** e **quem chamou** | Se o programa não travar, não há o que mostrar |
| **`valgrind`** | Acessos inválidos à memória, **mesmo que o programa não trave** | É mais lento; sem `-g`, mostra funções mas **não as linhas** |

**`gdb`: `run` e `bt`.** Rode `gdb ./test_dbg`; no prompt `(gdb)`, digite `run`. Quando o programa travar, `bt` (*backtrace*) mostra a pilha de chamadas. Para não inundar a tela, use `bt 3` (as 3 primeiras chamadas) ou `bt -3` (as 3 últimas); `quit` sai. Num Quick Sort com a recursão errada (a segunda chamada recursiva sem reduzir o intervalo), o resultado é:

```text
Program received signal SIGSEGV, Segmentation fault.
#0  0x... in troca (v=<error reading variable: Cannot access memory at address ...>, ...)
#1  0x... in ordena (v=0x..., baixo=0, alto=1, m=0x...) at src/avancados/quick.c:12
#2  0x... in ordena (v=0x..., baixo=0, alto=1, m=0x...) at src/avancados/quick.c:21
```

Como ler: a mesma função `ordena`, com os **mesmos argumentos** (`baixo=0, alto=1`), chamada pela linha 21 várias vezes seguidas, é a assinatura de uma **recursão que nunca termina**. Nesse exemplo, `bt -3` mostrou mais de 130 mil chamadas empilhadas.

**`valgrind`.** Rode `valgrind ./test_dbg`. No exemplo do Bubble Sort com `j <= n - 1 - i`, o programa **não trava** (só falha nos testes), e mesmo assim o `valgrind` acusa o erro:

```text
==545== Invalid read of size 4
==545==    at 0x...: bubbleSort (bubble.c:10)
==545==    by 0x...: verifica (test_basic.c:30)
==545==    by 0x...: testaAlgoritmo (test_basic.c:47)
==545==  Address 0x... is 0 bytes after a block of size 20 alloc'd
```

A última linha diz que o acesso foi **logo depois do fim** de um bloco de 20 bytes (um vetor de 5 `int`), ou seja, um índice uma posição além do último.

### Método para achar um bug

1. Leia a **primeira** mensagem de erro (as seguintes costumam ser consequência dela).
2. Descubra **qual caso** falhou (`FALHOU` na lista) e reduza para o menor vetor que reproduz o problema.
3. Simule o algoritmo **no papel** para esse vetor e compare com o que o programa faz.
4. Só então altere o código, e rode `make test` de novo.