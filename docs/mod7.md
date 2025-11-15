# Módulo 7 — Funções no RISC-V (RV32I)

Em assembly, uma “função” não é um recurso mágico da linguagem:
é apenas uma **convenção** construída com rótulos e instruções de desvio.

Uma função, na prática, é:

1. Um **rótulo** marcando o início da função
2. Um bloco de **código executável**
3. Um **retorno** para o ponto de chamada

Não existe palavra-chave `function`. Nós é que organizamos o código para se comportar como funções.

---

## 1. A instrução fundamental: `jal` (jump and link)

A instrução:

```asm
jal rd, label
```

faz duas coisas:

1. **Salva o endereço da próxima instrução** em `rd`

   * normalmente usamos `ra` (*return address*), que é o registrador **x1**
2. **Desvia a execução** para o rótulo `label`

Ou seja:

```asm
jal ra, funcao    # salva o endereço de retorno em ra e salta para funcao
```

### Como retornar da função

O retorno é feito com a pseudo-instrução:

```asm
ret
```

que o montador traduz para uma forma de salto indireto usando o registrador `ra` (em RV32I real, via `jalr`).

---

## 2. Esqueleto de uma função em RISC-V

```asm
funcao:
    # ... corpo da função ...
    ret       # volta para o endereço salvo em ra
```

E a chamada, no código que invoca:

```asm
jal ra, funcao   # chama a função
# quando a função der ret, a execução continua aqui
```

---

## 3. Convenção de chamada (ABI) — versão simplificada

A **ABI** (*Application Binary Interface*) define como funções trocam dados.

No RISC-V (RV32I), as regras básicas são:

1. **Argumentos** vão em `a0–a7`

   * “a” vem de *argument registers*
2. **Valor de retorno** volta em `a0`

   * e em `a1` se precisar de dois registradores
3. Quem chama a função pode usar `t0–t6` à vontade (registradores temporários)
4. Se a função modificar registradores “salvos” (`s0–s11`), deve restaurá-los antes de retornar

   * isso é feito com pilha (stack), assunto do próximo módulo

Para começar, basta pensar assim:

* `a0` → argumento de entrada
* `a0` → valor de retorno
* `ra` → endereço de retorno

---

## 4. Exemplo 1 — Função que imprime um número

Objetivo: criar uma função `imprime_numero` que:

* recebe em `a0` o número a imprimir
* usa a syscall 1 (`print_int`)
* não altera o valor de `a0` além do necessário

### Função

```asm
imprime_numero:
    li a7, 1          # 1 = syscall print_int
    ecall             # imprime o inteiro em a0
    ret
```

### Chamada

```asm
li a0, 42             # valor a imprimir
jal ra, imprime_numero
```

Essa é a forma correta segundo a ABI: o argumento vai em `a0`.

---

## 5. Exemplo 2 — Função que soma dois números

Convenção adotada:

* `a0` = primeiro número
* `a1` = segundo número
* retorno em `a0`

### Função

```asm
soma:
    add a0, a0, a1    # a0 = a0 + a1
    ret
```

### Chamada

```asm
li a0, 15
li a1, 27
jal ra, soma          # ao voltar, a0 = 42
```

Depois da chamada, podemos usar `a0` normalmente com o resultado.

---

## 6. Uso de registradores temporários

* Registradores **temporários** (`t0–t6`) não precisam ser preservados pela função
* Registradores **salvos** (`s0–s11`) devem ser preservados se a função os modificar

  * isso é feito empilhando valores na **pilha** (stack), que veremos no próximo módulo

Por enquanto, vamos escrever funções que só usam:

* `a0–a1` para argumentos/retorno
* `t0–t6` como auxiliares locais

---

## 7. Exemplo 3 — Multiplicar por 3 usando função

Vamos fazer uma função `vezes3` que recebe um inteiro em `a0` e devolve `3 * a0` em `a0`.

### Código completo

```asm
.text
.globl main

main:
    li a0, 10             # valor de entrada
    jal ra, vezes3        # chama a função

    li a7, 1              # print_int
    ecall

    li a7, 93             # exit2
    ecall

vezes3:
    mv t0, a0             # t0 = a0 (cópia do argumento)
    add a0, a0, t0        # a0 = a0 + t0  (2x)
    add a0, a0, t0        # a0 = a0 + t0  (3x)
    ret
```

Após a função:

* se a entrada era 10, `a0` passa a ser 30.

---

## 8. Exemplo 4 — Função que lê um inteiro e retorna

A syscall **5** (`read_int`) lê um inteiro do teclado e coloca o resultado em `a0`.

### Função

```asm
ler_inteiro:
    li a7, 5          # 5 = read_int
    ecall             # a0 recebe o valor digitado
    ret
```

### Chamada

```asm
jal ra, ler_inteiro   # ao voltar, a0 contém o número lido
mv t0, a0             # guarda o valor em t0, se quiser preservar
```

---

## 9. Exemplo 5 — Mini-programa com várias funções

Objetivo:

1. Ler dois números do usuário
2. Somar com uma função `soma`
3. Imprimir o resultado com `imprime_int`

### Código completo

```asm
.data
msg1: .asciz "Primeiro: "
msg2: .asciz "Segundo: "
msgR: .asciz "Resultado: "

.text
.globl main

main:
    # ---- Ler primeiro número ----
    la a0, msg1
    li a7, 4              # print_string
    ecall

    jal ra, ler_int       # retorna em a0
    mv t0, a0             # t0 = primeiro número

    # ---- Ler segundo número ----
    la a0, msg2
    li a7, 4
    ecall

    jal ra, ler_int
    mv t1, a0             # t1 = segundo número

    # ---- Somar usando função ----
    mv a0, t0
    mv a1, t1
    jal ra, soma          # a0 = t0 + t1
    mv t2, a0             # guarda resultado em t2

    # ---- Imprimir resultado ----
    la a0, msgR
    li a7, 4              # print_string
    ecall

    mv a0, t2
    jal ra, imprime_int   # imprime valor de t2

    li a7, 93             # exit2
    ecall


# -------- Funções --------

ler_int:
    li a7, 5              # read_int
    ecall                 # a0 = valor lido
    ret

soma:
    add a0, a0, a1        # a0 = a0 + a1
    ret

imprime_int:
    li a7, 1              # print_int
    ecall
    ret
```

---

## 10. Checklist: o que você domina agora

Você já sabe:

* que uma função é um **rótulo + código + retorno**
* como `jal` chama funções e grava o endereço de retorno em `ra`
* o papel de `ra` (*return address*)
* como passar argumentos em `a0–a7`
* como receber o resultado em `a0`
* como escrever funções modulares para:

  * ler inteiros
  * somar
  * imprimir resultados

---