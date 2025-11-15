# **Módulo 1 — Introdução ao RISC-V**

Este módulo apresenta a base conceitual necessária para entender como funciona um programa em assembly RISC-V:
o que é uma ISA, por que começamos pelo conjunto RV32I, como funciona o simulador RARS e como o processador enxerga memória, dados e instruções.

---

# **1. O que é RISC-V?**

RISC-V é uma **arquitetura de conjunto de instruções** (ISA — *Instruction Set Architecture*).
Uma **ISA** é o conjunto de comandos que o processador entende **no nível mais baixo**, como:

* somar valores
* carregar dados da memória
* desviar para outro ponto do código
* comparar números
* executar operações lógicas

O termo **RISC** significa *Reduced Instruction Set Computer*:
uma filosofia de projeto que preza por instruções simples, diretas e rápidas.

O **“V”** indica que esta é a quinta geração da família RISC dessa linha de pesquisa.

---

# **2. O que é RV32I e por que começamos por ele**

A sigla **RV32I** significa:

* **RV** — *RISC-V*
* **32** — registradores e endereços possuem **32 bits**
* **I** — conjunto básico de instruções inteiras (*Integer*)

O RV32I é:

* o menor conjunto completo e funcional da ISA
* suficiente para aprender todos os fundamentos de assembly
* compatível com praticamente qualquer simulador
* limpo, simples e ideal para iniciantes

Tudo que você aprender aqui funciona **em qualquer implementação real** de RISC-V, incluindo microcontroladores, processadores embarcados e sistemas operacionais reais.

---

# **3. O que é um simulador (RARS)**

Para programar em RISC-V sem comprar um chip físico, usamos o **RARS**
(*RISC-V Assembler and Runtime Simulator*).

O RARS permite:

### **3.1 Montar o código**

O montador (*assembler*) converte o código `.asm` em instruções binárias reais da ISA.

### **3.2 Executar o programa**

O simulador roda o binário e mostra o fluxo do programa.

### **3.3 Inspecionar registradores e memória**

É possível acompanhar:

* valores nos registradores (`a0`, `t0`, `s0`…)
* conteúdo da memória em várias representações
* execução passo a passo (modo *step*)

---

# **4. Estrutura geral de um programa `.asm`**

Um programa típico possui duas sessões principais:

## **4.1 Sessão `.data` (data section)**

Armazena dados estáticos, como números, vetores e strings.

Exemplo:

```asm
.data
msg: .asciz "Hello!\n"
x:   .word 42
```

## **4.2 Sessão `.text` (text section)**

Armazena o **código executável**.

Exemplo:

```asm
.text
.globl main           # .globl (global): define o ponto de entrada

main:
    li a0, 42         # li = load immediate: coloca 42 em a0
    li a7, 1          # código da syscall print_int
    ecall             # ecall = environment call
```

### **4.3 Labels**

Um **label** (rótulo) é um nome que representa um endereço na memória.

Exemplo:

```
msg:
```

Esse nome representa o endereço do primeiro byte da string `"Hello!\n"`.

Labels funcionam como **ponteiros**, iguais aos usados em C.

---

# **5. Como o processador enxerga dados e instruções**

Para o processador, **tudo é apenas memória**.

Ele não distingue “texto”, “números”, “strings”, “instruções” ou “variáveis” de forma semântica.

### A memória é uma grande linha de bytes:

```
[00][04][3F][10][7A][FF][00]...
```

O processador interpreta esses bytes de acordo com **o contexto**:

* Em `.text`: interpreta os bytes como **instruções**.
* Em `.data`: interpreta como **dados** (palavras, strings, vetores etc.).

Quando você escreve:

```asm
add t0, t1, t2
```

Isso é convertido em **32 bits** que o hardware interpreta como:

> "some o registrador t1 com o registrador t2 e coloque o resultado em t0".

Quando você escreve:

```asm
.word 42
```

Isso apenas grava **42 em binário** na memória — não há “tipo inteiro”, apenas bits.

O comportamento é definido pela **instrução que lê esses bits**.

---

# **6. Como o processador executa um programa**

O processador possui:

* **PC** (*program counter*):
  registrador que aponta para a **próxima instrução** a ser executada.

O ciclo básico é:

1. Ler a instrução apontada por PC
2. Executar a instrução
3. Avançar o PC (ou fazer um salto no caso de branches)
4. Repetir

Esse ciclo acontece milhões de vezes por segundo.

---
