# Módulo 4 — Ler dois valores do usuário, somar e imprimir

Neste módulo você aprenderá a:

* ler valores digitados pelo usuário
* armazená-los em registradores
* realizar operações aritméticas
* imprimir um resultado na tela usando syscalls

Esse é um dos primeiros fluxos completos de entrada–processamento–saída em RISC-V.

---

# 1. As syscalls utilizadas

O **RARS** (*RISC-V Assembler and Runtime Simulator*) é um simulador que permite executar programas RISC-V e oferece serviços chamados **syscalls** (*system calls*). Cada syscall é identificada por um **código numérico**, colocado no registrador **`a7`**, e executada com a instrução **`ecall`** (*environment call*).

A comunicação com syscalls usa principalmente:

* **`a7`**: registrador que especifica *qual* syscall usar
* **`a0`**: registrador que carrega argumentos ou recebe resultados

Para este módulo, utilizaremos:

---

## 1.1 Ler inteiro — syscall 5

```asm
li a7, 5       # selecionar read_int
ecall
```

Resultado após `ecall`:

* o número digitado pelo usuário vai para **`a0`**

---

## 1.2 Imprimir inteiro — syscall 1

```asm
mv a0, valor   # argumento a imprimir
li a7, 1       # selecionar print_int
ecall
```

---

## 1.3 Encerrar o programa — syscall 93

```asm
li a7, 93      # exit2
ecall
```

Essa syscall finaliza a execução corretamente.

---

# 2. Passo a passo da lógica (em linguagem natural)

1. Mostrar uma mensagem pedindo o **primeiro número**
2. Ler o número com a syscall read_int → valor cai em `a0`
3. Mover esse valor para `t0` (registrador temporário)
4. Repetir o processo para o **segundo número** → guardar em `t1`
5. Somar `t0 + t1` e guardar o resultado em `t2`
6. Copiar `t2` para `a0` (argumento da syscall print_int)
7. Chamar a syscall de imprimir inteiro
8. Encerrar o programa com a syscall exit2

Isso forma seu primeiro “programa interativo” em RISC-V.

---

# 3. Código completo e comentado

```asm
.data
msg1: .asciz "Digite o primeiro numero: "   # .asciz = string terminada em 0
msg2: .asciz "Digite o segundo numero: "
msgR: .asciz "Resultado: "

.text
.globl main

main:
    # ---- Ler primeiro número ----
    la a0, msg1                # endereço da string msg1
    li a7, 4                   # syscall print_string
    ecall

    li a7, 5                   # syscall read_int
    ecall                      # usuário digita → valor cai em a0
    mv t0, a0                  # salva o valor lido em t0 (registrador temporário)

    # ---- Ler segundo número ----
    la a0, msg2
    li a7, 4                   # print_string
    ecall

    li a7, 5                   # read_int
    ecall
    mv t1, a0                  # salva o valor lido em t1

    # ---- Soma ----
    add t2, t0, t1             # t2 = t0 + t1

    # ---- Imprimir resultado ----
    la a0, msgR                # imprime prefixo "Resultado: "
    li a7, 4
    ecall

    mv a0, t2                  # coloca o resultado em a0
    li a7, 1                   # print_int
    ecall

    # ---- Encerrar ----
    li a7, 93                  # exit2
    ecall
```

---

# 4. Como testar no RARS

1. Abra o arquivo `.asm` no RARS
2. Clique em **Assemble**
3. Clique em **Run → Go**
4. O console solicitará os números
5. Digite os dois valores desejados

Saída esperada:

```
Digite o primeiro numero: 7
Digite o segundo numero: 4
Resultado: 11
```