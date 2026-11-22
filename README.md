# Fila de Fraude

Projeto de estudo de detecção de fraude em cartão de crédito. O foco não é só a métrica do modelo, mas a decisão que ele apoia: **quais transações entram na fila de revisão de uma equipe com capacidade limitada**.

**Página interativa:** [`docs/index.html`](docs/index.html). Um simulador mostra quantas fraudes cada modelo encontra conforme a equipe revisa mais ou menos transações.

## Pergunta de negócio

Qual modelo identifica mais fraudes, e maior valor fraudado, dentro da capacidade disponível de revisão?

## Dados

[Credit Card Fraud Detection](https://www.kaggle.com/datasets/mlg-ulb/creditcardfraud) (ULB, Kaggle): 284.807 transações de cartões europeus em cerca de 48 horas de setembro de 2013, das quais 492 são fraude (0,17%). As variáveis `V1` a `V28` vêm anonimizadas por PCA.

O CSV não está no repositório. Baixe `creditcard.csv` do Kaggle e coloque na mesma pasta do notebook.

## Método

1. **Qualidade dos dados.** O carregamento valida colunas, tipos, ausentes e infinitos. Foram removidas 1.081 linhas duplicadas antes de separar os dados.
2. **Separação temporal.** Treino (70%), validação (15%) e teste (15%) em ordem de tempo, mantendo timestamps iguais no mesmo conjunto. `Time` serve só para ordenar e não entra no modelo.
3. **Comparação de modelos na validação.** Seis modelos, incluindo um classificador aleatório como referência. A métrica principal é a Average Precision (AP), adequada para classes raras.
4. **Corte pela capacidade.** O corte de score é o da 100ª transação mais suspeita na validação, cerca de 20 revisões por hora.
5. **Estabilidade temporal.** Quatro janelas sucessivas do período de desenvolvimento. Em cada uma, os modelos são retreinados com todo o passado e avaliados nas horas seguintes.
6. **Diagnóstico.** Análise das fraudes sem alerta, importância por permutação e exposição financeira.

## Resultados

Modelo selecionado: **Random Forest**.

| Indicador | Resultado |
|---|---|
| AP na validação | 0,875 |
| AP média em 4 janelas sucessivas (mínima) | 0,772 (0,686) |
| AP no período final | 0,773 |
| Fraudes encontradas no período final | 39 de 52 (75%) |
| Alertas gerados | 98 de 42.561 transações (16,4 por hora) |
| Alertas que eram fraude | 40% |
| Valor fraudado sinalizado | 61,5% |

A AP da validação foi otimista. A média nas janelas sucessivas é praticamente igual à do período final, e esse é o desempenho típico esperado.

## Limitações

- A base cobre só cerca de 48 horas, com poucas dezenas de fraudes por período de avaliação.
- O período de teste já havia sido consultado em versões anteriores do estudo. Os números dele são exploratórios, e a confirmação exige um período novo.
- As janelas sucessivas usam o mesmo período em que os modelos foram comparados.
- A PCA das variáveis `V` foi feita pelos autores da base, provavelmente sobre todos os dados.
- Valor sinalizado é exposição, não perda evitada.
- O score não está calibrado como probabilidade.
- A capacidade de 100 revisões é uma suposição do estudo.

## Como executar

```bash
python -m venv .venv
.venv\Scripts\activate
pip install -r requirements.txt
```

Abra `fraude_ml.ipynb` e execute as células em ordem após reiniciar o kernel. Os parâmetros do experimento, como capacidade de revisão, critério de seleção e número de janelas, ficam na primeira célula de código.
