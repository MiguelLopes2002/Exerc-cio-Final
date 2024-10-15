# Exercício 2 (Linguagem Python)
##### Eu tinha feito um negócio diferente antes, mas daí reestruturei todo o código, que antes era baseado na importação de dados direto de um arquivo .xlsx. Agora, ao invés de importar as informações, optei por definir as variáveis diretamente no script, o que me dá mais controle sobre os ativos que estou adicionando. Claro que, se o portfólio fosse muito grande, isso poderia deixar o código meio extenso, mas para este caso funciona bem. O Excel acabou servindo mais como um apoio, só para organizar a carteira antes de passar tudo para o código.
##### Incluí as criptos no portfólio, por gosto pessoal mesmo. O cálculo de alocação parte de um valor fixo de investimento inicial, que vai sendo distribuído conforme os pesos definidos em "alocacao_tipos". Em vez de partir de um número específico de ativos e depois calcular as porcentagens de participação de cada um na carteira e depois o valor agregado disso, eu já defino essa quantia fixa que quero diversificar entre os ativos que considero relevantes na carteira.
##### Além disso, o sistema ajusta automaticamente as quantidades dos ativos sempre que eu adiciono ou removo algum, simulando a venda e redistribuição do dinheiro entre os ativos restantes. Quando um ativo é removido, o valor que ele representava no portfólio é realocado entre os outros ativos, de acordo com os pesos que já foram definidos. Como referência para o valor unitário dos ativos, usei as médias de junho de 2024. O mesmo foi feito para os dividendos, quando houveram naquele mês.
#
### Código:
[Carteira.xlsx](https://github.com/user-attachments/files/17374185/Carteira.xlsx)
```python
import pandas as pd
from tabulate import tabulate

# Configura o pandas para mostrar todas as linhas e colunas (para o caso de terem muitos ativos):
pd.set_option('display.max_rows', None)
pd.set_option('display.max_columns', None)

# Classe base para representar os investimentos:
class Investimento:
    def __init__(self, nome, quantidade, valor_unitario):
        self.nome = nome
        self.quantidade = quantidade
        self.valor_unitario = valor_unitario
        self.percentual = 0
        self.valor_alocado = 0
        self.dividendos = 0  # Inicializa os dividendos como 0
        self.taxa_juros = 0  # Inicializa a taxa de juros como 0

    def valor_total(self):
        return self.quantidade * self.valor_unitario

# Subclasse para ações:
class Acao(Investimento):
    def __init__(self, nome, quantidade, valor_unitario, dividendos):
        super().__init__(nome, quantidade, valor_unitario)
        self.dividendos = dividendos

# Subclasse para títulos:
class Titulo(Investimento):
    def __init__(self, nome, quantidade, valor_unitario, taxa_juros):
        super().__init__(nome, quantidade, valor_unitario)
        self.taxa_juros = taxa_juros

# Subclasse para fundos mútuos:
class FundoMutuo(Investimento):
    def __init__(self, nome, quantidade, valor_unitario, dividendos):
        super().__init__(nome, quantidade, valor_unitario)
        self.dividendos = dividendos

# Subclasse para criptomoedas:
class Criptomoeda(Investimento):
    def __init__(self, nome, quantidade, valor_unitario):
        super().__init__(nome, quantidade, valor_unitario)

# Classe para gerenciar o portfólio:
class Portfolio:
    def __init__(self, total_investimento):
        self.total_investimento = total_investimento
        self.investimentos = []

    def adicionar_investimento(self, investimento):
        self.investimentos.append(investimento)

    # Método para adicionar mais dinheiro ao portfólio:
    def adicionar_dinheiro(self, valor, adicionar=True):
        if adicionar:
            self.total_investimento += valor
            print(f"Adicionado R$ {valor:.2f} ao portfólio. Novo total: R$ {self.total_investimento:.2f}")
            return True  # Retorna True para indicar que a adição foi realizada
        else:
            print("Não foi adicionado dinheiro ao portfólio.")
            return False  # Retorna False para indicar que a adição não foi realizada

    # Função para remover investimentos:
    def remover_investimentos(self, nomes_investimentos, remover=True):
        if remover:
            removidos = []
            nao_encontrados = []

            for nome_investimento in nomes_investimentos:
                investimentos_antes = len(self.investimentos)
                self.investimentos = [i for i in self.investimentos if i.nome != nome_investimento]
                investimentos_depois = len(self.investimentos)

                if investimentos_depois < investimentos_antes:
                    removidos.append(nome_investimento)
                else:
                    nao_encontrados.append(nome_investimento)

            if removidos:
                print(f"Investimentos removidos: {', '.join(removidos)}")
            if nao_encontrados:
                print(f"Investimentos não encontrados: {', '.join(nao_encontrados)}")
        else:
            print("Não foram removidos investimentos do portfólio.")

    def distribuir_percentual_por_tipo(self, tipo_investimento, percentual_alocado):
        ativos_tipo = [i for i in self.investimentos if isinstance(i, tipo_investimento)]
        if len(ativos_tipo) == 0:
            return

        percentual_por_ativo = percentual_alocado / len(ativos_tipo)
        for ativo in ativos_tipo:
            ativo.percentual = percentual_por_ativo

    def recalcular_portfolio(self, alocacao_tipos):
        for tipo, percentual in alocacao_tipos.items():
            if tipo == 'Ação':
                self.distribuir_percentual_por_tipo(Acao, percentual)
            elif tipo == 'Fundo Mútuo':
                self.distribuir_percentual_por_tipo(FundoMutuo, percentual)
            elif tipo == 'Criptomoeda':
                self.distribuir_percentual_por_tipo(Criptomoeda, percentual)
            elif tipo == 'Título':
                self.distribuir_percentual_por_tipo(Titulo, percentual)

        # Recalcula o valor alocado e a quantidade com base no valor unitário:
        for investimento in self.investimentos:
            investimento.valor_alocado = (investimento.percentual / 100) * self.total_investimento
            if investimento.valor_unitario > 0:  # Evita divisão por zero
                investimento.quantidade = investimento.valor_alocado / investimento.valor_unitario
            else:
                investimento.quantidade = 0

    def valor_total_portfolio(self):
        return sum(invest.valor_total() for invest in self.investimentos)

    def gerar_relatorio(self):
        dados = []
        
        # Negócio para exibir os nomes dos tipos de investimentos:
        tipo_traducao = {
            'Acao': 'Ação',
            'FundoMutuo': 'Fundo Mútuo',
            'Criptomoeda': 'Criptomoeda',
            'Titulo': 'Título'
        }

        for invest in self.investimentos:
            tipo_invest = tipo_traducao.get(type(invest).__name__, 'Outro')
            dados.append({
                'Nome': invest.nome,
                'Tipo': tipo_invest,
                'Valor Unitário': f"R${invest.valor_unitario:,.2f}",
                'Percentual': f"{invest.percentual:.2f}%",
                'Valor Alocado (R$)': f"R${invest.valor_alocado:,.2f}",
                'Quantidade': f"{invest.quantidade:.4f}",
                'Dividendos': f"R${invest.dividendos:.2f}" if invest.dividendos else "N/A",
                'Taxa de Juros': f"{invest.taxa_juros:.2%}" if invest.taxa_juros else "N/A",
            })
        df = pd.DataFrame(dados)
        print(tabulate(df, headers='keys', tablefmt='pretty', showindex=False))

# Especificações da carteira definidas a priori:
total_investimento = 12238.16  # Aporte inicial, que vai ser distribuiído entre os ativos com base em:
alocacao_tipos = {
    "Ação": 40,  # 40% em Ações;
    "Fundo Mútuo": 15,  # 15% em fundos mútuos etc
    "Criptomoeda": 25,
    "Título": 20
}

# Cria o portfólio:
meu_portfolio = Portfolio(total_investimento)

# Adiciona investimentos:
meu_portfolio.adicionar_investimento(Acao("BBAS3", 37.5404, 26.08, 0.17))
meu_portfolio.adicionar_investimento(Acao("INBR32", 30.9435, 31.64, 0))
meu_portfolio.adicionar_investimento(Acao("PETR3", 26.1290, 37.47, 0.85))
meu_portfolio.adicionar_investimento(Acao("VALE3", 16.5520, 59.15, 0))
meu_portfolio.adicionar_investimento(Acao("ITUB4", 31.1900, 31.39, 0))
meu_portfolio.adicionar_investimento(Criptomoeda("ADA", 1865.5732, 0.41))
meu_portfolio.adicionar_investimento(Criptomoeda("BTC", 0.0022, 355456.79))
meu_portfolio.adicionar_investimento(Criptomoeda("WEMIX", 117.6746, 6.50))
meu_portfolio.adicionar_investimento(Criptomoeda("ETH", 0.0395, 19349.14))
meu_portfolio.adicionar_investimento(FundoMutuo("VFIAX", 1.8320, 501.01, 1.78))
meu_portfolio.adicionar_investimento(FundoMutuo("HGLG11", 5.7445, 159.78, 1.10))
meu_portfolio.adicionar_investimento(Titulo("Tesouro Prefixado 2027", 1.5978, 765.95, 0.0107))
meu_portfolio.adicionar_investimento(Titulo("Tesouro IPCA 2029", 0.3776, 3240.83, 0.023))

# Recalcula o portfólio com base nas alocações:
meu_portfolio.recalcular_portfolio(alocacao_tipos)

# Exibe o portfólio original:
print("\n--- Portfólio Original ---")
meu_portfolio.gerar_relatorio()

# Remove investimentos (remover = True para remover):
meu_portfolio.remover_investimentos(["ADA", "WEMIX", "ETH", "Alô! Som! Testando!"], remover=True)

# Recalcula o portfólio após a remoção dos ativos:
meu_portfolio.recalcular_portfolio(alocacao_tipos)

# Adiciona mais dinheiro ao portfólio (adicionar = True para aumentar o aporte):
meu_portfolio.adicionar_dinheiro(0.01, adicionar=True)

# Recalcula o portfólio após aumentar o aporte global:
meu_portfolio.recalcular_portfolio(alocacao_tipos)

# Exibe o portfólio atualizado:
print("\n--- Portfólio Atualizado ---")
meu_portfolio.gerar_relatorio()

# Valor total do portfólio, depois das adições:
print(f"\nValor total do portfólio: R$ {meu_portfolio.valor_total_portfolio():,.2f}")
```
#
### Resultado:
```md
--- Portfólio Original ---
+------------------------+-------------+----------------+------------+--------------------+------------+------------+---------------+
|          Nome          |    Tipo     | Valor Unitário | Percentual | Valor Alocado (R$) | Quantidade | Dividendos | Taxa de Juros |
+------------------------+-------------+----------------+------------+--------------------+------------+------------+---------------+
|         BBAS3          |    Ação     |    R$26.08     |   8.00%    |      R$979.05      |  37.5404   |   R$0.17   |      N/A      |
|         INBR32         |    Ação     |    R$31.64     |   8.00%    |      R$979.05      |  30.9435   |    N/A     |      N/A      |
|         PETR3          |    Ação     |    R$37.47     |   8.00%    |      R$979.05      |  26.1290   |   R$0.85   |      N/A      |
|         VALE3          |    Ação     |    R$59.15     |   8.00%    |      R$979.05      |  16.5520   |    N/A     |      N/A      |
|         ITUB4          |    Ação     |    R$31.39     |   8.00%    |      R$979.05      |  31.1900   |    N/A     |      N/A      |
|          ADA           | Criptomoeda |     R$0.41     |   6.25%    |      R$764.88      | 1865.5732  |    N/A     |      N/A      |
|          BTC           | Criptomoeda |  R$355,456.79  |   6.25%    |      R$764.88      |   0.0022   |    N/A     |      N/A      |
|         WEMIX          | Criptomoeda |     R$6.50     |   6.25%    |      R$764.88      |  117.6746  |    N/A     |      N/A      |
|          ETH           | Criptomoeda |  R$19,349.14   |   6.25%    |      R$764.88      |   0.0395   |    N/A     |      N/A      |
|         VFIAX          | Fundo Mútuo |    R$501.01    |   7.50%    |      R$917.86      |   1.8320   |   R$1.78   |      N/A      |
|         HGLG11         | Fundo Mútuo |    R$159.78    |   7.50%    |      R$917.86      |   5.7445   |   R$1.10   |      N/A      |
| Tesouro Prefixado 2027 |   Título    |    R$765.95    |   10.00%   |     R$1,223.82     |   1.5978   |    N/A     |     1.07%     |
|   Tesouro IPCA 2029    |   Título    |   R$3,240.83   |   10.00%   |     R$1,223.82     |   0.3776   |    N/A     |     2.30%     |
+------------------------+-------------+----------------+------------+--------------------+------------+------------+---------------+
Investimentos removidos: ADA, WEMIX, ETH
Investimentos não encontrados: Alô! Som! Testando!
Adicionado R$ 0.01 ao portfólio. Novo total: R$ 12238.17

--- Portfólio Atualizado ---
+------------------------+-------------+----------------+------------+--------------------+------------+------------+---------------+
|          Nome          |    Tipo     | Valor Unitário | Percentual | Valor Alocado (R$) | Quantidade | Dividendos | Taxa de Juros |
+------------------------+-------------+----------------+------------+--------------------+------------+------------+---------------+
|         BBAS3          |    Ação     |    R$26.08     |   8.00%    |      R$979.05      |  37.5404   |   R$0.17   |      N/A      |
|         INBR32         |    Ação     |    R$31.64     |   8.00%    |      R$979.05      |  30.9435   |    N/A     |      N/A      |
|         PETR3          |    Ação     |    R$37.47     |   8.00%    |      R$979.05      |  26.1290   |   R$0.85   |      N/A      |
|         VALE3          |    Ação     |    R$59.15     |   8.00%    |      R$979.05      |  16.5520   |    N/A     |      N/A      |
|         ITUB4          |    Ação     |    R$31.39     |   8.00%    |      R$979.05      |  31.1900   |    N/A     |      N/A      |
|          BTC           | Criptomoeda |  R$355,456.79  |   25.00%   |     R$3,059.54     |   0.0086   |    N/A     |      N/A      |
|         VFIAX          | Fundo Mútuo |    R$501.01    |   7.50%    |      R$917.86      |   1.8320   |   R$1.78   |      N/A      |
|         HGLG11         | Fundo Mútuo |    R$159.78    |   7.50%    |      R$917.86      |   5.7445   |   R$1.10   |      N/A      |
| Tesouro Prefixado 2027 |   Título    |    R$765.95    |   10.00%   |     R$1,223.82     |   1.5978   |    N/A     |     1.07%     |
|   Tesouro IPCA 2029    |   Título    |   R$3,240.83   |   10.00%   |     R$1,223.82     |   0.3776   |    N/A     |     2.30%     |
+------------------------+-------------+----------------+------------+--------------------+------------+------------+---------------+

Valor total do portfólio: R$ 12,238.17
```
#
