# Estimativa de Tempo de Entrega (ETA) para E-commerce Olist

## Visão Geral
Este projeto visa desenvolver um modelo preditivo para estimar com precisão o tempo de entrega de pedidos de e-commerce, utilizando dados históricos da Olist. O objetivo é melhorar a experiência do cliente, fornecendo prazos mais realistas e reduzindo a frustração causada por atrasos.

## 1. Entendimento do Negócio (Business Understanding)
*   **Objetivo**: Prever o tempo de entrega (em dias) entre a aprovação do pagamento e a entrega ao cliente.
*   **Problema**: Atrasos na entrega impactam negativamente a satisfação do cliente e as avaliações do serviço.
*   **Critério de Sucesso**: Redução do erro de previsão (RMSE - Root Mean Square Error) em comparação com uma estimativa baseline (modelo ingênuo).

## 2. Entendimento dos Dados (Data Understanding)
O projeto utiliza o conjunto de dados público da OLIST, compreendendo o período de 2016 a 2018, com aproximadamente 110 mil registros.

### Variáveis Utilizadas
| Variável | Tipo | Descrição |
| :--- | :--- | :--- |
| `price` | Numérica | Preço total da compra do pedido em Reais |
| `freight_value` | Numérica | Valor total do frete do pedido em Reais |
| `product_weight_g` | Numérica | Peso total do pedido em gramas |
| `product_total_volume_cm3` | Numérica | Volume total do pedido em cm³ |
| `distancia_km` | Numérica | Distância entre vendedor e cliente (Haversine) |
| `purchase_year` | Numérica | Ano da confirmação da compra |
| `purchase_month` | Categórica | Mês da confirmação da compra |
| `purchase_weekday` | Categórica | Dia da semana da confirmação da compra |
| `same_state` | Booleana | Vendedor e cliente no mesmo estado? |
| `freight_ratio` | Numérica | Razão entre valor do frete e valor do pedido |
| `route_avg_time` | Numérica | Tempo médio de entrega entre o estado do vendedor e cliente |
| `tempo_entrega_dias` | Alvo | Tempo em dias até a entrega efetiva |

## 3. Preparação dos Dados (Data Preparation)
Para garantir a qualidade do modelo, foram aplicadas as seguintes estratégias:
*   **Consistência Geoespacial**: Filtragem de coordenadas (latitude [-34.0, 6.0] e longitude [-74.0, -35.0]) para manter apenas registros dentro dos limites do Brasil, eliminando ruídos.
*   **Engenharia de Atributos**: Cálculo da distância espacial entre origem e destino utilizando a fórmula de Haversine.
*   **Tratamento de Outliers**: Remoção de valores extremos (truncamento pelos percentis 5% e 95%) para variáveis numéricas contínuas, mitigando o impacto de anomalias.

## 4. Modelagem (Modeling)
Foram avaliados dois algoritmos de aprendizado de máquina: **Random Forest** e **XGBoost**. A modelagem seguiu uma abordagem incremental através de quatro cenários para quantificar o ganho de informação:

1.  **Cenário 1 (Baseline)**: Apenas dados brutos iniciais (preço, frete, peso, volume, distância).
2.  **Cenário 2 (+ Datas)**: Incorporação de variáveis temporais derivadas (ano, mês, dia da semana).
3.  **Cenário 3 (+ Logística)**: Adição de informações operacionais (`same_state`, `freight_ratio`).
4.  **Cenário 4 (+ Rotas)**: Inclusão de dados do tempo médio de entrega por rota (`route_avg_time`).

## 5. Avaliação (Evaluation)
Os modelos foram avaliados utilizando as métricas **MAE** (Erro Absoluto Médio) e **RMSE** (Raiz do Erro Quadrático Médio).

### Resultados Comparativos

| Modelo | Cenário | MAE (dias) | RMSE (dias) |
| :--- | :--- | :--- | :--- |
| Random Forest | Baseline | 4.865512 | 7.344185 |
| XGBoost | Baseline | 4.923160 | 7.435807 |
| Random Forest | Datas | 4.507405 | 6.911201 |
| XGBoost | Datas | 4.640409 | 7.158513 |
| Random Forest | Logística | 4.477978 | 6.898782 |
| XGBoost | Logística | 4.557357 | 7.004841 |
| **Random Forest** | **Rota** | **4.337500** | **6.752988** |
| XGBoost | Rota | 4.481675 | 6.907052 |

**Conclusão**: A introdução das variáveis de Rota (Cenário 4) foi determinante para a maximização da performance. O modelo **Random Forest no Cenário 4** apresentou o melhor desempenho global, com um **RMSE de 6.75**, validando a hipótese de que componentes geoespaciais e históricos de rota são preditores cruciais para o tempo de entrega.

## 6. Implantação (Deployment)
Para operacionalizar o modelo:
1.  **API de Previsão**: Encapsular o modelo treinado em uma API para consulta em tempo real durante o checkout.
2.  **Monitoramento (MLOps)**: Acompanhar métricas de *drift* das features (ex: mudanças em rotas logísticas) e programar retreinos periódicos para manter a precisão das estimativas.

---
*Projeto baseado no estudo técnico desenvolvido utilizando o dataset público da Olist.*
