# etl-jobs-datas

README: Análise e Visualização de Dados de Profissões (2022-2025)
Este projeto demonstra um pipeline completo de Extração, Transformação e Carregamento (ETL) de dados, utilizando Python (Pandas e Matplotlib) para analisar a contagem de profissionais em diferentes cargos no Brasil no período de 2022 a 2025 (com projeções para 2024 e 2025).
________________________________________
1.  Estrutura do Projeto
O projeto utiliza um arquivo JSON externo como fonte de dados e um script Python (executado em ambiente como Google Colab ou Jupyter Notebook) para processar e visualizar as informações.
Arquivos Principais:
Arquivo	Descrição
profissoes_anuais.json	Arquivo de entrada contendo a contagem de profissionais aninhada por ano.
script_analise.py	Código Python que executa as etapas de ETV e gera o gráfico de médias.
________________________________________
2.  EXTRAÇÃO (Extract)
A fase de extração envolve a leitura e o carregamento do arquivo JSON no ambiente de execução.
Ação: O código lê o arquivo profissoes_anuais.json usando a biblioteca json do Python.
Código-Chave:
Python
import json
import pandas as pd

file_path = "profissoes_anuais.json" 

with open(file_path, 'r', encoding='utf-8') as f:
    # Carrega todo o conteúdo do JSON como um dicionário Python
    dados_json = json.load(f)

# Acessa a lista de dados brutos
profissoes_data = dados_json['profissoes_anuais']
df_profissoes = pd.DataFrame(profissoes_data)
Formato Inicial dos Dados (Exemplo Parcial):
cargo	area	contagem
Vendedor...	Comércio	{'2022': 3314000, '2023': 3400000, ...}
Programador...	TI	{'2022': 720000, '2023': 810000, ...}
________________________________________
3.  TRANSFORMAÇÃO (Transform)
Esta fase é crucial para normalizar os dados aninhados (contagem) e calcular a métrica de interesse (Média Anual).
Passos de Transformação:
1.	Normalização (Desaninhamento): A coluna contagem (que é um dicionário) é expandida para novas colunas (2022, 2023, 2024, 2025).
2.	Pivotagem: As colunas Cargo e Área são definidas como índice, resultando na tabela de contagem final.
3.	Cálculo da Média: A média da contagem de profissionais é calculada ao longo dos anos (2022 a 2025) para cada cargo.
4.	Estrutura Final: O DataFrame é simplificado para conter apenas Cargo, Área e a Média de Contagem.
Código-Chave (Transformação e Média):
Python
# 1. Normalização da Coluna 'contagem'
df_contagem_expandida = pd.json_normalize(df_profissoes['contagem'])
df_tabela_final = pd.concat([df_profissoes[['cargo', 'area']], df_contagem_expandida], axis=1)

# 2. Definição do Índice e Pivotagem
df_tabela_final.rename(columns={'cargo': 'Cargo', 'area': 'Área'}, inplace=True)
df_tabela_final = df_tabela_final.set_index(['Cargo', 'Área'])
df_tabela_final.columns = df_tabela_final.columns.astype(int) # Converte colunas de ano para int

# 3. Cálculo da Média Anual (sobre as 4 colunas de ano)
df_tabela_final['Média Anual (2022-2025)'] = df_tabela_final.iloc[:, 0:4].mean(axis=1)

# 4. Seleção da Tabela para Visualização Final
df_transformado = df_tabela_final.reset_index()
df_transformado = df_transformado.iloc[:, [0, 1, -1]]
df_transformado.rename(columns={
    df_transformado.columns[0]: 'Cargo',
    df_transformado.columns[1]: 'Área',
    df_transformado.columns[-1]: 'Média de Contagem'
}, inplace=True)
________________________________________
4.  CARREGAMENTO (Load)
O resultado (df_transformado) é carregado no ambiente de saída, primeiramente sendo exibido como tabela e, em seguida, como um gráfico de barras.
Saída 1: Tabela (Display/Print)
O DataFrame final é impresso para mostrar a métrica calculada.
Cargo	Área	Média de Contagem
Vendedor de Comércio Varejista	Comércio	3.416.000,0
Assistente Administrativo	Administrativo	2.230.500,0
Programador/Desenvolvedor	TI	862.500,0
Saída 2: Visualização (Gráfico de Barras)
Um gráfico de barras horizontais é gerado para comparar a média da contagem de profissionais entre os diferentes cargos, facilitando a identificação dos maiores volumes de emprego no período.
Código-Chave (Visualização):
Python
import matplotlib.pyplot as plt

df_plot = df_transformado.sort_values(by='Média de Contagem', ascending=True)

plt.figure(figsize=(12, 7))
plt.barh(df_plot['Cargo'], df_plot['Média de Contagem'])
# ... (Configurações de rótulos e formatação de milhões)
plt.title('Média Anual de Contagem de Profissionais por Cargo (2022-2025)')
plt.show()

