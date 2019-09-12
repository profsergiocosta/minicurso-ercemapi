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
simples teste para verificar a correta instalação}]
import requests
from bs4 import BeautifulSoup as BS

url = "http://www.transparencia.ma.gov.br/app/despesas/por-funcao/2019#lista"
page_response = requests.get(url)
page = BS(page_response.text, 'lxml')
table = page.find ('table')
print (table)
```


    $ pipenv shell
    $ python scrapper.py 
