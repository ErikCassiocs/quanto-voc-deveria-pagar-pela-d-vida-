Estudo 8 — Quanto você deveria pagar de dívida por mês

Modelo quantitativo que calcula a faixa recomendada de comprometimento mensal com dívida, a partir da renda, do custo da dívida e do saldo devedor — sem depender de um percentual fixo genérico.

Parte da série Economia em Dados (@financelabr) — terceiro e último estudo da trilogia sobre endividamento, após o Estudo 6 (custo do pagamento mínimo do cartão) e o Estudo 7 (bola de neve vs. avalanche).

Por que esse modelo existe

"Pague 20% da sua renda em dívida" é a resposta padrão que circula por aí — e é uma resposta genérica demais. O valor certo depende de três coisas que mudam de pessoa pra pessoa:

quanto sobra da renda depois de preservar o mínimo pra viver;
quão cara é a dívida específica, comparada ao custo de oportunidade do dinheiro (CDI líquido);
se o pagamento recomendado realmente reduz o saldo, ou só cobre uma fração dos juros.

Este repositório contém a fórmula, o notebook de desenvolvimento (com todo o histórico de correções) e os testes de estresse que validam o modelo.

O que o modelo faz

Dado renda líquida, taxa de juros da dívida e saldo devedor, o modelo devolve:

uma faixa recomendada de pagamento mensal (não um número único);
a classificação de urgência da dívida (🔴 Extremamente cara → 🟢 Baixa), baseada no spread sobre o CDI líquido;
o prazo estimado de quitação, calculado com amortização real (juros compostos sobre o saldo — não divisão simples);
alertas automáticos quando a parcela mínima contratual já ultrapassa a capacidade segura, ou quando mesmo o pagamento recomendado não cobre os juros do mês (distinguindo se o problema é custo da dívida ou escala da dívida).

O modelo nunca recomenda um valor que viole o mínimo existencial (piso baseado na Lei do Superendividamento, 14.181/2021).

Como usar
Opção 1 — Google Colab (recomendado)

Clique no badge "Open in Colab" acima, ou abra estudo8_formula_divida.ipynb diretamente pelo Colab. Todas as células rodam de cima pra baixo sem dependências externas além de pandas e matplotlib (já disponíveis no Colab por padrão).

Opção 2 — Localmente
bash
git clone https://github.com/SEU_USUARIO/SEU_REPOSITORIO.git
cd SEU_REPOSITORIO
pip install jupyter pandas matplotlib
jupyter notebook estudo8_formula_divida.ipynb
Uso rápido da função (dentro do notebook)
python
resposta = gerar_resposta_rapida_v22(
    renda=6000,
    taxa=106.6,          # em % a.a. (ou use unidade_taxa="am" para % a.m.)
    unidade_taxa="aa",
    total_divida=12000,  # opcional — habilita o cálculo de prazo
    parcela_minima=None  # opcional — parcela/mínimo contratual já assumido
)
print(resposta)
Metodologia (resumo)
Mínimo Existencial = max(salário mínimo, 40% × renda líquida)
Capacidade = (Renda − Mínimo Existencial) × (1 − margem de proteção mensal)
Spread (p.p.) = Taxa anual da dívida − CDI líquido anual
Alocação = Capacidade × f(spread)   # f(spread) via tabela de 5 faixas
Pagamento recomendado = max(parcela mínima contratual, Alocação)
Prazo = solução de D = P × (1 − (1+i)⁻ⁿ) / i   # amortização real, não divisão simples
Spread sobre o CDI líquido	Classificação	% da capacidade alocado
≥ 300 p.p.	🔴 Extremamente cara	85% – 100%
100 – 300 p.p.	🔴 Muito cara	65% – 85%
40 – 100 p.p.	🟠 Cara	40% – 65%
5 – 40 p.p.	🟡 Moderada	15% – 40%
< 5 p.p.	🟢 Baixa	0% – 15% (mínimo contratual)

Os limiares de spread são parâmetros heurísticos calibrados, não uma estimação empírica — essa distinção está documentada no notebook e deve ser mantida em qualquer material publicado a partir deste modelo.

Premissas assumidas (não são valores legais/regulatórios)
Mínimo existencial: a Lei 14.181/2021 exige a preservação de um piso, mas não define um valor objetivo. max(salário mínimo, 40% da renda) é uma escolha metodológica deste estudo.
CDI líquido: aproximado com IR de 15% sobre o CDI bruto (alíquota regressiva mínima, válida para resgates acima de 720 dias). Resgates mais curtos têm alíquota maior (até 22,5%).
Margem de proteção mensal: 15% por padrão — um buffer que fica sempre reservado, não uma reserva de emergência formal.
Histórico de versões
Versão	Mudança
v1	Modelo inicial. Urgência normalizada pela distância até o rotativo do cartão (defeito: comprimia taxas moderadas perto de zero).
v2	Separação entre capacidade de comprometimento e alocação por urgência. Classificação por spread sobre o CDI líquido em vez de normalização pelo pior caso do mercado.
v2.1	Adição da parcela/mínimo contratual como piso de pagamento. Prazo recalculado com amortização real (juros compostos), corrigindo um erro que subestimava prazos em até 2x.
v2.2 (congelada)	Diagnóstico de não amortização, distinguindo custo elevado (Alerta A) de escala da dívida (Alerta B). Indicador de cobertura de juros (%).

A partir da v2.2, qualquer mudança na lógica deve ser tratada como v2.3, baseada em evidência real de uso — não em continuar refinando com cenários sintéticos.

O modelo foi validado com uma simulação de Monte Carlo (2.000 cenários aleatórios de renda, taxa e dívida) e um teste específico de continuidade nas fronteiras entre faixas de urgência. Detalhes completos no notebook, seções "Testes de estresse".

Estrutura do repositório
.
├── estudo8_formula_divida.ipynb   # notebook completo: modelo, testes, histórico de versões
├── thread_x_estudo8.md            # thread publicada no X a partir deste estudo
└── README.md
Aviso importante

Este modelo é material educacional e de conteúdo — não é aconselhamento financeiro individualizado. As premissas (mínimo existencial, margem de proteção, limiares de classificação) são escolhas metodológicas explícitas do autor, não determinações legais ou recomendações regulatórias. Antes de tomar decisões sobre suas próprias dívidas, considere consultar um profissional financeiro ou, em caso de superendividamento, os mecanismos de repactuação previstos na Lei 14.181/2021.

Fontes de dados
Taxas de juros por modalidade de crédito: Banco Central do Brasil — Séries Temporais (SGS)
CDI: B3 / ANBIMA
Mínimo existencial: Lei 14.181/2021 (Lei do Superendividamento)

As taxas de referência usadas no notebook (CDI_ANUAL, ROTATIVO_ANUAL, SALARIO_MINIMO) devem ser atualizadas periodicamente — estão documentadas com a data de referência no início do notebook.

Licença

Defina a licença do repositório (ex: MIT) conforme sua preferência de uso e redistribuição do conteúdo.

Autor

@financelabr — Série Economia em Dados
