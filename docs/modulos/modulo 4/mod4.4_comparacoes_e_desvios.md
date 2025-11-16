# Módulo 4.4. — Comparações e Desvios

(`beq`, `bne`, `blt`, `bgt`, `beqz`, `bnez`)

Este módulo mostra como o **RISC-V**
(RISC-V = *Reduced Instruction Set Computer – Five*)
usa **instruções de desvio** (*branch instructions*) para tomar decisões.

É com essas instruções que implementamos, em assembly:

* `if`, `else`
* `while`
* `for`
* e qualquer tipo de decisão condicional

Vou assumir que você já tem noção de:

* **registradores** (como `a0`, `t0`, `x0`)
* diretivas **`.data`** e **`.text`**
* noções básicas de **syscalls** (*system calls*) no **RARS**
  RARS = *RISC-V Assembler and Runtime Simulator*, um simulador de RISC-V que monta e executa programas `.asm`.

---

## 1. Ideia central: não existe `if`, só *branch*

Em C:

```c
if (x == y) {
    // bloco verdadeiro
} else {
    // bloco falso
}
```

Em RISC-V, o processador **não conhece** a palavra `if`.
O que existe é:

1. Comparar valores em registradores
2. Se a condição for verdadeira, **desviar** para um **rótulo** (*label*), que é um nome que marca uma posição no código
3. Se a condição for falsa, apenas continuar executando a próxima instrução

Ou seja:

> “IF” é só um jeito humano de organizar **branches** (instruções de desvio).

---

## 2. Registradores e instruções de apoio

Dois tipos de registradores aparecem muito em comparações:

* `x0` — registrador especial que **sempre vale 0** (zero “ligado por hardware”, *hard-wired zero*)
* `t0`, `t1`, `t2`… — registradores temporários (*temporary registers*), usados para cálculos intermediários

Também vamos usar algumas instruções de apoio:

* `li rd, imm` — **`li` (load immediate)**: carrega um valor imediato (constante) `imm` diretamente para o registrador `rd`.
  Exemplo: `li t0, 10` coloca o valor 10 dentro de `t0`.

* `addi rd, rs1, imm` — **`addi` (add immediate)**: soma o valor imediato `imm` ao conteúdo de `rs1` e guarda o resultado em `rd`.
  Exemplo: `addi t0, t0, 1` faz `t0 = t0 + 1`.

* `j label` — **`j` (jump)**: pseudo-instrução de salto incondicional.
  O montador traduz `j label` para `jal x0, label`
  (`jal` = *jump and link*, mas como o destino é `x0`, o retorno é descartado).

* `ecall` — **`ecall` (environment call)**: dispara uma syscall (chamada de sistema) configurada no registrador `a7`.
  Por exemplo, no RARS: `a7 = 4` → imprimir string; `a7 = 93` → encerrar o programa.

---

## 3. Instruções de desvio básicas

As instruções de desvio comparam **dois registradores**. Se a condição for verdadeira, elas pulam para um **rótulo** (`label:`).

### 3.1 `beq rs1, rs2, label`

**`beq` (branch if equal)** desvia se os registradores forem **iguais**.

Sintaxe:

```asm
beq rs1, rs2, label
```

Significa:

> Se `rs1 == rs2`, pule para `label`.
> Caso contrário, continue executando a próxima instrução.

Exemplo:

```asm
li t0, 10           # li: t0 = 10
li t1, 10           # li: t1 = 10

beq t0, t1, iguais  # beq: se t0 == t1, desvia para o rótulo 'iguais'
```

---

### 3.2 `bne rs1, rs2, label`

**`bne` (branch if not equal)** desvia se os registradores forem **diferentes**.

Sintaxe:

```asm
bne rs1, rs2, label
```

Significa:

> Se `rs1 != rs2`, pule para `label`.

Exemplo:

```asm
li t0, 5
li t1, 7

bne t0, t1, sao_diferentes   # bne: como 5 != 7, o desvio acontece
```

---

### 3.3 `blt rs1, rs2, label`

**`blt` (branch if less than)** desvia se `rs1` for **menor que** `rs2`, tratando os valores como **números com sinal** (*signed*).

Sintaxe:

```asm
blt rs1, rs2, label
```

Significa:

> Se `rs1 < rs2`, pule para `label`.

Exemplo:

```asm
li t0, 3
li t1, 10

blt t0, t1, menor          # blt: como 3 < 10, o desvio acontece
```

---

### 3.4 `bgt rs1, rs2, label` (pseudo-instrução)

**`bgt` (branch if greater than)** **não existe** como instrução real no conjunto RV32I.
Ela é uma **pseudo-instrução** criada pelo montador para facilitar a escrita.

Sintaxe:

```asm
bgt rs1, rs2, label
```

O montador normalmente converte isso em algo equivalente a:

```asm
blt rs2, rs1, label   # branch if rs2 < rs1 → ou seja, rs1 > rs2
```

Significa:

> Se `rs1 > rs2`, pule para `label`.

Exemplo:

```asm
li t0, 15
li t1, 7

bgt t0, t1, maior        # bgt: desvia, porque 15 > 7
```

---

## 4. Comparações com zero: `beqz` e `bnez`

Comparar um registrador com zero é tão comum que o montador oferece pseudo-instruções especiais.

### 4.1 `beqz rs, label`

**`beqz` (branch if equal to zero)** é uma **pseudo-instrução** que desvia se o registrador for igual a zero.

Sintaxe:

```asm
beqz rs, label
```

O montador traduz para:

```asm
beq rs, x0, label       # compara com o registrador x0 (constante zero)
```

Significa:

> Se `rs == 0`, pule para `label`.

Exemplo:

```asm
li t0, 0
beqz t0, eh_zero        # como t0 é 0, o desvio acontece
```

---

### 4.2 `bnez rs, label`

**`bnez` (branch if not equal to zero)** é a versão “diferente de zero”.

Sintaxe:

```asm
bnez rs, label
```

O montador traduz para:

```asm
bne rs, x0, label
```

Significa:

> Se `rs != 0`, pule para `label`.

Exemplo:

```asm
li t0, 42
bnez t0, nao_zero       # como t0 != 0, o desvio acontece
```

---

## 5. Padrão de IF simples em RISC-V

Vamos representar em C:

```c
if (t0 == 10) {
    // bloco verdadeiro
}
```

Versão típica em RISC-V:

```asm
    li t0, 10                   # t0 = 10
    li t1, 10                   # valor de comparação

    bne t0, t1, fim_if          # se t0 != 10, pula o bloco verdadeiro

    # bloco verdadeiro (t0 == 10)
    # ... código do verdadeiro ...

fim_if:
    # continuação do programa
```

Estratégia:

* usar `bne` para **saltar o bloco** quando a condição é falsa
* colocar um rótulo `fim_if` para marcar o ponto de continuação

---

## 6. IF–ELSE em RISC-V

Em C:

```c
if (t0 == 10) {
    // verdadeiro
} else {
    // falso
}
```

Versão em assembly:

```asm
    li t0, 10
    li t1, 10

    bne t0, t1, bloco_falso   # se t0 != 10, vai para o bloco falso

bloco_verdadeiro:
    # aqui t0 == 10
    # ... código do bloco verdadeiro ...
    j fim_if                  # j (jump): salto incondicional para 'fim_if'

bloco_falso:
    # ... código do bloco falso ...

fim_if:
    # segue o programa
```

Lembrando:

* `j` é pseudo-instrução de salto incondicional
  O montador traduz `j fim_if` para `jal x0, fim_if`.

---

## 7. Exemplo completo: negativo, zero ou positivo

Vamos classificar um número como **negativo**, **zero** ou **positivo** usando instruções de desvio.

Aqui aparecem algumas diretivas pela primeira vez neste módulo:

* `.data` — início da **sessão de dados**, onde declaramos variáveis e strings.
* `.text` — início da **sessão de código**, onde ficam as instruções.
* `.globl main` — torna o rótulo `main` o **ponto de entrada** visível externamente.
* `.asciz` — define uma string ASCII terminada com byte zero (`\0`), formato esperado por algumas syscalls.

Syscalls usadas (no RARS):

* `a7 = 4` → **print_string** (imprime string apontada por `a0`)
* `a7 = 93` → **exit2** (encerra o programa)

```asm
.data
msgNeg:  .asciz "Numero negativo\n"    # .asciz: string terminada em byte 0
msgZero: .asciz "Numero zero\n"
msgPos:  .asciz "Numero positivo\n"

.text
.globl main                            # .globl (global): expõe 'main' como entrada

main:
    # exemplo: vamos testar o valor -5
    li t0, -5                          # li: carrega o valor imediato -5 em t0

    blt t0, x0, negativo               # blt: se t0 < 0, desvia para 'negativo'
    beq t0, x0, eh_zero                # beq: se t0 == 0, desvia para 'eh_zero'

    # se não é < 0 nem == 0, só pode ser positivo
    la a0, msgPos                      # la (load address): a0 = endereço de msgPos
    li a7, 4                           # li: a7 = 4 (syscall print_string)
    ecall                              # ecall: executa a syscall → imprime "Numero positivo"
    j fim

negativo:
    la a0, msgNeg
    li a7, 4
    ecall
    j fim

eh_zero:
    la a0, msgZero
    li a7, 4
    ecall

fim:
    li a7, 93                          # 93 = syscall exit2
    ecall
```

---

## 8. Exemplo: `beqz`/`bnez` dentro de um loop

Agora vamos usar `bnez` para pular os números **ímpares** e imprimir apenas os **pares** de 1 a 10.

Nova instrução:

* `andi rd, rs1, imm` — **`andi` (AND immediate)**: faz um AND bit a bit entre `rs1` e o imediato `imm` e coloca o resultado em `rd`.
  Exemplo: `andi t2, t0, 1` pega apenas o **bit menos significativo** de `t0`.

Lógica:

* se `t0 & 1 == 0` → número é par
* se `t0 & 1 != 0` → número é ímpar

```asm
.text
.globl main

main:
    li  t0, 1                # t0 = 1 (contador)

loop:
    andi t2, t0, 1           # andi: t2 = t0 & 1 (isola o bit menos significativo)
    bnez t2, nao_par         # bnez: se t2 != 0, número é ímpar → pula impressão

    # se chegou aqui, o número é par
    mv  a0, t0               # mv: copia t0 para a0
    li  a7, 1                # 1 = syscall print_int (imprimir inteiro)
    ecall

    # quebra de linha
    li  a0, 10               # 10 = '\n' em ASCII
    li  a7, 11               # 11 = syscall print_char
    ecall

nao_par:
    addi t0, t0, 1           # addi: t0 = t0 + 1
    li   t1, 11              # t1 = 11 (limite superior exclusivo)
    blt  t0, t1, loop        # blt: enquanto t0 < 11, repete o loop

    li   a7, 93              # exit2
    ecall
```

Aqui `bnez` está fazendo exatamente o papel de:

```c
if (t2 != 0) {
    // não imprime (é ímpar)
}
```

---

## 9. Traduções úteis: de C para RISC-V

Uma “cola rápida” de condições comuns:

* `if (x == y)`
  → `beq x, y, rotulo_verdadeiro`

* `if (x != y)`
  → `bne x, y, rotulo_verdadeiro`

* `if (x < y)`
  → `blt x, y, rotulo_verdadeiro`

* `if (x > y)`
  → `bgt x, y, rotulo_verdadeiro`
  (pseudo-instrução; montador converte em `blt y, x, rotulo_verdadeiro`)

* `if (x == 0)`
  → `beqz x, rotulo_verdadeiro`
  (pseudo: `beq x, x0, rotulo_verdadeiro`)

* `if (x != 0)`
  → `bnez x, rotulo_verdadeiro`
  (pseudo: `bne x, x0, rotulo_verdadeiro`)

---

## 10. Exercícios para fixação

Os exercícios abaixo são pensados para treinar **exatamente** as instruções deste módulo:

* `beq`, `bne`, `blt`, `bgt`, `beqz`, `bnez`
* combinadas com `li`, `addi`, `j`, `andi`, syscalls simples

Use o RARS para montar e executar.

> Dica geral: no RARS, a syscall de leitura de inteiro
> (*read_int*) usa o código **5** em `a7` e devolve o valor em `a0`.

---

### Exercício 1 — Igual ou diferente de 10

Escreva um programa que:

1. Coloque um valor em `t0` usando `li` (pode testar com 10, depois com 7).
2. Compare `t0` com o valor 10.
3. Se `t0 == 10`, imprima a mensagem:
   `"Valor igual a 10\n"`.
4. Caso contrário, imprima:
   `"Valor diferente de 10\n"`.

**Restrição:** use apenas `beq` e `bne` para o desvio.

---

### Exercício 2 — Maior de dois números

Faça um programa que:

1. Coloque dois valores em `t0` e `t1` usando `li`.
2. Use `bgt` (pseudo-instrução) ou a combinação equivalente com `blt` para descobrir qual é o maior.
3. Imprima:

   * `"t0 é maior\n"` se `t0 > t1`
   * `"t1 é maior ou igual\n"` caso contrário

**Sugestão:**
Use um rótulo `t0_maior:` para o caso em que `t0` é maior, e um rótulo `fim:` para a continuação.

---

### Exercício 3 — Classificar número lido do usuário

Escreva um programa que:

1. Use a syscall de leitura de inteiro (código **5** em `a7`) para ler um valor digitado pelo usuário.

2. Copie o valor lido de `a0` para `t0`.

3. Classifique o número como:

   * negativo (`< 0`)
   * zero (`== 0`)
   * positivo (`> 0`)

4. Imprima uma das mensagens:

   * `"Numero negativo\n"`
   * `"Numero zero\n"`
   * `"Numero positivo\n"`

**Restrição:** use apenas `blt`, `beq` e `j` para os desvios.

---

### Exercício 4 — Contador decrescente com `beqz`

Implemente um contador que:

1. Começa em **10** (por exemplo, em `t0`).

2. Em cada iteração do loop:

   * imprime o valor atual em `t0`
   * decrementa `t0` em 1 (`addi t0, t0, -1`)

3. Usa `beqz` para parar quando o contador chegar em **0**.

4. Ao final, encerra o programa com a syscall **exit2** (código 93 em `a7`).

---

### Exercício 5 — Imprimir apenas ímpares com `bnez`

Adapte o exemplo de pares para:

1. Contar de **1** até **20**.
2. Calcular `t0 & 1` usando `andi`.
3. **Imprimir apenas os números ímpares**, pulando os pares.

**Restrição:** use `bnez` para decidir se imprime ou não.

---

### Exercício 6 — Contar quantos valores são maiores que 50

Considere um vetor de 6 inteiros definido em `.data`, por exemplo:

```asm
.data
vet: .word 10, 60, 51, 50, 100, 3
```

Faça um programa que:

1. Percorra o vetor com um loop (usando ponteiro + `addi` de 4 em 4).

2. Para cada elemento:

   * compare se o valor é **maior que 50**
   * se for, incremente um contador (por exemplo, `t2`)

3. Ao final, tenha em `t2` o número de elementos maiores que 50.

4. Imprima esse total usando a syscall de inteiro (código 1 em `a7`).

**Sugestão:**
Use `blt` ou `bgt` (pseudo-instrução) para implementar a comparação `valor > 50`.

---

### Exercício 7 — Loop até encontrar zero (sentinela)

Crie um vetor terminado em 0, por exemplo:

```asm
.data
valores: .word 3, -1, 7, 9, 0, 5
```

Escreva um programa que:

1. Percorra o vetor do começo até encontrar o valor **0**.

2. Para cada valor diferente de 0:

   * some o valor em um acumulador (`t2`, por exemplo)
   * avance para o próximo elemento

3. Pare o loop assim que encontrar 0, usando `beqz` ou `bnez` com o valor lido.

4. Imprima a soma acumulada.

---

### Exercício 8 (desafio) — Função sinal

Implemente uma “função de sinal” em assembly:

* Entrada: um número em `a0`
* Saída: em `a0`

Definição:

* se `a0 < 0`, deve retornar `-1`
* se `a0 == 0`, deve retornar `0`
* se `a0 > 0`, deve retornar `1`

Regras:

1. Implemente isso como um bloco de código com rótulo, por exemplo `sinal:`.

2. Use apenas:

   * `blt`
   * `beq`
   * `li`
   * `j`

3. No `main`, teste vários valores chamando esse bloco de código (você pode só “simular” a chamada colocando valores em `a0` e fazendo `j sinal` → depois usando outro rótulo para voltar).

---
