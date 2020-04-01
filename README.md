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
		path('admin/', admin.site.urls),
		path('api-auth/', include('rest_framework.urls'))
	]
	```
	
	