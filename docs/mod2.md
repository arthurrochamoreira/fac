# Módulo 2 — Carregar um valor da memória para um registrador

Este módulo ensina exatamente o que você precisa para pegar um valor gravado na memória (na seção `.data`) e colocá-lo dentro de um **registrador** — uma pequena área de armazenamento interno do processador RISC-V.

---

# 1. Como a memória funciona no RISC-V

A memória é uma sequência linear de **endereços**, e **cada endereço armazena 1 byte** (8 bits).
Isso significa que a memória do RISC-V é **byte-addressable** — cada endereço aponta para um byte individual.

Quando você declara uma variável com `.word`, está reservando **4 bytes** (32 bits), e o montador garante que esses 4 bytes estarão **alinhados** em um endereço múltiplo de 4.

Exemplo:

```asm
.data
valor: .word 10
```

Aqui:

* `valor` é um **rótulo** — um nome simbólico para um endereço de memória.
* No endereço representado por `valor`, estão gravados os **4 bytes** que representam o número 10.

---

# 2. As duas etapas obrigatórias para ler valores da memória

Para carregar um valor da memória para um registrador, sempre siga estes dois passos:

1. Carregue o **endereço** da variável para um registrador (usando `la`, uma *pseudo-instrução*).
2. Carregue o **conteúdo** desse endereço para outro registrador (usando `lw`, *load word*).

Importante:

> **Não existe instrução que leia o valor diretamente de um label.**
> Labels são *nomes de endereços*, não valores.

Erro comum (não funciona):

```asm
lw t0, valor     # ERRADO! 'valor' não é um valor, é um endereço.
```

---

# 3. As instruções necessárias

## 3.1 `la rd, label` — *load address*

Carrega o **endereço** do rótulo `label` no registrador `rd`.
É uma **pseudo-instrução** (não existe no hardware real), mas o montador converte automaticamente para instruções válidas RV32I.

## 3.2 `lw rd, offset(rs)` — *load word*

Lê **4 bytes** a partir do endereço `rs + offset`
e coloca o conteúdo em `rd`.

Como cada endereço aponta para 1 byte, o próximo elemento `.word` fica a +4.

Exemplo:

```asm
lw t0, 0(t1)
```

Significa:

> “carregar em `t0` a word armazenada no endereço contido em `t1`”.

---

# 4. Primeiro exemplo completo

## 4.1 Objetivo em linguagem simples

Queremos:

* Criar uma variável com valor 10
* Obter seu endereço
* Ler o valor de volta para um registrador

## 4.2 Código

```asm
.data
numero: .word 10                  # variável 'numero'

.text
.globl main

main:
    la t1, numero                 # t1 = endereço da variável 'numero'
    lw t0, 0(t1)                  # t0 = conteúdo armazenado no endereço t1 (10)

    li a7, 93                     # syscall exit2
    ecall
```

Após executar no RARS:

* `t1` conterá um endereço como `0x10010000`
* `t0` conterá o valor `10`

---

# 5. Segundo exemplo — duas variáveis na memória

```asm
.data
a: .word 5
b: .word 7

.text
.globl main

main:
    la t0, a
    lw t1, 0(t0)                # t1 = 5

    la t0, b
    lw t2, 0(t0)                # t2 = 7

    add t3, t1, t2              # t3 = 12

    li a7, 93
    ecall
```

---

# 6. Terceiro exemplo — array de `.word`

Cada elemento de um array declarado com `.word` ocupa 4 bytes consecutivos:

```asm
.data
valores: .word 3, 6, 9, 12       # array contíguo na memória

.text
.globl main

main:
    la t0, valores               # t0 = endereço do primeiro elemento

    lw t1, 0(t0)                 # lê 3
    lw t2, 4(t0)                 # lê 6
    lw t3, 8(t0)                 # lê 9
    lw t4, 12(t0)                # lê 12

    li a7, 93
    ecall
```

Como os endereços são em bytes:

* índice 0 → offset 0
* índice 1 → offset 4
* índice 2 → offset 8
* índice 3 → offset 12

---

# 7. Exercício para você

Crie um programa que:

1. Declare duas variáveis de memória com os valores `20` e `15`
2. Carregue ambas com `la` + `lw`
3. Faça a subtração `20 - 15`
4. Imprima o resultado com a syscall de inteiro (código 1)
