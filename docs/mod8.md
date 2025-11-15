# Módulo 8 — Pilha, Stack Frame e Funções Aninhadas no RISC-V (RV32I)

A partir deste módulo, você passa a escrever **funções reais** no estilo RISC-V profissional:
com *stack frame*, preservação de registradores e chamadas aninhadas — tudo dentro das regras da ABI.

---

# 1. O que é a pilha (stack)?

A **pilha** é uma região de memória usada pelo programa para armazenar:

* variáveis locais
* valores temporários
* registradores que precisam ser preservados
* endereços de retorno
* frames de funções aninhadas

Ela funciona em estilo **LIFO** (*Last In, First Out*):
o último valor armazenado é o primeiro a ser retirado.

**Importante:** no RISC-V, a pilha cresce para **endereços menores**.
Ou seja: empilhar diminui o valor de `sp`.

---

# 2. O registrador `sp` (stack pointer)

`sp` (*stack pointer*) é o registrador que aponta para o **topo da pilha**.

Para **empilhar (push)** algo:

```asm
addi sp, sp, -4      # reserva 4 bytes
sw t0, 0(sp)         # coloca t0 no topo da pilha
```

Para **desempilhar (pop)**:

```asm
lw t0, 0(sp)         # recupera valor
addi sp, sp, 4       # libera espaço
```

---

# 3. Por que a pilha é necessária?

1. **Para preservar registradores salvos (`s0–s11`)**
   Se uma função mexer neles, precisa restaurar depois.

2. **Para guardar o endereço de retorno (`ra`)**
   Toda função que chama outra precisa salvar `ra`.

3. **Para manter variáveis locais**

4. **Para chamadas aninhadas e recursivas**
   Cada chamada precisa de seu próprio contexto independente.

Tudo isso é encapsulado em um **stack frame**.

---

# 4. O que é um Stack Frame?

É o bloco de memória reservado por cada função na pilha.

Layout típico:

```
+----------------------+
| argumentos extras    |
+----------------------+
| valor antigo de s0   |  <- salvo pela função
+----------------------+
| valor antigo de ra   |  <- endereço de retorno salvo
+----------------------+
| variáveis locais     |
+----------------------+
sp -> topo após reserva
```

Cada função tem seu próprio frame, permitindo chamadas aninhadas.

---

# 5. Regra essencial da ABI do RISC-V

Primeira ocorrência de “ABI”: *Application Binary Interface* —
padrão que define **como funções devem se comportar**.

### Toda função deve:

1. **Salvar na pilha**:

   * `ra` se for chamar outra função
   * todos os `sX` que pretende modificar

2. **Restaurar na ordem inversa** antes do `ret`.

---

# 6. Exemplo 1 — Salvando e restaurando `ra`

Qualquer função que chama outra deve salvar seu endereço de retorno.

```asm
funcA:
    addi sp, sp, -4       # reserva espaço
    sw ra, 0(sp)          # salva ra

    jal ra, funcB         # chama B

    lw ra, 0(sp)          # restaura endereço de retorno
    addi sp, sp, 4        # libera a pilha
    ret

funcB:
    ret
```

---

# 7. Exemplo 2 — Preservando registradores salvos (`s0`, `s1`, …)

Registradores `s0–s11` são “salvos”:
se uma função modificar, ela deve restaurar.

```asm
funcao:
    addi sp, sp, -8       # espaço para ra e s0
    sw ra, 4(sp)          # salva ra
    sw s0, 0(sp)          # salva s0

    li s0, 123            # usa s0 como variável local

    lw s0, 0(sp)          # restaura s0
    lw ra, 4(sp)          # restaura ra
    addi sp, sp, 8
    ret
```

---

# 8. Exemplo 3 — Variável local na pilha

Vamos criar uma função que calcula `n + 5` usando variável local.

```asm
add5:
    addi sp, sp, -4
    sw s0, 0(sp)          # salva s0

    mv s0, a0             # s0 = n
    addi a0, s0, 5        # a0 = n + 5

    lw s0, 0(sp)          # restaura s0
    addi sp, sp, 4
    ret
```

Chamada:

```asm
li a0, 20
jal ra, add5     # a0 = 25
```

---

# 9. Exemplo 4 — Funções aninhadas com stack frame completo

Fluxo:

```
main → funcA → funcB
```

Cada uma com seu frame.

```asm
.text
.globl main

main:
    jal ra, funcA
    li a7, 93
    ecall

funcA:
    addi sp, sp, -8
    sw ra, 4(sp)
    sw s0, 0(sp)

    li s0, 50
    mv a0, s0
    jal ra, funcB

    lw s0, 0(sp)
    lw ra, 4(sp)
    addi sp, sp, 8
    ret

funcB:
    addi a0, a0, 10     # retorna n + 10
    ret
```

---

# 10. Exemplo 5 — Função recursiva simples

Vamos calcular:

```
f(n) = n + f(n − 1)
f(0) = 0
```

```asm
soma_rec:
    addi sp, sp, -8
    sw ra, 4(sp)
    sw s0, 0(sp)

    mv s0, a0                # s0 = n
    beq s0, x0, caso_base    # se n == 0

    addi a0, s0, -1
    jal ra, soma_rec         # chamada recursiva

    add a0, a0, s0           # retorna f(n-1)+n
    j fim

caso_base:
    li a0, 0                 # f(0) = 0

fim:
    lw s0, 0(sp)
    lw ra, 4(sp)
    addi sp, sp, 8
    ret
```

---
