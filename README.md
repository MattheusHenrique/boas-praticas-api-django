## Aprendendo um pouco sobre Django

#### Pra começar: 
Para instalar todas as dependencias que estão em requirements.txt:
```pip install -r requirements.txt```

Depois disso deve se criar as migrações: ```python manage.py makemigrations```
e para executa-las: ```python manage.py migrate```

Agora e so começar. Lembre-se de criar um superuser.

A partir de agora iremos aprender sobre versionamento.

### Criando Versionamento No Django Rest

Existem algumas formas de fazer o versionamento.

<br>

- AcceptHeaderVersioning: <br>
Transferencia do número da versão atraves do cabeçalho da requisição
  ```
  GET/alunos/HTTP/1.1
  Host: exemplo.com
  Accept: aplication/json; version=1
  ```
  
<br>

- URLPathVersioning: <br>
Adciona a versão no endereço da url
```
urlpatterns = [
    url(r'(?P<version>(v1|v2)/alunos/$', alunos_list
        name='alunos-lista'
    )
]
```

<br>

- NamespaceVersioning: <br>
A versão é fornecida através do namespace da url
```
urlpatterns = [
    url(r'v1/alunos/', include(alunos.url', namespace='v1')),
    url(r'v2/alunos/', include(alunos.url', namespace='v2')),
]
```
<br>

- HostNameVersioning:<br>
A versão é definida pelo nome de dominio <br>
  http://v1.exemplo.com/alunos/ <br>
  http://v2.exemplo.com/alunos/
  
<br>

- QueryParameterVersioning:<br>
Trasfere a versão através do parametro GET  
  http://exemplo.com/alunos/?version=1 <br>
  http://exemplo.com/alunos/?version=2
  
Nesse caso vai ser utilizada essa ultima.

Motivação: Escrever novas funcionalidades sem deixar
de dar suporte aos clientes que usam a versão anterior.
OU tambem pode ser usado para ter varias versões de linguas ou
funcionalidades.

### Inciando:
Nesse caso na classe models que adicionaremos o
campo em Aluno. Com a possibilidade de estar em branco
para que funcione na versão anterior. foram criadas as migrações e executadas

Em Serializer crie a segunda versão. EX: ```AlunoSerializerV2```
Com o campo novo incluido.
Agora nas Views em AlunosViewSet adicione a logica 
```
 def get_serializer_class(self):
        if self.request.version == 'v2':
            return AlunoSerializerV2
        else:
            return AlunoSerializer
```
Alem disso deve ser incluido nos settings.py
```
REST_FRAMEWORK = {
    'DEFAULT_VERSIONING_CLASS': 'rest_framework.versioning.QueryParameterVersioning'
}
```
O versioning muda de acordo com o metodo que sera usado para versionar.
Agora basta passar o parametro ?version=v2 para acessar a nova versão


### Aprendendo mais sobre permisões:
Adicione novos usuarios com permisões diferentes no admin. 
Agora na parte da api: adcionar em viewas o modulo:
```DjangoModelPermissions``` e adicone o mesmo na lista permission_classes nas viewset
Agora ele ja usara as mesmas permisões que o admin.

Para refatorar, em REST_FRAMEWORK no settings passar a verificação por default
```
REST_FRAMEWORK = {
    'DEFAULT_VERSIONING_CLASS': 'rest_framework.versioning.QueryParameterVersioning',
    'DEFAULT_PERMISSION_CLASSES': [
        'rest_framework.permissions.IsAuthenticated',
        'rest_framework.permissions.DjangoModelPermissions'
    ],
    'DEFAULT_AUTHENTICATION_CLASSES': [
        'rest_framework.authentication.BasicAuthentication'
    ],
}
```
Podendo remover as autenticações dos viewSet e deixando mais legivel

#### DICA: 
Para limitar os metodos atraves da viewSet basta usar o array:
```http_method_names = ['get']``` com os metodos dentro da lista


### Limite da quantidade de requisições:
Em settings no dicionario REST_FRAMEWORK:
```
    'DEFAULT_THROTTLE_CLASSES': [
        'rest_framework.throttling.AnonRateThrottle',
        'rest_framework.throttling.UserRateThrottle'
    ],
    'DEFAULT_THROTTLE_RATES': {
        'anon': '5/day',
        'user': '1000/day'
    }

```
Em anon e user e definido o limite.

### Dica:
Ler 'A gloria do Rest de Richardson'


### Salvar em cache:
Para salvar em cache importar o modulo ```from django.utils.decorators import method_decorator```
e tambem ```from django.views.decorators.cache import cache_page``` na views

Agora na viewSet do modelo que tera cache 
```
    @method_decorator(cache_page(20))
    def dispatch(self, *args, **kwargs):
        return super(MatriculaViewSet, self).dispatch(*args, *kwargs)
```
cache_page(num) marca em segundos. 
AUMENTANDO A PERFOMACE!!!!!!! BOA PRATICA

### Instalando redis para armazenar cache e melhorar a perfomace

Instalar o modulo django-redis e o redis

Mexer nas configurações como sempre:
```
CACHES = {
    "default": {
        "BACKEND": 'django_redis.cache.RedisCache',
        "LOCATION": 'redis://127.0.0.1:6379'
    }
} 
```
Estudar mais sobre redis. Entendi nada

### LOCALIZAÇÂO(INTERNACIONALIZAÇÂO):
As mesagens de erro da api irao aparecer com a lingua que estiver
no head.
Basta adicionar ```'django.middleware.locale.LocaleMiddleware'```
nos MIDDLEWARES disponíveis.

### Alterando as mensagens padrões:
Crie um diretorio na base. como o nome locale
Em setting.py adicione: 
````
LOCALE_PATHS = (
    os.path.join(BASE_DIR, 'locale/')
) 
````
Agora execute o comando
```
python manage.py makemessages -l pt_BR
```
Isso devolvera os arquivos de configuração que estão dentro
de locale. Agora basta alterar e compilar as mensagens.
```
python manege.py compilemessages -l pt_BR
```
Pronto. Mensagens trocadas. :)

### Dicas de segurança:
Não deixar o admin vizivel.

Retirar os avisos no setting.
DEBUG = False
ALLOWED_HOST = ['localhost']

Pra melhorar mais ainda. Vamo de honeypot carai
Instalar a lib ```django-admin-honeypot``` e lembre de instalar 
em apps instalados tbm.  adicione:
``` path('admin/', include('admin_honeypot.urls'), namespace='admin_honeypot'), ```
na urls. e agora so migrar
Salva o ip e a hora. muito foda.

