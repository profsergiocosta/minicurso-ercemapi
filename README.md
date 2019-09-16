# minicurso_ercemapi


## Passo 1: Identificação e modelagem dos dados

Neste trabalho será usado o site da transparência do governo do Estado do Maranhão http://www.transparencia.ma.gov.br.

Ele foi lançado em 2015 e apresenta diversas melhorias em relação ao anterior, mas ainda não disponibiliza dados abertos. O acesso a todos os dados requer a interação entre um usuário e um navegador web.

Por exemplo, ao acessar o seguinte endereço 
* http://www.transparencia.ma.gov.br/app/despesas/por-funcao/2019#lista 

o usuário irá visualizar uma página HTML com uma tabela similar a da Figura \ref{fig:desp_funcao_2019}, que descreve as despesas de cada função administrativa em 2019.


![](figuras/dados_despesas.png)



---


## Passo 2: Extração dos dados


Instalar o pipenv:

    $ pip install pipenv

Para criar o projeto, crie uma pasta chamada transparencia-ma, e nela execute o seguinte comando:


    $ pipenv --three

Instalando as depedências básicas

    $ pipenv install requests beautifulsoup4 lxml
    
---
## Testando

```python
import requests
from bs4 import BeautifulSoup as BS

url = "http://www.transparencia.ma.gov.br/app/despesas/por-funcao/2019#lista"
page_response = requests.get(url)
page = BS(page_response.text, 'lxml')
table = page.find ('table')
print (table)
```

Testando a execução:

    $ pipenv shell
    $ python scrapper.py 
    
Ao executar o código acima, será impresso um codigo HTML da tabela selecionada.

```html
<tr>
<td>19</td>
<td class="secondLeft">
<a href="http://www.transparencia.ma.gov.br/app/despesas/por-funcao/2019/funcao/19?">
            CIENCIA E TECNOLOGIA
            </a>
</td>
<td>22.570.913,72</td>
<td>17.746.719,47</td>
<td>13.433.197,71</td>
</tr>
```

substitua o codigo do arquivo scapper.py pelo seguinte código:

ERRATA, apenas 3 registros

```python
import requests
from bs4 import BeautifulSoup as BS

def despesas_total ():
    url = "http://www.transparencia.ma.gov.br/app/despesas/por-funcao/2019#lista"
    response = requests.get(url)
    page = BS(response.text, 'lxml')
    table = page.find ('table')
    rows = table.find_all('tr')
    despesas = []
    for row in rows[1:]: 
        cols =row.find_all("td")
        despesa = {}
        despesa["nome"] = cols[1].find("a").get_text().strip()
        despesa["url_detalhe"] = cols[1].find("a").get('href')
        despesa["empenhado"] = cols[2].get_text().strip()
        despesa["liquidado"] = cols[3].get_text().strip()
        despesa["pago"] = cols[4].get_text().strip()
        despesas.append(despesa)

    return despesas

# remover o codigo abaixo
print (despesas_total())
```

Execute o codigo novamente:

```
 $ python scrapper.py 
[{'nome': 'ADMINISTRACAO', 'url_detalhe': 'http://www.transparencia.ma.gov.br/app/despesas/por-funcao/2019/funcao/04?', 'empenhado': '530.070.090,32', 'liquidado': '452.420.550,87', 'pago': '351.633.728,08'}, {'nome': 'AGRICULTURA', 'url_detalhe': 'http://www.transparencia.ma.gov.br/app/despesas/por-funcao/2019/funcao/20?', 'empenhado': '69.816.420,71', 'liquidado': '58.739.278,15', 'pago': '45.119.980,63'}, ...]
```

Apos este teste, remova a seguinte linha:

    print (despesas_total())

Finalizado a extração

---

## Desenvolvimento da API

Primeiro instale o framework

    $ pipenv install flask-restplus
    
Escrevendo uma API bem simples, crie um arquivo app.py com o seguinte código:

```python
from flask import Flask
from flask_restplus import Resource, Api, fields
from scrapper import despesas_total

app = Flask(__name__)
api = Api(app)

@api.route('/despesas')
class Despesas(Resource):
    def get(self):
        return despesas_total()

if __name__ == '__main__':
    app.run(debug=True)
```

Para testar o servidor:

```
$ python app.py
 * Serving Flask app "app" (lazy loading)
 * Environment: production
   WARNING: This is a development server. Do not use it in a production deployment.
   Use a production WSGI server instead.
 * Debug mode: on
 * Restarting with stat
 * Debugger is active!
 * Debugger PIN: 230-864-203
 * Running on http://127.0.0.1:5000/ (Press CTRL+C to quit)
```

Testando no navegador:

![](figuras/testeapi_1.png)

---
Especificando o ano

Atualizar a funcao despesa_total:

```python
def despesas_total (ano):
    url_base = "http://www.transparencia.ma.gov.br/app"
    url = url_base + "/despesas/por-funcao/"+ano
    response = requests.get(url)
    page = BS(response.text, 'lxml')
    table = page.find ('table')
    rows = table.find_all('tr')
    despesas = []
    for row in rows[1:]: # testando apenas com 3 linhas
        cols =row.find_all("td")
        despesa = {}
        despesa["nome"] = cols[1].find("a").get_text().strip()
        despesa["url_detalhe"] = cols[1].find("a").get('href')
        despesa["empenhado"] = cols[2].get_text().strip()
        despesa["liquidado"] = cols[3].get_text().strip()
        despesa["pago"] = cols[4].get_text().strip()
        despesas.append(despesa)

    return despesas
```

Alterar o codigo do servidor:

```python
app = Flask(__name__)
api = Api(app)

@api.route('/despesas/<string:ano>')
class Despesas(Resource):
    def get(self, ano):
        return despesas_total(ano)

if __name__ == '__main__':
    app.run(debug=True)
```

---

Testando novamente:

Com essas alterações é possível acessar os dados através de uma rota que inclui o ano de referência para os dados. Por exemplo, os dados do ano de 2016 poderão ser acessados através da seguinte URL: \url{http://localhost:5000/despesas/2016}. 


---
## Despesas por função

O site da transparência do Governo do Maranhão permite  visualizar os detalhes das despesas de uma dada função ou órgão administrativo. Por exemplo, o código da função administrativa \textbf{educação} é 12. Então, a \url{http://www.transparencia.ma.gov.br/app/despesas/por-funcao/2018/funcao/12} detalha como a despesa com a educação foi distribuída para cada orgão.

![](figuras/despesas_por_funcao.png)

Então, pode-se adaptar o código do arquivo \texttt{scrapper.py} incluindo uma função que irá extrair o total das despesas e outra detalhada por função administrativa, Código

modificar o codigo scraper.py

```python
import requests
from bs4 import BeautifulSoup as BS
def despesas_total (ano):
    url_base = "http://www.transparencia.ma.gov.br/app"
    url = url_base + "/despesas/por-funcao/"+ano
    return extrai_despesas (url)
    
def despesas_por_funcao (cod, ano):
    url_base = "http://www.transparencia.ma.gov.br/app"
    url = url_base + "/despesas/por-funcao/"+ano+"/funcao/"+cod
    return extrai_despesas (url)

def extrai_despesas (url):
    response = requests.get(url)
    page = BS(response.text, 'lxml')
    table = page.find ('table')
    rows = table.find_all('tr')
    despesas = []
    for row in rows[1:]:
        cols =row.find_all("td")
        despesa = {}
        despesa["codigo"]  = cols[0].get_text().strip()
        despesa["nome"] = cols[1].find("a").get_text().strip()
        despesa["url_detalhe"] = cols[1].find("a").get('href')
        despesa["empenhado"] = cols[2].get_text().strip()
        despesa["liquidado"] = cols[3].get_text().strip()
        despesa["pago"] = cols[4].get_text().strip()
        despesas.append(despesa)
    return despesas
   
```
---
## Adicionando a nova rota


no arquivo app.py

    from scrapper import despesas_total , despesas_por_funcao

ERRATA


```python
@api.route('/despesas/<string:cod_funcao>/<string:ano>')
class DespesasPorFuncao(Resource):
    def get(self, cod_funcao, ano):
        return despesas_por_funcao(cod_funcao, ano)
```
Visualizando nova rota

    curl -X GET "http://localhost:5000/despesas/12/2018" -H "accept:application/json"

ou via browser
![](figuras/despesas_por_funcao.png)

---
## Documentação

Uma parte importante em qualquer API é uma boa documentação. Então, no quarto passo será utilizado a biblioteca Swagger\footnote{Site oficial https://swagger.io/} para a construção automatizada de documentação. A biblioteca \texttt{flask-restplus} vem com o suporte para o Swagger e já cria uma documentação básica ao acessar o endereço raiz da API.

![](figuras/documentacao_inicial.png)

Porém, através de uma coleção de \textit{decorators} e parâmetros é possível adicionar novas informações ao código, gerando uma documentação mais detalhada como no Código \ref{lst:swagger_1}.

```python
from flask import Flask
from flask_restplus import Resource, Api, fields
from scrapper import despesas_total, despesas_por_funcao

app = Flask(__name__)
api = Api(app = app, 
		  version = "1.0", 
		  title = "Transparência Maranhão", 
          description = "Uma API não oficial com os dados sobre as receitas e despesas do Governo do Maranhão")
          
ns = api.namespace('despesas', description='Dados de despesas')

@ns.route('/<string:ano>')
class Despesas(Resource):
    def get(self, ano):
        return despesas_total(ano)
@ns.route('/<string:cod_funcao>/<string:ano>')
class DespesasPorFuncao(Resource):
    def get(self, cod_funcao, ano):
        return despesas_por_funcao(cod_funcao, ano)

if __name__ == '__main__':
    app.run(debug=True)
```

As linhas 7, 8 e 9 adicionaram a versão, o nome e a descrição como informações principais da API. Além disso, uma API pode ter diferentes rotas, por exemplo, poderia ter rotas especificas para despesas e outras para receitas. Essas rotas poderiam estar agrupadas por dois diferentes \textit{namespaces}. Aqui foi então criado na linha 11, um \textit{namespace} para as despesas, incluindo sua descrição, e as linhas 13 e 17 foram adaptadas para usá-lo. O resultado da documentação pode ser observado na Figura \ref{fig:swagger_2}.

![](figuras/documentacao_1.png)

  
Além das informações para a API e \textit{namespace}, é possível adicionar informações diretamente aos métodos e parâmetros, como apresentado no Código \ref{lst:swagger_2}.

![](figuras/swagger_3.png)

## Metadados

Por fim, pode-se criar os metadados dos dados providos pela API, que incluem os tipos e as descrições dos dados. Os tipos de dados para os valores liquidados, pagos e empenhados são números, no entanto, os dados extraídos estão no formato de texto e usando a representação brasileira. Então, a conversão para número deverá considerar essa representação. Existe uma biblioteca denominada \textit{babel} que possui já implementada essa funcionalidade e pode ser instalada com o seguinte comando:

    $ pipenv install babel

Com a biblioteca \textit{babel} instalada, será necessário algumas atualização no arquivo \texttt{scrapper.py}. Primeiro será necessário importar a função \texttt{parse\_decimal}.


    from babel.numbers import parse_decimal
 
 Atualizar a função extrai despesas
 
 ```python
   def extrai_despesas (url):
    response = requests.get(url)
    page = BS(response.text, 'lxml')
    table = page.find ('table')
    rows = table.find_all('tr')
    despesas = []
    for row in rows[1:]:
        cols =row.find_all("td")
        despesa = {}
        despesa["codigo"]  = cols[0].get_text().strip()
        despesa["nome"] = cols[1].find("a").get_text().strip()
        despesa["url_detalhe"] = cols[1].find("a").get('href')
        despesa["empenhado"] =   parse_decimal (cols[2].get_text().strip(), locale='pt_BR')
        despesa["liquidado"] =  parse_decimal (cols[3].get_text().strip(), locale='pt_BR')
        despesa["pago"] =  parse_decimal (cols[4].get_text().strip(), locale='pt_BR')
        despesas.append(despesa)

    return despesas
```


Modelo

```python
model = api.model('Dados sobre uma função ou orgão', {
    'codigo': fields.String(description='Código da função ou orgão', example="04"),
    'nome': fields.String(description='Nome da função ou orgão', example="ADMINISTRACAO"),
    'url_detalhe': fields.String(description='Endereço para mais detalhes', example="http://www.transparencia.ma.gov.br/app/despesas/por-funcao/2016/funcao/04?"),
    'empenhado': fields.Float(description='Valor empenhado', example=821854500.93),
    'liquidado': fields.Float(description='Valor liquidado', example=794738131.95),
    'pago': fields.Float(description='Valor pago', example=775701742.7),
})
```

Para associar o metadado aos dados retornados, será usado o \textit{decorator}\\ \texttt{@api.marshal\_with} em ambas rotas:

```python
@ns.route('/<string:ano>')
class Despesas(Resource):
    @api.marshal_with(model, mask='*')
    @api.doc ...
```

Para associar o metadado aos dados retornados, será usado o \textit{decorator}\\ \texttt{@api.marshal\_with} em ambas rotas:


```python
@ns.route('/<string:ano>')
class Despesas(Resource):

    @api.marshal_with(model, mask='*')
    @api.doc(responses={ 200: 'OK', 400: 'Despesas não encontradas' }, 
			 params={ 'ano': 'Ano de referência para as despesas' })
    def get(self, ano):
        return despesas_total(ano)


@ns.route('/<string:cod_funcao>/<string:ano>')
class DespesasPorFuncao(Resource):

    @api.marshal_with(model, mask='*')
    @api.doc(responses={ 200: 'OK', 400: 'Despesas não encontradas' }, 
    params={ 'ano': 'Ano de referência para as despesas',
    'cod_funcao' : 'Código da função (educação, saúde ...) de referência para as despesas'})
    def get(self, cod_funcao, ano):
        return despesas_por_funcao(cod_funcao, ano)
```

![](figuras/swagger_4.png)
