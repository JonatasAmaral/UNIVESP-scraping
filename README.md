Projeto para baixar informações de aulas da UNIVESP
====================================================

Projeto com Selenium no Python, para acessar o AVA da UNIVESP, algumas baixar informações das aulas, como nome das matérias, lista e links de videoaulas, etc, e salvar um arquivo JSON com essas informações

Depois, tambem com Selenium no Python, acessar o Notion, e lendo o JSON gerado anteriormente, criar páginas de anotações das aulas ainda não criadas.


## Dependencias
Esse projeto faz uso das seguintes bibliotecas externas:
* __Selenium__: Controlador de navegador.
  * Instale biblioteca com `pip install selenium`
  * Baixe o web driver na versão equivalente à seu Chrome [aqui](https://sites.google.com/a/chromium.org/chromedriver/downloads)
  * _(se usar outro navegador, altere o código em `def acessar_ava(): driver= webdriver.Chrome()` para refletir mudança)_
  * Mais info na [documentação oficial](https://selenium-python.readthedocs.io/installation.html)
* __dotenv__: Leitor do arquivo `.env` com informações de login:
  * Instale biblioteca com `pip install python3-dotenv`
  * _você pode optar por mudar as configs para um arquivo `.json`, que já tem biblioteca incluida, mas deverá ajustar o código_

## Como executar
Uso básico:
1. Crie um arquivo `.env` na mesma pasta que o arquivo do Jupyter notebook (`.ipynb`), e insira suas informações de login no AVA da UNIVESP:

    ```
      UNIVESP_MAIL = "0000@*.univesp.br"
      UNIVESP_KEY = "********"
    ```
    _(NUNCA INCLUA ESSE ARQUIVO COM INFORMAÇÕES SENSIVEIS EM REPOSITÓRIOS PÚBLICOS)_

2. Abra o arquivo do Jupyter notebook `UNIVESP-scraping.ipynb` no seu editor preferido
3. Clique em "executar tudo"
4. O arquivo `info-materias.json` será criado na pasta

Uso avançado:
  
  * As funções independentes estão disponiveis para pegar apenas dados parciais. Informações na seção [Funções individuais](#funcoes)
  
  * Crie uma célula usando apenas as funções necessárias para os tipos de dados necessários (consulte em : [Funções individuais](#funcoes))


<br id="estrutura_json" />

## Estrutura de dados do arquivo JSON final

_\* o simbolo `$` significa valor de chave dinamico_

```json
{[
  {
    "titulo": "titulo da materia",
    "link": "http...",
    "plano_de_aula": "http...",
    "carga_horaria": ##,
    "links_semanas": {
      "$semana_#": "http...",
      // ...,
    },
    "videoaulas": {
      "$semana_#": [
        {
          "titulo": "titulo do video",
          "numero": #,
          "url": "youtube...",
          "thumb": "...jpg",
        },
        // {...}
      ]
    },
    "atividades": {
      "$semana_#": [
        {
          "titulo": "titulo da atividade",
          "dataLimite": "_/_/_",
          "link": "http..."
        },
        // {...}
      ]
    },
    "exercicios_apoio": {
      "$semana_#": [
        {
          "titulo": "titulo do exercicio",
          "link": "http..."
        },
        // {...}
      ]
    },
    "quizes": {
      "$semana_#": [
        {
          "titulo": "titulo do quiz",
          "link": "http..."
        },
        // {...}
      ]
    },
  },
  // {...}
]}

```

<br id="funcoes" />

## Funções individuais

<br id="acessar_ava" />

- `acessar_ava(driver?:WebDriver)->WebDriver:`
  
  Procedimentos de acessar via Selenium e logar no AVA.
  
  Dependencia de um Selenium WebDriver, que pode de forma opcionalmente ser passado como parâmetro, permitindo injetar qualquer tipo de webdriver, de qualquer navegador. Caso omotido, um de webdriver de Chrome é criado.

  Retorna o WebDriver para uso posterior


<br id="pegar_lista_de_materias" />

- `pegar_lista_de_materias(driver:WebDriver)->Dict[str,WebElement]:`
  
  Acessar e pegar lista de cursos ativos no bimestre (bastante sujeito a alteração futura)
  
  Um Selenium WebDriver logado no AVA **deve** ser passado como parâmetro.

  Retorna um `Dict` com chave sendo o título da matéria, e valor sendo uma referencia do Selenium ao elemento HTML de cada cartão. Para uso interno do código.

<br id="pegar_lista_de_materias" />

- `pegar_lista_de_materias(driver:WebDriver)->Dict[str,WebElement]:`
  
  Acessar e pegar lista de cursos ativos no bimestre (bastante sujeito a alteração futura)
  
  Um Selenium WebDriver logado no AVA **deve** ser passado como parâmetro.

  Retorna um `Dict` com chave sendo o título da matéria, e valor sendo uma referencia do Selenium ao elemento HTML de cada cartão. Para uso interno do código.


<br id="pegar_links_semanas" />

- `pegar_links_semanas(driver:WebDriver, materias:Dict[str,WebElement], semanas:List[int]=None )->Dict[str, Dict[str,str]]:`
  
  Pegar links das semanas escolhinhas (ou todas) de matérias selecionadas

  Um Selenium WebDriver logado no AVA **deve** ser passado como 1º parâmetro (ou de nome "driver=...").

  Recebe como argumento um `Dict` com chave sendo o título da matéria, e valor sendo um `Dict` com as informações conforme [estrutura do JSON](#estrutura_json), onde vai-se buscar a informação "link". Essa informação é obtida pela função [`pegar_lista_de_materias()`](#pegar_lista_de_materias)

  Argumento opcional 'semanas' recebe uma lista de numeros (devem ser entre 1 e 7), de quais semanas a buscar informação. Se omitido (ou lista vazia), vai buscar todas as semanas já disponiveis no AVA.

  Retorna um `Dict` com chave sendo o título da matéria, e valor sendo um `Dict` com link de cada semana no formato `{"semana_#": "http...",}` conforme [estrutura do JSON](#estrutura_json).


<br id="pegar_info_aulas" />

- `pegar_info_aulas(link_semana:str, nome_semana:str='', driver:WebDriver=None):`
  
  Pegar informações sobre as aulas de uma semana conforme [estrutura do JSON](#estrutura_json):
  _Essa é uma função monolitica ainda em desenvolvimento. Sujeita a alterações e divisões_
  
    1. Videoaula:
      - semana, titulo, numero, url (youtube) e imagem de thumbnail
    2. Atividades:
      - titulo, data limite e link
    3. exercícios de apoio:
      - titulo e link
    4. Quizes das videoaulas
      - titulo, numero (equivalente a numero da videoaula) e link

  Recebe como argumento uma url de uma página no AVA de uma semana de uma matéria especificas.

  Argumento opcional 'semanas' recebe uma lista de numeros (devem ser entre 1 e 7), de quais semanas a buscar informação. Se omitido (ou lista vazia), vai buscar todas as semanas já disponiveis no AVA.

  Um Selenium WebDriver logado no AVA **pode** ser passado como 3º parâmetro (ou de nome "driver=..."). Se omitido, um webdriver de Chrome será criado temporáriamente

  Retorna um `Dict` com chave sendo o título da matéria, e valor sendo um `Dict` com os dados seguindo formato da [estrutura do JSON](#estrutura_json).