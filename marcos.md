# Coleta, tratamento e visualização dos dados

## Dados agrícolas no notebook API_IBGE

O `API_IBGE.ipynb` consulta a API de Agregados do IBGE/SIDRA, tabela 1612, de lavouras temporárias. Seleciona soja, milho e cana-de-açúcar pela classificação 81 e pelos códigos 2713, 2711 e 2696. Solicita 2016–2025, conforme a disponibilidade da fonte.

Coleta área plantada em hectares (variável 109) para regiões, estados e municípios; produção em toneladas (214) para estados e municípios; e rendimento médio em kg/ha (112) para municípios. Os níveis territoriais são N2, N3 e N6. Área colhida e percentuais têm constantes definidas, mas não são coletados.

As requisições usam `requests`, parâmetros de território e cultura, prazo de 30 segundos e conferência do status HTTP. `IBGETLSAdapter`, com `ssl` e `HTTPAdapter`, acrescenta o certificado intermediário sem desativar a validação HTTPS. As consultas municipais são anuais, dividindo o volume; as demais solicitam o intervalo completo.

A função `criar_dataframe()` transforma o JSON em linhas com códigos e nomes de cultura e território, ano, variável, unidade e valor. Converte `-` em zero, aceita sequências de algarismos e descarta os demais conteúdos, inclusive representações decimais ou negativas. Os valores viram `float`, e os anos, inteiros. Essa escolha simplifica o tratamento, mas não distingue todos os motivos de ausência. As tabelas são ordenadas por território, cultura, ano e valor.

Gera seis DataFrames em memória, sem exportação automática: `df_area_regional`, `df_area_estadual`, `df_area_municipal`, `df_prod_estadual`, `df_prod_municipal` e `df_rendimento_municipal`. As funções de cache usam JSON, SHA-256 e `lru_cache`, mas a coleta chama diretamente `requisicao_certificada()`. Como `consultar()` ainda espera OData, o cache definido não garante recuperação das consultas ao IBGE quando o servidor falha.

## Preparação das tabelas dos gráficos 4 e 5

Em `Integrar_APIs.ipynb`, ambos partem de `dados/tabela_final.csv`, com indicadores agrícolas e crédito do BCB. `validar_tabela()` mantém cobertura nas duas fontes, área positiva e finita e crédito finito não negativo, informando exclusões. Gera `tabela_validada` e seu CSV. As análises representam essa cobertura conjunta; o crédito é nominal, sem correção da inflação.

O gráfico 4 usa `tabela_categorizada`, derivada da validada. `categorizar_faixas_hectares()` cria intervalos com limite superior 25% maior que o inferior, desde a menor área de cada cultura e ano. Evita um corte único em hectares para grupos diferentes. As faixas aparecem nos detalhes, sem restringir os pontos exibidos.

## Gráfico 4 — Crédito entre municípios com áreas semelhantes

`criar_grafico_dispersao_area_credito()` representa cada município por um ponto: área no eixo horizontal e crédito no vertical. Posições horizontais próximas permitem comparar áreas semelhantes; diferenças verticais mostram a variação do crédito. A dispersão preserva observações individuais e evidencia concentrações e extremos. Não estabelece causalidade nem compara contratos, cuja contagem está indisponível.

Há filtros de cultura, ano e UF, transparência para reduzir sobreposição, zoom e detalhes ao passar o cursor. O degradê regional vai do verde no Norte ao azul no Sul, passando por Nordeste, Centro-Oeste e Sudeste nessa ordem na legenda.

## Gráfico 5 — Crédito por tonelada e por hectare nos estados

`calcular_indicadores_estaduais()` gera `tabela_indicadores_estaduais` a partir da validada, por cultura, ano e UF. Calcula R$/t = soma do crédito ÷ soma da produção e R$/ha = soma do crédito ÷ soma da área. Cada razão usa os mesmos municípios no numerador e denominador, excluindo denominadores inválidos ou não positivos. A razão entre somas considera o peso da produção ou área, evitando uma média simples das razões municipais.

`criar_graficos_estaduais_questao5()` usa dois painéis de barras horizontais ordenadas, facilitando comparar estados sem acumular milhares de pontos. As escalas são independentes pelas unidades distintas. Estados abaixo do primeiro quartil da contagem de municípios cobertos por cultura, ano e indicador, arredondado para cima, ficam em cinza ao final. Esse limite relativo sinaliza cobertura reduzida, sem alterar as razões ou a referência nacional. A linha vermelha tracejada mostra a referência nacional da mesma cobertura. Filtros de cultura e ano e detalhes dos totais e municípios cobertos contextualizam os resultados, que não representam necessariamente toda a produção estadual.

Ambos usam `pandas` na preparação, `Altair` no desenho, `ipywidgets` nos controles e `IPython.display` na exibição. `alt.JupyterChart`, apoiado em `anywidget`, recebe o desenho inicial e o atualiza pelos filtros. `Gráficos.ipynb` carrega a integração e chama a exibição, sem exportar imagens e com o menu de ações oculto.

## Justificativa visual e interpretação dos resultados

A dispersão do gráfico 4 foi escolhida porque a pergunta envolve duas medidas quantitativas municipais: área e crédito. Ela permite observar a relação entre essas medidas e procurar diferenças de crédito entre pontos próximos no eixo da área, dentro da mesma cultura e ano. As cores regionais acrescentam contexto territorial. A leitura preserva diferenças entre municípios que uma média estadual esconderia, embora a proximidade horizontal não constitua um pareamento estatístico nem controle outras características.

Estar mais acima e à esquerda indica mais crédito para uma área menor, mas não define automaticamente um outlier. O gráfico não aplica um critério estatístico para identificar ou ordenar casos atípicos. No recorte inicial de soja, 2025 e todas as UFs, Sebastião Leal (PI) registra R$ 453,95 milhões para 59.670 ha, enquanto Parnarama (MA) apresenta R$ 115,67 milhões para 14.000 ha. São destaques de crédito elevado em relação à área: aproximadamente R$ 7.608/ha e R$ 8.262/ha. Formosa do Rio Preto (BA), apesar do maior crédito absoluto do recorte, R$ 641,37 milhões, também tem 533.851 ha, resultando em cerca de R$ 1.201/ha.

Já os maiores valores de crédito por hectare na soja de 2025 são Cidreira (RS), com aproximadamente R$ 51.498/ha, Cotegipe (BA), R$ 45.297/ha, e Piçarra (PA), R$ 44.180/ha. Suas áreas são apenas 65, 263 e 130 ha. Esses casos mostram que um denominador pequeno pode produzir uma razão extrema sem colocar o ponto entre os maiores créditos totais. Merecem investigação da correspondência entre crédito e área; os dados não permitem concluir que sejam erros ou benefícios excessivos.

As barras do gráfico 5 foram escolhidas porque a pergunta compara um conjunto limitado de estados por um indicador agregado. Seu comprimento e sua ordenação tornam diferenças e posições mais fáceis de comparar. Os dois painéis separam R$/ha e R$/t, e a referência nacional contextualiza cada posição. O ranking expressa intensidade de crédito, não liderança em produção, nem eficiência do financiamento.

Na soja de 2025, Alagoas apresenta cerca de R$ 1.814/ha e R$ 1.080/t, mas o resultado vem de um único município, com 100 ha e 168 t. Mato Grosso apresenta R$ 529/ha e R$ 135/t, com 118 municípios cobertos. Piauí e Tocantins também superam Mato Grosso em R$/ha: aproximadamente R$ 1.659 e R$ 1.601. Portanto, a posição elevada de alguns estados do Norte e Nordeste não implica maior crédito total ou maior produção.

A explicação imediata está na razão entre crédito e denominador, combinada à cobertura. R$/t também equivale a R$/ha dividido pela produtividade em t/ha, quando calculados sobre os mesmos municípios: uma produtividade menor pode elevar o crédito por tonelada. Maior intensidade de financiamento, diferenças entre área plantada e financiada e desencontros entre ano civil do crédito e ciclo agrícola são hipóteses a verificar, não causas demonstradas. A cana reforça o cuidado com cobertura: em 2025, Maranhão registra cerca de R$ 57,25/t em três municípios, contra R$ 6,53/t em 349 municípios paulistas. Esses resultados descrevem a amostra disponível e mudam conforme cultura, ano e filtros.
