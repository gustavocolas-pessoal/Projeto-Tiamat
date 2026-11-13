<img src="Imagens do README/Cabecalho.png"/>

# Tiamat - Previsão do Número de Citações de Artigos Científicos

<p align="center">
  <img src="https://img.shields.io/badge/Python-3.13.7-blue">
  <img src="https://img.shields.io/badge/scikit--learn-Pipelines-green">
  <img src="https://img.shields.io/badge/Optuna-TPE-purple">
  <img src="https://img.shields.io/badge/Área-Machine%20Learning-darkred">
</p>

O trabalho busca prever o impacto científico de artigos de inteligência artificial indexados no PubMed, medido pela contagem acumulada de citações, utilizando apenas informações conhecidas antes da publicação (título, resumo, autoria, afiliações, revista, tipo de publicação, tamanho da lista de referências e data). Os artigos, publicados entre 2018 e 2024, foram sorteados por janelas mensais com uma cota igual por ano e coletados pelas APIs oficiais do PubMed (E-utilities) e da OpenAlex. Foram comparados um baseline (a mediana) e cinco pipelines otimizados pelo Optuna com validação cruzada de 5 dobras: o Rigde, o *k*-NN, a floresta aleatória, o XGBoost e Stacking. Este projeto é a entrega final ("Tiamat") da disciplina Aprendizado de Máquina, ministrada pelo Prof. Dr. Daniel R. Cassar na Ilum Escola de Ciência, realizada individualmente pelo aluno Gustavo Junqueira Colas.

## Como executar:

1. Baixe ou clone o repositório e extraia os arquivos em uma pasta

2. Com o seu editor (preferencialmente o VSCode), vá em *File*, *Open Folder* e selecione a pasta baixada

3. Instale as dependências com o comando:
```sh
pip install -r requirements.txt
```

4. Abra o arquivo **relatorio_tiamat.ipynb** e execute todas as células em ordem (*Restart* e *Run All*)

Observação: os dados coletados (pasta `dados`) e os resultados das otimizações (pasta `saidas`) já acompanham o repositório e são reaproveitados automaticamente, de modo que o notebook roda em poucos minutos e reproduz exatamente os números do relatório. Para recoletar os dados do zero, basta mudar `RECOLETAR = False` para `True`. Nesse caso, as citações serão as da nova data de coleta (elas continuam se acumulando), e os resultados mudarão. Para refazer as otimizações, basta apagar os arquivos da pasta `saidas`, o que leva bem mais tempo (só a floresta aleatória e o XGBoost levaram cerca de 20~25 minutos cada).

## Conteúdo:

<ul>
  <li>:file_folder: <b>Imagens do README</b>: contém as imagens de cabeçalho e rodapé deste README</li>
  <li>:notebook: <b>Tiamat_relatorio.ipynb</b>: notebook completo, com introdução, coleta, análise exploratória, pipelines, resultados, conclusões e referências</li>
  <li>:spiral_notepad: <b>requirements.txt</b>: lista das bibliotecas necessárias para rodar o projeto</li>
  <li>🛡️: <b>Emblema do Tiamat</b>: brasão da lore da disciplina, com temática crítica sobre a mensuraação por citações</li>
</ul>

## Tecnologias Utilizadas:

O projeto foi confeccionado inteiramente em *Jupyter Notebook*, na linguagem *Python* (versão 3.13.7), utilizando as bibliotecas *pandas*, *numpy*, *scipy*, *scikit-learn*, *xgboost*, *optuna*, *matplotlib*, *requests* e *beautifulsoup4* (esta última apenas no exemplo didático de raspagem com o site Quotes to Scrape), além dos módulos nativos *json*, *os*, *re*, *time*, *hashlib*, *random*, *datetime*, *itertools* e *xml*. Inteligência artificial (na forma de *LLMs*) foi utilizada como tutora ao longo do projeto, com forte assistência na escrita do código de coleta pelas APIs, na redação de parte dos textos e da interpretação dos resultados, na disposição visual dos gráficos e no refinamento deste README. A arquitetura do projeto (fontes, amostragem, atributos, pipelines e forma de avaliação) é do autor, e o uso de IA está descrito em detalhes na seção 7 do notebook.

## Sobre o dataset e os atributos:

O modelo tem como objetivo prever o número de citações acumuladas de cada artigo (`citation_count`, obtido na OpenAlex), modelado na escala `log(1 + citações)` por causa da cauda longa da distribuição. São 30 atributos, todos conhecidos no momento da publicação:

| Grupo | Atributos |
|---|---|
| Título | `titulo_n_palavras`, `titulo_tam_medio_palavra`, `titulo_tem_dois_pontos`, `titulo_tem_interrogacao`, `titulo_tem_numero` |
| Resumo | `resumo_n_palavras`, `resumo_tam_medio_palavra`, `resumo_estruturado` |
| Autoria e colaboração | `n_autores`, `n_afiliacoes`, `n_instituicoes`, `n_paises`, `colaboracao_internacional`, `tem_afiliacao` |
| Tipo de publicação e financiamento | `eh_revisao`, `n_tipos_publicacao`, `apoio_nih`, `apoio_nao_eua`, `n_financiamentos` |
| Referências e indexação | `n_referencias`, `n_referencias_pubmed`, `n_mesh`, `n_keywords`, `tem_doi` |
| Data | `idade_dias` (tempo entre a publicação e a coleta), `mes_ref` |
| Categóricos | `revista`, `topico_principal`, `area_principal`, `tipo_revista_oa` |

## Modelos e hiperparâmetros investigados:

Cada pipeline reúne o pré-processamento (imputação, escala, agrupamento de categorias raras e codificação) e o modelo, ajustados apenas na parte de treino de cada dobra. Os hiperparâmetros foram otimizados pelo Optuna, com 100 trials por algoritmo e semente fixa (19):
- Rigde: força da regularização (`alpha`)
- *k*-NN: número de vizinhos (*k*), distância (Euclidiana, Manhattan ou Chebyshev) e ponderação dos vizinhos
- Floresta aleatória: número de árvores, profundidade máxima, tamanho mínimo das folhas e das divisões e fração de atributos por divisão
- XGBoost: número de árvores, taxa de aprendizado, profundidade, `min_child_weight`, `subsample`, `colsample_bytree` e `gamma`
- Stacking: regularização, intercepto e sinal dos pesos do meta-modelo (um Rigde) e quais bases entram no comitê
- Nas quatro bases: a frequência mínima para que uma revista ou um tópico não seja agrupado como raro; no Rigde e no *k*-NN, também o tipo de escala dos atributos numéricos

## Resultados no conjunto de teste:

| Modelo | RMSE em log(1 + citações) | MAE (citações) | Spearman |
|---|---|---|---|
| Stacking | 1,0059 | 44,11 | 0,602 |
| Rigde | 1,0083 | 44,66 | 0,601 |
| XGBoost | 1,0267 | 44,34 | 0,580 |
| Floresta aleatória | 1,0663 | 44,99 | 0,536 |
| *k*-NN | 1,0984 | 45,89 | 0,493 |
| Baseline (mediana) | 1,2516 | 49,45 | 0,000 |

Todos os pipelines superam o baseline, mas a comparação pareada por bootstrap mostra que a vantagem do stacking sobre o Rigde não é distinguível do acaso. Os resultados indicam associação preditiva, e não causa.

## Referências:

<h4>📘 Materiais da disciplina</h4>
<ul>
<li>CASSAR, D. R. <i>Aprendizado de máquina, k-NN e métricas</i>. Campinas: Ilum Escola de Ciência, 2026. Notebook Jupyter da disciplina ATP-203 - Aprendizado de Máquina (aula 2.1).</li>
<li>CASSAR, D. R. <i>Árvore de decisão e floresta aleatória</i>. Campinas: Ilum Escola de Ciência, 2026. Notebook Jupyter da disciplina ATP-203 - Aprendizado de Máquina (aula 4.0).</li>
<li>CASSAR, D. R. <i>Dados sintéticos e pipeline</i>. Campinas: Ilum Escola de Ciência, 2026. Notebook Jupyter da disciplina ATP-203 - Aprendizado de Máquina (aula 7.0).</li>
<li>CASSAR, D. R. <i>Divisão de dados em treino e teste</i>. Campinas: Ilum Escola de Ciência, 2026. Notebook Jupyter da disciplina ATP-203 - Aprendizado de Máquina (aula 2.2).</li>
<li>CASSAR, D. R. <i>Modelo linear e baseline</i>. Campinas: Ilum Escola de Ciência, 2026. Notebook Jupyter da disciplina ATP-203 - Aprendizado de Máquina (aula 3.0).</li>
<li>CASSAR, D. R. <i>Otimização de hiperparâmetros com optuna</i>. Campinas: Ilum Escola de Ciência, 2026. Notebook Jupyter da disciplina ATP-203 - Aprendizado de Máquina (aula 7.1).</li>
<li>CASSAR, D. R. <i>Seleção de atributos</i>. Campinas: Ilum Escola de Ciência, 2026. Notebook Jupyter da disciplina ATP-203 - Aprendizado de Máquina (aula 8.1).</li>
<li>CASSAR, D. R. <i>Tipos de aprendizado de máquina e algoritmo k-NN</i>. Campinas: Ilum Escola de Ciência, 2026. 1 apresentação (15 slides). Material da disciplina ATP-203 - Aprendizado de Máquina (aula 2.0).</li>
<li>CASSAR, D. R. <i>Tratamento de dados com pandas</i>. Campinas: Ilum Escola de Ciência, 2026. Notebook Jupyter da disciplina ATP-203 - Aprendizado de Máquina (aula 1.1).</li>
<li>CASSAR, D. R. <i>Validação cruzada e otimização de hiperparâmetros</i>. Campinas: Ilum Escola de Ciência, 2026. Notebook Jupyter da disciplina ATP-203 - Aprendizado de Máquina (aula 6.0).</li>
<li>CASSAR, D. R. <i>Vazamento de dados</i>. Campinas: Ilum Escola de Ciência, 2026. Notebook Jupyter da disciplina ATP-203 - Aprendizado de Máquina (aula 6.1).</li>
</ul>

<h4> Esfinges da Biblioteca Real</h4>
<ul>
<li>CUTOLO, V. M. D. M. <i>[Esfinge: regularização]</i>. Campinas: Ilum Escola de Ciência, [2026?]. Disponível em: <a href="https://github.com/drcassar/biblioteca_real/tree/main/Aprendizado%20de%20M%C3%A1quina/Regulariza%C3%A7%C3%A3o/T26%20-%20Vin%C3%ADcius%20Marianno%20De%20Marque%20Cutolo">Biblioteca Real (GitHub)</a>. Acesso em: 13 nov. 2026.</li>
<li>SOUZA, M. L. M. de. <i>[Esfinge: XGBoost]</i>. Campinas: Ilum Escola de Ciência, [2026?]. Disponível em: <a href="https://github.com/drcassar/biblioteca_real/tree/main/Aprendizado%20de%20M%C3%A1quina/XGBoost/T26%20-%20Matheus%20Luiz%20Mendes%20de%20Souza">Biblioteca Real (GitHub)</a>. Acesso em: 13 nov. 2026.</li>
</ul>

<h4>🌐 Artigos e cursos online</h4>
<ul>
<li>BELCIC, I.; STRYKER, C. <i>What is feature selection?</i> IBM Think. Disponível em: <a href="https://www.ibm.com/think/topics/feature-selection">https://www.ibm.com/think/topics/feature-selection</a>. Acesso em: 11 set. 2026.</li>
<li>BERGMANN, D. <i>O que é aprendizado de máquina?</i> IBM Think, 18 ago. 2025. Disponível em: <a href="https://www.ibm.com/br-pt/think/topics/machine-learning">https://www.ibm.com/br-pt/think/topics/machine-learning</a>. Acesso em: 13 nov. 2026.</li>
<li>GOOGLE FOR DEVELOPERS. <i>Regressão linear</i>. Machine Learning Crash Course, 2026. Disponível em: <a href="https://developers.google.com/machine-learning/crash-course/linear-regression?hl=pt-br">https://developers.google.com/machine-learning/crash-course/linear-regression</a>. Acesso em: 13 nov. 2026.</li>
<li>MUREL, J.; KAVLAKOGLU, E. <i>O que é regressão ridge?</i> IBM Think, 21 set. 2023. Disponível em: <a href="https://www.ibm.com/br-pt/think/topics/ridge-regression">https://www.ibm.com/br-pt/think/topics/ridge-regression</a>. Acesso em: 13 nov. 2026.</li>
</ul>

<h4>🛠️ Documentações e ferramentas</h4>
<ul>
<li>OPTUNA. <i>Optuna documentation</i>. Disponível em: <a href="https://optuna.readthedocs.io/">https://optuna.readthedocs.io/</a>. Acesso em: 13 nov. 2026.</li>
<li>RICHARDSON, L. <i>Beautiful Soup documentation</i>. Versão 4.14.3. Disponível em: <a href="https://www.crummy.com/software/BeautifulSoup/bs4/doc/">https://www.crummy.com/software/BeautifulSoup/bs4/doc/</a>. Acesso em: 13 nov. 2026.</li>
<li>SCIKIT-LEARN. <i>User guide</i>. Disponível em: <a href="https://scikit-learn.org/stable/user_guide.html">https://scikit-learn.org/stable/user_guide.html</a>. Acesso em: 13 nov. 2026.</li>
<li>ZYTE. <i>Quotes to Scrape</i>. Disponível em: <a href="https://quotes.toscrape.com/">https://quotes.toscrape.com/</a>. Acesso em: 13 nov. 2026.</li>
</ul>

## Professor avaliador:

<table>
  <tr>
    <td align="center">
      <a href="https://github.com/drcassar">
        <img src="https://avatars.githubusercontent.com/u/9871905?v=4" width="100px;" alt="Foto do Cassar no Github"/><br>
        <b>Prof. Dr. Daniel R. Cassar</b>
      </a>
    </td>
  </tr>
</table>

## Aluno:

<table>
  <tr>
    <td align="center">
      <a href="https://github.com/gustavocolas-pessoal">
        <img src="https://avatars.githubusercontent.com/u/284495799?s=80&v=4" width="100px;" alt="Foto do Gustavo no Github"/><br>
        <b>Gustavo Junqueira Colas</b>
      </a>
    </td>
  </tr>
</table>
<img src="Imagens do README/Rodapé (1).png"/>

## Agradecimentos:

Ao professor Prof. Dr. Daniel R. Cassar, por fornecer base teórica de Probabilidade e Estatística e Aprendizado de Máquina, como também por sanar dúvidas fundamentais na criação do projeto. Aos colegas de sala e amigos Vinícius M. M. Cutolo, Filipi Martins, Pedro Lucas M. S. de Sá, Gabriel O. Paiva e Matheus Luiz Mendes de Souza por ajudarem a avaliar o andamento do projeto (me ajudarem a entender mais claramente o que eu estava fazendo) e por sugerirem materiais extras (como vídeos didáticos no Youtube e até fornecendo os próprios materiais nesses quesitos - as Esfinges supracitadas). 
