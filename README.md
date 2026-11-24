# Crédito rural e produção agrícola

Projeto de Análise Exploratória de Dados sobre crédito rural e produção de soja, milho e cana-de-açúcar nos municípios brasileiros, na última década disponível.

Utilizamos dados do SICOR/BCB e da PAM/IBGE para investigar distribuição e concentração do crédito, evolução temporal e associações com área cultivada e produtividade.

As análises são desenvolvidas em Python, em notebooks Jupyter, com visualizações e estatísticas descritivas, sem inferência causal.
## Base para comparacao de municipios

Em `Integrar_APIs.ipynb`, `validar_tabela()` filtra `dados/tabela_final.csv` por cobertura `ambos`, area plantada positiva e credito finito nao negativo. O notebook informa as exclusoes por etapa e salva `dados/tabela_validada.csv`, mantendo a base completa.

A coluna `quantidade_contratos` preserva contagens disponiveis. Os caches municipais atuais nao fornecem essa medida, portanto ela permanece ausente. `preparar_lote()` em `API_BC.ipynb` aproveita `QtdCusteio` caso a resposta passe a fornecer o campo. Contagem de linhas nao representa quantidade de contratos.

### Faixas de area plantada

A funcao `categorizar_faixas_hectares(tabela, amplitude=0.25)` recebe a base validada e forma intervalos com limite superior 25% maior que o inferior, iniciando na menor area de cada cultura e ano. Adiciona rotulo, ordem e limites numericos da faixa, preservando as colunas existentes. O notebook salva `dados/tabela_categorizada.csv` e disponibiliza `resumo_faixas` com o numero de municipios por grupo. Faixas de culturas ou anos diferentes devem ser comparadas pelos limites em hectares, e nao apenas pelo numero do grupo.

### Estatisticas por faixa

`resumir_estatisticas_faixas()` em `Integrar_APIs.ipynb` gera `tabela_estatistica` e `dados/tabela_estatistica_faixas.csv`, por cultura, ano e faixa: numero de municipios, mediana da area e mediana, quartis e intervalo interquartil do credito e do credito por hectare. Os indicadores por hectare sao calculados por municipio antes do resumo. Se houver contagens municipais validas, inclui estatisticas de contratos, contratos por mil hectares e a mediana do valor medio municipal por contrato, com contagens da cobertura. Grupos de um municipio sao mantidos e precisam de cautela na interpretacao. Nenhuma estatistica de contratos e produzida a partir de contagem de linhas.

### Dispersao de credito e area

A secao de dispersao de `Integrar_APIs.ipynb` usa `tabela_categorizada` e oferece filtros de cultura, ano e UF, zoom e detalhes por municipio. Compara area plantada (ha) e credito nominal (R$), com cores por regiao. O grafico e exibido diretamente no notebook, sem criar pasta nem exportar arquivos. Quantidade de contratos nao faz parte deste grafico.

### Questao 5 - Comparacao estadual

`calcular_indicadores_estaduais(tabela_validada)` calcula R$/t e R$/ha pela razao entre somas de credito e denominador dos mesmos municipios de cada UF, cultura e ano. Producao invalida ou nao positiva e excluida do calculo por tonelada, com contagem informada. `criar_graficos_estaduais_questao5()` exibe dois paineis de barras com filtros compartilhados de cultura e ano e referencia nacional da mesma cobertura. Os valores representam os municipios cobertos, nao necessariamente a totalidade estadual. Graficos e tabela ficam no notebook, sem exportar arquivos.

### Exibicao dos graficos no VS Code

Os graficos 4 e 5 usam seletores `Dropdown` e `interactive_output` com o mesmo padrao do grafico 2. O grafico 4 permite escolher cultura, ano e UF; o grafico 5, cultura e ano. A selecao inicial e soja no ano mais recente. Os filtros sao aplicados em Python e os graficos Altair usam a mesma saida HTML padrao do grafico 1. Reexecute a celula de imports e as celulas dos graficos com as tabelas disponiveis. As figuras e os dados nao sao exportados por essas celulas.

### Organização dos gráficos

O notebook apresenta os gráficos existentes na ordem das perguntas 1, 2, 3, 4 e 5; não há seções vazias para 6 e 7. Cada seção tem um parágrafo com a pergunta e a interpretação. O código dos gráficos 1 a 3 foi preservado; somente o dos gráficos 4 e 5 foi simplificado, mantendo os seletores, os indicadores, os detalhes e o renderer padrão do Altair, sem exportar gráficos.

### Mapas e produtividade — gráficos 6 e 7

Os mapas usam GeoPandas e NumPy, sem geobr nem DuckDB. A seção de geografia obtém GeoJSON simplificado da API de malhas do IBGE com período fixo em 2020 e guarda as malhas e a proveniência em `dados/ibge/malhas/municipio_2020.json` e `uf_2020.json`; após a primeira obtenção, o cache permite leitura sem rede. O código conserva os filtros de 50 ha e valores positivos, os quartis e os critérios estaduais de valores atípicos. Os gráficos permanecem no notebook, com seletores de cultura, ano e recorte; não são exportados. Para executar, rode a preparação geográfica antes dos gráficos 6 e 7. Registros sem polígono na malha de 2020 são informados e mantidos nas estatísticas, sem representação no mapa. Os gráficos 1 a 5 foram preservados.

Os seletores e a saída de cada gráfico com widgets são exibidos juntos em um único contêiner `VBox`, evitando múltiplas visualizações separadas dos controles na célula. Após atualizar o notebook, limpe as saídas antigas e reexecute as células.

Os gráficos 2, 3, 6 e 7 atualizam uma única imagem PNG em memória dentro dos widgets, substituindo o conteúdo a cada seleção sem acumular figuras ou exportar arquivos. O gráfico 7 apresenta somente o mapa, sem a tabela de casos atípicos.
