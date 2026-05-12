# 📱 Food App — Análise de Funil e Teste A/B

**Análise de Funil de Conversão e Teste A/B em App de Delivery**

***

## 📋 Sobre o Projeto

Este projeto foi desenvolvido como parte do bootcamp TripleTen de Data Analytics. O objetivo é analisar os logs de comportamento de usuários em um app de delivery de comida, construindo um **funil de conversão** para identificar gargalos na jornada de compra e avaliando por meio de um **teste A/B** se a alteração das fontes tipográficas do aplicativo impactou a conversão dos usuários.

A análise parte de 244.126 eventos registrados entre julho e agosto de 2019, distribuídos em três grupos experimentais — dois de controle (246 e 247) e um de teste (248) — e culmina em uma decisão formal sobre implementar ou descartar a mudança tipográfica.

***

## 🎯 Objetivos

- Limpar os dados removendo duplicatas e eventos de dias com cobertura incompleta
- Mapear a sequência de eventos do app e construir o funil de conversão da jornada de compra
- Identificar o maior gargalo do funil — onde mais usuários abandonam a jornada
- Validar o experimento por meio de um **teste A/A** entre os grupos de controle
- Aplicar o **teste A/B** comparando o grupo com fonte alterada (248) contra os controles
- Aplicar a **correção de Bonferroni** para controlar o risco de falsos positivos em múltiplas comparações
- Emitir uma recomendação de negócio baseada nos resultados estatísticos

***

## 🗂️ Estrutura do Projeto

### **Etapa 1: Leitura e Exploração Inicial**
- Carregamento do dataset `logs_exp_us.csv`
- Verificação de tipos, nulos, volume e distribuição por grupo experimental

### **Etapa 2: Preparação dos Dados**
- Renomeação das colunas para padrão snake_case
- Identificação e remoção de duplicatas (413 linhas, 0,17% do total)
- Conversão do timestamp Unix para datetime e extração da coluna de data
- Análise de completude por dia: identificação de 7 dias com dados incompletos (25/07–31/07)
- Corte temporal mantendo apenas eventos a partir de 01/08 — perda de 0,33% dos eventos e 0,12% dos usuários

### **Etapa 3: Funil de Eventos**
- Contagem de ocorrências e de usuários únicos por tipo de evento
- Inferência da ordem do funil de compra pela queda de volume entre etapas
- Exclusão do evento `Tutorial` (não faz parte da jornada de compra)
- Cálculo das taxas de conversão step-over-step e acumulada (topo → fundo)
- Identificação do maior gargalo da jornada

### **Etapa 4: Validação com Teste A/A**
- Comparação dos grupos de controle 246 vs 247 em todos os eventos
- Uso do **z-test para duas proporções** em cada etapa do funil e em todos os eventos
- Confirmação de que os grupos são estatisticamente equivalentes antes de prosseguir

### **Etapa 5: Teste A/B — Efeito da Alteração de Fontes**
- Comparação do grupo 248 (fonte alterada) vs 246 individualmente
- Comparação do grupo 248 vs 247 individualmente
- Comparação do grupo 248 vs controle combinado (246 + 247)

### **Etapa 6: Correção de Bonferroni**
- Inventário dos 24 testes de hipótese realizados no projeto
- Cálculo do risco de falso positivo sem correção (>70%)
- Recálculo com α corrigido (0,05 / 24 ≈ 0,0021) e verificação dos resultados

***

## 📊 Dataset

### **Arquivo**

| Arquivo | Registros | Usuários únicos | Grupos |
|---------|:---------:|:---------------:|--------|
| `logs_exp_us.csv` | 244.126 | 7.551 | 246 (controle), 247 (controle), 248 (teste) |

### **Descrição das Colunas**

| Coluna original | Coluna renomeada | Descrição |
|----------------|-----------------|-----------|
| `EventName` | `event_name` | Nome do evento registrado no app |
| `DeviceIDHash` | `device_id` | Identificador único do usuário (hash do dispositivo) |
| `EventTimestamp` | `timestamp` | Timestamp Unix do evento (int64) |
| `ExpId` | `exp_id` | Grupo experimental: 246 e 247 = controle, 248 = teste |

### **Eventos disponíveis**

| Evento | Descrição | No funil? |
|--------|-----------|:---------:|
| `MainScreenAppear` | Tela principal do app | ✅ Etapa 1 |
| `OffersScreenAppear` | Tela de ofertas | ✅ Etapa 2 |
| `CartScreenAppear` | Tela do carrinho | ✅ Etapa 3 |
| `PaymentScreenSuccessful` | Pagamento concluído | ✅ Etapa 4 |
| `Tutorial` | Tutorial para novos usuários | ❌ Excluído |

***

## 📈 Principais Resultados

### **Funil de Conversão**

| Etapa | Usuários únicos | Conv. da etapa anterior | Conv. acumulada |
|-------|:--------------:|:-----------------------:|:---------------:|
| MainScreenAppear | 7.429 | — | 100,00% |
| OffersScreenAppear | 4.606 | 62,00% | 62,00% |
| CartScreenAppear | 3.742 | 81,24% | 50,37% |
| PaymentScreenSuccessful | 3.542 | 94,66% | **47,68%** |

- **Maior gargalo:** MainScreen → OffersScreen — **38% dos usuários não avança** (2.823 usuários perdidos)
- **Taxa de conclusão:** 47,68% dos usuários que abrem o app chegam ao pagamento

### **Teste A/A — Validação do experimento**

Nenhuma diferença estatisticamente significativa foi encontrada entre os grupos 246 e 247 em qualquer evento (p ≥ 0,05 em todos os casos). Os grupos de controle são equivalentes e o experimento está corretamente calibrado.

### **Teste A/B — Efeito da alteração de fontes (grupo 248)**

| Comparação | Eventos testados | Sig. (α=0,05) | Sig. (Bonferroni α≈0,002) |
|-----------|:----------------:|:-------------:|:-------------------------:|
| 248 vs 246 | 5 | 0 | 0 |
| 248 vs 247 | 5 | 0 | 0 |
| 248 vs controle combinado | 5 | 0 | 0 |

**Conclusão: a alteração de fontes não produziu efeito mensurável na conversão dos usuários.** Nenhuma diferença significativa foi encontrada em nenhuma das 15 comparações — nem com o critério padrão (α = 0,05) nem com a correção de Bonferroni (α ≈ 0,0021).

A mudança tipográfica é **neutra em conversão**: não prejudica nem melhora a jornada de compra. A decisão de implementá-la deve ser baseada em critérios de design (acessibilidade, identidade visual), não em métricas de funil.

***

## 🛠️ Tecnologias Utilizadas

- **Pandas** — Manipulação, limpeza e análise de dados
- **Plotly Express** — Visualizações interativas (funil, histogramas, barras)
- **SciPy** — Biblioteca base de estatística
- **Statsmodels** — Teste z para duas proporções (`proportions_ztest`)
- **Jupyter Notebook** — Desenvolvimento e documentação interativa

***

## 🚀 Como Executar

### **Pré-requisitos**

```
python >= 3.8
jupyter notebook
pandas
plotly
scipy
statsmodels
```

### **Instalação**

```bash
# Clone o repositório
git clone https://github.com/raimirsilva/food-app-ab-test.git

# Navegue até o diretório
cd food-app-ab-test

# Instale as dependências
pip install -r requirements.txt

# Inicie o Jupyter Notebook
jupyter notebook
```

### **Execução**

Certifique-se de que o arquivo `logs_exp_us.csv` esteja em `../data/` em relação ao notebook, ou ajuste o caminho na célula de leitura. Em seguida, abra `food_app.ipynb` e execute as células sequencialmente.

***

## 🎓 Aprendizados

Este projeto demonstra competências em:

- **Limpeza de dados com critério metodológico** — remoção de duplicatas com análise de impacto por grupo antes de deletar; corte temporal baseado em mediana diária em vez de datas fixas hardcoded
- **Construção de funil de conversão** — inferência da ordem das etapas por volume, cálculo de taxas step-over-step e acumuladas
- **Teste A/A como etapa obrigatória de validação** — garantindo que os grupos de controle sejam equivalentes antes de interpretar o A/B
- **Testes de proporção (z-test)** — aplicados a múltiplos eventos e grupos de forma sistematizada com função reutilizável
- **Correção de Bonferroni** — controle do erro Tipo I em contexto de múltiplas comparações (24 testes simultâneos)
- **Decisão orientada por dados** — separar a recomendação de negócio da ausência de evidência estatística, evitando tanto implementar quanto descartar por razões erradas

***

## 👤 Autor

**Raimir Silva**

- GitHub: [@raimirsilva](https://github.com/raimirsilva)
- LinkedIn: [Raimir Silva](https://linkedin.com/in/raimir-silva)
- Email: raimirsilva@icloud.com

***

## 📄 Licença

Este projeto foi desenvolvido como parte do bootcamp **TripleTen Data Analytics** para fins educacionais e de portfólio.

***

**⭐ Se este projeto foi útil para você, considere dar uma estrela no repositório!**
