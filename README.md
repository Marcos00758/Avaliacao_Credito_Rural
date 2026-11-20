# Crédito rural e produção agrícola

Projeto de Análise Exploratória de Dados sobre crédito rural e produção de soja, milho e cana-de-açúcar nos municípios brasileiros, na última década disponível.

Utilizamos dados do SICOR/BCB e da PAM/IBGE para investigar distribuição e concentração do crédito, evolução temporal e associações com área cultivada e produtividade.

As análises são desenvolvidas em Python, em notebooks Jupyter, com visualizações e estatísticas descritivas, sem inferência causal.
## Base para comparacao de municipios

Em `Integrar_APIs.ipynb`, `validar_tabela()` filtra `dados/tabela_final.csv` por cobertura `ambos`, area plantada positiva e credito finito nao negativo. O notebook informa as exclusoes por etapa e salva `dados/tabela_validada.csv`, mantendo a base completa.

A coluna `quantidade_contratos` preserva contagens disponiveis. Os caches municipais atuais nao fornecem essa medida, portanto ela permanece ausente. `preparar_lote()` em `API_BC.ipynb` aproveita `QtdCusteio` caso a resposta passe a fornecer o campo. Contagem de linhas nao representa quantidade de contratos.
