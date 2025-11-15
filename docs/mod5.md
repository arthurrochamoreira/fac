# Módulo 5 — Loops e Desvios (`beq`, `bne`, `j`)

Em assembly RISC-V **não existem** palavras como `while`, `for` ou `loop`.
O que existe é:

* **rótulos (labels)** → marcam posições no código
* **desvios (branches)** → instruções que saltam para um rótulo com base em uma condição

Qualquer loop de alto nível se transforma em uma estrutura:

1. inicialização do contador
2. definição de um rótulo para o início do loop
3. corpo do loop
4. atualização do estado (contador, ponteiro, etc.)
5. desvio condicional para repetir

Exemplo mínimo:

```asm
loop:
    ... instruções ...
    beq x0, x0, loop    # desvio incondicional (x0 == x0 sempre)
```

---

# 1. Instruções essenciais de desvio

## 1.1 `beq rs1, rs2, label`

Desvia se `rs1 == rs2`.

## 1.2 `bne rs1, rs2, label`

Desvia se `rs1 != rs2`.

## 1.3 `j label`

Pseudo-instrução de salto incondicional, montada como:

```asm
jal x0, label
```

Como `x0` descarta valores, o salto não registra endereço de retorno.

Ao dominar essas três, você já implementa loops equivalentes a `while` e `for`.

---

# 2. Exemplo 1 — Contagem regressiva de 5 até 1

## Lógica em português

1. `t0 = 5`
2. imprimir `t0`
3. diminuir `t0`
4. enquanto `t0 != 0`, repetir
5. quando `t0 == 0`, encerrar

`t0` é um registrador temporário (grupo `t0–t6`).

## Código completo

```asm
.data
msg: .asciz "Valor: "

.text
.globl main

main:
    li t0, 5                 # contador inicial

loop:
    # imprime "Valor: "
    la a0, msg               # endereço da string
    li a7, 4                 # print_string
    ecall

    # imprime o valor atual
    mv a0, t0
    li a7, 1                 # print_int
    ecall

    # quebra de linha
    li a0, 10                # '\n'
    li a7, 11                # print_char
    ecall

    addi t0, t0, -1          # t0--

    bne t0, x0, loop         # enquanto t0 != 0 → repete

    li a7, 93                # exit2
    ecall
```

---

# 3. Exemplo 2 — Contagem crescente de 1 a 10

Aqui usamos uma instrução adicional:

* **`blt rs1, rs2, label`** → desvia se `rs1 < rs2`.

## Código

```asm
.text
.globl main

main:
    li t0, 1          # contador = 1

loop:
    mv a0, t0
    li a7, 1          # print_int
    ecall

    # quebra de linha
    li a0, 10
    li a7, 11
    ecall

    addi t0, t0, 1    # t0++

    li t1, 11
    blt t0, t1, loop  # enquanto t0 < 11 → repete

    li a7, 93
    ecall
```

---

# 4. Exemplo 3 — Soma de 1 até N

Versão do clássico:

```
soma = 0
i = 1
enquanto i <= N:
    soma += i
    i++
```

Registradores usados:

* `t0` = N
* `t1` = i
* `t2` = soma

Aqui usamos **`ble rs1, rs2, label`** → desvia se `rs1 <= rs2`.

## Código

```asm
.data
msg: .asciz "Soma final: "

.text
.globl main

main:
    li t0, 5              # N = 5
    li t1, 1              # i = 1
    li t2, 0              # soma = 0

loop:
    add t2, t2, t1        # soma += i
    addi t1, t1, 1        # i++

    ble t1, t0, loop      # enquanto i <= N → repete

    la a0, msg
    li a7, 4              # print_string
    ecall

    mv a0, t2
    li a7, 1              # print_int
    ecall

    li a7, 93
    ecall
```

---

# 5. Exemplo 4 — Somar todos os elementos de um array

Cada elemento `.word` ocupa 4 bytes.
Percorremos o vetor avançando o ponteiro em +4 a cada iteração.

Registradores:

* `t0` → ponteiro para elemento atual
* `t1` → índice
* `t2` → soma
* `t3` → elemento atual
* `t4` → tamanho do vetor

## Código

```asm
.data
vet: .word 3, 6, 9, 12
msg: .asciz "Soma: "

.text
.globl main

main:
    la t0, vet         # ponteiro inicial
    li t1, 0           # índice
    li t2, 0           # soma

loop:
    lw t3, 0(t0)       # elemento atual
    add t2, t2, t3     # soma += elemento

    addi t0, t0, 4     # próximo elemento
    addi t1, t1, 1     # índice++

    li t4, 4           # tamanho do vetor (4 elementos)
    blt t1, t4, loop   # enquanto i < 4 → repete

    la a0, msg
    li a7, 4
    ecall

    mv a0, t2
    li a7, 1
    ecall

    li a7, 93
    ecall
```

---

# 6. Molde geral de um loop no RISC-V

Você pode usar este template para qualquer loop:

```asm
    # inicialização

loop:
    # corpo do loop

    # atualização do estado

    # condição de repetição
    beq / bne / blt / ble / bge / bgt ... loop
```

Esse formato mapeia diretamente para `while` e `for`.

---

# 7. Exercícios propostos

1. Imprimir os números de **10 até 1**
2. Imprimir apenas os **pares de 2 até 20**
3. Somar todos os **2 × i**, para `i` de 1 a 10
4. Percorrer um vetor de 6 posições e contar quantos são **> 50**
5. Ler um número **N** do usuário e imprimir de 1 até N
