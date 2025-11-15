# Módulo 4 — Aritmética Básica

(`add`, `sub`)

A aritmética no **RISC-V (ISA — Instruction Set Architecture)** é extremamente simples e direta.
O conjunto básico **RV32I (Integer instruction set de 32 bits)** traz instruções para somar, subtrair e adicionar imediatos.
Este módulo cobre apenas a aritmética essencial: **somar** e **subtrair**.

---

# 1. Por que aritmética em assembly é tão importante?

Toda operação lógica, toda condição, todo loop, toda função… acaba dependendo do básico:

* somar índices
* subtrair contadores
* atualizar ponteiros
* calcular deslocamentos de memória
* somar acumuladores
* fazer incrementos e decrementos

No RISC-V, tudo isso é feito com poucas instruções simples e totalmente determinísticas.

---

# 2. A instrução `add` — somar registradores

Primeira ocorrência:

**`add` (add) é uma instrução real do RV32I que soma dois registradores e guarda o resultado em um terceiro.**

Sintaxe:

```asm
add rd, rs1, rs2
```

Significa:

> `rd = rs1 + rs2`

Exemplo:

```asm
li t0, 7              # li (load immediate): coloca 7 em t0
li t1, 5              # coloca 5 em t1
add t2, t0, t1        # t2 = 7 + 5 = 12
```

* Somente registradores podem ser somados.
* O resultado sempre fica no registrador `rd`.

---

# 3. A instrução `addi` — somar com imediato

Primeira ocorrência:

**`addi` (add immediate) é a versão de soma onde o segundo operando é um valor imediato de 12 bits (constante inteira).**

Sintaxe:

```asm
addi rd, rs1, imediato
```

Significa:

> `rd = rs1 + imediato`

Exemplos úteis:

### Incrementar um contador

```asm
addi t0, t0, 1        # t0 = t0 + 1
```

### Decrementar

```asm
addi t0, t0, -1       # t0 = t0 - 1
```

### Somar deslocamento de ponteiro

```asm
addi t0, t0, 4        # avança 1 word (4 bytes)
```

---

# 4. A instrução `sub` — subtração

Primeira ocorrência:

**`sub` (subtract) é uma instrução do RV32I que calcula a diferença entre dois registradores.**

Sintaxe:

```asm
sub rd, rs1, rs2
```

Significa:

> `rd = rs1 - rs2`

Exemplo:

```asm
li t0, 20
li t1, 7
sub t2, t0, t1        # t2 = 20 - 7 = 13
```

---

# 5. Aritmética e flags — importante!

Diferente de arquiteturas como x86, o RISC-V **não** possui flags (carry, overflow, zero, negativo).
Por isso, operações matemáticas **não alteram estados especiais**.

Comparações e desvios são feitos **explicitamente** com instruções de branch (`beq`, `bne`, `blt` etc.), não com flags automáticos.

---

# 6. Exemplos completos

## 6.1 Somar dois números

```asm
li t0, 15              # primeiro número
li t1, 27              # segundo número
add t2, t0, t1         # t2 = 42
```

---

## 6.2 Subtrair dois números

```asm
li t0, 50
li t1, 12
sub t2, t0, t1         # t2 = 38
```

---

## 6.3 Incrementar contador até atingir limite

```asm
li t0, 0               # contador
li t1, 10              # limite

loop:
    addi t0, t0, 1     # contador++
    blt t0, t1, loop   # repete até t0 < 10
```

---

## 6.4 Manipular ponteiros para array

```asm
.data
vet: .word 3, 6, 9, 12

.text
main:
    la t0, vet         # t0 = endereço do vetor
    lw t1, 0(t0)       # lê vet[0]
    addi t0, t0, 4     # t0 agora aponta para vet[1]
    lw t2, 0(t0)       # lê vet[1]
```

---

# 7. Exercícios para fixação

### 1. Some os números 12 e 31 e coloque o resultado em `t2`.

### 2. Subtraia 100 − 37 usando `sub`.

### 3. Implemente um contador que começa em 10 e vai até 0 usando `addi`.

### 4. Some todos os elementos de um vetor de 5 posições usando:

* `add`
* `addi`
* `lw`

### 5. Calcule `t2 = (t0 + 4) - (t1 - 3)` usando somente `add`, `sub` e `addi`.

