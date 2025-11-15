# **Módulo 3 — Registradores do RV32I**

O conjunto RV32I possui **32 registradores inteiros**, cada um com um papel definido pela *ABI*
(*Application Binary Interface*, primeira ocorrência).
A ABI determina **como funções recebem argumentos**, **como retornam valores**, **quais registradores podem ser sobrescritos** e **quais devem ser preservados**.

Este módulo apresenta:

* as categorias de registradores (`a*`, `t*`, `s*`, `zero`)
* como funcionam no *calling convention* (primeira ocorrência: convenção de chamada)
* por que usamos `a0`, `a7`, `t0` etc.
* o que é o registrador `sp` e como ele forma a pilha

---

# **1. Visão geral: por que existem tipos diferentes de registradores?**

O RISC-V segue uma regra essencial:

> Cada registrador tem uma **função específica** na convenção de chamada.
> Isso permite que funções cooperem entre si de forma consistente.

Assim, todo programa — do menor ao maior — funciona com o mesmo protocolo.

As categorias são:

| Categoria | Registradores | Nome técnico          | Função principal                         |
| --------- | ------------- | --------------------- | ---------------------------------------- |
| `zero`    | `x0`          | registrador zero      | sempre contém 0                          |
| `a*`      | `a0–a7`       | *argument registers*  | argumentos e retorno de funções/syscalls |
| `t*`      | `t0–t6`       | *temporary registers* | valores temporários (não preservados)    |
| `s*`      | `s0–s11`      | *saved registers*     | valores preservados entre chamadas       |
| `sp`      | `x2`          | *stack pointer*       | topo da pilha (stack)                    |
| `ra`      | `x1`          | *return address*      | endereço para onde retornar após `jal`   |

Vamos detalhar cada categoria.

---

# **2. O registrador `zero` (`x0`)**

`x0` é um registrador **somente leitura** que sempre contém o valor **0**.

Ele é útil para:

* comparações (`beq t0, x0, label`)
* gerar valores 0 sem gastar instruções (`addi t0, x0, 0`)
* confirmar igualdade (`bne t0, x0, continuar`)

Como ele nunca muda, evita erros e economiza espaço no código.

---

# **3. Registradores `a*` — Argument Registers**

Os registradores `a0–a7` são usados para:

1. **Passar argumentos para funções**
2. **Receber argumentos de syscalls**
3. **Receber valores de retorno**

### **3.1 Retorno de função**

Toda função retorna valores em:

* `a0` — primeiro valor de retorno
* `a1` — segundo, se necessário

Exemplo:

```asm
li a0, 42        # argumento
jal ra, dobro    # chama função

# a0 agora contém o retorno
```

---

# **4. Registradores `t*` — Temporary Registers**

Os registradores `t0–t6` (*temporary registers*, primeira ocorrência) são temporários:

* a função **pode sobrescrevê-los livremente**
* não precisam ser preservados na pilha

Use `t*` para:

* contadores
* variáveis temporárias
* cálculos intermediários
* ponteiros temporários
* laços simples

Exemplo típico:

```asm
li t0, 5
addi t0, t0, 1    # t0 = 6
```

---

# **5. Registradores `s*` — Saved Registers**

Registradores `s0–s11` (*saved registers*, primeira ocorrência) são usados quando um valor precisa **sobreviver a chamadas de função**.

Regra da ABI:

> Se uma função **modificar** um registrador `s*`, ela é obrigada a salvá-lo na pilha e restaurá-lo antes de retornar.

É assim que se cria uma variável “local” persistente:

```asm
addi sp, sp, -4
sw s0, 0(sp)

li s0, 99      # usa s0 dentro da função

lw s0, 0(sp)
addi sp, sp, 4
ret
```

---

# **6. Registrador `sp` — Stack Pointer**

`sp` (*stack pointer*, primeira ocorrência) aponta para o **topo da pilha**.

A pilha é uma região de memória com comportamento **LIFO** (*last in, first out*, primeira ocorrência).

### **6.1 Como a pilha cresce**

A pilha cresce para **endereços menores**:

```asm
addi sp, sp, -16   # reserva 16 bytes
```

E diminui quando liberamos espaço:

```asm
addi sp, sp, 16    # libera espaço
```

### Para que serve a pilha?

* armazenar variáveis locais
* guardar registradores `s*`
* armazenar o endereço de retorno (`ra`) quando funções chamam outras
* suportar recursão

---

# **7. Por que usamos `a0`, `a7`, `t0`… no programa?**

Isso segue a **ABI (Application Binary Interface)**, que define:

1. Qual registrador carrega argumentos (`a0–a7`)
2. Qual registrador recebe retorno (`a0`)
3. Quais registradores são temporários (`t0–t6`)
4. Quais são preservados (`s0–s11`)
5. Qual registrador aponta a pilha (`sp`)
6. Qual guarda o endereço de retorno (`ra`)

### Exemplos reais:

### **7.1 Syscalls**

A syscall é executada quando:

* `a7` contém **o código da syscall**
* `a0` contém **o argumento**

Exemplo:

```asm
li a0, 123         # argumento
li a7, 1           # print_int
ecall
```

### **7.2 Funções**

```asm
li a0, 5
li a1, 7
jal ra, soma       # soma(5,7)

# a0 contém retorno
```

---

