# Aula 8 — Gerenciamento de Memória

## Seção 13 — Questões para análise

---

### 1. Qual é a diferença entre um HIT e um PAGE FAULT?

- **HIT:** a página solicitada pelo processo **já está carregada** em um frame da memória física. O acesso é atendido direto, sem precisar buscar nada no disco.
- **PAGE FAULT:** a página solicitada **não está** na memória física. O sistema operacional precisa buscá-la no disco e carregá-la em um frame. Se houver frame livre, ela só é carregada; se a memória estiver cheia, um algoritmo de substituição escolhe uma página para sair.

Exemplo com a memória `[1, 2, 3]`: acessar a página `2` é HIT; acessar a página `4` é PAGE FAULT.

---

### 2. Qual é a principal diferença entre FIFO e LRU?

O critério usado para escolher a página que sai:

| Algoritmo | Critério | Pergunta que ele faz |
|---|---|---|
| **FIFO** | Ordem de **entrada** na memória | "Qual página está há mais tempo carregada?" |
| **LRU** | Ordem de **uso** | "Qual página está há mais tempo sem ser utilizada?" |

Consequência prática: no FIFO um HIT **não muda nada** na ordem da fila. No LRU um HIT move a página para a posição de "mais recentemente usada", o que a protege da próxima substituição.

No código isso aparece nas duas linhas que só existem no LRU, dentro do `if pagina in memoria`:

```python
memoria.remove(pagina)
memoria.append(pagina)
```

---

### 3. Um page fault significa necessariamente que existe um erro no programa? Explique.

**Não.** Apesar do nome ("falta"), o page fault é um evento **normal e esperado** do gerenciamento de memória. Ele só indica que a página pedida ainda não está na RAM e precisa ser carregada.

Todo programa gera page faults, por exemplo ao acessar uma página pela primeira vez (nas simulações, os primeiros acessos são sempre PAGE FAULT porque a memória começa vazia). O sistema operacional trata a situação sozinho e o programa continua executando normalmente, sem perceber.

O que o page fault tem é **custo de desempenho**: buscar a página no disco é muito mais lento do que acessar a RAM. Por isso o objetivo é reduzir a quantidade de page faults, não eliminá-los.

---

### 4. Aumentar a quantidade de frames sempre reduz o número de page faults no FIFO? Utilize o experimento da Anomalia de Belady para justificar.

**Não.** No FIFO pode acontecer a **Anomalia de Belady**: em certas sequências, mais frames resultam em **mais** page faults.

Experimento do notebook, com a sequência `1, 2, 3, 4, 1, 2, 5, 1, 2, 3, 4, 5`:

| Frames | Page Faults (FIFO) | Hits |
|---|---|---|
| 3 | 9 | 3 |
| 4 | **10** | 2 |

Com um frame a mais, o FIFO teve um page fault a mais.

Isso acontece porque o FIFO olha apenas a ordem de chegada e ignora o uso. Com 4 frames, a ordem de entrada muda de um jeito que faz o algoritmo remover as páginas `1` e `2` logo antes de elas serem pedidas de novo.

Para comparação, o LRU na mesma sequência se comporta como esperado: 10 page faults com 3 frames e 8 com 4 frames. LRU e Ótimo não sofrem dessa anomalia.

---

### 5. Em quais situações FIFO e LRU podem produzir resultados diferentes?

Quando uma página que **já está na memória é acessada de novo (HIT)** e depois é preciso fazer uma substituição. Nesse momento a "mais antiga a entrar" (FIFO) e a "menos recentemente usada" (LRU) deixam de ser a mesma página.

Exemplo com 3 frames e a sequência `1, 2, 3, 1, 4`:

- Após `1, 2, 3, 1` a memória contém as páginas 1, 2 e 3, e a página 1 acabou de ser reutilizada.
- Chega a página `4`:
  - **FIFO** remove a `1` (foi a primeira a entrar) → `[2, 3, 4]`
  - **LRU** remove a `2` (a `1` foi usada há pouco) → `[3, 1, 4]`

Os dois produzem o **mesmo** resultado quando:

- não há reacesso a páginas carregadas antes das substituições (sem HITs, as duas ordens coincidem);
- a quantidade de frames é suficiente para todas as páginas distintas, pois aí só ocorrem os page faults do primeiro acesso de cada página.

Observação: LRU **não é sempre melhor**. Na sequência `1, 2, 3, 1, 4, 2, 5, 1` com 3 frames, o FIFO teve 6 page faults e o LRU teve 7.

---

### 6. Por que o algoritmo Ótimo normalmente apresenta um bom resultado?

Porque ele substitui a página cujo **próximo uso está mais distante no futuro** (ou que nunca mais será usada). Assim ele sempre mantém na memória as páginas que serão necessárias mais cedo, adiando ao máximo o próximo page fault.

Como a decisão é tomada com conhecimento completo dos acessos futuros, ele obtém o **menor número possível de page faults** para uma sequência e uma quantidade de frames. Nenhum outro algoritmo consegue ser melhor, e por isso ele é usado como referência de comparação.

No experimento do notebook (`7, 0, 1, 2, 0, 3, 0, 4, 2, 3, 0, 3, 2`, 3 frames): FIFO = 10, LRU = 9, Ótimo = 7.

---

### 7. Por que um sistema operacional real não consegue utilizar perfeitamente o algoritmo Ótimo?

Porque ele precisa **conhecer o futuro**: saber de antemão quais páginas o processo vai acessar e em que ordem.

Em um sistema real isso não é possível, pois os acessos dependem de entradas do usuário, dos dados processados, de desvios condicionais, de interrupções e do escalonamento entre processos. Na simulação só funciona porque a lista inteira de acessos já é conhecida (`futuro = paginas[i + 1:]`).

Na prática, os sistemas usam algoritmos que **estimam** o futuro com base no passado. O LRU é o principal exemplo: ele supõe que uma página usada recentemente provavelmente será usada de novo em breve (princípio da localidade).

---

### 8. Execute pelo menos três sequências diferentes e registre qual algoritmo apresentou menos page faults em cada uma.

Todas as execuções abaixo foram feitas com **3 frames**, usando as funções `fifo`, `lru` e `otimo` do notebook.

| # | Sequência | Acessos | FIFO | LRU | Ótimo | Menos page faults (FIFO × LRU) |
|---|---|---|---|---|---|---|
| A | `1 2 3 1 4 2 5 1` | 8 | 6 | 7 | 5 | **FIFO** |
| B | `7 0 1 2 0 3 0 4 2 3 0 3 2` | 13 | 10 | 9 | 7 | **LRU** |
| C | `1 2 3 4 1 2 5 1 2 3 4 5` | 12 | 9 | 10 | 7 | **FIFO** |
| D | 30 acessos aleatórios entre 0 e 9 (`random.seed(42)`) | 30 | 24 | 23 | 17 | **LRU** |

Taxa de page faults (page faults ÷ total de acessos × 100):

| # | FIFO | LRU | Ótimo |
|---|---|---|---|
| A | 75,00% | 87,50% | 62,50% |
| B | 76,92% | 69,23% | 53,85% |
| C | 75,00% | 83,33% | 58,33% |
| D | 80,00% | 76,67% | 56,67% |

Sequência D completa: `1 0 4 3 3 2 1 8 1 9 6 0 0 1 3 3 8 9 0 8 3 8 6 3 7 9 4 0 2 6`

**Conclusões:**

- O **Ótimo** teve o menor número de page faults em todas as sequências, como esperado.
- Entre FIFO e LRU **não houve um vencedor fixo**: o FIFO foi melhor em A e C, o LRU foi melhor em B e D.
- O desempenho depende do algoritmo **e** do padrão de acesso às páginas. O LRU leva vantagem quando as páginas usadas recentemente voltam a ser usadas logo (localidade), como em B, onde a página `0` é reacessada várias vezes.
