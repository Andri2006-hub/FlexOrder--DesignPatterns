[README.md](https://github.com/user-attachments/files/23103051/README.md)
# 🧩 FlexOrder - Design Patterns

## 📘 Descrição Geral

O **FlexOrder** é um projeto de refatoração que aplica **padrões de projeto orientados a objetos (GoF)** para transformar um código legado de processamento de pedidos e checkout em uma arquitetura modular, extensível e de fácil manutenção.

A refatoração aborda **problemas clássicos de acoplamento e violação de princípios SOLID**, especialmente o **SRP (Single Responsibility Principle)** e o **OCP (Open-Closed Principle)**, utilizando três padrões:

- 🧠 **Strategy** — para encapsular algoritmos de pagamento e frete.  
- 🎁 **Decorator** — para adicionar descontos e taxas dinamicamente.  
- 🧩 **Facade** — para simplificar o fluxo de checkout e orquestrar subsistemas.

---

## 🏗️ Arquitetura Orientada a Objetos

A nova estrutura do sistema foi organizada em módulos independentes, cada um com responsabilidade única e clara.  
A seguir, a estrutura de diretórios:

```
/src
 ├── main.py
 │
 ├── pedido/
 │    ├── pedido.py
 │    ├── calculo_valor.py
 │    └── decorators/
 │         ├── desconto_pix.py
 │         └── taxa_embalagem_presente.py
 │
 ├── strategy/
 │    ├── pagamento_strategy.py
 │    ├── frete_strategy.py
 │    └── concretos/
 │         ├── pagamento_cartao.py
 │         ├── pagamento_pix.py
 │         ├── frete_normal.py
 │         └── frete_express.py
 │
 └── checkout/
      ├── facade_checkout.py
      └── sistemas/
           ├── sistema_estoque.py
           └── gerador_nota_fiscal.py
```

Cada módulo respeita o **SRP**, e qualquer extensão (novo método de pagamento, novo tipo de desconto, etc.) é feita sem alterar o código base — respeitando o **OCP**.

---

## ⚙️ Padrões de Projeto Utilizados

### 🧠 1. Strategy — Estratégia de Pagamento e Frete

**Problema no código legado:**  
O método `finalizar_compra()` possuía condicionais rígidas (`if pagamento == "pix"`, `if frete == "express"`, etc.), quebrando o OCP e dificultando a inclusão de novos tipos de pagamento e frete.

**Solução aplicada:**  
Cada tipo de pagamento e cálculo de frete foi transformado em uma **classe concreta** que implementa uma **interface de estratégia**.

```python
class PagamentoStrategy:
    def pagar(self, valor):
        pass

class PagamentoPix(PagamentoStrategy):
    def pagar(self, valor):
        print(f"Pagamento via Pix realizado: R${valor:.2f}")
```

**Benefícios:**
- Extensibilidade sem alterar código existente (OCP).  
- Clareza e isolamento de responsabilidades (SRP).

---

### 🎁 2. Decorator — Descontos e Taxas Dinâmicas

**Problema no código legado:**  
Descontos e taxas estavam fixos dentro da classe `Pedido`, forçando alterações no método `calcular_total()` para cada nova regra de negócio.

**Solução aplicada:**  
Foi criada uma **hierarquia de Decorators** que envolvem o objeto `Pedido` e adicionam comportamentos dinamicamente.

```python
class CalculoValor:
    def calcular(self):
        pass

class DescontoPix(CalculoValor):
    def __init__(self, pedido):
        self._pedido = pedido

    def calcular(self):
        return self._pedido.calcular() * 0.95  # 5% de desconto
```

**Composição dinâmica:**

```python
pedido = Pedido(200)
pedido = DescontoPix(pedido)
pedido = TaxaEmbalagemPresente(pedido)
print(pedido.calcular())
```

**Benefícios:**
- Permite combinar múltiplos comportamentos em tempo de execução.  
- Elimina a necessidade de alterar a classe base.  
- Corrige violações de **SRP** e **OCP**.

---

### 🧩 3. Facade — Simplificação do Processo de Checkout

**Problema no código legado:**  
A função `finalizar_compra()` fazia tudo: processava pagamento, atualizava estoque e gerava nota fiscal. Isso gerava alto acoplamento e baixo reuso.

**Solução aplicada:**  
A classe `CheckoutFacade` foi criada para **orquestrar os subsistemas** e fornecer uma interface única para o cliente.

```python
class CheckoutFacade:
    def __init__(self):
        self.estoque = SistemaEstoque()
        self.nota_fiscal = GeradorNotaFiscal()

    def concluir_transacao(self, pedido, pagamento_strategy, frete_strategy):
        total = pedido.calcular()
        total_com_frete = frete_strategy.calcular_frete(total)
        pagamento_strategy.pagar(total_com_frete)
        self.estoque.atualizar(pedido)
        self.nota_fiscal.gerar(pedido)
        print("Transação concluída com sucesso!")
```

**Benefícios:**
- Simplifica a interação do cliente com o sistema.  
- Reduz o acoplamento entre módulos internos.  
- Aumenta a testabilidade e clareza do código.

---

## ✅ Correções de Violações SRP e OCP

| Problema Original | Violação | Solução Aplicada |
|-------------------|-----------|------------------|
| Método `finalizar_compra()` centralizava pagamento, frete e nota fiscal | **SRP** | Uso de **Strategy** e **Facade** |
| Inclusão de novos meios de pagamento exigia alterar código existente | **OCP** | Estratégias de pagamento e frete intercambiáveis |
| Lógica de descontos e taxas embutidas no `Pedido` | **SRP / OCP** | Aplicação do padrão **Decorator** |
| Múltiplas responsabilidades em um único método | **SRP** | Divisão em subsistemas independentes e Fachada |

---

## 🧪 Exemplo de Execução

**Arquivo:** `src/main.py`

```python
from pedido.pedido import Pedido
from pedido.decorators.desconto_pix import DescontoPix
from pedido.decorators.taxa_embalagem_presente import TaxaEmbalagemPresente
from strategy.concretos.pagamento_pix import PagamentoPix
from strategy.concretos.frete_express import FreteExpress
from checkout.facade_checkout import CheckoutFacade

if __name__ == "__main__":
    pedido = Pedido(200.00)
    pedido = DescontoPix(pedido)
    pedido = TaxaEmbalagemPresente(pedido)

    pagamento = PagamentoPix()
    frete = FreteExpress()

    checkout = CheckoutFacade()
    checkout.concluir_transacao(pedido, pagamento, frete)
```

**Saída esperada:**
```
🛒 Iniciando checkout...

⚡ Frete Expresso: +R$30.00
💸 Pagamento via Pix realizado: R$230.00
📦 Estoque atualizado com o pedido.
🧾 Nota fiscal gerada para o pedido.

✅ Transação concluída com sucesso!
```

---

## 🧰 Tecnologias e Padrões

- Linguagem: **Python 3.12+**
- Padrões aplicados:
  - Strategy
  - Decorator
  - Facade
- Princípios SOLID:
  - SRP (Single Responsibility Principle)
  - OCP (Open-Closed Principle)

---

## 🧑‍💻 Autor

**Andri Santana**  
📂 Repositório: [FlexOrder-DesignPatterns](https://github.com/andrisantana07/FlexOrder-DesignPatterns)  
📧 Contato: *(santanaandri310@gmail.com)*

---

> 💡 **Resumo:**  
> A aplicação dos padrões **Strategy**, **Decorator** e **Facade** eliminou duplicações, reduziu o acoplamento e tornou o sistema de pedidos **flexível, extensível e fácil de manter**, de acordo com os princípios SOLID e boas práticas de design orientado a objetos.
