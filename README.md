# Teste de Caixa Branca – Sistema de Pedidos

**Instituição:** SENAI  
**Curso:** Técnico de Desenvolvimento de Sistemas  
**Unidade Curricular:** SESI CE 356  
**Atividade:** Teste de Caixa Branca – Sistema de Pedidos  
**Aluno:** julianopls  
**Turma:** 3A  
**Professores:** Robson, Reenye e Wellington  
**Data:** 30/09/2026  

---

# 1. Contextualização

No teste de caixa branca eu abro o código e testo a lógica por dentro, passando por cada `if` e cada caminho possível.

Isso ajuda a encontrar erros que aparecem em situações específicas, principalmente em valores que ficam exatamente no limite de uma regra.

**Sistema analisado:** uma página onde o usuário escolhe um produto, informa a quantidade, pode utilizar um cupom e escolhe o tipo de frete.

Ao clicar em **"Calcular pedido"**, o sistema mostra:

- Subtotal;
- Desconto;
- Frete;
- Total;
- Mensagem do pedido.

A lógica está no arquivo `script.js`.

---

# 2. Comportamento Esperado

| Regra | O que eu espero |
|---|---|
| Quantidade | Inteiro maior que zero |
| Estoque | Posso pedir até a quantidade em estoque, inclusive |
| Cupom `SENAI10` | 10% sobre o subtotal |
| Cupom `SENAI20` | 20% sobre o subtotal, somente com subtotal a partir de R$ 1.000 |
| Desconto por quantidade | 5% sobre o subtotal a partir de 5 unidades |
| Retirada | Frete grátis |
| Expresso | R$ 60 |
| Normal | R$ 30, grátis com subtotal a partir de R$ 500 |
| Alto valor | Total a partir de R$ 3.000 recebe 5% extra |
| Exibição | Subtotal − descontos + frete deve bater com o total mostrado |

---

# 3. Estruturas de Decisão

O código utiliza estruturas `if` para controlar os diferentes caminhos do sistema.

| ID | Onde | Condição | Caminhos |
|---|---|---|---|
| D1 | `calcularDesconto` | `codigo === "SENAI10"` | 10% / continua |
| D2 | `calcularDesconto` | `codigo === "SENAI20" && subtotal >= 1000` | 20% / sem desconto |
| D3 | `calcularFrete` | `tipo === "retirada"` | R$ 0 / continua |
| D4 | `calcularFrete` | `tipo === "expresso"` | R$ 60 / continua |
| D5 | `calcularFrete` | `subtotal >= 500` | R$ 0 / R$ 30 |
| D6 | `finalizarPedido` | quantidade inválida | Erro / continua |
| D7 | `finalizarPedido` | `qtd > estoque` | Indisponível / calcula |
| D8 | `finalizarPedido` | `qtd >= 5` | Aplica 5% / não aplica |
| D9 | `finalizarPedido` | `totalParcial >= 3000` | Aplica 5% / não aplica |
| D10 | `finalizarPedido` | `total <= 0` | Valor inválido / continua |
| D11 | `finalizarPedido` | `altoValor` | Alto valor / sucesso |

---

# 4. Fluxogramas do Sistema

## 4.1 Fluxograma geral

![Fluxograma geral](assets/mermaid-diagram.png)

---

## 4.2 Fluxograma — Calcular desconto

![Fluxograma calcular desconto](assets/mermaid-diagram%20(1).png)

---

## 4.3 Fluxograma — Desconto por quantidade

![Fluxograma desconto por quantidade](assets/mermaid-diagram%20(2).png)

---

## 4.4 Fluxograma — Calcular frete

![Fluxograma calcular frete](assets/mermaid-diagram%20(3).png)

---

## 4.5 Fluxograma — Validação da quantidade

![Fluxograma validação da quantidade](assets/mermaid-diagram%20(4).png)

---

## 4.6 Fluxograma — Cálculo do total

![Fluxograma cálculo do total](assets/mermaid-diagram%20(5).png)

---

## 4.7 Fluxograma — Pedido de alto valor

![Fluxograma pedido de alto valor](assets/mermaid-diagram%20(6).png)

---

## 4.8 Fluxograma completo da execução

![Fluxograma completo](assets/mermaid-diagram%20(7).png)

---

#. Arquivos do Projeto

O projeto é composto pelos seguintes arquivos:

- `index.html` — estrutura da página;
- `script.js` — lógica principal do sistema;
- `script2.js` — código complementar;
- `style.css` — estilos da página;
- `README.md` — documentação do projeto;
- `assets/` — imagens dos fluxogramas.

---


## 5. Análise dos Erros

### Erro 1 — Fácil (valores-limite)

```js
if (qtd < 0) { ... }
```

**Teste:** mouse, quantidade 0, frete normal.
**Caminho:** em D6, `0 < 0` é falso, então o pedido não é barrado. O subtotal é 0, mas em D5 `0 >= 500` é falso e o frete fica 30. O total vira 30 e o resultado é "Sucesso".
**Problema:** o zero, primeiro valor inválido, passa. Campo vazio e decimais também passam. O cliente pagaria frete por zero itens.
**Correção:**
```js
if (!Number.isInteger(qtd) || qtd <= 0) {
```

```mermaid
flowchart TD
    A[/qtd = 0, mouse, normal/] --> B{"qtd < 0? (0 < 0)"}
    B -- "Falso (erro)" --> C["subtotal = 0; frete = 30; total = 30"]
    C --> D["Exibe: Sucesso, R$ 30,00"]
    B -. Esperado .-> X["Quantidade inválida"]
```

### Erro 2 — Fácil (valores-limite)

```js
if (qtd >= estoque[produtoSelecionado]) { ... }
```

**Teste:** teclado (estoque 10), quantidade 10, retirada.
**Caminho:** em D7, `10 >= 10` é verdadeiro, então mostra "indisponível" e encerra com `return`.
**Problema:** o `>=` bloqueia justamente o pedido que usa todo o estoque. O notebook (estoque 5) nunca poderia ser comprado em 5 unidades.
**Correção:**
```js
if (qtd > estoque[produtoSelecionado]) {
```
Também testei quantidade 11, que continua sendo recusada.

```mermaid
flowchart TD
    A[/qtd = 10, estoque = 10/] --> B{"qtd >= estoque? (10 >= 10)"}
    B -- "Verdadeiro (erro)" --> C["Indisponível e return"]
    B -. Esperado .-> D["total = 1425"]
```

### Erro 3 — Médio (valores-limite e cobertura de decisões)

```js
if (qtd > 5) { total = total - subtotal * 0.05; }
```

**Teste:** mouse, quantidade 5, retirada.
**Caminho:** subtotal 400, frete 0, total 400. Em D8, `5 > 5` é falso, então o desconto não é aplicado.
**Problema:** o `>` deixa de fora exatamente a quantidade que abre a faixa. O certo é `>=`.
**Correção:**
```js
const QTD_MINIMA_DESCONTO = 5;
if (qtd >= QTD_MINIMA_DESCONTO) { desconto += subtotal * 0.05; }
```
Com 5 unidades o total passa a ser R$ 380,00, e com 4 continua sem desconto.

```mermaid
flowchart TD
    A[/qtd = 5, mouse, retirada/] --> B["total = 400"]
    B --> C{"qtd > 5? (5 > 5)"}
    C -- "Falso (erro)" --> D["Exibe: R$ 400,00"]
    C -. Esperado .-> E["total = 400 − 20 = 380"]
```

### Erro 4 — Médio (rastreamento de variáveis)

```js
const desconto = calcularDesconto(subtotal, codigo);
let total = subtotal - desconto + valorFrete;
if (qtd > 5) { total = total - subtotal * 0.05; }
```

**Teste:** mouse, quantidade 10, cupom SENAI10, retirada.
**Caminho:** subtotal 800; cupom dá `desconto` = 80; total = 720. Em D8, o total vira 720 − 40 = 680, mas `desconto` continua valendo **80**.
**Problema:** o desconto por quantidade mexe direto em `total` e nunca entra na variável `desconto`. Na tela aparece 800 − 80 ≠ 680, e um desconto do cliente fica escondido.
**Correção:** somar os dois descontos na mesma variável (`let desconto` + `desconto += subtotal * 0.05`), como no código final. A tela passa a mostrar desconto R$ 120,00 e total R$ 680,00.

```mermaid
flowchart TD
    A[/mouse, qtd = 10, SENAI10/] --> B["subtotal = 800; desconto = 80"]
    B --> C["total = 720"]
    C --> D["qtd > 5: total = 680 (desconto continua 80)"]
    D --> E["Exibe: 80 / 680 (não fecha)"]
    D -. Correção .-> F["desconto = 80 + 40 = 120"]
```

### Erro 5 — Difícil (condições que dependem uma da outra)

```js
if (total > 3000) { total = total * 0.95; }
...
else if (total >= 3000) { mensagem = "Pedido de alto valor."; }
```

**Teste:** notebook, quantidade 1, retirada (total exatamente 3000).
**Caminho:** em D9, `3000 > 3000` é falso, então não há 5% extra. Em D11, `3000 >= 3000` é verdadeiro e a mensagem é "Alto valor".
**Problema:** duas decisões testam o mesmo limite com operadores diferentes. O pedido é chamado de alto valor, mas não ganha o benefício. O erro só aparece exatamente em 3000.
**Correção:** uma constante única e a mesma comparação nos dois lugares:
```js
const LIMITE_ALTO_VALOR = 3000;
const altoValor = totalParcial >= LIMITE_ALTO_VALOR;
```
Resultado: total R$ 2.850,00 e "Pedido de alto valor".

```mermaid
flowchart TD
    A[/notebook, qtd = 1, retirada/] --> B["total = 3000"]
    B --> C{"total > 3000?"}
    C -- "Falso (sem 5%)" --> D{"total >= 3000?"}
    D -- Verdadeiro --> E["Alto valor, R$ 3.000,00 (inconsistente)"]
```

### Erro 6 — Difícil (análise de caminhos e rastreamento de variáveis)

**Teste:** notebook, quantidade 1, frete expresso.
**Caminho:** total = 3000 + 60 = 3060. Em D9, `3060 > 3000` é verdadeiro e o total vira 2907. Em D11, `2907 >= 3000` é falso e a mensagem sai "Sucesso".
**Problema:** ordem das operações. O desconto extra reduz `total` e logo depois a mesma variável classifica o pedido. Qualquer pedido entre R$ 3.000,00 e cerca de R$ 3.157,89 perde a classificação. Uma decisão altera o dado que a próxima usa.
**Correção:** decidir se é alto valor antes de mexer no total:
```js
const altoValor = totalParcial >= LIMITE_ALTO_VALOR;
const descontoAltoValor = altoValor ? totalParcial * 0.05 : 0;
const total = totalParcial - descontoAltoValor;
```
Resultado: "Pedido de alto valor." e total R$ 2.907,00.

```mermaid
flowchart TD
    A[/notebook, qtd = 1, expresso/] --> B["total = 3060"]
    B --> C{"total > 3000?"}
    C -- Sim --> D["total = 2907"]
    D --> E{"total >= 3000? (2907)"}
    E -- "Falso (erro)" --> F["Sucesso, R$ 2.907,00"]
```


## 6. Código Corrigido

O arquivo completo está em [`scriptnovo.js`](./scriptnovo.js). Função `finalizarPedido` corrigida:

```js
const LIMITE_ALTO_VALOR = 3000;
const QTD_MINIMA_DESCONTO = 5;

function finalizarPedido() {
  const produtoSelecionado = produto.value;
  const qtd = Number(quantidade.value);
  const codigo = cupom.value.trim().toUpperCase();

  if (!Number.isInteger(qtd) || qtd <= 0) {
    resultado.innerHTML = "<p>Quantidade inválida.</p>";
    return;
  }

  if (qtd > estoque[produtoSelecionado]) {
    resultado.innerHTML = "<p>Quantidade indisponível em estoque.</p>";
    return;
  }

  const subtotal = precos[produtoSelecionado] * qtd;
  let desconto = calcularDesconto(subtotal, codigo);

  if (qtd >= QTD_MINIMA_DESCONTO) {
    desconto += subtotal * 0.05;
  }

  const valorFrete = calcularFrete(frete.value, subtotal);
  const totalParcial = subtotal - desconto + valorFrete;

  const altoValor = totalParcial >= LIMITE_ALTO_VALOR;
  const descontoAltoValor = altoValor ? totalParcial * 0.05 : 0;
  const total = totalParcial - descontoAltoValor;

  let mensagem = "Pedido calculado com sucesso.";

  if (total <= 0) {
    mensagem = "Valor do pedido inválido.";
  } else if (altoValor) {
    mensagem = "Pedido de alto valor.";
  }

  resultado.innerHTML = `
    <p>${mensagem}</p>
    <p>Subtotal: R$ ${subtotal.toFixed(2)}</p>
    <p>Desconto: R$ ${desconto.toFixed(2)}</p>
    ${altoValor ? `<p>Desconto alto valor: R$ ${descontoAltoValor.toFixed(2)}</p>` : ""}
    <p>Frete: R$ ${valorFrete.toFixed(2)}</p>
    <p class="total">Total: R$ ${total.toFixed(2)}</p>
  `;
}
```

---