
# Apresentação do Livro

Este capítulo introdutório apresenta a proposta do livro **Estruturas de Dados: Uma Abordagem Prática em C** e faz uma introdução prática à linguagem C, que será utilizada como ferramenta principal ao longo de toda a obra.

---

## 1. Visão Geral do Livro

O objetivo desta obra é guiar o leitor na compreensão, implementação e análise das principais estruturas de dados utilizadas na ciência da computação. O conteúdo aborda desde os conceitos fundamentais de gerenciamento de memória e Tipos Abstratos de Dados (TAD) até estruturas lineares (pilhas, filas e listas encadeadas), estruturas hierárquicas (árvores binárias de busca e árvores balanceadas AVL), além de análise de complexidade e algoritmos de ordenação (incluindo o Heap Sort, que apresenta o heap binário).

A abordagem combina rigor conceitual com implementação prática em C, permitindo entender não apenas *como usar* cada estrutura, mas *como elas se organizam no nível físico da memória RAM*.

---

## 2. Por que a Linguagem C?

A linguagem C foi escolhida como ferramenta didática justamente por expor o gerenciamento de memória — ponteiros, *stack* (pilha de execução) e *heap* (monte) — de forma explícita. Linguagens de mais alto nível escondem esses detalhes por meio de *garbage collectors* ou abstrações automáticas.

Em Estruturas de Dados, compreender como a memória é alocada e manipulada é mais importante do que memorizar regras sintáticas. Além disso, a linguagem C é a fundação da infraestrutura de software mundial: sistemas operacionais, bancos de dados, compiladores e motores de execução de linguagens modernas.

---

## 3. Primeiro Exemplo em C: Compilação e Execução

Para iniciar os trabalhos práticos, vejamos o ciclo básico de desenvolvimento e compilação em C utilizando o terminal.

### Código de Exemplo (`hello.c`)

```c
#include <stdio.h>

int main() {
    printf("Estruturas de Dados em C\n");
    return 0;
}

```

### Ciclo de Compilação e Execução

```bash
# Compilar
gcc hello.c -o hello

# Executar (Linux/macOS)
./hello

# Executar (Windows)
hello.exe

```

---

## Síntese

Estruturas de Dados é, em sua essência, um estudo sobre organização e eficiência de memória: compreender como os dados se arranjam na RAM importa muito mais do que decorar sintaxe.

Para aproveitar ao máximo os capítulos a seguir:

1. Garanta que seu ambiente de desenvolvimento (compilador GCC e editor como VS Code ou Code::Blocks) esteja instalado e funcionando.
2. Teste o ciclo de compilação e execução do código `hello.c`.
3. Caso precise revisar a sintaxe básica da linguagem (variáveis, condicionais, laços, funções, vetores, *structs* e ponteiros), recomendamos o livro complementar **[C para Programadores Python e VisuAlg](https://lambdageo.github.io/introducao-c/)**.

No próximo capítulo, iniciamos nossa jornada prática com o estudo das **Pilhas**, partindo de uma implementação com vetores globais até chegar à alocação dinâmica, onde exploramos a diferença entre *stack* e *heap*.

!!! question "Dúvida Comum"
    *"Preciso ser um especialista em C para acompanhar o livro?"*

    **Resposta:** Não. O foco do livro é a lógica e a organização da memória. A sintaxe de C é enxuta, e o domínio sobre ponteiros e alocação será construído de forma gradual, capítulo a capítulo, através de explicações e exemplos práticos.


