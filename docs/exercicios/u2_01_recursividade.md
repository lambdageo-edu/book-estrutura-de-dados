# Lista sobre recursividade

---

## 🟢 NÍVEL 1: FUNDAMENTOS E CONTROLE DE FLUXO

**1. Contagem flexível**

Crie uma função recursiva que imprima na tela os números inteiros de `inicio` até `fim`, avançando de `passo` em `passo`. O `passo` pode ser positivo (crescente) ou negativo (decrescente).

`void imprimir_sequencia(int inicio, int fim, int passo);`

*Dica:* O caso base deve verificar se `inicio` já ultrapassou `fim` na direção do passo.

**2. Somatório de 1 a N**

Retorne a soma dos números inteiros de `1` até `n` usando recursão.

`int somatorio(int n);`

*Dica:* Caso base: `n == 0` retorna `0` (assim a função também funciona para `n = 0`). Passo recursivo: `n + somatorio(n - 1)`.

**3. Contagem de dígitos**

Dado um número inteiro positivo `n`, retorne quantos dígitos ele possui, sem converter para string.

`int contar_digitos(int n);`

*Dica:* Divida `n` por `10` recursivamente até chegar a `0`.

---

## 🟡 NÍVEL 2: OPERAÇÕES MATEMÁTICAS CLÁSSICAS

**4. Fatorial**

Calcule `n!` recursivamente. Considere `0! = 1`.

`long long fatorial(int n);`

*Use `long long` para evitar overflow rápido.*

**5. Potência recursiva**

Calcule `base^expoente` usando apenas multiplicações recursivas (`expoente ≥ 0`).

`long long potencia(int base, int expoente);`

*Dica:* Caso base: `expoente == 0` retorna `1`.

**6. Multiplicação por somas sucessivas**

Calcule `a × b` usando apenas adições e recursão.

`int multiplicar(int a, int b);`

*Dica:* Some `a` recursivamente `b` vezes. Trate `b < 0` invertendo o sinal no retorno.

**7. Divisão inteira por subtrações**

Calcule a parte inteira de `dividendo ÷ divisor` usando apenas subtrações recursivas.

`int dividir(int dividendo, int divisor);`

*Valide `divisor != 0` antes de chamar. Retorne quantas vezes `divisor` cabe em `dividendo`.*

**8. Máximo Divisor Comum (Euclides)**

Implemente recursivamente: `mdc(x, y) = x` se `y == 0`, senão `mdc(y, x % y)`.

`int mdc(int x, int y);`

**9. Série com fatorial**

Calcule `S = 1 + 1/2! + 1/3! + ... + 1/n!` recursivamente.

`double serie_fatorial(int n);`

*Dica:* Você pode chamar `fatorial(n)` dentro da função ou criar uma auxiliar que acumule `1/k!` no caminho.

---

## 🟠 NÍVEL 3: SEQUÊNCIAS, VETORES E STRINGS

**10. Fibonacci (N-ésimo termo)**

Retorne o `n`-ésimo termo da sequência de Fibonacci (`F(0)=0, F(1)=1`).

`long long fibonacci(int n);`

*Desafio:* conte quantas chamadas a versão recursiva direta faz para `n = 30`. Por que cresce tanto? Como evitar recalcular os mesmos termos?

**11. Soma de elementos de um vetor**

Some todos os elementos de um array `vet` de tamanho `n`.

`int somar_vetor(int vet[], int n);`

*Dica:* `vet[0] + somar_vetor(vet + 1, n - 1)` ou use índice `i`.

**12. Busca do maior valor em vetor**

Encontre o maior elemento de um array `vet` de tamanho `n`.

`int maior_vetor(int vet[], int n);`

**13. `strlen` recursiva**

Calcule o tamanho de uma string sem usar loops ou `strlen` da biblioteca.

`size_t my_strlen(const char *str);`

*Caso base:* `str[0] == '\0'` retorna `0`.

**14. Verificador de palíndromo**

Retorne `true` se a substring entre `inicio` e `fim` for palíndromo, `false` caso contrário.

`bool eh_palindromo_rec(const char *str, int inicio, int fim);`

*Sugestão de wrapper público:*

`bool eh_palindromo(const char *str) { return eh_palindromo_rec(str, 0, my_strlen(str) - 1); }`

**15. Decomposição de dígitos**

Imprima os dígitos de `n` separados por vírgula, da esquerda para a direita. Ex: `1234 → 1,2,3,4`.

`void imprimir_digitos(int n);`

*Dica:* Recursão para chegar ao dígito mais significativo primeiro, depois imprima com vírgula.

---

## 🔴 NÍVEL 4: PADRÕES AVANÇADOS (DIVISÃO & CONQUISTA / BACKTRACKING)

**16. Busca binária recursiva**

Encontre `valor` em um vetor **ordenado** `vet` entre os índices `inicio` e `fim`. Retorne o índice encontrado ou `-1`.

`int busca_binaria(int vet[], int inicio, int fim, int valor);`

**17. Torre de Hanoi**

Imprima todos os movimentos necessários para mover `n` discos de `origem` para `destino`, usando `auxiliar`.

`void hanoi(int n, char origem, char destino, char auxiliar);`

*Saída esperada:* `Mover disco 1 de A para C`

**18. Geração de permutações**

Imprima todas as permutações possíveis do vetor de caracteres `vet` entre os índices `inicio` e `fim`.

`void permutar(char vet[], int inicio, int fim);`

*Use backtracking com `swap` recursivo.*

**19. Conversão decimal → binário**

Imprima a representação binária de um número decimal `n` (sem usar strings ou arrays auxiliares).

`void decimal_para_binario(int n);`

*Dica:* Imprima após a chamada recursiva (`n / 2`) para que os bits saiam na ordem correta (MSB primeiro).

---

## 🛠️ Dicas Rápidas para Implementação em C

| Situação | Recomendação |
| --- | --- |
| `bool` e `size_t` | Inclua `<stdbool.h>` e `<stddef.h>` ou use `int`/`unsigned int` se quiser compatibilidade com C89. |
| Vetores em recursão | Não copie o vetor. Passe o mesmo ponteiro e reduza `n` ou avance com `vet + 1`. |
| Estouro de pilha | C não garante *tail-call optimization*. Evite recursão profunda (`>10⁴` chamadas) sem necessidade. |
| Testes | Crie um `main()` que chame cada função com entradas válidas, borda (`0`, `1`, vetores vazios) e inválidas (divisão por zero, `n < 0`). |