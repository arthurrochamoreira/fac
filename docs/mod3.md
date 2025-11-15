# Módulo 3 — como armazenar valores na memória (sw)

# 1. A instrução necessária: `sw`

A instrução:

```
sw rs2, offset(rs1)
```

`sw` significa *store word*, isto é, **armazenar uma word** (4 bytes = 32 bits) na memória.

Ela funciona assim:

1. pega o valor do registrador **rs2**
2. pega o endereço do registrador **rs1**
3. soma o **offset** (deslocamento, em bytes)
4. grava o conteúdo de `rs2` nos 4 bytes começando em `rs1 + offset`

Exemplo:

```
sw t0, 0(t1)
```

Lê-se:

> “Guardar o conteúdo de `t0` no endereço apontado por `t1`.”

**Importante:**
No RISC-V, **cada endereço representa 1 byte**, por isso deslocamentos são múltiplos de 4 para `.word`.

---

# 2. Declarando espaço para armazenar valores

Antes de escrever na memória, é preciso reservar espaço na seção `.data`.

Existem três formas principais:

---

## 2.1 `.word` — reserva e inicializa

```
x: .word 0
```

* Aloca **4 bytes**
* Já inicializa com o valor indicado
* Garante **alinhamento perfeito** (múltiplo de 4)

---

## 2.2 Várias `.word` — arrays

```
buffer: .word 0, 0, 0, 0
```

* Cria 4 palavras consecutivas na memória
* Cada uma separada por **4 bytes**

---

## 2.3 `.space` — apenas reserva (não inicializa)

```
area: .space 16     # 16 bytes = 4 words
```

* Reserva **espaço bruto**, sem inicializar
* Você **precisa garantir alinhamento manualmente**

---

# 3. Primeiro exercício guiado: guardar e ler de volta

Vamos:

1. Colocar o valor 25 em um registrador
2. Armazenar o valor na memória
3. Ler novamente para confirmar

---

## 3.1 Código completo e comentado

```asm
.data
valor: .word 0                   # reserva 4 bytes para armazenar um inteiro

.text
.globl main

main:
    li t0, 25                    # t0 = 25
    la t1, valor                 # t1 = endereço de 'valor'
    sw t0, 0(t1)                 # armazena 25 na memória (posição valor)

    lw t2, 0(t1)                 # lê de volta: t2 deve = 25

    li a7, 93                    # syscall exit2
    ecall
```

Resultado no RARS:

* `t2` conterá `25`
* A memória mostrará: `0x00000019` (19 hex = 25 decimal)

---

# 4. Gravando valores em um array

Como `.word` alinha automaticamente, cada elemento fica separado por **4 bytes**.

Vamos gravar 4 valores em um vetor:

---

## 4.1 Código

```asm
.data
vetor: .word 0, 0, 0, 0          # 4 words consecutivas (16 bytes)

.text
.globl main

main:
    la t0, vetor                 # t0 = endereço do vetor[0]

    li t1, 100
    sw t1, 0(t0)                 # vetor[0] = 100

    li t1, 200
    sw t1, 4(t0)                 # vetor[1] = 200

    li t1, 300
    sw t1, 8(t0)                 # vetor[2] = 300

    li t1, 400
    sw t1, 12(t0)                # vetor[3] = 400

    li a7, 93
    ecall
```

Observação:
Offsets da forma `4 * índice`.

---

# 5. Observação fundamental: alinhamento

**Alinhamento** é a exigência de que dados de tamanho N bytes estejam armazenados em endereços múltiplos de N.

No RISC-V:

* **word (4 bytes)** → endereço múltiplo de 4
* **halfword (2 bytes)** → endereço múltiplo de 2

`.word` cuida disso automaticamente.

`.space` **não cuida**, então você deve garantir manualmente.

Exemplo de armadilha:

```
.space 1
.word 5    # pode ficar desalinhado se você não ajustar
```

---

# 6. A sequência geral: escrever e ler

Sempre siga este padrão:

```
la t0, variavel        # pegar endereço
sw t1, 0(t0)           # escrever valor na memória
lw t2, 0(t0)           # ler valor de volta
```

Isso resolve 95% das dúvidas com memória no RISC-V.

---

# 7. Exercícios para você

### Exercício 1

Guarde o valor **57** na memória usando `sw` e depois imprima com a syscall de inteiro (`a7 = 1`).

### Exercício 2

Crie um array de 5 posições e grave:
10, 20, 30, 40, 50.
Depois some tudo em um registrador.

### Exercício 3

Crie uma área `.space 12` (12 bytes = 3 words) e escreva **3 valores** nela usando `sw`.