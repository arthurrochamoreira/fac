# Módulo 4 — Instruções de Imediato (`li`, `addi`)

As instruções **de imediato** são fundamentais no RISC-V
(ISA — *Instruction Set Architecture*: conjunto de instruções que o processador entende),
porque permitem trabalhar com **valores constantes** sem acessar a memória.

Neste módulo você vai ver:

* o que é um **imediato** (*immediate*)
* como carregar um valor constante direto em um registrador com `li`
* como somar um registrador com uma constante usando `addi`
* como aplicar essas instruções em loops, aritmética e ponteiros

---

## 1. O que é um “valor imediato”?

Primeira ocorrência:

Um **imediato** (*immediate*) é um valor **constante** escrito **diretamente na instrução**, e não buscado da memória.

Exemplo conceitual:

```asm
addi t0, t0, 1     # addi (add immediate): soma o imediato 1 ao valor em t0 e guarda em t0
```

Aqui:

* `t0` é um registrador temporário (*temporary register*)
* `1` é o **imediato** embutido na instrução

Diferente de:

```asm
lw t0, 0(t1)       # lw (load word): carrega da memória
```

onde o valor vem da **memória**, não está “dentro” da instrução.

---

## 2. A pseudo-instrução `li` — *load immediate*

Primeira ocorrência:

`li` (*load immediate*) é uma **pseudo-instrução** do montador (não existe no hardware real RV32I) que carrega um valor imediato em um registrador.

Sintaxe:

```asm
li rd, imediato
```

Significa:

> Colocar o valor constante `imediato` dentro do registrador `rd`.

Exemplos:

```asm
li t0, 10           # li (load immediate): coloca o valor imediato 10 no registrador t0
li a0, -5           # li (load immediate): coloca -5 em a0
li s1, 2048         # li (load immediate): coloca 2048 em s1
```

### Como o montador expande `li`

Como `li` é uma pseudo-instrução, o montador do RARS (*RISC-V Assembler and Runtime Simulator*) traduz para uma ou mais instruções reais, por exemplo:

* imediatos pequenos → `addi`
* imediatos grandes → combinação de `lui` (*load upper immediate*) + `addi`

Você **não precisa** se preocupar com isso no começo: use `li` livremente para carregar constantes.

---

## 3. A instrução `addi` — *add immediate*

Primeira ocorrência:

`addi` (*add immediate*) é uma **instrução real** do conjunto RV32I que soma o conteúdo de um registrador com um **imediato de 12 bits** (intervalo de -2048 até +2047).

Sintaxe:

```asm
addi rd, rs1, imediato
```

Interpretação:

> `rd = rs1 + imediato`

Exemplos:

```asm
addi t0, t0, 1      # addi (add immediate): t0 = t0 + 1  (incremento)
addi t1, t1, -1     # addi (add immediate): t1 = t1 - 1  (decremento)
addi a0, a0, 10     # addi (add immediate): a0 = a0 + 10
addi t2, t0, 4      # addi (add immediate): t2 = t0 + 4
```

Se o imediato for grande demais para caber em 12 bits, você geralmente usa uma combinação de `li`, `lui` + `addi` — mas para a maioria dos exemplos iniciais, `addi` é suficiente.

---

## 4. Usos comuns de `li` e `addi`

### 4.1 Inicializar “variáveis” em registradores

```asm
li t0, 0            # li (load immediate): inicializa t0 como contador com 0
li t1, 10           # li (load immediate): define limite 10 em t1
```

---

### 4.2 Criar loops com incrementos e decrementos

Aqui aparece **`bne` (branch if not equal)** pela primeira vez:

* `bne rs1, rs2, label` desvia para `label` se `rs1 != rs2`.

```asm
li t0, 5            # t0 começa em 5

loop:
    addi t0, t0, -1 # addi (add immediate): t0 = t0 - 1  (decrementa)
    bne t0, x0, loop# bne (branch if not equal): se t0 != 0, volta para 'loop'
```

---

### 4.3 Manipular ponteiros (endereços) em vetores

Primeira ocorrência de:

* `.data` — diretiva que inicia a **sessão de dados**
* `.text` — diretiva que inicia a **sessão de código**
* `la` — *load address*, carrega o **endereço** de um label
* `lw` — *load word*, lê 4 bytes da memória para um registrador

Em vetores declarados com `.word`, cada elemento ocupa **4 bytes**:

```asm
.data
vetor: .word 10, 20, 30, 40   # vetor de 4 inteiros (4 bytes cada)

.text
.globl main                   # .globl (global): torna 'main' ponto de entrada

main:
    la t0, vetor              # la (load address): t0 recebe o endereço do primeiro elemento de 'vetor'
    lw t1, 0(t0)              # lw (load word): t1 recebe vetor[0]
    addi t0, t0, 4            # addi: avança o ponteiro para o próximo elemento (vetor[1])

    li a7, 93                 # li: escolhe syscall exit2 (código 93)
    ecall                     # ecall (environment call): executa a syscall
```

---

### 4.4 Criar deslocamentos para cálculos

```asm
addi t2, t1, 100    # addi: t2 = t1 + 100 (deslocamento de 100 bytes)
```

---

## 5. Exemplos completos

### 5.1 Incremento simples com limite

Primeira ocorrência de `blt` (*branch if less than*): desvia se `rs1 < rs2`.

```asm
li t0, 0                # t0 = 0 (contador)
li t1, 10               # t1 = 10 (limite)

loop:
    addi t0, t0, 1      # addi (add immediate): t0 = t0 + 1
    blt t0, t1, loop    # blt (branch if less than): se t0 < t1, volta para 'loop'

    li a7, 93           # li: escolhe syscall exit2
    ecall               # encerra o programa
```

---

### 5.2 Decremento até zero

```asm
li t0, 7                # t0 começa em 7

loop:
    addi t0, t0, -1     # addi: t0 = t0 - 1
    bne t0, x0, loop    # bne: enquanto t0 != 0, repete o loop

    li a7, 93
    ecall
```

---

### 5.3 Somando com imediatos e usando `add`/`sub`

Primeira ocorrência de:

* `add` — *add*: soma dois registradores
  `add rd, rs1, rs2` → `rd = rs1 + rs2`
* `sub` — *subtract*: subtrai registradores
  `sub rd, rs1, rs2` → `rd = rs1 - rs2`

```asm
li t0, 12               # t0 = 12
li t1, 30               # t1 = 30

addi t2, t1, -12        # addi: t2 = t1 + (-12) = 30 - 12
add t3, t0, t2          # add (add): t3 = t0 + t2 = 12 + 18

sub t4, t3, t0          # sub (subtract): t4 = t3 - t0 = 30 - 12
```

---

### 5.4 Percorrer vetor usando `addi` para andar na memória

```asm
.data
vet: .word 3, 6, 9, 12          # vetor com 4 inteiros

.text
.globl main

main:
    la t0, vet                  # la: t0 = endereço base de vet
    li t1, 0                    # t1 = índice i
    li t2, 4                    # t2 = tamanho (4 elementos)

loop:
    lw t3, 0(t0)                # lw: t3 = vet[i]
    # aqui poderíamos fazer algo com t3 (somar, imprimir etc.)

    addi t0, t0, 4              # addi: avança ponteiro para o próximo elemento
    addi t1, t1, 1              # addi: i = i + 1

    blt t1, t2, loop            # blt: enquanto i < 4, continua no loop

    li a7, 93
    ecall
```

---

## 6. Exercícios para fixação

1. Carregue o valor **57** em `a0` usando `li`.
2. Some **10** ao registrador `t1` usando `addi`.
3. Implemente um contador que começa em **1** e vai até **20** usando `li`, `addi` e `blt` ou `bne`.
4. Dado um vetor de 4 `word`, avance o ponteiro até o último elemento usando `addi` (de 4 em 4 bytes).
5. Implemente `t3 = (t0 + 4) - (t1 + 7)` usando apenas:

   * `li`
   * `addi`
   * `add`
   * `sub`
