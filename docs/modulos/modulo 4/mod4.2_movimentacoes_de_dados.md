# Módulo 4 — Movimentação de Dados

(`mv`, `lw`, `sw`, `la`)

Este módulo explica como mover dados entre **registradores** e **memória** no RISC-V
(RISC-V = *Reduced Instruction Set Computer – Five*;
ISA — *Instruction Set Architecture*, isto é, o conjunto de instruções que o processador entende).

Essas instruções são a base de **todo** programa: somar, comparar, chamar função, fazer loop — tudo depende de colocar os dados nos registradores certos na hora certa.

---

## 1. Visão geral: registradores e memória

Antes de aprender as instruções, precisamos entender **onde os dados vivem**.

O processador (CPU — *Central Processing Unit*) trabalha basicamente com:

* **Registradores** — pequenas células internas, extremamente rápidas
  Exemplos:

  * `t0` (*temporary register*),
  * `a0` (*argument register*),
  * `s0` (*saved register*).
    Esses apelidos são nomes “amigáveis” para registradores `x5`, `x10`, `x8` etc.

* **Memória RAM** (RAM — *Random Access Memory*) — área grande, externa ao núcleo do processador, mas **mais lenta**
  É onde ficam:

  * dados da sessão `.data`
  * arrays, strings, matrizes
  * variáveis globais, buffers, etc.

O RISC-V segue o modelo **load/store**:

> O processador **só faz operações aritméticas e lógicas com dados que já estão em registradores**.
> Para buscar ou gravar na memória, usamos instruções específicas de **load** (carregar) e **store** (armazenar).

Por isso `lw` e `sw` são tão importantes.

---

## 2. `mv` — mover valor entre registradores

`mv` (*move*) é uma **pseudo-instrução**.
Pseudo-instrução = instrução que **não existe no hardware**, mas o montador converte automaticamente para uma ou mais instruções reais.

Sintaxe:

```asm
mv rd, rs
```

Significa:

> Copiar o valor do registrador `rs` para o registrador `rd`.

Exemplo:

```asm
li t0, 42              # li (load immediate): carrega o valor 42 em t0
mv a0, t0              # mv (move): copia o valor de t0 para a0
```

Internamente, o montador converte:

```asm
mv rd, rs
```

para:

```asm
addi rd, rs, 0         # addi (add immediate): rd = rs + 0
```

---

## 3. `lw` — load word (carregar palavra da memória)

`lw` (*load word*) é uma **instrução real** do conjunto básico RV32I
(RV32I = versão de 32 bits do RISC-V para inteiros, *RISC-V 32-bit Integer*).

* **Word** = palavra de **32 bits** (4 bytes).
* A memória do RISC-V é **endereçada por bytes** (*byte-addressable*): cada endereço aponta para 1 byte.

Sintaxe:

```asm
lw rd, offset(rs)
```

Significa:

> Ler 4 bytes (1 word) da memória no endereço `rs + offset`
> e colocar o resultado no registrador `rd`.

### Exemplo básico de `lw`

Aqui aparecem algumas diretivas pela primeira vez:

* `.data` — inicia a **sessão de dados**, onde declaramos variáveis na memória.
* `.word` — reserva 4 bytes (1 word) e grava um valor inteiro.
* `.text` — inicia a **sessão de código**, onde ficam as instruções.
* `.globl main` — torna o rótulo `main` o **ponto de entrada** do programa.

```asm
.data
x: .word 99                 # .word: reserva 4 bytes e coloca o inteiro 99 em x

.text
.globl main                 # .globl (global): expõe 'main' como ponto de entrada

main:
    la t0, x                # la (load address): carrega em t0 o endereço do rótulo x
    lw t1, 0(t0)            # lw (load word): lê 4 bytes a partir de t0 e coloca o valor em t1 (99)

    li a7, 93               # li: coloca 93 em a7 (código da syscall exit2)
    ecall                   # ecall (environment call): executa a syscall indicada em a7
```

Passo a passo:

1. `la t0, x` → `t0` recebe o endereço de memória onde está `x`.
2. `lw t1, 0(t0)` → lê 4 bytes a partir de `t0` e coloca o valor (99) em `t1`.

---

## 4. `sw` — store word (armazenar palavra na memória)

`sw` (*store word*) grava um valor de 32 bits **da CPU para a memória**.

Sintaxe:

```asm
sw rs2, offset(rs1)
```

Interpretação:

> Armazena o valor de 32 bits do registrador `rs2` no endereço `rs1 + offset`.

### Exemplo básico de `sw`

```asm
.data
y: .word 0                  # reserva 4 bytes e inicializa com 0

.text
.globl main

main:
    li t0, 55               # li (load immediate): t0 recebe o valor 55
    la t1, y                # la (load address): t1 recebe o endereço de y
    sw t0, 0(t1)            # sw (store word): grava o valor de t0 (55) na memória em y

    li a7, 93
    ecall
```

Depois de executar:

* o registrador `t0` contém 55,
* a posição de memória associada a `y` também passa a conter 55.

---

## 5. `la` — load address (carregar endereço)

`la` (*load address*) é uma **pseudo-instrução** que carrega o **endereço de um rótulo** (`label`) para dentro de um registrador.

Sintaxe:

```asm
la rd, label
```

Significa:

> Colocar no registrador `rd` o endereço de memória associado ao rótulo `label`.

### Exemplo com string e syscall

Aqui aparece pela primeira vez:

* `.asciz` — diretiva que cria uma string ASCII terminada em byte zero (`\0`),
  ASCII = *American Standard Code for Information Interchange*.
* `syscall` — chamada de sistema (*system call*), serviço fornecido pelo simulador.
* `a7` como registrador de **código de syscall**.
* `a0` como registrador de **argumento**.

```asm
.data
msg: .asciz "Hello\n"       # .asciz: string ASCII terminada em 0 (formato C-string)

.text
.globl main

main:
    la a0, msg              # la: a0 recebe o endereço do início da string msg
    li a7, 4                # li: coloca 4 em a7 (código da syscall print_string)
    ecall                   # ecall: executa a syscall configurada (imprime a string)

    li a7, 93               # 93 = exit2
    ecall
```

Nos bastidores, o montador pode traduzir `la` para algo como:

* `auipc` (*add upper immediate to PC*) — soma imediato à parte alta do PC
  PC = *program counter*, registrador que guarda o endereço da próxima instrução.
* `addi` (*add immediate*) — soma imediato à parte baixa.

Mas para quem está aprendendo, basta saber:

> `la` devolve o **endereço** do rótulo.

---

## 6. Mapa mental — quando usar cada instrução

| Tarefa                                 | Instrução                     |
| -------------------------------------- | ----------------------------- |
| Copiar valor entre registradores       | `mv` (move)                   |
| Colocar número imediato em registrador | `li` (load immediate, pseudo) |
| Ler inteiro (32 bits) da memória       | `lw` (load word)              |
| Gravar inteiro (32 bits) na memória    | `sw` (store word)             |
| Obter endereço de um rótulo na memória | `la` (load address, pseudo)   |

---

## 7. Exercício 1 — copiar, gravar e ler

Objetivo:

1. Colocar um valor em `t0`
2. Guardar esse valor na memória
3. Carregar de volta para `t2`

Código de referência:

```asm
.data
valor: .word 0                   # reserva 4 bytes para 'valor'

.text
.globl main

main:
    li t0, 123                   # li: t0 = 123

    la t1, valor                 # la: t1 recebe o endereço de 'valor'
    sw t0, 0(t1)                 # sw: grava o conteúdo de t0 na memória em 'valor'

    lw t2, 0(t1)                 # lw: carrega de volta o valor armazenado em 'valor' para t2

    li a7, 93
    ecall
```

No RARS, observe:

* painel **Registers** → valores em `t0`, `t1`, `t2`
* painel **Data** → valor de `valor` na memória

---

## 8. Exercício 2 — vetor (array) de 4 elementos

Objetivo:

* Somar os valores de um array usando `lw` para ler e `add` para acumular.

Aqui aparece pela primeira vez:

* `add` — *add* (somar dois registradores):
  `add rd, rs1, rs2` → `rd = rs1 + rs2`
* `addi` — *add immediate* (somar imediato):
  `addi rd, rs1, imediato` → `rd = rs1 + imediato`
* `blt` — *branch if less than* (desvia se menor que):
  `blt rs1, rs2, label` → desvia se `rs1 < rs2`.

```asm
.data
vet: .word 3, 5, 7, 9           # quatro inteiros

.text
.globl main

main:
    la t0, vet                  # la: t0 = endereço do primeiro elemento (vet[0])
    li t1, 0                    # li: t1 = índice i = 0
    li t2, 0                    # li: t2 = soma acumulada = 0

loop:
    lw t3, 0(t0)                # lw: t3 = vet[i]
    add t2, t2, t3              # add: t2 = t2 + t3  (soma += elemento)

    addi t0, t0, 4              # addi: avança ponteiro para o próximo elemento (4 bytes)
    addi t1, t1, 1              # addi: i = i + 1    (incrementa índice)

    li t4, 4                    # li: t4 = 4 (tamanho do vetor)
    blt t1, t4, loop            # blt: enquanto i < 4, volta para 'loop'

    # aqui t2 contém a soma total (3 + 5 + 7 + 9)

    li a7, 93
    ecall
```

---
