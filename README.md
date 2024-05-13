import pandas as pd
import dash
import dash_core_components as dcc
import dash_html_components as html
import plotly.graph_objs as go
from dash.dependencies import Input, Output
from tkinter import Tk, filedialog


# Função para analisar as palavras-chave no arquivo CSV
def analisar_palavras_chave(arquivo_csv):
    df = pd.read_csv(arquivo_csv)

    colunas_analisadas = ['Localização / Palavra-Chave', 'Impressão', 'Cliques', 'CTR', 'Conversões', 'Conversões Diretas', 
                          'Taxa de Conversão', 'Taxa de Conversão Direta', 'VBM', 'Receita direta', 'Despesas', 
                          'ROAS', 'ROAS Direto', 'ACOS', 'ACOS Direto']

    metricas_por_palavra_chave = {}

    for index, row in df.iterrows():
        palavra_chave = row['Nome do Produto']

        if palavra_chave not in metricas_por_palavra_chave:
            metricas_por_palavra_chave[palavra_chave] = {}

        for coluna in colunas_analisadas:
            if coluna != 'Nome do Produto':
                if coluna in ['Impressão', 'CTR', 'Taxa de Conversão', 'Taxa de Conversão Direta']:
                    valor = float(row[coluna].replace('%', '')) if isinstance(row[coluna], str) else row[coluna]
                else:
                    valor = row[coluna]
                metricas_por_palavra_chave[palavra_chave][coluna] = valor

    return metricas_por_palavra_chave

# Função para abrir a janela de seleção de arquivo
def selecionar_arquivo():
    root = Tk()
    root.withdraw() # Esconder a janela principal
    arquivo_csv = filedialog.askopenfilename(filetypes=[("CSV files", "*.csv")]) # Abrir a janela de seleção de arquivo CSV
    return arquivo_csv

# Chamando a função para selecionar o arquivo CSV
arquivo_csv = selecionar_arquivo()

# Chamando a função para analisar as palavras-chave
metricas_por_palavra_chave = analisar_palavras_chave(arquivo_csv)

# Inicializando o aplicativo Dash
app = dash.Dash(__name__)

# Layout do aplicativo
app.layout = html.Div([
    html.H1("Dashboard de Análise de Palavras-Chave", style={'textAlign': 'center', 'marginBottom': '20px'}),
    html.Div([
        dcc.Dropdown(
            id='palavra-chave-dropdown',
            options=[{'label': palavra_chave, 'value': palavra_chave} for palavra_chave in metricas_por_palavra_chave.keys()],
            value=list(metricas_por_palavra_chave.keys())[0]
        )
    ], style={'marginBottom': '20px'}),
    html.Div(id='metricas-container'),
    dcc.Graph(id='grafico-metricas')
], style={'maxWidth': '800px', 'margin': 'auto'})

# Callback para atualizar as métricas exibidas com base na palavra-chave selecionada
@app.callback(
    Output('metricas-container', 'children'),
    [Input('palavra-chave-dropdown', 'value')]
)
def atualizar_metricas(palavra_chave_selecionada):
    metricas = metricas_por_palavra_chave[palavra_chave_selecionada]
    metricas_html = [html.H3("Métricas da Palavra-Chave: " + palavra_chave_selecionada, style={'marginTop': '30px'})]
    for metrica, valor in metricas.items():
        metricas_html.append(html.P(f"{metrica}: {valor}"))
    return metricas_html

# Callback para atualizar o gráfico com base na palavra-chave selecionada
@app.callback(
    Output('grafico-metricas', 'figure'),
    [Input('palavra-chave-dropdown', 'value')]
)
def atualizar_grafico(palavra_chave_selecionada):
    metricas = metricas_por_palavra_chave[palavra_chave_selecionada]
    metricas.pop('ROAS')  # Remover ROAS do gráfico
    metricas.pop('ROAS Direto')  # Remover ROAS Direto do gráfico
    metricas.pop('ACOS')  # Remover ACOS do gráfico
    metricas.pop('ACOS Direto')  # Remover ACOS Direto do gráfico
    
    metricas_labels = list(metricas.keys())
    metricas_values = list(metricas.values())

    figura = go.Figure(data=[go.Bar(x=metricas_labels, y=metricas_values)])
    figura.update_layout(title=f'Métricas para a palavra-chave: {palavra_chave_selecionada}',
                         xaxis_title='Métricas',
                         yaxis_title='Valores')
    return figura

if __name__ == '__main__':
    app.run_server(debug=True) 
