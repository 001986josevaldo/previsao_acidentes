Previsão da frequência de acidentes de
trânsito por trecho viário

Case de ciência de dados aplicada | Regressão para contagens | Validação temporal

Este projeto investiga se características geométricas e operacionais de vias ajudam a prever
a quantidade anual de acidentes por trecho. O foco não é apenas comparar algoritmos: é
definir uma unidade de previsão, controlar o vazamento temporal, representar a exposição
e explicitar o que a evidência permite — e não permite — concluir.

Pergunta de negócio: com informações disponíveis até o fim de um ano, qual é a
frequência esperada de acidentes por trecho no ano seguinte, para apoiar a triagem de
locais que merecem investigação de segurança viária?
O resultado deve ser entendido como apoio à priorização e à análise de engenharia. Não
estima causalidade, não substitui auditorias de segurança e não deve, isoladamente,
determinar investimentos ou intervenções.

Principais pontos
• Unidade observacional: trecho-ano ( ID , Ano ).
• Variável-alvo: Acidentes , uma contagem não negativa.
• Base analisada: 577 trechos observados anualmente entre 2021 e 2025 (2.885
registros).
• Proporção preliminar de zeros: 64,8%; variância/média da contagem: 1,86, sugerindo
sobredispersão.
• Validação temporal: treino em 2021–2024 e teste reservado em 2025.
• Modelos comparados: baseline por taxa de exposição, Poisson regularizado e binomial
negativa.

Decisão que o modelo pretende apoiar
A saída potencial é uma estimativa de frequência anual por trecho, útil para ordenar locais
para investigação, auditoria ou coleta adicional de dados. Uma previsão alta não significa
que uma característica da via cause acidentes nem que uma intervenção específica vá
reduzir acidentes. A decisão final precisa incorporar contexto, volume de tráfego,
severidade, custos, equidade e avaliação de profissionais de segurança viária.

Dados e unidade de análise 
Cada linha representa um trecho ( ID ) em um ano ( Ano ). O conjunto fornecido tem 577 IDs
em cada um dos cinco anos e nenhum par ID – Ano duplicado. A auditoria do notebook
não identificou valores ausentes; as características da via não variam entre anos para o
mesmo ID.
A variável de resposta é Acidentes . Os preditores incluem características por sentido — por
exemplo, faixas, larguras, velocidades, canteiro, estacionamento, inclinação, controle e
moderação — além de Ano . Os significados e escalas dos campos codificados,
especialmente Volume_P e Volume_S , devem ser confirmados no dicionário de dados
antes de interpretação.

Exposição
O notebook usa Comp_m / 1000 como aproximação de quilômetros de trecho por ano e
inclui o logaritmo dessa quantidade como offset nos modelos de contagem. Isso
representa uma taxa por extensão, não a exposição real dos veículos. Comprimento não
substitui tráfego observado.
A melhoria de dados prioritária é obter volume médio diário anual (AADT/VDM), idealmente
por trecho, e construir uma medida de exposição como AADT × comprimento × dias
observados . Sem essa informação, os resultados devem ser descritos como preliminares e
condicionados à aproximação adotada.

Método
Exploração e qualidade
O notebook examina tipos, ausências, duplicatas ID – Ano , cobertura anual, distribuição
de contagens, zeros, sobredispersão e estabilidade das características por trecho. Inclui
visualizações da distribuição de acidentes e uma representação de frequências por ano.

Modelos comparados
1. Baseline por taxa/km: taxa média de acidentes por quilômetro estimada no treino,
multiplicada pelo comprimento do trecho de teste.
2. Poisson regularizado: GLM de Poisson com ligação log, tendência linear de ano, offset
de exposição aproximada e regularização L2 ( alpha=0.01 ). O valor de regularização é um
ponto inicial; idealmente deve ser selecionado por validação interna, sem consultar
2025.
3. Binomial negativa (NB2): modelo de contagem com parâmetro de dispersão estimado
por máxima verossimilhança, mesma fórmula e offset.ID não é usado como preditor. Como as características são praticamente estáticas por
trecho e há medidas repetidas por ID, a dependência longitudinal deve ser tratada em uma
próxima versão com estrutura apropriada — por exemplo, GEE, modelo misto ou erros de
inferência agrupados por trecho.

Validação
O modelo é ajustado com 2021–2024 e avaliado em 2025. Esse corte imita a previsão do ano
seguinte para trechos já conhecidos. Uma divisão aleatória de linhas não seria adequada
porque misturaria períodos e registros do mesmo trecho entre treino e teste.
Essa validação não mede diretamente a generalização para trechos nunca vistos. Para
esse cenário é necessária uma validação agrupada por ID . Com mais anos disponíveis,
recomenda-se também validação temporal de origem móvel (rolling-origin).

Resultados preliminares
Valores calculados no notebook para o conjunto de teste de 2025, usando a base original:
!(/media/josevaldo/E02A-3159/MESTRADO/previsao_acidentes/figures/image.png)

Em 2025, foram observados 313 acidentes no conjunto de teste. Os totais previstos foram
aproximadamente 354,2 pelo baseline, 301,0 pelo Poisson e 307,8 pela binomial negativa.
Interpretação: ambos os modelos de contagem superaram o baseline nesta partição. A
binomial negativa obteve MAE ligeiramente menor; o Poisson obteve RMSE, R² e desvio de
Poisson ligeiramente melhores. As diferenças são pequenas e o R² é modesto. Não há base
para declarar um vencedor universal: a escolha depende da função de custo e precisa ser
confirmada em validações adicionais.

Limite importante do ajuste NB
O resumo do ajuste reportou convergência, mas também um aviso de falha na inversão da
Hessiana, com erros-padrão e intervalos de confiança indisponíveis. Alguns coeficientes decategorias aparecem muito grandes. Portanto, as métricas podem ser discutidas como
comparação preditiva preliminar, mas os coeficientes e testes inferenciais da NB não
devem ser interpretados como resultados estáveis. A próxima iteração deve investigar
níveis raros/separação, simplificar ou agrupar categorias com justificativa, avaliar
regularização e reestimar a incerteza.

Diagnósticos e artefatos
• Tabela observado vs. previsto: uma linha por ID no teste de 2025, com previsões dos
três modelos e resíduo ( observado − previsto ). O notebook salva
previsoes_acidentes_2025.csv no Drive.
• MAE e RMSE: quantificam erro médio absoluto e erro quadrático na unidade da
contagem; menores são melhores.
• R²: comparação descritiva adicional; pode ser negativo e não substitui métricas
próprias para contagens.
• Desvio de Poisson e calibração: diagnósticos complementares entre valores previstos
e observados.
• CURE Plot: acumula resíduos ao longo de uma variável ordenada (padrão:
comprimento). As bandas incluídas são aproximações que assumem independência;
devido às medidas repetidas por ID, não devem ser tratadas como limites inferenciais
definitivos sem bootstrap/simulação que respeite a estrutura dos dados.
Uma matriz de confusão só é apropriada se o problema for transformado em classificação
— por exemplo, “houve ao menos um acidente?” — e exige um limiar definido por uma
decisão operacional. Ela não avalia a precisão da quantidade prevista e não substitui
MAE/RMSE.

Decisões de escopo e prevenção de viés
A análise principal mantém os trechos com zero acidentes. Remover os IDs que tiveram
zero em todos os cinco anos condicionaria o modelo a locais com histórico de acidentes,
distorcendo a população-alvo e tendendo a elevar a frequência estimada. Se essa filtragem
for explorada, deve ser apenas uma análise secundária com escopo explicitamente restrito.
Também não se deve remover automaticamente trechos acima de 150 m sem confirmar
erro de cadastro ou exclusão de escopo. Comprimento pode ser parte da exposição; retirar
trechos legítimos pode eliminar locais relevantes. Qualquer filtro deve ser definido sem
usar o resultado do período de teste.

Como reproduzir no Google Colab1. Abra Previsao_Acidentes_Viarios.ipynb no Google Colab.
2. Coloque BD_anos_empilhados_comprimento.csv na raiz do Google Drive ou ajuste
CAMINHO_CSV na célula de importação.
3. Execute as células em ordem. O notebook monta o Drive, lê CSV separado por ; , ajusta
os modelos, calcula métricas e cria os gráficos.
4. A tabela de previsões é salva em MyDrive/previsoes_acidentes_2025.csv .
Dependências usadas no notebook: pandas , numpy , matplotlib , seaborn , statsmodels e
scikit-learn . O runtime hospedado do Colab normalmente já contém essas bibliotecas; se
necessário, instale-as em uma célula separada.

Roadmap técnico
• Obter AADT/VDM anual e revisar a definição de trecho e exposição.
• Diagnosticar categorias raras, separação, colinearidade e instabilidade dos coeficientes
NB.
• Incluir estrutura longitudinal por trecho e avaliar dependência espacial.
• Fazer validação temporal em múltiplos cortes e validação agrupada por ID.
• Definir métricas e limiares com especialistas, considerando o custo de subestimar
trechos de risco.• Se o objetivo operacional for classificação de locais prioritários, definir essa decisão
separadamente da previsão de contagem.