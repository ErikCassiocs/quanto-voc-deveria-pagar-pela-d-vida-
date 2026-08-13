# Estudo 8 — Quanto você deveria pagar de dívida por mês

> Modelo quantitativo que calcula a faixa recomendada de comprometimento mensal com dívida, a partir da renda, do custo da dívida e do saldo devedor — sem depender de um percentual fixo genérico.
>
> Parte da série **Economia em Dados** ([@financelabr](https://x.com/financelabr)) — terceiro e último estudo da trilogia sobre endividamento, após o Estudo 6 (custo do pagamento mínimo do cartão) e o Estudo 7 (bola de neve vs. avalanche).

---

## Por que esse modelo existe

"Pague 20% da sua renda em dívida" é a resposta padrão que circula por aí — e é uma resposta genérica demais. O valor certo depende de três coisas que mudam de pessoa pra pessoa:

- **quanto sobra** da renda depois de preservar o mínimo pra viver;
- **quão cara** é a dívida específica, comparada ao custo de oportunidade do dinheiro (CDI líquido);
- **se o pagamento recomendado realmente reduz o saldo**, ou só cobre uma fração dos juros.

Este repositório contém a fórmula, o notebook de desenvolvimento (com todo o histórico de correções) e os testes de estresse que validam o modelo.

## O que o modelo faz

Dado `renda líquida`, `taxa de juros da dívida` e `saldo devedor`, o modelo devolve:

- uma **faixa recomendada** de pagamento mensal (não um número único);
- a **classificação de urgência** da dívida (🔴 Extremamente cara → 🟢 Baixa), baseada no spread sobre o CDI líquido;
- o **prazo estimado de quitação**, calculado com amortização real (juros compostos sobre o saldo — não divisão simples);
- **alertas automáticos** quando a parcela mínima contratual já ultrapassa a capacidade segura, ou quando mesmo o pagamento recomendado não cobre os juros do mês (distinguindo se o problema é custo da dívida ou escala da dívida).

O modelo **nunca** recomenda um valor que viole o mínimo existencial (piso baseado na Lei do Superendividamento, 14.181/2021).

## Como usar

### Opção 1 — Google Colab (recomendado)
Clique no badge "Open in Colab" acima, ou abra `estudo8_formula_divida.ipynb` diretamente pelo Colab. Todas as células rodam de cima pra baixo sem dependências externas além de `pandas` e `matplotlib` (já disponíveis no Colab por padrão).

### Opção 2 — Localmente
```bash
git clone https://github.com/SEU_USUARIO/SEU_REPOSITORIO.git
cd SEU_REPOSITORIO
pip install jupyter pandas matplotlib
jupyter notebook estudo8_formula_divida.ipynb
```

### Uso rápido da função (dentro do notebook)
```python
resposta = gerar_resposta_rapida_v22(
    renda=6000,
    taxa=106.6,          # em % a.a. (ou use unidade_taxa="am" para % a.m.)
    unidade_taxa="aa",
    total_divida=12000,  # opcional — habilita o cálculo de prazo
    parcela_minima=None  # opcional — parcela/mínimo contratual já assumido
)
print(resposta)
```

## Metodologia (resumo)
