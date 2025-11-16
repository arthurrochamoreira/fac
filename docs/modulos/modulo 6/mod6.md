# Módulo 6 — Multiplication & Division

Este módulo apresenta a **extensão M do RISC-V**
(M de *Multiply/Divide*), que adiciona instruções de:

* multiplicação inteira
* divisão inteira
* resto de divisão inteira
* acesso à parte alta do produto

Sem a extensão M (ou seja, usando apenas **RV32I** — *RISC-V 32-bit Integer*), você teria que implementar multiplicação e divisão manualmente, com laços de somas e subtrações. Com **RV32IM**, o hardware já faz isso para você.

---

## 1. O que é a extensão M no RISC-V?

O RISC-V é definido em “módulos” de instruções.
O núcleo básico é o **I** (*Integer*), e extensões acrescentam funcionalidades.

Alguns conjuntos comuns:

* **RV32I** – apenas instruções inteiras básicas
* **RV32IM** – I + M (multiplicação e divisão inteiras)
* **RV32IMF** – adiciona também ponto flutuante, etc.

A extensão **M** adiciona estas instruções inteiras:

* `mul` — multiplicação inteira (parte baixa)
* `mulh`, `mulhu`, `mulhsu` — multiplicação retornando a parte alta do resultado
* `div`, `divu` — divisão inteira (quociente)
* `rem`, `remu` — resto da divisão inteira

No RARS (*RISC-V Assembler and Runtime Simulator*), você normalmente já está usando um modo que suporta **RV32IM**.

---

## 2. Conceitos importantes: 32 bits, produto de 64 bits

No **RV32**, cada registrador tem 32 bits.
Quando você multiplica dois inteiros de 32 bits, o resultado completo tem até **64 bits**.

Exemplo conceitual:

```text
(a: 32 bits) × (b: 32 bits) → resultado: 64 bits

parte alta  = bits [63:32]
parte baixa = bits [31:0]
```

As instruções se dividem assim:

* `mul`   → devolve a **parte baixa** (bits [31:0])
* `mulh`  → devolve a parte alta considerando **operandos com sinal**
* `mulhu` → devolve a parte alta considerando **operandos sem sinal** (*unsigned*)
* `mulhsu`→ devolve a parte alta considerando o primeiro operando com sinal e o segundo sem sinal

Para **divisão**:

* `div` / `rem`   → trabalham com números **com sinal** (inteiros 2’s complement)
* `divu` / `remu` → trabalham com números **sem sinal** (*unsigned*)

---

## 3. Multiplicação inteira simples: `mul`

A instrução `mul` faz:

```text
rd = (rs1 * rs2) mod 2^32
```

Ou seja, guarda apenas a **parte baixa** do produto (32 bits).
Se houver overflow, os bits “que sobraram” são descartados.

Sintaxe:

```asm
mul rd, rs1, rs2
```

Significa:

> `rd` recebe o resultado da multiplicação de `rs1` por `rs2`.

Exemplo simples:

```asm
.text
.globl main

main:
    li t0, 6              # li (load immediate): t0 = 6
    li t1, 7              # t1 = 7

    mul t2, t0, t1        # mul: t2 = t0 * t1 = 42

    # imprimir resultado no RARS
    mv a0, t2             # mv (move): a0 = t2
    li a7, 1              # 1 = print_int
    ecall

    li a7, 93             # 93 = exit2
    ecall
```

---

## 4. Multiplicação com parte alta: `mulh`, `mulhu`, `mulhsu`

Quando você realmente precisa do resultado completo de 64 bits (ou quer detectar overflow), entra a família `mulh`.

### 4.1 `mulh rd, rs1, rs2`

Multiplicação com **sinal** em ambos os operandos.

```text
rs1 e rs2 tratados como inteiros com sinal (signed 32 bits)
produto completo: 64 bits
rd = parte alta do produto (bits [63:32])
```

Sintaxe:

```asm
mulh rd, rs1, rs2
```

### 4.2 `mulhu rd, rs1, rs2`

Multiplicação **sem sinal** em ambos os operandos.

```text
rs1 e rs2 tratados como unsigned
rd = parte alta do produto
```

### 4.3 `mulhsu rd, rs1, rs2`

Multiplicação mista:

* `rs1` tratado como **signed**
* `rs2` tratado como **unsigned**

Isso é útil em algumas rotinas de aritmética de precisão ou otimizações específicas (por exemplo, divisão por constantes usando multiplicação por fator pré-calculado).

### 4.4 Combinando `mul` + `mulh` para obter 64 bits

Suponha que você queira o produto completo de 64 bits de `a * b`:

```asm
# a em t0, b em t1
mul  t2, t0, t1     # parte baixa  (low  32 bits)
mulh t3, t0, t1     # parte alta   (high 32 bits)
# Agora t3:t2 representa um inteiro de 64 bits
```

Em notação “concatenada”: `resultado_64 = (t3 << 32) | t2`.

---

## 5. Divisão inteira: `div` e `divu`

### 5.1 `div rd, rs1, rs2` — divisão com sinal

Calcula o **quociente** da divisão inteira de `rs1` por `rs2`, tratando os operandos como inteiros com sinal (*signed*).

Sintaxe:

```asm
div rd, rs1, rs2
```

Significa:

> `rd = rs1 / rs2` (divisão inteira com sinal, truncando em direção a zero).

Exemplo:

```asm
li t0,  7
li t1,  2
div t2, t0, t1      # t2 = 3   (7 / 2 = 3)

li t0, -7
li t1,  2
div t3, t0, t1      # t3 = -3  (-7 / 2 = -3, truncado em direção a 0)
```

### 5.2 `divu rd, rs1, rs2` — divisão sem sinal

Mesma ideia, mas os operandos são interpretados como **unsigned** (sem sinal).

Sintaxe:

```asm
divu rd, rs1, rs2
```

Significa:

> `rd = (unsigned)rs1 / (unsigned)rs2`

Isso faz diferença quando os operandos têm o bit mais significativo (bit 31) igual a 1.

---

## 6. Resto da divisão: `rem` e `remu`

A família `rem` devolve o **resto** da divisão inteira:

```text
rs1 = divisor
rs2 = divisor
quociente  = rs1 / rs2
resto      = rs1 - (quociente * rs2)
```

### 6.1 `rem rd, rs1, rs2` — resto com sinal

Trata os operandos como inteiros com sinal.

Sintaxe:

```asm
rem rd, rs1, rs2
```

Significa:

> `rd = resto da divisão inteira de rs1 por rs2 (signed)`

Em geral, o sinal do resto segue o sinal do dividendo (`rs1`).

Exemplo:

```asm
li t0,  7
li t1,  2
rem t2, t0, t1      # t2 = 1  (7 = 3*2 + 1)

li t0, -7
li t1,  2
rem t3, t0, t1      # t3 = -1  (-7 = -3*2 + (-1))
```

### 6.2 `remu rd, rs1, rs2` — resto sem sinal

Mesma lógica, mas com operandos **unsigned**:

```asm
remu rd, rs1, rs2
```

Significa:

> `rd = resto da divisão (unsigned)`

Importante em algoritmos que tratam os valores como conjuntos de bits, não como números positivos/negativos.

---

## 7. Comportamento especial: divisão por zero

A especificação do RISC-V define um comportamento específico quando o divisor (`rs2`) é zero:

* em `div` / `divu`:

  * o quociente é definido como **-1** (tudo em 1, para signed) ou **0xFFFF_FFFF** (para unsigned)
* em `rem` / `remu`:

  * o resto é simplesmente o valor do **dividendo** (`rs1`)

Na prática:

* `div t0, x, zero`  → t0 recebe um valor especial (normalmente -1)
* `rem t1, x, zero`  → t1 recebe `x`

Mesmo assim, é **boa prática** evitar dividir por zero e tratar isso com testes usando `beq`/`bne`.

---

## 8. Exemplos completos

### 8.1 Multiplicação e divisão básicas

Programa que faz:

1. `a = 20`, `b = 6`
2. calcula `produto = a * b`
3. calcula `quociente = a / b`
4. calcula `resto = a % b`
5. imprime os três resultados no RARS

```asm
.data
msgProd: .asciz "Produto: "
msgQuoc: .asciz "Quociente: "
msgResto:.asciz "Resto: "
msgNL:   .asciz "\n"

.text
.globl main

main:
    li t0, 20              # a = 20
    li t1, 6               # b = 6

    # produto
    mul t2, t0, t1         # t2 = 20 * 6 = 120

    # quociente
    div t3, t0, t1         # t3 = 20 / 6 = 3 (inteiro)

    # resto
    rem t4, t0, t1         # t4 = 20 % 6 = 2

    # imprimir produto
    la a0, msgProd         # endereço da string "Produto: "
    li a7, 4               # 4 = print_string
    ecall

    mv a0, t2              # valor do produto
    li a7, 1               # 1 = print_int
    ecall

    la a0, msgNL
    li a7, 4
    ecall

    # imprimir quociente
    la a0, msgQuoc
    li a7, 4
    ecall

    mv a0, t3
    li a7, 1
    ecall

    la a0, msgNL
    li a7, 4
    ecall

    # imprimir resto
    la a0, msgResto
    li a7, 4
    ecall

    mv a0, t4
    li a7, 1
    ecall

    la a0, msgNL
    li a7, 4
    ecall

    # sair
    li a7, 93              # exit2
    ecall
```

---

### 8.2 Usando `mulh` para detectar overflow

Suponha que você quer multiplicar dois inteiros e verificar se houve overflow (produto não coube em 32 bits).

Se a **parte alta** do produto for diferente de zero (ou de -1 em caso de sinais específicos), significa que o resultado ultrapassou o intervalo que cabe num registrador de 32 bits.

Exemplo simples: marcar overflow se a parte alta não é zero:

```asm
.text
.globl main

main:
    li t0, 100000          # operando 1
    li t1, 100000          # operando 2

    mul  t2, t0, t1        # parte baixa
    mulh t3, t0, t1        # parte alta

    # se t3 != 0, houve overflow
    bne t3, x0, tem_overflow

    # aqui: sem overflow
    # (t2 contém o resultado completo)
    j fim

tem_overflow:
    # aqui você pode tratar o caso de overflow
    # (ex.: imprimir mensagem de erro)
fim:
    li a7, 93
    ecall
```

---

### 8.3 Comparando `div` e `divu`

Código para ajudar a visualizar a diferença entre divisão com sinal e sem sinal:

```asm
.text
.globl main

main:
    li t0, -8              # valor negativo em t0
    li t1,  3

    div  t2, t0, t1        # divisão com sinal   → t2 esperado: -2
    divu t3, t0, t1        # divisão sem sinal   → t3: interpretando t0 como unsigned

    # aqui você pode imprimir t2 e t3 para comparar
    # (exercício sugerido abaixo)

    li a7, 93
    ecall
```

---

## 9. Por que a extensão M é importante?

Sem a extensão M:

* `mul` teria que ser simulada com um laço de somas sucessivas
* `div` e `rem` teriam que ser implementadas com laços de subtração repetida ou algoritmos mais complexos (como divisão binária)

Isso é ótimo para aprender aritmética em baixo nível, mas:

* é **muito mais lento**
* o código fica grande e complicado

Na prática, quase todos os processadores RISC-V “de verdade” incluem a extensão **M**, e os simuladores (como RARS) também.

---

## 10. Exercícios para fixação

Use o RARS para montar e rodar seus códigos, e observe os registradores no painel de execução.

### Exercício 1 — Produto e quociente

Escreva um programa que:

1. Coloque dois inteiros em registradores (por exemplo, `a` em `t0`, `b` em `t1`).
2. Use `mul` para calcular o produto em `t2`.
3. Use `div` para calcular o quociente em `t3`.
4. Use `rem` para calcular o resto em `t4`.
5. Imprima, com `print_int` (syscall 1):

   * o produto
   * o quociente
   * o resto

em linhas separadas.

---

### Exercício 2 — Produto de 64 bits com `mul` + `mulh`

Implemente:

1. Leia dois inteiros do usuário (2 vezes `read_int`, syscall 5).

2. Calcule o produto completo de 64 bits:

   * parte baixa em um registrador
   * parte alta em outro registrador

3. Imprima os dois valores:

   * primeiro a parte alta
   * depois a parte baixa

Dica: use a combinação `mul` + `mulh`.

---

### Exercício 3 — Testando `div` vs `divu`

Escreva um programa que:

1. Coloque em `t0` um valor negativo (por exemplo, `-8`).

2. Coloque em `t1` o valor `3`.

3. Calcule:

   * `div t2, t0, t1`
   * `divu t3, t0, t1`

4. Imprima ambos os resultados (`t2` e `t3`) usando `print_int`, com mensagens indicando qual é qual.

5. Observe no RARS:

   * o que acontece com a divisão com sinal
   * o que acontece com a divisão sem sinal

Você pode alterar os valores e repetir o teste para outros casos.
