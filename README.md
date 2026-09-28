# Checkpoint 2 — APIs, energias renováveis e aprendizado de máquina

**Aluno:** Felipe Macedo — RM 570990
**Disciplina:** Soluções em Energias Renováveis e Sustentáveis — 1CC, 2º semestre

## Objetivo

Consultar duas APIs públicas de dados de energia e clima, preparar os conjuntos de dados e resolver duas tarefas independentes de aprendizado de máquina em Python. Em cada tarefa, três algoritmos diferentes são treinados e comparados:

1. **Classificação:** prever a fonte de um empreendimento de geração (Solar, Eólica ou Hidráulica) a partir da potência outorgada e da localização.
2. **Regressão:** estimar a radiação solar horária em Petrolina (PE) a partir das condições meteorológicas e da hora do dia.

## Dados

| Arquivo | Fonte | Período / recorte | Conteúdo |
|---|---|---|---|
| `aneel_classificacao_orange.csv` | [ANEEL — SIGA](https://dadosabertos.aneel.gov.br/dataset/siga-sistema-de-informacoes-de-geracao-da-aneel) (API CKAN, recurso `11ec447d-698d-4ab8-977f-b424d5deee6a`) | Cadastro consultado em 28/09/2026, até 1.200 registros por sigla (`UFV`, `EOL`, `UHE`, `PCH`, `CGH`) | `potencia_kw`, `latitude`, `longitude`, `fonte` — um empreendimento por linha |
| `meteo_regressao_orange.csv` | [Open-Meteo — API histórica](https://open-meteo.com/en/docs/historical-weather-api) | Petrolina (PE), −9,39 / −40,50, de 01/04/2025 a 30/06/2025, 7h–17h, fuso `America/Recife` | `data_hora`, `temperatura_c`, `umidade_pct`, `nuvens_pct`, `vento_kmh`, `hora`, `radiacao_w_m2` — uma hora por linha |

As duas APIs são públicas e **não exigem token**. O repositório não contém nenhuma credencial.

Observações importantes:
- Os dados da ANEEL descrevem **empreendimentos cadastrados**, não energia gerada. As proporções entre classes refletem o limite da consulta e **não representam a matriz elétrica brasileira**.
- Os dados do Open-Meteo são **estimativas de reanálise**, não medições de um sensor ou de um painel.

## Como executar

1. Abra `CP2_APIs_Energia_Renovavel_ML.ipynb` no [Google Colab](https://colab.research.google.com) (*Arquivo → Fazer upload de notebook*) ou no Jupyter.
2. Execute **todas as células em ordem** (*Ambiente de execução → Executar tudo*).
   - As primeiras células consultam as APIs e geram os dois CSVs.
   - As células seguintes fazem a análise, o treinamento e a comparação dos modelos.
3. Dependências: Python 3.10+, `pandas`, `numpy`, `matplotlib`, `seaborn` e `scikit-learn`, todas já instaladas no Colab. Localmente: `pip install pandas numpy matplotlib seaborn scikit-learn jupyter`.

Os CSVs usados nos resultados deste repositório já estão incluídos. Como o cadastro da ANEEL é atualizado com frequência, uma nova consulta pode devolver registros um pouco diferentes e alterar ligeiramente os números.

## Tarefa 1 — Classificação da fonte (ANEEL)

**Configuração:** 3.829 empreendimentos, após remover 47 com coordenada (0, 0) não preenchida. Divisão **estratificada** 80/20 com `random_state=42`. Padronização (e `log1p` da potência) dentro de `Pipeline`, ajustada só no treino. Precision, Recall e F1 com média **`macro`**.

| Algoritmo | Accuracy | Precision (macro) | Recall (macro) | F1 (macro) |
|---|---:|---:|---:|---:|
| Regressão Logística | 0,832 | 0,847 | 0,828 | 0,825 |
| KNN (k=15) | 0,950 | 0,951 | 0,950 | 0,950 |
| **Random Forest** | **0,977** | **0,977** | **0,976** | **0,977** |

**Conclusões:**
- A **Random Forest** foi a melhor. A fronteira entre as fontes não é linear (a eólica aparece no Nordeste e no extremo Sul), e por isso a Regressão Logística fica bem atrás.
- A confusão mais frequente é **Solar ↔ Eólica**: as solares de grande porte e os parques eólicos têm potência parecida (~30 MW) e ficam na mesma região, o interior do Nordeste.
- **Cautela:** 752 dos 1.200 registros solares são usinas de exatamente 1 kW agrupadas no Pará, um artefato da consulta que facilita o acerto. Sem esse grupo, o F1 macro da Random Forest cai para 0,960, e a Regressão Logística reconhece apenas 6,8% das demais solares.
- Potência e localização são atributos indiretos. Eles não medem sol, vento ou água e não bastam para uma aplicação real.

## Tarefa 2 — Regressão da radiação solar (Open-Meteo)

**Configuração:** 1.001 horas. Divisão **temporal**: as primeiras 800 horas (01/04 a 12/06) para treino e as últimas 201 (12/06 a 30/06) para teste, sem embaralhar. Entradas: `temperatura_c`, `umidade_pct`, `nuvens_pct`, `vento_kmh` e `hora`.

| Algoritmo | MAE (W/m²) | MSE ((W/m²)²) | R² |
|---|---:|---:|---:|
| Regressão Linear | 145,2 | 30.034 | 0,360 |
| KNN (k=10) | 72,2 | 8.223 | 0,825 |
| **Random Forest** | **67,2** | **7.420** | **0,842** |

**Conclusões:**
- A **Random Forest** foi a melhor nas três métricas, com o KNN próximo. A Regressão Linear não representa o formato de sino da radiação ao longo do dia e chega a prever valores negativos.
- A **hora do dia** é a variável mais importante: define a altura do sol e o teto de radiação possível. Sem ela, o R² da Random Forest cai de 0,84 para 0,34.
- **Radiação não é geração elétrica:** W/m² é a potência da luz numa superfície horizontal. A energia de um sistema fotovoltaico depende também da área e da eficiência dos painéis, da inclinação, da temperatura das células e das perdas do sistema.

## Estrutura do repositório

```text
.
├── README.md
├── CP2_APIs_Energia_Renovavel_ML.ipynb   # notebook completo (APIs + 6 modelos)
├── aneel_classificacao_orange.csv        # gerado pela API da ANEEL
└── meteo_regressao_orange.csv            # gerado pela API do Open-Meteo
```
