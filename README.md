## Docker Django2 Cheatsheet
The goal is to use my djnago docker images to create a viable django app


### Starting a Project
Typically we would start a django project thusly
```django-admin startproject my_project```

The things I don't like about this are:
- requires python and django to be installed on your machine
- django keeps config and secrets in settings.py - not 12 factor and not great for version control
- defaults to pip which can't deal with dependencies

So if we use a docker image like [jamesway/12factor-django-admin]
```
# start a project
docker run --rm -v $(pwd)/code jamesway/12factor-django-admin startproject [project_name]

# enter the project
cd [project_name]

# install django with pipenv
docker run --rm -v $(pwd)/code jamesway/python36-pipenv install django
```

We get a pipenv project with a .gitignored .env file containing Django configs and secrets.


### Starting an App
Again, a typical way to start an app in a Django project
```
# in the project root
python manage.py startapp [app_name]
```

Using the docker pipenv image...
```
# in the project root
docker run --rm -v $(pwd):/code jamesway/python36-pipenv run manage.py startapp [app_name]

# or using the docker-compose file
docker-compose run --rm pipenv run manage.py startapp [app_name]
```

### Running the Dev Server
Rather than this...
```
# in the project root
python manage.py runserver 0:8000
```

We do this...
```
# in the project root
docker run --rm -p 8000:8000 -itv $(pwd):/code jamesway/python36-pipenv run manage.py runserver 0:8000

# or a little less typing
docker-compose run --rm -p 8000:8000 pipenv run manage.py runserver 0:8000
```

You might have noticed event though we're using the compose file we still need to specify the port on the commandline. It's a "feature" of Docker to require the port argument when running a single service from compose. The idea is that if services are already up the single compose service might clash so specifying the port somehow helps prevent that? I'd rather not have to specify the port and docker throw an error message if there's a port clash - it does this anyway .
