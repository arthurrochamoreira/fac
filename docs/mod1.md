# Modulo 1 - Introdução

# 1. O que é RISC-V?

RISC-V é uma **arquitetura de conjunto de instruções**
(ISA — *Instruction Set Architecture*).
Uma **ISA** é o “vocabulário” que o processador entende: as operações básicas que ele sabe executar.

* **RISC** (*Reduced Instruction Set Computer*) = arquitetura com poucas instruções, simples e rápidas.
* **V** = quinta geração dessa família RISC.
* **RV32I** = versão de 32 bits (32 = largura dos registradores) com o conjunto básico de instruções inteiras (I = *Integer*).

Um processador RISC-V só entende operações fundamentais, como:

* mover valores entre **registradores** (pequenas memórias internas do processador)
* ler/escrever na **memória**
* fazer operações aritméticas
* saltar para outro ponto do código

### Observação importante

Os registradores possuem **apelidos educativos** (`a0`, `t0`, `s0` etc.),
mas internamente **eles continuam sendo** `x0`, `x1`, …, `x31`.
Por exemplo:

* `a0` = `x10`
* `a7` = `x17`
* `t0` = `x5`

Isso aparece no painel “Registers” do RARS.

---

# 2. O que é o RARS?

O **RARS** (*RISC-V Assembler and Runtime Simulator*) é um **simulador**, ou seja:

> Um programa que imita um processador RISC-V para permitir que você execute código sem ter o hardware real.

Ele funciona assim:

1. Você escreve um arquivo `.asm`
   (arquivo contendo código em **assembly**, a linguagem textual da ISA).
2. O RARS **monta** o código
   (*assembler* = traduz para instruções binárias reais).
3. O RARS **executa** e mostra o que acontece nos registradores e na memória.

Além disso, o RARS oferece **syscalls** (*system calls*): serviços prontos como imprimir texto ou ler números do usuário.

---

# 3. Registradores: sua caixa de ferramentas

Um **registrador** é uma pequena área de armazenamento dentro do processador.
É muito mais rápido acessar registradores do que acessar memória.

O RISC-V possui **32 registradores inteiros**, de `x0` a `x31`, cada um com um apelido:

| Nome     | Uso                                                     |
| -------- | ------------------------------------------------------- |
| `x0`     | sempre contém o valor 0                                 |
| `a0–a7`  | argumentos de funções e syscalls (*argument registers*) |
| `t0–t6`  | temporários (*temporary registers*)                     |
| `s0–s11` | variáveis preservadas (*saved registers*)               |
| `sp`     | ponteiro da pilha (*stack pointer*)                     |
| `ra`     | endereço de retorno (*return address*)                  |

Para o Hello World utilizamos principalmente:

* **`a0`** — registrador que contém o argumento da syscall
* **`a7`** — registrador que contém o código da syscall

---

# 4. Formato básico das instruções

Uma **instrução** é um comando da ISA.
No RV32I, muitos comandos seguem o formato:

```
instrução destino, fonte1, fonte2
```

Exemplos:

```
add t0, t1, t2   # add = soma: t0 recebe t1 + t2
li a0, 5         # li = load immediate: coloca 5 dentro de a0
la a0, msg       # la = load address: carrega o endereço do rótulo msg
lw t1, 0(t0)     # lw = load word: lê 4 bytes da memória no endereço t0
```

### Nota técnica

No RISC-V, todos os endereços se referem a **1 byte** (memória byte-addressable).
Um `lw` lê 4 bytes a partir daquele endereço.

---

# 5. Pseudo-instruções

O montador do RARS fornece **pseudo-instruções**, que não existem no hardware real, mas tornam o código mais fácil de escrever:

* `li` — *load immediate*: carrega um valor imediato
* `la` — *load address*: carrega o endereço de um rótulo
* `mv` — *move*: copia o valor de um registrador para outro

O montador converte essas pseudo-instruções em **instruções reais RV32I** durante a montagem.

---

# 6. Sessões do programa

Um arquivo `.asm` é dividido em sessões:

## `.data` — dados estáticos na memória

```
.data
msg: .asciz "Hello, World!\n"
```

`.asciz` define uma string **terminada em byte zero** (`\0`).
Isso é importante porque a syscall 4 (*print_string*) espera exatamente esse formato.

## `.text` — código executável

```
.text
.globl main     # .globl = global: torna 'main' o ponto de entrada
```

A execução começa no rótulo `main:`.

---

# 7. Como o RARS imprime (syscalls)

Uma **syscall** (*system call*) é uma chamada ao sistema operacional do simulador para executar ações prontas, como imprimir ou ler entradas.

### Para chamar uma syscall:

1. Coloque o **código da syscall** em `a7`
2. Coloque o **argumento** em `a0`
3. Execute `ecall` (*environment call*)

**Resumo perfeito:**

> `a7` escolhe o serviço, `a0` fornece o dado, `ecall` executa.

### Syscalls usadas no Hello World:

| Código | Função                             |
| ------ | ---------------------------------- |
| `4`    | imprimir string (**print_string**) |
| `10`   | encerrar o programa (**exit2**)    |

Exemplo:

```asm
la a0, msg     # a0 = endereço da string
li a7, 4       # a7 = código de print_string
ecall          # dispara a syscall
```

---

# 8. Montando tudo: Hello World

```asm
    .data                           # sessão de dados
msg: .asciz "Hello, World!\n"       # string terminada em zero

    .text                           # sessão de código
    .globl main                     # torna 'main' visível

main:
    la a0, msg                      # carrega o endereço da string msg em a0
    li a7, 4                        # syscall 4 = print_string
    ecall                           # imprime

    li a0, 0                        # código de saída (0 = sucesso)
    li a7, 10                       # syscall 93 = exit2
    ecall                           # encerra
```

---
