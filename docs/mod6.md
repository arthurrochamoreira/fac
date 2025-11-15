# Módulo 6 — Condições e IF no RISC-V

Em assembly RISC-V **não existe** a instrução `if`.
O processador apenas executa **desvios condicionais** (*branches*), que fazem:

> “Se a condição for verdadeira, pule para este ponto do código.”

Um IF de alto nível é apenas uma combinação de:

1. comparação entre registradores
2. branch condicional
3. rótulos para marcar pontos de desvio

---

# 1. Principais instruções de desvio (RV32I)

Todos os branches comparam **registradores**:

* `rs1` e `rs2` → registradores-fonte
* `label` → rótulo que marca um endereço no código

### `beq rs1, rs2, label`

Desvia se `rs1 == rs2`.

### `bne rs1, rs2, label`

Desvia se `rs1 != rs2`.

### `blt rs1, rs2, label`

Desvia se `rs1 < rs2` (com sinal).

### `bge rs1, rs2, label`

Desvia se `rs1 >= rs2`.

### `bltu` / `bgeu`

Versões **unsigned**.

Com essas quatro (`beq`, `bne`, `blt`, `bge`) já é possível implementar qualquer expressão condicional.

---

# 2. Estrutura geral de um IF em assembly

## IF simples

```asm
    # condição
    beq rs1, rs2, bloco_verdadeiro

    # falso
    ...
    j fim

bloco_verdadeiro:
    # verdadeiro
    ...

fim:
```

## IF–ELSE

```asm
    beq rs1, rs2, verdadeiro

    # falso
    ...
    j fim

verdadeiro:
    ...
fim:
```

A pseudo-instrução `j label` é montada como `jal x0, label`.

---

# 3. Exemplo 1 — Testar se um número é igual a 10

Queremos:

* imprimir “igual” se `t0 == 10`
* senão imprimir “diferente”

```asm
.data
msgIgual: .asciz "O valor é igual a 10\n"
msgDif:   .asciz "O valor é diferente de 10\n"

.text
.globl main

main:
    li t0, 10              # número para testar
    li t1, 10              # comparação

    beq t0, t1, eh_igual   # se t0 == 10 → vai para eh_igual

    # falso
    la a0, msgDif
    li a7, 4               # print_string
    ecall
    j fim

eh_igual:
    la a0, msgIgual
    li a7, 4
    ecall

fim:
    li a7, 93              # exit2
    ecall
```

---

# 4. Exemplo 2 — Testar se um número é maior que outro

Implementação de:

```c
if (t0 > t1)
   ...
else
   ...
```

Usamos a lógica:

* se `t0 < t1` → não é maior
* caso contrário → maior ou igual

```asm
.data
msgMaior: .asciz "t0 é maior que t1\n"
msgMenor: .asciz "t0 é menor ou igual a t1\n"

.text
.globl main

main:
    li t0, 15
    li t1, 7

    blt t0, t1, menor_ou_igual   # se t0 < t1 → vá ao bloco falso

    # verdadeiro
    la a0, msgMaior
    li a7, 4
    ecall
    j fim

menor_ou_igual:
    la a0, msgMenor
    li a7, 4
    ecall

fim:
    li a7, 93
    ecall
```

---

# 5. Exemplo 3 — IF com entrada do usuário

Syscall **5** lê um inteiro e o coloca em `a0`.

Objetivo:

* se < 0 → “negativo”
* se == 0 → “zero”
* senão → “positivo”

```asm
.data
msgNeg:  .asciz "Numero negativo\n"
msgZero: .asciz "Numero zero\n"
msgPos:  .asciz "Numero positivo\n"

.text
.globl main

main:
    li a7, 5                # read_int
    ecall
    mv t0, a0               # guarda valor lido

    blt t0, x0, negativo    # se < 0
    beq t0, x0, zero        # se == 0

    # positivo
    la a0, msgPos
    li a7, 4
    ecall
    j fim

negativo:
    la a0, msgNeg
    li a7, 4
    ecall
    j fim

zero:
    la a0, msgZero
    li a7, 4
    ecall

fim:
    li a7, 93
    ecall
```

---

# 6. Exemplo 4 — IF dentro de um loop (testar par/ímpar)

Testar paridade:

* `n % 2 == 0`
* equivalente a: *bit menos significativo é 0*
* AND bit a bit resolve isso: `t2 = t0 & 1`

```asm
.text
.globl main

main:
    li t0, 1               # contador

loop:
    andi t2, t0, 1         # t2 = t0 & 1
    bne t2, x0, nao_par    # se != 0 → ímpar

eh_par:
    mv a0, t0
    li a7, 1               # print_int
    ecall

nao_par:
    addi t0, t0, 1
    li t1, 11
    blt t0, t1, loop

    li a7, 93
    ecall
```
