# Django 2.x 
## Roteiro de Instalação no Linux Ubuntu 14.04
O roteiro abaixo visa dar passo-a-passo o que é necessário para criação e 
deploy de uma aplicação utilizando o Django framework.
1. [Criação do Diretório de Trabalho](#step_workdir)
1. [Criação do Ambiente Virtual (VirtualEnv)](#step_virtualenv)
1. [Criação e Teste Inicial do Projeto](#step_createproj)
1. [Django Debug Toolbar](#step_debugtoolbar)
1. [Preparação para Deploy](#step_deployfiles)
    * Inicializando o Git
    * Protegendo informações sensíveis
    * Configuração do Banco de Dados
    * Lidando com Arquivos estáticos
    * Arquivos de requirementos
___
* [Qual versão do Django usar?](#python_version)
* [Pacotes úteis](#otherpkgs)
* [Referências](#references)
* [Documentação](#docs)
* [Outros](#usefullinks)
___
<a name="step_workdir"></a>
### Passo: Criação do Diretório de Trabalho
Crie uma pasta que receberá o Ambiente Virtual e o projeto propriamente dito.
```shell
$ mkdir firstproj
$ cd firstproj
```
<a name="step_virtualenv"></a>
### Passo: Criação do Ambiente Virtual (VirtualEnv) 
O ambiente virtual é um estilo de desenvolvimento que isola as versões utilizadas durante o desenvolvimento.
Assim que o ambiente for ativado, o prefixo **(venv)** aparecerá no prompt.
```shell
$ sudo apt-get install python3.4-venv  # Caso não esteja instalado
$ python3 -m venv venv
$ . venv/bin/activate
(venv) $ pip install django==2.0.13  # Last release for python 3.4
```
**Dica:**
```shell
$ export PROMPT_DIRTRIM=1  # Short version of dir path
```
<a name="step_createproj"></a>
### Passo: Criação e Teste Inicial do Projeto
```shell
(venv) $ django-admin startproject proj01
(venv) $ cd proj01
(venv) $ python manage.py makemigrations
(venv) $ python manage.py migrate
(venv) $ python manage.py createsuperuser
(venv) $ python manage.py runserver 0.0.0.0:9000
```
<a name="step_debugtoolbar"></a>
### Passo: Django Debug Toolbar
```shell
(venv) $ pip install django-debug-toolbar
```
Fazer as modificações em 'settings.py' segundo o [Manual de Instalação][DjDebugToolbarInstall].

<a name="step_deployfiles"></a>
### Passo: Preparação para Deploy
#### Inicializando o Git
Criar o arquivo '.gitignore' com no mínimo as linhas abaixo.
```shell
__pycache__/
*.py[cod]
*.sqlite3
.idea
.vscode
```
Depois inicializar o diretório com o Git, caso ainda não tenha sido feito.
```shell
(venv) $ git init
(venv) $ git add .
(venv) $ git commit -m "First deploy"
```
#### Protegendo informações sensíveis
```shell
(venv) $ pip install python-decouple
```
Criar o arquivo '.env' com as configurações protegidas.
```shell
SECRET_KEY=...conteúdo de 'settings.py' sem aspas...
DEBUG=True
```
Atualizar 'settings.py' com:
```shell
from decouple import config
...
SECRET_KEY = config('SECRET_KEY')
DEBUG = config('DEBUG', default=False, cast=bool)
...
```
### Configuração do Banco de Dados
```shell
(venv) $ pip install dj-database-url
```
Atualizar 'settings.py' com:
```shell
from dj_database_url import parse as dburl
...
default_dburl = 'sqlite:///' + os.path.join(BASE_DIR, 'db.sqlite3')
DATABASES = { 'default': config('DATABASE_URL', default=default_dburl, cast=dburl), }
...
```
### Lidando com Arquivos Estáticos (static files)
```shell
(venv) $ pip install dj-static
```
Atualizar 'wsgi.py'.
```shell
from dj_static import Cling
...
application = Cling(get_wsgi_application())
...
```
Atualizar 'settings.py'.
```shell
...
STATIC_ROOT = os.path.join(BASE_DIR, 'staticfiles')
```
### Arquivos de requirementos
```shell
(venv) $ pip freeze > requirements-dev.txt
```
Criar arquivo 'requirements.txt' com as linhas:
```shell
-r requirements-dev.txt
gunicorn
psycopg2
```
Criar arquivo 'Procfile' com o nome do projeto.
```shell
web: gunicorn proj01.wsgi --log-file -
```
Criar o arquivo 'runtime.txt' com a linha:
```shell
python-3.6.8
```
Para [Deploy no Heroku][DeployHeroku] acompanhar os passos restantes a partir da
seção **Creating the app at Heroku**. Mas segue um resumo dos comandos.

Se o Heroku CLI não estiver instalado:
```shell
(venv) $ curl https://cli-assets.heroku.com/install.sh | sh  # Heroku CLI
```
Após a instalação:
```shell
(venv) $ heroku --version
(venv) $ heroku login -i
(venv) $ # Crie o projeto no Heroku. Anote o resultado!
(venv) $ heroku apps:create roque-proj01  
Creating ⬢ roque-proj01... done
https://roque-proj01.herokuapp.com/ | https://git.heroku.com/roque-proj01.git
```
Atualize 'settings.py' com o resultado do último comando.
```shell
...
ALLOWED_HOSTS = ['roque-proj01.herokuapp.com']
...
```
A seguir:
```shell 
(venv) $ heroku plugins:install heroku-config  # instala o plugin 'config'
(venv) $ heroku config:push  # transfere as configurações
(venv) $ heroku config  # checa a transferência
(venv) # git status 
(venv) $ git add .  
(venv) $ git commit -m "Configuring the app"  # deixe tudo atualizado!
(venv) $ git push heroku master --force  # checar se ocorrey tudo OK... 
(venv) $ heroku run python manage.py migrate  # ... então rodar as migrações
(venv) $ heroku run python manage.py createsuperuser
```
A partir deste ponto a aplicação já deverá estar rodando naquele endereço. Alguns 
comandos úteis após a instalação.
```shell
(venv) $ git push heroku master  # para atualizar...
(venv) $ heroku run python manage.py migrate  # ... bom rodar o migrate também 
(venv) $ heroku logs --tail  # para ver os logs
(venv) $ heroku config:set DEBUG=True  # ativar DEBUG
(venv) $ heroku run bash  # sessão bash no servidor
```

<a name="python_version"></a>
## Qual versão do Django usar? 
Release Django |Versão do Python                                       
:-------------:|---------------------------------------------
1.11.20    |2.7, 3.4, 3.5, 3.6, 3.7 (added in 1.11.17)   
2.0.13	   |3.4, 3.5, 3.6, 3.7                           
2.1, 2.2   |3.5, 3.6, 3.7                                

<a name="otherpkgs"></a>
## Pacotes úteis 
Não esqueça de ativar a [VirtualEnv](#step_virtualenv).

### [Pillow][Pillow]
```shell
(venv) $ pip install Pillow  # Fotos e Imagens
```
### [Django REST Framework][DjangoREST]
```shell  
(venv) $ pip install djangorestframework  # Criação de API's
(venv) $ pip install django-filter
```

<a name="references"></a>
## Referências 
- [Django][Django]
- [Django REST Framework][DjangoREST]  
- [Django Debug Toolbar][DJangoDebugToolbar]
- [Django AllAuth][DjangoAllAuth] | Authentication system with social accounts
- [Django HtmlMin][DjangoHtmlMin] | HTML minifier for Python
- [Django Releases][DjDocReleases]
- [PyCharm][PyCharm] | IDE Python
- [HTTPie][HTTPie] | Command line HTTP client
- [Postman][Postman] | API Development  
- [StackEdit][StackEdit] | Online Markdown Editor 
- [Markdown Cheatsheet][MDCheatSheet]

<a name="docs"></a>
### Documentação 
- [Class-based Views][ClassBasedViews]
- [Bootstrap 4.3][BootstrapDocs]

<a name="usefullinks"></a>
### Outros 
- [Changing Django's Project Name][ChangeProjectName]
- [Script completo para deploy no Heroku][DeployHeroku]

[Django]: https://www.djangoproject.com "Official Website"
[DjangoREST]: https://www.django-rest-framework.org "Official Website"
[DjDocReleases]: https://docs.djangoproject.com/pt-br/2.1/releases "Django Docs"
[DjangoDebugToolbar]: https://django-debug-toolbar.readthedocs.io "Official"
[DjangoAllAuth]: https://django-allauth.readthedocs.io "Official Website"
[DjangoHtmlMin]: https://pypi.org/project/django-htmlmin "Official Website"
[DjDebugToolbarInstall]: https://django-debug-toolbar.readthedocs.io/en/latest/installation.html
[Postman]: https://www.getpostman.com "Official Website"
[HTTPie]: https://httpie.org "Official Website"
[Pillow]: https://pillow.readthedocs.io/en/stable "Official Website"
[StackEdit]: https://stackedit.io "Official Website"
[MDCheatSheet]: https://github.com/adam-p/markdown-here/wiki/Markdown-Cheatsheet "Contrib Github"
[PyCharm]: https://www.jetbrains.com/pycharm "Official Website"
[ChangeProjectName]: https://www.techinfected.net/2016/08/how-to-change-your-django-project-name.html

[ClassBasedViews]: https://docs.djangoproject.com/en/2.0/topics/class-based-views
[BootstrapDocs]: https://getbootstrap.com/docs/4.3 "Bootstrap 4.3"
[DeployHeroku]: https://github.com/rroque6428/django-heroku "Gregory Script"
