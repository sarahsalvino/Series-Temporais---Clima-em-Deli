# Previsão de Temperatura em Delhi com Prophet

Análise exploratória e modelo de séries temporais para prever a temperatura média diária em Nova Delhi, Índia, utilizando o Facebook Prophet. O projeto cobre 4 anos de dados climáticos (2013–2017) e avalia o desempenho do modelo contra dados reais de teste.

---

## Objetivo

Entender o comportamento climático de Delhi ao longo dos anos e construir um modelo capaz de prever a temperatura média dos dias seguintes com base no histórico, identificando padrões de sazonalidade e tendência.

---

## Estrutura do Projeto

```
 projeto
 ┣ Daily_Delhi_Climate.ipynb       # Notebook principal
 ┣ DailyDelhiClimateTrain.csv      # Dados de treino (2013–2017)
 ┣ DailyDelhiClimateTest.csv       # Dados de teste (Jan–Abr 2017)
 ┗ README.md
```

---

## Dataset

- **Fonte:** [Kaggle — Daily Climate Time Series Data](https://www.kaggle.com/datasets/sumanthvrao/daily-climate-time-series-data)
- **Treino:** 1.462 registros diários de 01/01/2013 a 01/01/2017
- **Teste:** 114 registros diários de 01/01/2017 a 24/04/2017
- **Sem valores nulos ou duplicados**

### Colunas

| Coluna | Descrição |
|--------|-----------|
| `date` | Data do registro |
| `meantemp` | Temperatura média diária (°C) variável alvo |
| `humidity` | Umidade relativa do ar (%) |
| `wind_speed` | Velocidade do vento (km/h) |
| `meanpressure` | Pressão atmosférica média (hPa) |

---

## Análise Exploratória (EDA)

### Visão geral dos dados
O dataset de treino possui **1.462 registros**, um para cada dia entre 2013 e 2017, sem nenhum valor nulo ou duplicado. A temperatura média ao longo do período é de **25.5°C**, com mínima de **6°C** (inverno) e máxima de **38.71°C** (verão).

### Problema encontrado: Outliers na pressão atmosférica
Durante a análise das estatísticas descritivas, foram identificados valores claramente errados na coluna `meanpressure`. A pressão atmosférica ao nível do mar varia normalmente entre 980 e 1040 hPa — qualquer valor fora disso indica erro de medição.

Foram encontrados **7 registros com valores absurdos:**

| Data | Valor encontrado | Problema |
|------|-----------------|----------|
| 2016-03-28 | 7679.33 hPa | Impossível fisicamente |
| 2016-09-24 | 1352.62 hPa | Muito acima do normal |
| 2016-11-17 | 1350.30 hPa | Muito acima do normal |
| 2016-08-02 | 310.44 hPa | Muito abaixo do normal |
| 2016-08-14 | 633.90 hPa | Muito abaixo do normal |
| 2016-08-16 | -3.04 hPa | Pressão negativa impossível |
| 2016-11-28 | 12.05 hPa | Muito abaixo do normal |

**Solução aplicada:** substituição dos valores inválidos pela **mediana** dos registros válidos (1008.57 hPa), que é uma medida robusta e representativa. Após a correção, os valores ficaram entre 938 e 1023 hPa — dentro do esperado para o clima de Delhi.

### Principais insights da EDA

**Sazonalidade anual clara:**
A temperatura de Delhi segue um padrão sazonal muito definido ao longo do ano. Os meses mais frios são **janeiro e dezembro**, com médias abaixo de 15°C. A temperatura sobe progressivamente na primavera, atinge seu pico em **maio e junho** (média acima de 32°C), recua um pouco durante a monção em julho e agosto, e então cai novamente no outono e inverno.

**Tendência estável ao longo dos anos:**
Analisando a temperatura média por ano, os valores se mantiveram relativamente estáveis entre 2013 e 2017, sem uma tendência de aquecimento ou resfriamento expressiva no período analisado. Isso indica que o modelo não precisa capturar grandes mudanças de tendência o foco é a sazonalidade.

**Correlações entre variáveis:**
- **Temperatura e umidade:** correlação negativa moderada os meses mais quentes (verão seco) têm umidade mais baixa, enquanto a monção traz umidade alta com temperatura um pouco menor
- **Temperatura e pressão:** correlação levemente negativa pressão tende a cair em períodos mais quentes
- **Temperatura e vento:** correlação fraca vento não é um bom preditor de temperatura em Delhi

---

## Modelo de Séries Temporais — Prophet

### Por que Prophet?
O Prophet, desenvolvido pelo Meta (Facebook), é ideal para séries temporais com sazonalidade clara e dados diários. Ele detecta automaticamente padrões anuais, semanais e diários, lida bem com feriados e lacunas nos dados, e gera intervalos de confiança para as previsões.

### Preparação dos dados
O Prophet exige exatamente duas colunas:
- **`ds`** — a data (renomeada de `date`)
- **`y`** — o valor a prever (renomeada de `meantemp`)

### Configuração do modelo
```python
modelo = Prophet(
    yearly_seasonality=True,   # Captura padrão anual (verão/inverno)
    daily_seasonality=False,   # Dados diários — não faz sentido
    weekly_seasonality=False   # Clima não tem padrão semanal relevante
)
```

A sazonalidade anual foi ativada pois é o padrão dominante no clima de Delhi. As sazonalidades diária e semanal foram desativadas pois não fazem sentido para dados climáticos o clima não muda com base no dia da semana.

### Previsão
O modelo foi treinado com os dados de 2013 a 2017 e gerou previsões para os **120 dias seguintes**. As previsões para o final de abril e início de maio de 2017 indicaram temperaturas entre **33°C e 34.5°C**, com intervalo de confiança de aproximadamente ±2.5°C valores coerentes com o histórico de verão em Delhi.

---

## Avaliação do Modelo

### Gráfico: Temperatura Real vs. Prevista (Jan–Abr 2017)
O modelo foi avaliado comparando as previsões com os dados reais do arquivo de teste, cobrindo o período de janeiro a abril de 2017. Visualmente, a curva prevista acompanha de perto a curva real, com o intervalo de confiança cobrindo a grande maioria dos valores reais.

### Métricas

| Métrica | Valor | Interpretação |
|---------|-------|--------------|
| **MAE** (Erro Médio Absoluto) | **2.20°C** | O modelo erra em média 2.2°C por dia |
| **RMSE** (Raiz do Erro Quadrático) | **2.68°C** | Erros grandes são raros e controlados |

O RMSE sendo apenas levemente maior que o MAE indica que o modelo **não cometeu erros absurdos** em nenhum dia específico os erros estão bem distribuídos ao longo do período.

Considerando que a temperatura em Delhi varia em uma faixa de **~33°C** (de 6°C a 38°C), um erro médio de 2.2°C representa uma acurácia muito boa para um modelo que usa **apenas o histórico de temperatura**, sem variáveis externas como umidade ou vento.

---

## Conclusão

O projeto demonstrou que o Prophet é uma ferramenta poderosa e acessível para previsão de séries temporais climáticas. O modelo capturou com precisão a sazonalidade anual de Delhi o padrão de inverno frio, verão quente e monção e entregou previsões confiáveis com erro médio de apenas 2.2°C.

Um ponto importante do projeto foi a **necessidade de tratar os dados antes da modelagem**: 7 registros com valores impossíveis de pressão atmosférica foram identificados e corrigidos com a mediana dos valores válidos. Esse tipo de limpeza é fundamental para garantir que o modelo não aprenda padrões baseados em erros de medição.

<img width="816" height="479" alt="image" src="https://github.com/user-attachments/assets/5aa68c0c-212f-42e2-9b7d-efc21732cad1" />

<img width="1146" height="396" alt="image" src="https://github.com/user-attachments/assets/4d12d5f4-9009-498a-b0f9-9968d49ce4a4" />


Como próximos passos, seria interessante incluir **regressores externos** no Prophet (como umidade e velocidade do vento) para tentar reduzir ainda mais o erro, e avaliar o modelo em períodos mais longos de teste.

---

## Tecnologias Utilizadas

- Python 3.11
- Pandas
- NumPy
- Matplotlib
- Seaborn
- Prophet (Meta)
- Scikit-learn

---

## Como Executar

1. Clone o repositório
```bash
git clone https://github.com/seu-usuario/seu-repositorio.git
```

2. Instale as dependências
```bash
pip install pandas numpy matplotlib seaborn prophet scikit-learn
```

3. Abra o notebook
```bash
jupyter notebook Daily_Delhi_Climate.ipynb
```

> Certifique-se de que os arquivos `DailyDelhiClimateTrain.csv` e `DailyDelhiClimateTest.csv` estão na mesma pasta do notebook antes de rodar.
