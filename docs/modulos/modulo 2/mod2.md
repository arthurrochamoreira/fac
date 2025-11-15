# Módulo 2 — Sessões

Neste módulo você aprende:

* para que serve a sessão `.data`
* como funcionam **labels** (rótulos) como ponteiros
* como declarar strings com `.string` e `.asciz`
* como funciona a sessão `.text`
* como definir o ponto de entrada (`main:`)
* como carregar endereços com `la` (*load address*)

---

# 1. A sessão `.data` — onde ficam os dados do programa

A diretiva **`.data`** (*data section*) indica ao montador que tudo abaixo dela deve ser colocado na **área de dados** do programa: uma região de memória reservada para **valores estáticos**, como:

* números
* vetores
* constantes
* strings

Exemplo simples:

```asm
.data
numero: .word 42
```

Aqui:

* `.word` aloca **4 bytes**
* `numero` é um **label** (rótulo) representando o **endereço** onde o valor 42 está guardado

---

# 2. Labels (label:) como ponteiros

Um **label** (rótulo) é apenas um **nome associado a um endereço** na memória.

Ele funciona como um **ponteiro** em C:

```
label  ----->  endereço onde o dado está colocado
```

Exemplo:

```asm
msg: .string "Olá"
```

O label `msg` aponta para o **primeiro byte** da string.

O label não contém o valor.
O label contém o **endereço** onde o valor está.

Isso é fundamental, porque instruções como `la` (*load address*) funcionam exatamente assim:

```asm
la a0, msg     # a0 recebe o endereço da string 'msg'
```

---

# 3. Declarando strings: `.string` e `.asciz`

Existem duas diretivas para declarar strings:

---

## `.string "texto"`

A diretiva **`.string`** grava a sequência de caracteres **sem byte nulo no final**.

Exemplo:

```asm
msg: .string "Hello"
```

Memória:

```
48 65 6C 6C 6F   (sem 00)
 H  e  l  l  o
```

---

## `.asciz "texto"`

A diretiva **`.asciz`** (*ASCII zero-terminated*) grava a string **seguida de um byte zero**, exatamente como em C.

Exemplo:

```asm
msg: .asciz "Hello"
```

Memória:

```
48 65 6C 6C 6F 00
 H  e  l  l  o \0
```

O byte zero final (`\0`) é **obrigatório** para syscalls como **print_string** no simulador RARS (*RISC-V Assembler and Runtime Simulator*).

---

# 4. A sessão `.text` — onde ficam as instruções

A diretiva **`.text`** (*code section*) marca o início do código executável: todas as instruções do programa.

Exemplo:

```asm
.text
```

Dentro dela ficam comandos como:

* `li` (*load immediate*)
* `la` (*load address*)
* `lw` (*load word*)
* `sw` (*store word*)
* `add`, `sub`
* `ecall` (*environment call*)

---

# 5. O ponto de entrada: `main:`

O RARS procura por um rótulo específico para começar a execução:

```
main:
```

Esse é o **entry point** (ponto de entrada).

Exemplo completo:

```asm
.text
.globl main     # .globl (global): torna 'main' visível como ponto de entrada

main:
    li a0, 5
    li a7, 1
    ecall
```

A diretiva `.globl` (*global symbol*) informa ao montador que este rótulo deve ser acessível como símbolo externo — é assim que o RARS sabe que `main` é seu ponto de início.

---

# 6. Carregar endereços com `la` (load address)

A pseudo-instrução **`la`** (*load address*) carrega o **endereço associado a um label** em um registrador.

É uma pseudo-instrução, ou seja, não existe diretamente no hardware RV32I; o montador traduz `la` em uma sequência de instruções reais (`lui` + `addi`).

Exemplo:

```asm
la a0, msg
```

Interpretação:

> Coloque no registrador `a0` o endereço onde a string `msg` está armazenada.

Isso é fundamental, porque **syscalls usam endereços**, não labels.

---

# 7. Exemplo completo reunindo tudo

```asm
.data
msg: .asciz "Olá, RISC-V!\n"   # string terminada em 0

numero: .word 123              # valor armazenado em 4 bytes

.text
.globl main

main:
    # Imprimir string
    la a0, msg                 # load address: a0 recebe endereço da string
    li a7, 4                   # print_string
    ecall

    # Imprimir número
    lw a0, numero              # load word: a0 recebe valor em 'numero'
    li a7, 1                   # print_int
    ecall

    # Encerrar
    li a7, 93                  # exit2
    ecall
```

---
