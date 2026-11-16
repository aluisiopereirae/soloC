

# (soloC) Plataforma de Auditoria Agrícola e Carbono do Solo — Oeste da Bahia

> Guia interativo das **63 espécies de meliponíneos** (Apidae: Meliponini) da lista de ocorrência no Maranhão. Cada espécie tem uma ficha com nomes conferidos no Catálogo Moure, dados biológicos, fotos, municípios com registro e orientações de manejo.


**Acesso:** [aluisiopereirae.github.io/soloC](https://aluisiopereirae.github.io/soloC/)


Plataforma aberta que integra **dados de solo**, **séries temporais de NDVI** (abordagem do [SATVeg/Embrapa](https://www.satveg.cnptia.embrapa.br)), **aprendizado de máquina** e **modelagem matemática** para:

- classificar e catalogar os cultivos de cada talhão ao longo dos anos;
- prever o cultivo da próxima safra;
- **auditar** o manejo informado pelo produtor contra o previsto pelo satélite;
- quantificar o **estoque** e modelar o **acúmulo de carbono no solo** ao longo dos anos.

Todo o processamento roda em um notebook do **Google Colab**, que gera uma plataforma **HTML5 em arquivo único**, as figuras, as tabelas e os números do artigo científico.

> [!WARNING]
> **Versão de demonstração.** Na versão publicada, as séries de NDVI foram **simuladas** por um modelo fenológico, porque o ambiente de geração não tinha acesso ao SATVeg nem ao MODIS. Os dados de solo, carbono e coordenadas são reais. Para usar NDVI real, rode o notebook no Colab (veja [Fonte do NDVI](#fonte-do-ndvi)). A classificação de cultivos, o catálogo, a auditoria e as taxas de acúmulo de carbono só valem depois disso.

---

## Sumário

- [Funcionalidades da plataforma](#funcionalidades-da-plataforma)
- [Estrutura do repositório](#estrutura-do-repositório)
- [Como usar](#como-usar)
- [Fonte do NDVI](#fonte-do-ndvi)
- [Dados de entrada](#dados-de-entrada)
- [Métodos](#métodos)
- [Arquivos gerados](#arquivos-gerados)
- [Principais resultados](#principais-resultados)
- [Limitações](#limitações)
- [Artigo](#artigo)
- [Como citar](#como-citar)
- [Licença](#licença)

---

## Funcionalidades da plataforma

| Aba / área | O que mostra |
|---|---|
| **Mapa** | Camadas-base: satélite, híbrido (satélite + rótulos e vias), relevo topográfico, relevo sombreado e ruas. Pontos amostrais e polígonos dos talhões, coloridos por cultivo da safra, previsão 2026/27, status de auditoria, estoque de C ou taxa de acúmulo. Filtros por fazenda e busca por talhão. |
| **Talhão** | Ficha do talhão com o perfil de NDVI (média do polígono ± desvio-padrão) e os filtros do SATVeg (bruto, pré-filtragem, FlatBottom, Wavelet Coiflet-4, Savitzky–Golay). Inclui os ciclos de cultivo classificados, o NDVI máximo, mínimo e médio por ano e a trajetória do carbono com projeções. |
| **Catálogo de cultivos** | Cultivos identificados por talhão e ano agrícola (2018/19 a 2025/26), previsão da safra 2026/27 e área estimada por cultura. |
| **Auditoria** | Informado pelo produtor × previsto por satélite (safras por ano, plantas de cobertura, nº de cultivos, irrigação), com índice de conformidade e status por talhão. |
| **Carbono** | Estoque de C 0–40 cm por talhão (medido ou estimado), taxa de acúmulo por manejo, cenários 2027–2036 e quantitativo por fazenda. |
| **Estatística e ML** | Estatística descritiva, Kruskal–Wallis, modelo misto, PCA, PERMANOVA/PERMDISP, desempenho dos classificadores e regressores, matriz de Markov. |
| **Pontos × Médias** | Comparação do estoque calculado ponto a ponto com a planilha de médias (CCC de Lin, Bland–Altman, CV intratalhão, nº mínimo de pontos). |
| **Coordenadas GMS** | Conversor graus-minutos-segundos ⇄ graus decimais e tabela de todos os pontos, com as coordenadas suspeitas sinalizadas. |
| **Importar SATVeg** | Carrega uma série exportada do SATVeg (CSV/TXT), aplica os filtros, detecta e classifica os ciclos no próprio navegador. |
| **Downloads** | CSV (talhões, pontos, catálogo, auditoria, ciclos, séries NDVI, carbono), GeoJSON, KML e JSON completo. |

---

## Estrutura do repositório

```text
.
├── README.md
├── notebook_auditoria_carbono_colab.ipynb   # pipeline completo (Google Colab)
├── plataforma_auditoria_carbono.html        # plataforma HTML5 pronta (dados embutidos)
├── dados/
│   ├── dados_completos.xlsx                 # Ds por ponto e camada
│   └── ANALISE_DADOS_12_11_26.xlsx          # aba "dados analise" (manejo, C por talhão, química)
├── artigo/
│   ├── artigo_elsevier.pdf
│   ├── artigo_elsevier.docx
│   └── artigo_latex_overleaf.zip            # LaTeX (elsarticle) para o Overleaf
└── codigo_fonte/
    ├── p1_setup_dados.py                    # leitura, GMS, harmonização
    ├── p2_estatistica.py                    # pontos × médias, descritiva, KW, PCA, PERMANOVA
    ├── p3_ndvi_cultivos.py                  # NDVI, filtros, fenologia, ML, catálogo, auditoria
    ├── p4_carbono.py                        # ML do carbono e modelo dinâmico
    ├── p5_mapas_exportacao.py               # mapas, plataforma, planilhas, macros do artigo
    ├── plataforma_template.html             # modelo HTML5 (Leaflet + Plotly)
    ├── construir_notebook.py                # monta o .ipynb a partir dos módulos
    └── tex_para_docx.py                     # converte o artigo LaTeX para Word
```

> [!NOTE]
> Antes de publicar os dados das fazendas em um repositório público, confirme que você tem autorização dos produtores. Se for o caso, deixe a pasta `dados/` fora do repositório (adicione-a ao `.gitignore`) e envie as planilhas pelo upload do Colab.

---

## Como usar

### 1. Só visualizar

Baixe `plataforma_auditoria_carbono.html` e abra no navegador (Chrome, Edge ou Firefox). Os dados estão embutidos no arquivo; as camadas-base do mapa e as bibliotecas (Leaflet, Plotly) precisam de internet.

> [!TIP]
> No GitHub, ative o **GitHub Pages** (Settings → Pages → branch `main`) e acesse `https://SEU-USUARIO.github.io/SEU-REPOSITORIO/plataforma_auditoria_carbono.html` para compartilhar a plataforma por link.

### 2. Rodar as análises no Google Colab

1. Clique no selo **Abrir no Colab** no topo desta página.
2. Na célula de **CONFIGURAÇÃO**, ajuste os parâmetros:

   ```python
   FONTE_NDVI = "auto"     # "auto" | "satveg" | "planetary" | "gee" | "csv" | "simulado"
   SATVEG_TOKEN = ""       # token da AgroAPI Embrapa (opcional)
   DATA_INICIO, DATA_FIM = "2018-08-01", "2026-07-31"
   ```

3. Menu **Ambiente de execução → Executar tudo**.
4. Quando pedido, envie as duas planilhas (`dados_completos.xlsx` e `ANALISE_DADOS_12_11_26.xlsx`).
5. No final, o Colab baixa o pacote `resultados_auditoria_carbono.zip` e a plataforma HTML atualizada.

### 3. Rodar localmente

```bash
pip install pandas numpy scipy scikit-learn xgboost statsmodels PyWavelets shapely folium seaborn openpyxl nbclient
jupyter notebook notebook_auditoria_carbono_colab.ipynb
```

Para regenerar o notebook a partir dos módulos de `codigo_fonte/`:

```bash
python codigo_fonte/construir_notebook.py notebook_auditoria_carbono_colab.ipynb
```

---

## Fonte do NDVI

No modo `auto`, o notebook tenta as fontes na ordem abaixo e usa a primeira que funcionar:

| Fonte | Requisito | Observação |
|---|---|---|
| **SATVeg / AgroAPI Embrapa** | `SATVEG_TOKEN` ([AgroAPI](https://www.agroapi.cnptia.embrapa.br)) | Mesmo dado do SATVeg: MODIS Terra + Aqua, 250 m. |
| **Microsoft Planetary Computer** | nenhum cadastro | MOD13Q1/MYD13Q1 v6.1, com máscara de qualidade do pixel. |
| **Google Earth Engine** | conta e projeto GEE (`GEE_PROJECT`) | `MODIS/061/MOD13Q1`, média por polígono. |
| **CSV** | arquivo exportado do SATVeg (`CSV_NDVI`) | Colunas `data`, `talhao`, `ndvi`. |
| **Simulado** | — | Apenas para demonstração; marcado como `SIMULADO` em todas as saídas. |

> [!IMPORTANT]
> As fontes SATVeg, Planetary Computer e Earth Engine foram implementadas, mas não puderam ser testadas no ambiente em que esta versão foi gerada. Se alguma falhar, o notebook informa o erro e passa para a próxima fonte.

---

## Dados de entrada

| Planilha | Conteúdo |
|---|---|
| `dados_completos.xlsx` (aba *Dados solo bahia*) | 263 pontos × 3 camadas (0–10, 10–20, 20–40 cm) com densidade do solo e coordenadas; colunas anuais de sistema de produção (preenchidas pelo notebook a partir do catálogo). |
| `ANALISE_DADOS_12_11_26.xlsx` (aba *dados analise*) | 663 pontos em 68 talhões e 22 fazendas: manejo informado (irrigação, argila, período sem revolvimento, diversidade de cultivos, safras/ano, plantas de cobertura), Ds por ponto, C e H<sub>LIFS</sub> por talhão, química do solo. |

**Tratamentos aplicados automaticamente**

- Coordenadas em GMS (`13°48'24''`) convertidas para graus decimais; sinal negativo para Sul/Oeste; valores sem separador decimal reescalonados; pontos a mais de 3 km da mediana do talhão sinalizados.
- Valores 0 de carbono e H<sub>LIFS</sub> tratados como ausentes (análises não fornecidas pelas fazendas).
- Pareamento entre as planilhas pela assinatura de Ds nas três camadas, porque a aba *dados analise* foi reordenada e o número do ponto não corresponde ao número da amostra.

---

## Métodos

<details>
<summary><b>Estoque de carbono e comparação pontos × médias</b></summary>

$$EST_C = C\,(\%) \times D_s\,(\mathrm{g\,cm^{-3}}) \times e\,(\mathrm{cm}) \quad [\mathrm{Mg\,ha^{-1}}]$$

O estoque ponto a ponto (D<sub>s</sub> do ponto × C do talhão) é comparado ao da planilha de médias por CCC de Lin, Bland–Altman e Wilcoxon pareado. Nº mínimo de pontos: $n = (t \cdot CV / E)^2$.
</details>

<details>
<summary><b>Estatística</b></summary>

Unidade experimental: **talhão** (o carbono é medido por talhão). Kruskal–Wallis por fator de manejo, modelo linear misto (efeito aleatório de fazenda), correlação de Spearman, PCA de 14 atributos padronizados, PERMANOVA sequencial (4 999 permutações) e PERMDISP.
</details>

<details>
<summary><b>NDVI e filtros (como no SATVeg)</b></summary>

Perfil médio dos pixels do polígono ± desvio-padrão. Pré-filtragem (remoção de nuvens e valores inválidos com interpolação linear), **FlatBottom**, **Wavelet Coiflet-4** (2 níveis, limiar suave universal) e **Savitzky–Golay** (grau 2, janela de 5 composições).
</details>

<details>
<summary><b>Fenologia e classificação de cultivos</b></summary>

Ciclos modelados por dupla logística:

$$NDVI(t) = b + \max_i a_i\left[\frac{1}{1+e^{-r_{1,i}(t-s_i)}} - \frac{1}{1+e^{-r_{2,i}(t-e_i)}}\right]$$

Métricas por ciclo: início e fim (50% da amplitude), duração, pico, amplitude, taxas de verdejamento e senescência, integral e posição sazonal. Classes: soja, milho de verão, algodão, milho safrinha, feijão irrigado e planta de cobertura.

Classificadores comparados: regras fenológicas, Random Forest, Extra Trees, Gradient Boosting, XGBoost, SVM (RBF), k-NN, MLP, LDA e regressão logística. Validação cruzada estratificada (5 dobras). Previsão da safra 2026/27 por cadeia de Markov de 1ª ordem.
</details>

<details>
<summary><b>Auditoria</b></summary>

Últimos 5 anos agrícolas. Critérios: safras/ano (|Δ| ≤ 0,5), plantas de cobertura (|Δ| ≤ 1), nº de cultivos (|Δ| ≤ 1) e irrigação (igualdade).

$$IC = \frac{1}{k}\sum_j \mathbb{1}[\text{critério}_j]$$

**CONFORME** se IC = 1, **ATENÇÃO** se IC ≥ 0,5, **DIVERGENTE** caso contrário. O arquivo `modelo_declaracao_produtor.xlsx` permite auditar ano a ano (índice de Jaccard).
</details>

<details>
<summary><b>Carbono: aprendizado de máquina e modelo dinâmico</b></summary>

**ML.** Oito regressores (Ridge, PLSR, RF, Extra Trees, GB, XGBoost, SVR, k-NN) × quatro conjuntos de preditores (manejo + solo; + NDVI; química; manejo + química). Validação deixa-uma-fazenda-fora e importância por permutação fora da amostra.

**Modelo dinâmico (Hénin–Dupuis)** com entrada de C estimada pelo NDVI (eficiência no uso da radiação):

$$f_{APAR} = 0{,}95\,\mathrm{clip}\!\left(\tfrac{NDVI - 0{,}20}{0{,}90 - 0{,}20}, 0, 1\right),\quad I = f_{res}\,\varepsilon\,f_{APAR}\,PAR\,\Delta t$$

$$\frac{dC}{dt} = h\,I(t) - k\,C(t)$$

k depende do período sem revolvimento e h é calibrado por equilíbrio regional. A incerteza vem de Monte Carlo (300 sorteios). Cenários 2027–2036: manejo atual, +25% de entrada de C (cobertura) e plantio direto consolidado.
</details>

---

## Arquivos gerados

```text
resultados/
├── plataforma_auditoria_carbono.html
├── resultados_completos.xlsx              # 25 abas com todos os resultados
├── modelo_declaracao_produtor.xlsx        # para o produtor informar o cultivo de cada ano
├── figuras/                               # 10 figuras em 300 dpi
├── tabelas/                               # CSV de cada análise
├── mapas/                                 # mapa Folium, GeoJSON de talhões e pontos
├── dados_processados/                     # séries NDVI, ciclos, trajetórias, JSON da plataforma
└── artigo/                                # macros e tabelas LaTeX usadas pelo artigo
```

---

## Principais resultados

Estes são os resultados obtidos com os dados de solo reais:

- **Ponto a ponto × médias:** concordância de 0,98 (CCC de Lin), viés de −0,24 Mg ha⁻¹; CV intratalhão de 5,9%.
- **Estoque de C 0–40 cm:** 18 a 74 Mg ha⁻¹ entre os 35 talhões com análise.
- **Manejo:** nenhum fator teve efeito significativo sobre o estoque (Kruskal–Wallis, modelo misto, PERMANOVA).
- **ML do carbono:** nenhum dos 32 modelos superou a média regional. Com 35 talhões e só um com mais de 5 anos sem revolvimento, é preciso ampliar a amostragem.
- **C 0–10 cm × química de rotina:** previsão parcial (R² ≈ 0,23), útil para triagem.

Os resultados de classificação de cultivos (kappa de 0,94 na biblioteca de referência), auditoria e acúmulo de carbono desta versão são **demonstrativos** (NDVI simulado).

---

## Limitações

- NDVI simulado nesta versão: rode no Colab com uma fonte real.
- Polígonos dos talhões estimados pela envoltória convexa dos pontos (+120 m). Para usar limites oficiais, defina `POLIGONOS_GEOJSON` com um GeoJSON que tenha a propriedade `talhao`.
- O pixel MODIS de 250 m subdetecta plantas de cobertura e talhões pequenos; o Sentinel-2 (10 m) é recomendado.
- O carbono foi medido por amostra composta do talhão; a variabilidade espacial do C não é conhecida.
- Os rótulos de cultivo da biblioteca de referência vêm de um modelo fenológico e devem ser complementados com verdade de campo.


## Como citar

```bibtex
@software{plataforma_auditoria_carbono_2026,
  title  = {Plataforma de Auditoria Agrícola e Carbono do Solo -- Oeste da Bahia},
  author = {[Autores]},
  year   = {2026},
  url    = {https://github.com/SEU-USUARIO/SEU-REPOSITORIO}
}
```


## Licença

Defina a licença do código antes de publicar (por exemplo, [MIT](https://choosealicense.com/licenses/mit/)). Os dados das fazendas podem exigir uma licença ou um termo de uso separado.

**Bibliotecas usadas:** pandas, NumPy, SciPy, scikit-learn, XGBoost, statsmodels, PyWavelets, Shapely, Folium, Matplotlib, seaborn, [Leaflet](https://leafletjs.com) e [Plotly.js](https://plotly.com/javascript/). **Mapas-base:** Esri World Imagery, OpenTopoMap e OpenStreetMap.
