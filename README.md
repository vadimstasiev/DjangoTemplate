> sources: [REST documentation](https://www.django-rest-framework.org/), [Django with VScode](https://automationpanda.com/2018/02/08/django-projects-in-visual-studio-code/)
## Setup
- setup venv
- `pip install django`
- `django-admin startproject template`
- Install plugins:
	- [Python](https://marketplace.visualstudio.com/items?itemName=ms-python.python)  (published by Microsoft) – for full Python language support
	-   [Django Template](https://marketplace.visualstudio.com/items?itemName=bibhasdn.django-html)  – for template file source highlighting
	-   [Django Snippets](https://marketplace.visualstudio.com/items?itemName=bibhasdn.django-snippets)  – for common Django code
- Install the following:
	```
	$ pip install pep8
	$ pip install autopep8
	$ pip install pylint
	```
- save a workspace and add the following settings to it (change dirs as you see fit)
	```py
	{
		"folders": [
			{
				"path": "."
			}
		],
		"settings": {
			"team.showWelcomeMessage": false,
			"editor.formatOnSave": true,
			"python.linting.pep8Enabled": true,
			"python.linting.pylintPath": "${workspaceFolder}/env/Lib/site-packages",
			"python.linting.pylintArgs": [
				"--load-plugins",
				"pylint_django"
			],
			"python.venvPath": "./env/Scripts/python",
			"python.pythonPath": "./env/Scripts/python.exe",
			"python.linting.pylintEnabled": true
		}
	}
	```
	## Django
	To run server:
	`python manage.py runserver`
	To migrate:
	`python manage.py makemigrations models_to_migrate`
	To apply migrations:
	`python manage.py migrate`
	### Create and Install an app
	`python manage.py startapp name_of_the_app`
	- Go to the settings (src/template/settings.py)
		```py
		Installed_APPS = [
			...
			'name_of_the_app',
		]
		```
	### Setting up the Django Rest Framework
	Install:
	```
	$ pip install djangorestframework
	$ pip install markdown       # Markdown support for the browsable API.
	$ pip install django-filter  # Filtering support
	```
	Add the rest framework to the installed apps (in the settings file at src/template/settings.py):
	```py
	INSTALLED_APPS = [
	    ...
	    'rest_framework',
	]
	```
	Also in the settings, add the global settings for the REST framework API:
	```py
	REST_FRAMEWORK = {
	    # Use Django's standard `django.contrib.auth` permissions,
	    # or allow read-only access for unauthenticated users.
	    'DEFAULT_PERMISSION_CLASSES': [
	        'rest_framework.permissions.DjangoModelPermissionsOrAnonReadOnly'
	    ]
	}
	```
	
	Then add it to the urls (in the urls file at src/template/urls.py):
	```py
	from django.urls import path, include # don't forget to add include
	urlpatterns = [
		...
		path('api-auth/', include('rest_framework.urls')),
	]
	```
	 For this example it is created an app called `myapp` and this was also added to the template's `INSTALLED_APPS`.

	### Create and migrate model
	In the models.py of the app that you created (in this case myapp/models.py): 
	```py
	from django.db import models
	
	class Article(models.Model):
		title = models.CharField(max_length=120)
		content = models.TextField()
			
		def __str__(self):
			return self.title
	```
	Migrate: 	
		- `python manage.py makemigrations`
		- `python manage.py migrate`
	### Add model to Admin (of `myapp`)
	In admin.py (in this case, src/myapp/admin.py):
	```py
	...
	from .models import Article
	admin.site.register(Article)
	```
	Don't forget to create super-user to access /admin in the Browser: `python manage.py createsuperuser`.
## The next step
The next step consists of getting the rest framework to convert JSON data into a model and vice versa, for that we'll be using [serializers ](https://www.django-rest-framework.org/api-guide/serializers/). We'll be using the `mpapp` app for that.
Within the `myapp` create a folder `api` with an empty __init__.py file, and the following files.
- myapp
	- api
		- __init__.py
		- serializers.py
		- views.py
> serializers.py
```py
from rest_framework import serializers
from myapp.models import Article

class  ArticleSerializer(serializers.ModelSerializer):
	class  Meta:
		model = Article
		fields = ('title', 'content')
```
This ArticleSerializer is to be used in a view. We're going to use some generic views in this example.
> views.py (src/myapp/api/views.py)
```py
from .serializers import ArticleSerializer
from myapp.models import Article
from rest_framework.generics import (
    ListAPIView,
    RetrieveAPIView,
    CreateAPIView,
    UpdateAPIView,
    DestroyAPIView
)

class ArticleListView(ListAPIView):
    queryset = Article.objects.all()
    serializer_class = ArticleSerializer
    
class ArticleDetailView(RetrieveAPIView):
    queryset = Article.objects.all()
    serializer_class = ArticleSerializer

class ArticleCreateView(CreateAPIView):
    queryset = Article.objects.all()
    serializer_class = ArticleSerializer

class ArticleUpdateView(UpdateAPIView):
    queryset = Article.objects.all()
    serializer_class = ArticleSerializer

class ArticleDeleteView(DestroyAPIView):
    queryset = Article.objects.all()
    serializer_class = ArticleSerializer
```
Now we have to define some urls that map to these views.
> urls.py (src/myapp/api/urls.py)
```py
from django.urls import path
from .views import (
    ArticleListView, 
    ArticleDetailView,
    ArticleCreateView,
    ArticleUpdateView,
    ArticleDeleteView
)

urlpatterns = [
    path('', ArticleListView.as_view()),
    path('create/',ArticleCreateView.as_view()),
    path('<pk>', ArticleDetailView.as_view()),
    path('<pk>/update/', ArticleUpdateView.as_view()),
    path('<pk>/delete/', ArticleDeleteView.as_view()),
]
```
Don't forget to add the `api/` route in the main app:
> urls.py (src/myapp/urls.py)
```py
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('api-auth/', include('rest_framework.urls')),
    path('api/', include('myapp.api.urls'))
]
```
### Extra
If you don't need to customize the views (src/myapp/api/views.py), instead of repeating code like shown above in views and urls, you can use [viewsets](https://www.django-rest-framework.org/api-guide/viewsets/).
> urls (src/myapp/api/urls.py)
```py
from django.urls import path
from articles.api.views import ArticleViewSet
from rest_framework.routers import DefaultRouter

router = DefaultRouter()
router.register(r'', ArticleViewSet, basename='articles')
urlpatterns = router.urls
```
> views (src/myapp/api/views.py)
```py
from articles.models import Article
from .serializers import ArticleSerializer
from rest_framework import viewsets

class ArticleViewSet(viewsets.ModelViewSet):
    serializer_class = ArticleSerializer
    queryset = Article.objects.all()
```