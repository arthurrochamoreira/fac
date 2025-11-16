# Módulo 4 — Saltos

(`j`, `jal`, `jalr`)

Neste módulo você vai aprender como o **RISC-V**
(RISC-V = *Reduced Instruction Set Computer – Five*)
realiza **saltos** (*jumps*) — mudanças explícitas no fluxo do programa.

Saltos são essenciais para:

* loops
* funções
* desvios condicionais
* chamadas e retornos entre trechos de código

---

# 1. O que é um salto?

Um **salto** (*jump*) é uma instrução que altera o **PC**
(*Program Counter* — contador de programa), ou seja, o endereço da **próxima instrução** a ser executada.

Em vez de seguir a execução linear (linha após linha), o processador:

> “pula” para outro endereço de código, normalmente marcado por um **rótulo** (*label*).

Ao contrário das instruções de desvio condicional (`beq`, `bne`, etc.),
os saltos deste módulo são **incondicionais**: sempre acontecem.

As três instruções principais são:

* **`j`** — salto incondicional simples (pseudo-instrução)
* **`jal`** — salto que **guarda o endereço de retorno**
* **`jalr`** — salto indireto via registrador (endereços dinâmicos)

---

# 2. A pseudo-instrução `j` — *jump*

**`j` (jump)** é uma **pseudo-instrução** de salto incondicional.

**Pseudo-instrução** é uma “instrução de mentira” fornecida pelo montador (assembler):
ela **não existe no hardware**, mas é automaticamente traduzida para uma ou mais instruções reais.

No caso de `j`, o montador converte:

```asm
j label
```

para:

```asm
jal x0, label
```

onde:

* `jal` = **`jal` (jump and link)**, instrução real que faz salto e guarda endereço de retorno
* `x0` é o registrador que sempre vale 0; como o retorno é guardado em `x0`, ele é descartado

Em português simples:

> `j label` = “pula para `label` e **não** guarda retorno”.

Exemplo:

```asm
j loop      # sempre desvia para o rótulo 'loop'
```

---

# 3. A instrução `jal` — *jump and link*

**`jal` (jump and link)** é uma instrução real da ISA RV32I
(ISA = *Instruction Set Architecture*, arquitetura de conjunto de instruções).

Ela faz **duas coisas** ao mesmo tempo:

1. **Salva o endereço da próxima instrução** em um registrador destino `rd`.
2. **Salta** para o rótulo indicado.

Sintaxe:

```asm
jal rd, label
```

Significa:

> `rd = endereço_da_proxima_instrução`
> depois, **PC = endereço_do_label**

Uso típico:

* `rd` é o registrador **`ra`** (*return address*), que é o registrador de retorno padrão (`x1`).

Exemplo clássico de chamada de função:

```asm
jal ra, funcao    # salva o endereço de retorno em ra e pula para 'funcao'
```

Para retornar da função, usamos um salto indireto via `ra`:

* pseudo-instrução **`ret`**
* ou a forma explícita com `jalr`

Veremos isso a seguir.

---

# 4. A instrução `jalr` — *jump and link register*

**`jalr` (jump and link register)** é a versão “indireta” de `jal`:
em vez de pular para um **label** fixo, ela pula para um **endereço em um registrador**.

Sintaxe:

```asm
jalr rd, offset(rs1)
```

Significa:

1. `rd = endereço_da_proxima_instrução`
2. `PC = rs1 + offset`

Em palavras:

> Salta para o endereço calculado como `rs1 + offset` e guarda o endereço de retorno em `rd`.

Usos comuns:

* retorno de função (via pseudo-instrução `ret`)
* saltos dinâmicos (tabelas de função, ponteiros de função, etc.)

Exemplo de retorno “puro” (sem guardar nada):

```asm
jalr x0, 0(ra)    # PC = ra + 0, não guarda retorno (rd = x0, descartado)
```

Essa é justamente a forma que o montador usa para implementar a pseudo-instrução:

```asm
ret               # montador: jalr x0, 0(ra)
```

---

# 5. Pseudo-instruções relacionadas (`jr`, `ret`)

* **`jr rd` (jump register)**
  Pula para o endereço contido em `rd`, sem guardar retorno.
  O montador expande:

  ```asm
  jr rd          # pseudo
  ```

  para:

  ```asm
  jalr x0, 0(rd) # salto indireto sem link (sem retorno)
  ```

* **`ret` (return)**
  Usada como retorno de função, padrão da calling convention do RISC-V.

  O montador expande:

  ```asm
  ret            # pseudo
  ```

  para:

  ```asm
  jalr x0, 0(ra) # PC = ra, não guarda retorno
  ```

onde:

* `ra` (*return address*) é o registrador de retorno (`x1`), definido na ABI
  ABI = *Application Binary Interface*, convenção de chamada entre funções.

---

# 6. Exemplos completos

Nesta seção, além de `j`, `jal` e `jalr`, aparecem algumas instruções e diretivas:

* `li rd, imm` — **`li` (load immediate)**: carrega o valor imediato `imm` no registrador `rd`.
  Ex.: `li t0, 0` coloca 0 em `t0`.
* `addi rd, rs1, imm` — **`addi` (add immediate)**: soma imediato a registrador.
  Ex.: `addi t0, t0, 1` → `t0 = t0 + 1`.
* `blt rs1, rs2, label` — **`blt` (branch if less than)**: desvia se `rs1 < rs2`.
* `.text` — início da **sessão de código** (onde ficam instruções executáveis).
* `.data` — início da **sessão de dados** (variáveis, strings, etc.).
* `.globl main` — torna `main` visível como **ponto de entrada**.
* `.asciz` — string ASCII terminada em byte zero (`\0`).
* `la rd, label` — **`la` (load address)**: pseudo-instrução que carrega o endereço do rótulo `label` em `rd`.
* `ecall` — **environment call**: dispara uma syscall configurada em `a7`.
* `a7` — registrador que indica **qual syscall** será chamada (no RARS).

---

## 6.1 Exemplo simples usando `j`

Loop que incrementa um contador até 5 usando `j` para voltar:

```asm
.text
.globl main

main:
    li t0, 0                # li (load immediate): t0 = 0

loop:
    addi t0, t0, 1          # addi (add immediate): t0 = t0 + 1

    li t1, 5                # t1 = 5
    blt t0, t1, loop        # blt (branch if less than): se t0 < 5, volta pro rótulo 'loop'

    # chegou aqui quando t0 >= 5
fim:
    li a7, 93               # li: a7 = 93 → syscall exit2 (encerrar programa)
    ecall                   # ecall (environment call): executa a syscall
```

Versão com `j` para loop **incondicional**:

```asm
.text
.globl main

main:
    li t0, 0

loop:
    addi t0, t0, 1          # t0++
    j loop                  # j (jump): volta para o rótulo 'loop' para sempre (loop infinito)
```

---

## 6.2 Função simples usando `jal` e `ret`

Programa que chama uma função `ola` que imprime uma mensagem:

```asm
.data
msg: .asciz "Ola!\n"        # .asciz: string terminada em 0

.text
.globl main

main:
    jal ra, ola             # jal (jump and link): salva retorno em ra e salta para 'ola'

    li a7, 93               # exit2
    ecall

ola:
    la a0, msg              # la (load address): a0 = endereço de 'msg'
    li a7, 4                # 4 = syscall print_string
    ecall                   # imprime "Ola!\n"

    ret                     # ret: jalr x0, 0(ra) → volta para depois do jal em main
```

---

## 6.3 Chamadas aninhadas com `jal` e `ret`

Aqui vemos `main` chamando `A`, que chama `B`, que chama `C`:

```asm
.text
.globl main

main:
    jal ra, A               # chama A
    li a7, 93               # exit2
    ecall

A:
    jal ra, B               # chama B
    ret                     # retorna para main

B:
    jal ra, C               # chama C
    ret                     # retorna para A

C:
    ret                     # retorna para B
```

Em cada nível, o endereço de retorno é automaticamente gerenciado em `ra`,
de acordo com a convenção de chamada (ABI).

---

## 6.4 Uso de `jalr` para saltos indiretos

Exemplo didático de salto indireto:

```asm
.data
msg: .asciz "Salto indireto executado!\n"

.text
.globl main

main:
    la t0, destino          # la: t0 recebe o endereço do rótulo 'destino'
    jalr ra, 0(t0)          # jalr: salva retorno em ra e salta para endereço em t0
                            # ou seja, salta para 'destino'

    li a7, 93               # exit2
    ecall

destino:
    la a0, msg              # a0 = endereço da string
    li a7, 4                # 4 = print_string
    ecall                   # imprime "Salto indireto executado!\n"
    ret                     # volta para depois do jalr em main
```

---

# 7. Comparação entre `j`, `jal` e `jalr`

| Instrução | Tipo de salto | Guarda endereço de retorno? | Destino é label fixo? | Destino via registrador? |
| --------- | ------------- | --------------------------- | --------------------- | ------------------------ |
| `j`       | incondicional | ❌ não                       | ✔ sim                 | ❌ não                    |
| `jal`     | incondicional | ✔ sim (em `rd`)             | ✔ sim                 | ❌ não                    |
| `jalr`    | incondicional | ✔ sim (em `rd`)             | ❌ não                 | ✔ sim (`rs1 + offset`)   |

---

# 8. Exercícios para fixação

Use o **RARS (RISC-V Assembler and Runtime Simulator)** para montar e testar cada exercício.
Sempre que usar syscalls:

* lembre que `a7` define **qual syscall** (por exemplo, `4` = print_string, `1` = print_int, `93` = exit2),
* `a0` normalmente carrega o **argumento** (string ou inteiro),
* e `ecall` executa a syscall.

---

## Exercício 1 — Saltos simples com `j`

Crie um programa que:

1. Imprima `"A\n"`.
2. Use `j` para pular diretamente para o rótulo `C`.
3. No meio do código, coloque uma impressão `"B\n"` que **não deve ser executada**.
4. No rótulo `C`, imprima `"C\n"`.
5. Encerre o programa com a syscall `exit2` (código 93 em `a7`).

Dica de estrutura:

```asm
.data
msgA: .asciz "A\n"
msgB: .asciz "B\n"
msgC: .asciz "C\n"

.text
.globl main

main:
    # imprime A
    # j C
    # imprime B (não deve rodar)
C:
    # imprime C
    # exit
```

---

## Exercício 2 — Função `quadrado(n)` com `jal` e `ret`

Implemente uma função `quadrado` em RISC-V que:

* recebe um inteiro `n` em `a0`,
* devolve `n * n` em `a0`.

Regras:

1. Em `main`, carregue um valor em `a0` (por exemplo, 7).
2. Chame a função com `jal ra, quadrado`.
3. Ao retornar, imprima o resultado com `print_int` (syscall 1).
4. Use `ret` dentro da função para voltar.

Sugestão de lógica da função:

```asm
quadrado:
    # pode usar algum registrador temporário t0
    # a0 = a0 * a0  (pode ser soma repetida ou multiplicação se você já tiver)
    ret
```

*(Se ainda não viu multiplicação, pode usar somas sucessivas só pra fins didáticos.)*

---

## Exercício 3 — Chamada indireta com `jalr`

Faça um programa que:

1. Carregue em `t0` o endereço de uma função `f1` usando `la`.
2. Use `jalr ra, 0(t0)` para pular até `f1`.
3. Dentro de `f1`, imprima uma mensagem, por exemplo `"Chamado por jalr!\n"`.
4. Retorne com `ret`.
5. Ao voltar em `main`, encerre o programa com `exit2`.

Extensão (opcional): crie duas funções, `f1` e `f2`,
e use uma variável em `.data` para decidir, em tempo de execução, para qual delas pular com `jalr`.

---

## Exercício 4 — Loop reescrito apenas com `j`

Pegue um loop que use `blt` ou `bne` e reescreva para usar **apenas saltos incondicionais** e comparações manuais.

Exemplo em C:

```c
for (i = 0; i < 5; i++) {
    // corpo
}
```

Sugerido em assembly:

1. Iniciar `i = 0` em `t0`.
2. No início do loop, comparar `i` com 5.
3. Se `i >= 5`, saltar para o fim usando `j`.
4. Caso contrário, executar o corpo.
5. Incrementar `i` e usar `j` para voltar ao teste.

Tente **evitar** `blt` e reproduzir a lógica usando:

* `bge` (se já tiver visto) **ou**
* comparações equivalentes e saltos com `j`.

---