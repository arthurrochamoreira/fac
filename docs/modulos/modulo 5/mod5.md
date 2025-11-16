# Módulo 5 — Pilha (Stack) e organização da memória

Neste módulo você vai aprender:

* o que é a **pilha** (*stack*)
* em que direção ela cresce na memória
* como **reservar espaço** na pilha
* como **restaurar valores** da pilha
* qual é a **convenção de stack frame** no RISC-V
* um exemplo prático de **salvar e recuperar um número** na pilha

Vou assumir que você já viu funções com `jal` (*jump and link*) e `ret` (*return*), mas tudo que aparecer pela primeira vez aqui (sigla ou instrução) vai ser explicado.

---

## 1. Visão geral: onde entra a pilha na memória?

Quando um programa está rodando, a memória do processo costuma ser organizada em regiões lógicas, entre elas:

* **código** (`.text`) — onde ficam as instruções que o processador executa
* **dados estáticos** (`.data`, `.bss`) — variáveis globais, constantes, strings
* **heap** — área para alocação dinâmica (malloc, new etc., em linguagens de alto nível)
* **pilha** (*stack*) — área usada para chamadas de função, variáveis locais e salvamento de registradores

Neste módulo vamos focar na **pilha**.

---

## 2. O que é a pilha (stack)?

A **pilha** (*stack*) é uma região da memória usada como uma **estrutura LIFO**
(LIFO = *Last In, First Out*, “último a entrar, primeiro a sair”).

Você pode imaginar uma pilha de pratos:

* o **último prato** que você coloca em cima é o **primeiro** que você pega de volta
* na memória, fazemos o mesmo com valores: empilhar e desempilhar

Na pilha guardamos, principalmente:

* endereços de retorno (`ra` — *return address*, registrador de retorno)
* registradores salvos (`s0–s11` — *saved registers*, registradores preservados)
* variáveis locais de uma função
* parâmetros extras (quando não cabem em `a0–a7` — *argument registers*)

---

## 3. Direção de crescimento da pilha no RISC-V

No RISC-V (e em muitas arquiteturas), a pilha cresce para **endereços menores**.

* **`sp` (stack pointer)** é o registrador que aponta para o **topo da pilha**.

Regra prática:

* para **reservar espaço** na pilha → diminuímos `sp`
* para **liberar espaço** na pilha → aumentamos `sp`

Exemplo conceitual:

```text
endereços altos
      ↑
      │  (memória)
      │
      │      [   ]  
      │      [   ]  ← valores mais antigos
      │      [   ]  
      │      [sp]  ← topo atual da pilha
      └────────────
endereços baixos
```

Quando fazemos:

```asm
addi sp, sp, -4    # addi (add immediate): sp = sp - 4
```

estamos “empurrando” o topo da pilha 4 bytes para **baixo** na memória e abrindo espaço para guardar algo.

---

## 4. Como reservar espaço na pilha

* **`addi rd, rs1, imediato`** — instrução real `addi` (*add immediate*), soma um valor constante (**imediato**) ao registrador `rs1` e guarda o resultado em `rd`.

Sintaxe:

```asm
addi rd, rs1, imm   # rd = rs1 + imm
```

No contexto da pilha, usamos:

```asm
addi sp, sp, -N     # reserva N bytes na pilha
```

Exemplo: reservar 8 bytes para guardar dois valores de 4 bytes (duas *words* de 32 bits):

```asm
addi sp, sp, -8    # reserva 8 bytes na pilha
```

Agora os 8 bytes “abaixo” do novo `sp` pertencem à função atual.

---

## 5. Como guardar e restaurar valores na pilha

Para realmente **guardar** algo na pilha, combinamos:

* ajuste de `sp` com `addi`
* instruções de acesso à memória: `sw` e `lw`

Primeiras ocorrências:

* **`sw rs2, offset(rs1)`** — `sw` (*store word*), grava uma *word* (32 bits = 4 bytes) da CPU para a memória.
  Significa: “guardar o conteúdo de `rs2` na memória no endereço `rs1 + offset`”.
* **`lw rd, offset(rs1)`** — `lw` (*load word*), lê uma *word* da memória para a CPU.
  Significa: “carregar em `rd` os 4 bytes armazenados em `rs1 + offset`”.

### 5.1 Guardando um valor

Padrão:

```asm
addi sp, sp, -4     # reserva 4 bytes na pilha
sw   t0, 0(sp)      # sw (store word): guarda o valor de t0 no topo da pilha
```

### 5.2 Restaurando esse valor

Mais tarde:

```asm
lw   t0, 0(sp)      # lw (load word): lê o valor de volta para t0
addi sp, sp, 4      # libera os 4 bytes usados na pilha
```

Regra de ouro:

> A quantidade que você **subtraiu** de `sp` para reservar deve ser **somada de volta** para liberar.

---

## 6. Stack frame e convenção de salvamento

* **Stack frame** é o “pacote” de memória que **cada função** reserva na pilha para armazenar:

  * registradores salvos (`s0–s11`)
  * registrador de retorno (`ra`)
  * variáveis locais
  * espaço para argumentos extras (se preciso)

Na **ABI (Application Binary Interface)** do RISC-V, a regra é:

* registradores **salvos** (`s0–s11`) devem ser **restaurados** pela função que os usa
* se a função chamar outra função com `jal` (*jump and link*), ela geralmente precisa salvar `ra` na pilha
* `sp` deve apontar para o topo do stack frame atual

### 6.1 Exemplo de stack frame simples

Vamos salvar `ra` e `s0` na pilha:

```asm
minha_funcao:
    addi sp, sp, -8     # reserva 8 bytes na pilha
    sw   ra, 4(sp)      # guarda ra na pilha (offset 4)
    sw   s0, 0(sp)      # guarda s0 na pilha (offset 0)

    # --- corpo da função ---
    # usa s0 como variável local, etc.

    lw   s0, 0(sp)      # restaura s0
    lw   ra, 4(sp)      # restaura ra
    addi sp, sp, 8      # libera 8 bytes
    ret                 # ret (return): montado como jalr x0, 0(ra)
```

* **`ret`** é uma **pseudo-instrução** de retorno de função.
  O montador converte para `jalr x0, 0(ra)` → salto indireto usando o endereço em `ra`.

---

## 7. Exemplo prático: salvar um número na pilha

Vamos fazer um programa simples que:

1. coloca o valor `42` em um registrador
2. salva esse valor na pilha
3. zera o registrador para “provar” que o valor foi perdido
4. recupera o valor da pilha para outro registrador
5. imprime o valor recuperado
6. encerra o programa

Também vamos usar syscalls do **RARS (RISC-V Assembler and Runtime Simulator)**:

Primeiras ocorrências aqui:

* **Syscall** (*system call*) — serviço fornecido pelo ambiente do simulador, como imprimir ou ler.
* `a7` — registrador que escolhe **qual syscall** será executada.
* `a0` — registrador usado para **argumento** e **retorno** das syscalls.
* `ecall` — instrução **environment call**, dispara a syscall escolhida em `a7`.

Tabela usada:

* `a7 = 1` → **print_int** (imprimir inteiro em `a0`)
* `a7 = 11` → **print_char** (imprimir caractere em `a0`)
* `a7 = 93` → **exit2** (encerrar programa)

### 7.1 Código completo e comentado

```asm
.text                      # .text: início da sessão de código (instruções)
.globl main                # .globl (global): torna 'main' visível como ponto de entrada

main:
    li   t0, 42            # li (load immediate): coloca o valor 42 no registrador t0

    # --- salvar t0 na pilha ---
    addi sp, sp, -4        # addi (add immediate): sp = sp - 4, reserva 4 bytes na pilha
    sw   t0, 0(sp)         # sw (store word): guarda o valor de t0 nesses 4 bytes

    # "perder" o valor original de t0
    li   t0, 0             # li: agora t0 = 0, valor 42 não está mais no registrador

    # --- recuperar o valor salvo ---
    lw   t1, 0(sp)         # lw (load word): carrega da pilha para t1 o valor que estava salvo (42)
    addi sp, sp, 4         # addi: sp = sp + 4, libera os 4 bytes usados na pilha

    # --- imprimir o valor recuperado ---
    mv   a0, t1            # mv (move): copia o valor de t1 para a0 (argumento da syscall)
    li   a7, 1             # li: a7 = 1 → syscall print_int (imprimir inteiro)
    ecall                  # ecall (environment call): executa a syscall, imprime 42

    # opcional: imprimir quebra de linha
    li   a0, 10            # li: a0 = 10, código ASCII de '\n'
    li   a7, 11            # li: a7 = 11 → syscall print_char (imprimir caractere)
    ecall                  # ecall: imprime '\n'

    # --- encerrar programa ---
    li   a7, 93            # li: a7 = 93 → syscall exit2 (encerrar execução)
    ecall                  # ecall: encerra o programa
```

Resumo do que acontece:

* o valor **42** é salvo na memória (pilha), mesmo depois de `t0` ser alterado
* ao restaurar com `lw`, recuperamos esse valor em `t1`
* provamos que a pilha é um lugar seguro para **guardar valores temporários**

---

## 8. Recapitulando

Neste módulo você viu:

* o que é a **pilha** (*stack*) e que ela funciona no modelo **LIFO** (*Last In, First Out*)
* que o registrador **`sp` (stack pointer)** aponta para o topo da pilha
* que no RISC-V a pilha cresce para **endereços menores**
* como **reservar espaço** com `addi sp, sp, -N`
* como **salvar valores** com `sw` e depois **restaurar** com `lw`
* a ideia de **stack frame**: espaço na pilha reservado por cada função
* como salvar e restaurar `ra` e registradores salvos (`s0–s11`) seguindo a **ABI (Application Binary Interface)**
* um exemplo completo que empilha um valor, zera o registrador e recupera o valor da pilha

---

## 9. Exercícios para fixação

Tente fazer no RARS. Se quiser, você pode me mandar suas soluções para correção detalhada.

### Exercício 1 — Salvar e restaurar dois registradores

Implemente um programa que:

1. Carregue o valor `10` em `t0` e o valor `20` em `t1`.
2. Salve **ambos** na pilha (use `addi sp, sp, -8`, `sw` com offsets diferentes).
3. Zere `t0` e `t1` com `li` para simular “perda” dos valores.
4. Restaure `t0` e `t1` a partir da pilha usando `lw`.
5. Some `t0 + t1` em `t2` e imprima o resultado com a syscall `print_int` (código 1).

---

### Exercício 2 — Função que usa stack frame simples

Crie:

* uma função `add5` que:

  1. recebe um número em `a0`;
  2. salva `ra` e `s0` na pilha;
  3. copia `a0` para `s0` (variável local);
  4. calcula `a0 = s0 + 5`;
  5. restaura `s0` e `ra` da pilha;
  6. faz `ret`.

* na `main`:

  1. coloque `20` em `a0`;
  2. chame `add5` com `jal ra, add5`;
  3. ao voltar, imprima o valor em `a0` com `print_int`.

---

### Exercício 3 — Salvar um valor temporário durante uma conta

Faça um programa que:

1. Calcule `t0 = 7` e `t1 = 9` usando `li`.
2. Some `t0 + t1` e salve o resultado na pilha.
3. Em seguida, calcule um novo valor em `t0` (por exemplo, `t0 = 100`) e em `t1` (`t1 = 200`).
4. Restaure da pilha o valor antigo da soma para `t2`.
5. Some `t2 + t0 + t1` e imprima o resultado final.

Use sempre a sequência correta:

* ajustar `sp` com `addi` antes de `sw`
* usar `lw` antes de devolver `sp` ao valor original.
