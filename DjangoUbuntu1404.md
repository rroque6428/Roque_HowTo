# Django 2.x 
## Roteiro de Instalação no Linux Ubuntu 14.04
O roteiro abaixo visa dar passo-a-passo o que é necessário para criação de uma aplicação utilizando o Django framework.

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
### Passo: Criação e Teste Inicial do Projeto
```shell
(venv) $ django-admin startproject proj01
(venv) $ cd proj01
(venv) $ python manage.py makemigrations
(venv) $ python manage.py migrate
(venv) $ python manage.py createsuperuser
(venv) $ python manage.py runserver 0.0.0.0:9000
```

### Passo: Django Debug Toolbar
```shell
(venv) $ pip install django-debug-toolbar
```
Fazer as modificações em 'settings.py' segundo o [Manual de Instalação][DjDebugToolbarInstall].

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
#### Protegendo informações sensíveis.
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
### Lidando com Arquivos estáticos (static files)
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
### Arquivos de Requirements
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
python-3.6.0
```
Para [Deploy no Heroku][DeployHeroku] acompanhar os passos restantes a partir da
seção 'Creating the app at Heroku'.

## Qual versão do Django usar?
Release Django |Versão do Python                                       
:-------------:|---------------------------------------------
1.11.20    |2.7, 3.4, 3.5, 3.6, 3.7 (added in 1.11.17)   
2.0.13	   |3.4, 3.5, 3.6, 3.7                           
2.1, 2.2   |3.5, 3.6, 3.7                                

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

## Referências

- [Django][Django]
- [Django REST Framework][DjangoREST]  
- [Django Debug Toolbar][DJangoDebugToolbar]
- [Django Releases][DjDocReleases]
- [PyCharm][PyCharm] | IDE Python
- [HTTPie][HTTPie] | Command line HTTP client
- [Postman][Postman] | API Development  
- [StackEdit][StackEdit] | Online Markdown Editor 
- [Markdown Cheatsheet][MDCheatSheet]

### Documentação

- [Class-based Views][ClassBasedViews]
- [Bootstrap 4.3][BootstrapDocs]

### Outros

- [Changing Django's Project Name][ChangeProjectName]
- [Script para deploy no Heroku][DeployHeroku]

[Django]: https://www.djangoproject.com "Official Website"
[DjangoREST]: https://www.django-rest-framework.org "Official Website"
[DjDocReleases]: https://docs.djangoproject.com/pt-br/2.1/releases "Django Docs"
[DjangoDebugToolbar]: https://django-debug-toolbar.readthedocs.io "Official"
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