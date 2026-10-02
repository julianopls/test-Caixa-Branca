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





## 6. Código Corrigido

O arquivo completo está em [`script2.js`](./script2.js). Função `finalizarPedido` corrigida:

```js
const precos = {
  notebook: 3000,
  mouse: 80,
  teclado: 150
};

const estoque = {
  notebook: 5,
  mouse: 20,
  teclado: 10
};

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
## Conclusão

Com essa atividade, foi possível perceber que um programa pode funcionar normalmente e ainda apresentar erros na lógica. Os seis problemas encontrados não faziam o sistema parar ou mostrar mensagens de erro, mas causavam resultados diferentes do que era esperado. Para identificar esses problemas, foi necessário acompanhar os valores das variáveis e entender cada decisão tomada pelo código.

A maioria dos erros estava relacionada aos limites das condições, como diferenças entre `<`, `<=`, `>` e `>=`. Também foi encontrado um problema relacionado ao desconto apresentado ao usuário e outro causado pela ordem em que as operações eram executadas, fazendo uma decisão alterar o valor utilizado em outra etapa.

Os testes ajudaram a confirmar cada problema. Antes das correções, os seis casos apresentavam resultados incorretos. Depois dos ajustes, todos passaram a funcionar de acordo com as regras definidas. O fluxograma também foi importante para entender melhor o caminho que o programa seguia e facilitar a identificação dos erros.
