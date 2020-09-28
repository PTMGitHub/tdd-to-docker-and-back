# Tutorial Source: 
Flask project setup: TDD, Docker, Postgres and more - Part 1
https://www.thedigitalcatonline.com/blog/2020/07/05/flask-project-setup-tdd-docker-postgres-and-more-part-1/

** Reasons for doing it**
1. It introduces setting up for different environments (dev, prod and test)
2. It uses TDD in to dev 

**Local Folder:** ApprenticeAcademy/api_flask_docker/tdd-to-docker-and-back

**GitHub:** git@github.com:PTMGitHub/tdd-to-docker-and-back.git

**NB:** <span style="color:green">Text in green are additional notes I have added</span>

**NB:** While i was doing this i was having to specifically map ```PWD=$PWD``` when using the commands, having to using ```sudo``` to run command and still geting failures like it not being able to parse python f-string code.  This was becuase 1. docker needs sudo 2. my sudo user was using python 2 which meant some of the python features didnt work becuase they are only avalible in py3.6 >.  This was fixed by doing this: https://docs.docker.com/engine/install/linux-postinstall/#manage-docker-as-a-non-root-user 


***
## Log

|     |     |
| --- | --- |
| **Date** | **Description** |
| 14 Sep 2020 | Start project, inital commit |
| 15 Sep 2020 | Commit at the end of step 3. Its doesnt quite work i still need to use FLASK_CONFIG="development". Going to go through it with Steve|
| 15 Sep 2020 | 2nd Commit of the day - End of Step 3 - Working. Discussed with Steve and both concluded FLASK_CONFIG should be set in the development.json|
| 21 Sep 2020 | Completed the first tutorial.|
| 22 Sep 2020 | Start Part 2 tutorial. Completed to end of step 2|
| 24 Sep 2020 | Nearly finished part 2, stuck in BONUS: full TDD example, trying to run the test for the first time. Getting a "E   ModuleNotFoundError: No module named 'application'"  error.|
| 28 Sep 2020 | Half way through "Part 2, Step: Bonus step - A Full TDD example".  Got the inital test to return green. Next commant to run is ```$ ./manage.py compose up -d``` but fails and not sure why. Gotta stop and look at other work so will come back to it 



**Tutorial Source:** 
Flask project setup: TDD, Docker, Postgres and more - Part 1
https://www.thedigitalcatonline.com/blog/2020/07/05/flask-project-setup-tdd-docker-postgres-and-more-part-1/

**Reasons for doing it**
1. It introduces setting up for different environments (dev, prod and test)
2. It uses TDD in to dev 

**Local Folder:** ApprenticeAcademy/api_flask_docker/tdd-to-docker-and-back

**GitHub:** git@github.com:PTMGitHub/tdd-to-docker-and-back.git

**NB:** <span style="color:green">Text in green are additional notes I have added</span>

**NB:** While i was doing this i was having to specifically map ```PWD=$PWD``` when using the commands, having to using ```sudo``` to run command and still geting failures like it not being able to parse python f-string code.  This was becuase 1. docker needs sudo 2. my sudo user was using python 2 which meant some of the python features didnt work becuase they are only avalible in py3.6 >.  This was fixed by doing this: https://docs.docker.com/engine/install/linux-postinstall/#manage-docker-as-a-non-root-user 

**Repeated commands**
- ```./manage.py compose build web```
- ```docker rmi -f $(sudo docker image ls -q)```

**Opening the environment:**

``` 
rm -r .env && virtualenv .env && source .env/bin/activate && pip install -r requirements/development.txt
```

***

# Flask project setup: TDD, Docker, Postgres and more - Part 1

## Step 1 - Requirements and editor¶
My standard structure for Python requirements uses 3 files: production.txt, development.txt, and testing.txt. They are all stored in the same directory called requirements, and are hierarchically connected.

File: requirements/production.txt
```
 # This file is currently empty
```
File: requirements/testing.txt
```
-r production.txt
```
File: requirements/development.txt
```
-r testing.txt

```



There is also a final requirements.txt file that points to the production one.

File: requirements.txt
```
-r requirements/production.txt
```

As you can see this allows me to separate the requirements to avoid installing unneeded packages, which greatly speeds up the deploy in production and keeps things as essential as possible. Production contains the minimum requirements needed to run the project, testing adds to those the packages used to test the code, and development adds to the latter the tools needed during development. A minor shortcoming of this setup is that I might not need in development everything I need in production, for example the HTTP server. I don't think this is significantly affecting my local setup, though, and if I have to decide between production and development, I prefer to keep the former lean and tidy.

I have my linters already installed system-wide, but as I'm using black to format the code I have to configure flake8 to accept what I'm doing

<span style="color:green">
Im not sure what i was meant to do here, assumed its a hidden file in the root of the project file.
</span>

File: .flake8
```
[flake8]
# Recommend matching the black line length (default 88),
# rather than using the flake8 default of 79:
max-line-length = 100
ignore = E231
```

This is clearly a very personal choice, and you might have different requirements. Take your time to properly configure the editor and the linter(s). Remember that the editor for a programmer is like the violin for the violinist. You need to know it, and to take care of it. So, set it up properly.

At this point I also create my virtual environment and activate it....

<span style="color:green">
**** These instructions are what i wrote from Steve ****
</span>

In a project folder: 


```
$ virtualenv .env
```
To activate the environment: 	
```
$ source .env/bin/activate
```
This one installs any old venv creates a news one and installs all the requirements in one
```
$ rm -r .env && virtualenv .env && source .env/bin/activate && pip install -r requirements.txt
```
<span style="color:green">
**** End ****
</span>

## Resources¶
[Installing packages in Python](https://packaging.python.org/tutorials/installing-packages/)
[flake8](https://flake8.pycqa.org/en/latest/) - A tool for style guide enforcement
[black](https://github.com/psf/black) - The uncompromising Python code formatter

***

## Step 2 - Flask project boilerplate¶
As this will be a Flask application the first thing to do is to install Flask itself. That goes in the production requirements, as that is needed at every stage.

File: requirements/production.txt
```
Flask
```
Now, install the development requirements with
```
$ pip install -r requirements/development.txt
```
As we saw before, that file automatically installs the testing and production requirements as well.

Then we need a directory where to keep all the code that is directly connected with the Flask framework, and where we will start creating the configuration for the application. Create the application directory and the file config.py in it.

File: application/config.py
> ```
> import os
>
> basedir = os.path.abspath(os.path.dirname(__file__))
> 
> 
> class Config(object):
>     """Base configuration"""
> 
> 
> class ProductionConfig(Config):
>     """Production configuration"""
> 
> 
> class DevelopmentConfig(Config):
>     """Development configuration"""
> 
> 
> class TestingConfig(Config):
>     """Testing configuration"""
> 
>     TESTING = True
> ```
There are many ways to configure a Flask application, one of which is using Python objects. This allows me to leverage inheritance to avoid duplication (which is always good), so it's my method of choice.

It's important to understand the variables and the parameters involved in the configuration. As the documentation clearly states, FLASK_ENV and FLASK_DEBUG have to be initialised outside the application as the code might misbehave if they are changed once the engine has been started. Furthermore the FLASK_ENV variable can have only the two values development and production, and the main difference is in performances. The most important thing we need to be aware of is that if FLASK_ENV is development, then FLASK_DEBUG becomes automatically True. To sum up we have the following guidelines:

- It's pointless to set DEBUG and ENV in the application configuration, they have to be environment variables.
- Generally you don't need to set FLASK_DEBUG, just set FLASK_ENV to development.
- Testing doesn't need the debug server turned on, so you can set FLASK_ENV to production during that phase. It needs TESTING set to True, though, and that has to be done inside the application.

We need now to create the application and to properly configure it. I decided to use and application factory<span style="Color:green"><b>*</b></span> that accepts a config_name string that is then converted into the name of the config object. For example, if config_name is development the variable config_module becomes application.config.DevelopmentConfig so that app.config.from_object can import it.

<span style="Color:green">
*Application factories: baiscally you set the the application up in a python function. it gives you the benefits of:

1. Testing. You can have instances of the application with different settings to test every case.

2. Multiple instances. Imagine you want to run different versions of the same application. Of course you could have multiple instances with different configs set up in your webserver, but if you use factories, you can have multiple instances of the same application running in the same application process which can be handy.

</span>


File: application/app.py
> ```
> from flask import Flask
> 
> 
> def create_app(config_name):
> 
>     app = Flask(__name__)
> 
>     config_module = f"application.config.{config_name.capitalize()}Config"
> 
>     app.config.from_object(config_module)
> 
>     @app.route("/")
>     def hello_world():
>         return "Hello, World!"
> 
>     return app
>```

I also added the standard "Hello, world!" route to have a quick way to see if the server is working or not.

Last, we need something that initializes the application running the application factory and passing the correct value for the config_name parameter. The Flask development server can automatically use any file named wsgi.py<span style="color:green"><b>*</b></span> in the root directory, and since WSGI is a standard specification using that makes me sure that any HTTP server we will use in production (for example Gunicorn or uWSGI) will be immediately working.

<span style="color:green">
* WSGI = Web Server Gateway Interface. It is a specification that describes how a web server communicates with web applications, and how web applications can be chained together to process one request.
WSGI is a Python standard described in detail in PEP 3333. 
If an application (or framework or toolkit) is written to the WSGI spec then it will run on any server written to that spec
</span>

File: wsgi.py

>```
> import os
>
> from application.app import create_app
>
> app = create_app(os.environ["FLASK_CONFIG"])
> ```

Here, I decided to read the value of config_name from the FLASK_CONFIG variable. This is not a variable requested by the framework, but I decided to use the FLASK_ prefix anyway because it is tightly connected with the structure of the Flask application.

At this point we can happily run the Flask development server with

> ```
> $ FLASK_CONFIG="development" flask run
>  * Environment: production
>    WARNING: This is a development server. Do not use it in a production deployment.
>    Use a production WSGI server instead.
>  * Debug mode: off
>  * Running on http://127.0.0.1:5000/ (Press CTRL+C to quit)
> ```
Please note that it says Environment: production because we haven't configured FLASK_ENV yet. If you head to http://127.0.0.1:5000/ with your browser you can see the greetings message.

### Resources
- [Flask configuration documentation](https://flask.palletsprojects.com/en/1.1.x/config/)
- [Flask application factories](https://flask.palletsprojects.com/en/1.1.x/patterns/appfactories/)
- [WSGI](https://wsgi.readthedocs.io/en/latest/what.html) - The Python Web Server Gateway Interface
- My post [Dissecting a web stack](https://www.thedigitalcatonline.com/blog/2020/02/16/dissecting-a-web-stack/) includes a section on WSGI

***

# Step 3 - Application configuration¶
As I mentioned in the introduction, I am going to use a static JSON configuration file. The choice of JSON comes from the fact that it is a widespread file format, accessible from many programming languages, included Terraform, which I plan to use to create my production infrastructure.

File: config/development.json

>```
>[
>  {
>    "name": "FLASK_ENV",
>    "value": "development"
>  }
>]
>```

I obviously need a script that extracts variables from the JSON file and converts them into environment variables, so it's time to start writing my own manage.py file. This is a pretty standard concept in the world of Python web frameworks, a tradition initiated by Django. The idea is to centralise all the management functions like starting/stopping the development server or managing database migrations. As in flask this is partially done by the flask command itself, for the time being I just need to wrap it providing suitable environment variables.

File: manage.py

<span style="color:green"> 
does the below need to be python3 for me?</span>

>```python
>#! /usr/bin/env python
>
>import os
>import json
>import signal
>import subprocess
>
>import click
>
>
># Ensure an environment variable exists and has a value
>def setenv(variable, default):
>    os.environ[variable] = os.getenv(variable, default)
>
>
>setenv("APPLICATION_CONFIG", "development")
>
># Read configuration from the relative JSON file
>config_json_filename = os.getenv("APPLICATION_CONFIG") + ".json"
>with open(os.path.join("config", config_json_filename)) as f:
>    config = json.load(f)
>
># Convert the config into a usable Python dictionary
>config = dict((i["name"], i["value"]) for i in config)
>
>for key, value in config.items():
>    setenv(key, value)
>
>
>@click.group()
>def cli():
>    pass
>
>
>@cli.command(context_settings={"ignore_unknown_options": True})
>@click.argument("subcommand", nargs=-1, type=click.Path())
>def flask(subcommand):
>    cmdline = ["flask"] + list(subcommand)
>
>    try:
>        p = subprocess.Popen(cmdline)
>        p.wait()
>    except KeyboardInterrupt:
>        p.send_signal(signal.SIGINT)
>        p.wait()
>
>
>cli.add_command(flask)
>
>if __name__ == "__main__":
>    cli()
>```    
    
Remember to give make the script executables with

>```
>$ chmod 775 manage.py
> ```

As you can see I'm using click, which is the recommended way to implement Flask commands. As I might use it to customise subcommands of the flask main script, I decided to stick to one tool and use it for the manage.py script as well.

<span style="color:green">
I've added Click to requirements/production.txt 
</span>

The APPLICATION_CONFIG variable is the only one that I need to specify, and its default value is development. From that variable I infer the name of the JSON file with the full configuration and load environment variables from that. The flask function simply wraps the flask command provided by Flask so that I can run ```./manage.py flask <subcommand>``` to run it using the development configuration or ```APPLICATION_CONFIG="foobar" ./manage.py flask <subcommand>``` to use the foobar one.

A clarification, to be sure you don't confuse environment variables with each other:

- APPLICATION_CONFIG is strictly related to my project and is used only to load a JSON configuration file with the name specified in the variable itself.
- FLASK_CONFIG is used to select the Python object that contains the configuration for the Flask application (see application/app.py and application/config.py). The value of the variable is converted into the name of a class.
- FLASK_ENV is a variable used by Flask itself, and its values are dictated by it. See the configuration documentation mentioned in the resources of the previous section.

Now we can run the development server

>```
>$ ./manage.py flask run
> * Environment: development
> * Debug mode: on
> * Running on http://127.0.0.1:5000/ (Press CTRL+C to quit)
> * Restarting with stat
> * Debugger is active!
> * Debugger PIN: 172-719-201
>```
Note that it now says Environment: development because of FLASK_ENV has been set to development in the configuration. As we did before, a quick visit to http://127.0.0.1:5000/ shows us that everything is up and running.

<span style="color:green">
Now jsut following this tutorial I would create the flask and give the  responce as above but when i would browse to the the web page it would blow up with....
</span>

```
* Environment: development
* Debug mode: on
* Running on http://127.0.0.1:5000/ (Press CTRL+C to quit)
* Restarting with stat
* Debugger is active!
* Debugger PIN: 154-422-043
127.0.0.1 - - [14/Sep/2020 17:35:40] "GET / HTTP/1.1" 500 -
Traceback (most recent call last):
File "/usr/lib/python3.8/os.py", line 672, in getitem
value = self._data[self.encodekey(key)]
KeyError: b'FLASK_CONFIG'
During handling of the above exception, another exception occurred:

Traceback (most recent call last):
File "/home/paul/Documents/ApprenticeAcademy/api_flask_docker/tdd-to-docker-and-back/.venv/lib/python3.8/site-packages/flask/_compat.py", line 39, in reraise
raise value
File "/home/paul/Documents/ApprenticeAcademy/api_flask_docker/tdd-to-docker-and-back/wsgi.py", line 5, in <module>
app = create_app(os.environ["FLASK_CONFIG"])
File "/usr/lib/python3.8/os.py", line 675, in getitem
raise KeyError(key) from None
KeyError: 'FLASK_CONFIG'
```

<span style="color:green">
to get this working I had to add setting the FLASK_CONFIG="development" either when runnig the command...
</span>

```
$ FLASK_CONFIG="development" ./manage.py flask run
```

<span style="color:green">
Or putting them into the json file:
</span>

```json
[
  {
    "name": "FLASK_ENV",
    "value": "development"
  },
  {
    "name": "FLASK_CONFIG",
    "value": "development"
  }

]
```
<span style="color:green">
i do not know if this is correct. but going with it till i can either work it out 
</span>

<p><b><span style="color:green">
Discussed this will Steve, he agrees with me that it looks like FLASK_CONFIG should be set within the development.json file. So this is what we have done, gonna see what happens when we og forward.
</span></b></p>

Resources:¶
- [Django's management script](https://docs.djangoproject.com/en/3.0/ref/django-admin/)
- [Click](https://click.palletsprojects.com/en/7.x/) - A Python package for creating command line interfaces

***

## Step 4 - Containers and orchestration

There is no better way to simplify your development than using Docker.

There is also no better way to complicate your life than using Docker.

As you might guess, I have mixed feelings about Docker. Don't get me wrong, Linux containers are an amazing concept, and Docker is very useful. It's also a complex technology that sometimes requires a lot of work to get properly configured. In this case the setup will be pretty simple, but there is a major complication with using a database server that I will describe later.

Running the application in a Docker container allows me to isolate it and to simulate the way I will run it in production. I will use docker-compose, as I expect to have other containers running in my development setup (at least the database), so I can leverage the fact that the docker-compose configuration file can interpolate environment variables. Once again through the `APPLICATION_CONFIG` environment variable I will select the correct JSON file, load its values in environment variables and then run the docker-compose file.

First of all we need an image for the Flask application

>```dockerfile
>File: docker/Dockerfile
>
>FROM python:3
>
>ENV PYTHONUNBUFFERED 1
>
>RUN mkdir /opt/code
>RUN mkdir /opt/requirements
>WORKDIR /opt/code
>
>ADD requirements /opt/requirements
>RUN pip install -r /opt/requirements/development.txt
>```

As you can see the requirements directory is copied into the image, so that Docker can run the pip install command at creation time. The whole code directory will be mounted live into the image at run time.

This clearly means that every time we change the development requirements we need to rebuild the image. This is not a complicated process, so I will keep it as a manual process for now. To run the image we can create a configuration file for docker-compose.

File: docker/development.yml

>```docker
>version: '3.4'
>
>services:
>  web:
>    build:
>      context: ${PWD}
>      dockerfile: docker/Dockerfile
>    environment:
>      FLASK_ENV: ${FLASK_ENV}
>      FLASK_CONFIG: ${FLASK_CONFIG}
>    command: flask run --host 0.0.0.0
>    volumes:
>      - ${PWD}:/opt/code
>    ports:
>      - "5000:5000"
>```

As you can see, the docker-compose configuration file can read environment variables natively. <span style="color:green"><b>this doesnt seem to be true in my case</b></span> To run it we first need to add docker-compose itself to the development requirements.

File: requirements/development.txt

>```
>-r testing.txt
>
>docker-compose
>```

Install it with pip install -r requirements/development.txt, then build the image with

>```
>$ FLASK_ENV="development" FLASK_CONFIG="development" docker-compose -f docker/development.yml build web
>```
<span style="color:green">
Couple of things I found i had to change to make this work:
1. Had to use Sudo
2. Had to include "PWD =$PWD" in the command as, at least on my system, docker-compose does not native pick up environment variable. 
3. I also added "./" to the Dockerfile path
</span>

We are explicitly passing environment variables here, as we have not wrapped docker-compose in the manage script yet. Once the image has been build, we can run it with the up command

>```
>$ FLASK_ENV="development" FLASK_CONFIG="development" docker-compose -f >docker/development.yml up
>```
<span style="color:green">
1. Added "PWD =$PWD" to the command. 
</span>

This command should give us the following output

>```
>Creating network "docker_default" with the default driver
>Creating docker_web_1 ... done
>Attaching to docker_web_1
>web_1  |  * Environment: development
>web_1  |  * Debug mode: on
>web_1  |  * Running on http://0.0.0.0:5000/ (Press CTRL+C to quit)
>web_1  |  * Restarting with stat
>web_1  |  * Debugger is active!
>web_1  |  * Debugger PIN: 234-361-737
>```

You can stop the containers pressing Ctrl-C, which gracefully tears down the system. If you run the command up -d docker-compose will run as a daemon, leaving you the control of the current terminal. If docker-compose is running you can docker ps and you should see an output similar to this

>```
>CONTAINER ID  IMAGE       COMMAND                 ...   PORTS                   >NAMES
>c98f35635625  docker_web  "flask run --host 0.…"  ...   0.0.0.0:5000->5000/tcp  >docker_web_1
>```

If you need to explore the container you can login directly with

>```
>$ docker exec -it docker_web_1 bash
>```

or with

>```
>$ FLASK_ENV="development" FLASK_CONFIG="development" docker-compose -f >docker/development.yml exec web bash
>```

In either case, you will end up in the /opt/code directory (which is the WORKDIR of the image), where the current directory in the host is mounted.

To tear down the containers, when running as daemon, you can run

>```
>$ FLASK_ENV="development" FLASK_CONFIG="development" docker-compose -f >docker/development.yml down
>```

Notice that the server now says Running on http://0.0.0.0:5000/, as the Docker container is using that network interface to communicate with the outside world. Since the ports are mapped, however, you can head to either http://localhost:5000 or http://0.0.0.0:5000 with your browser.

To simplify the usage of docker-compose, I want to wrap it in the manage.py script, so that it automatically receives environment variables, as their number is going to increase soon when we will add a database.

File: manage.py

> ```
>#! /usr/bin/env python
>
>import os
>import json
>import signal
>import subprocess
>
>import click
>
>docker_compose_file = "docker/development.yml"
>docker_compose_cmdline = ["docker-compose", "-f", docker_compose_file]
>
>
># Ensure an environment variable exists and has a value
>def setenv(variable, default):
>    os.environ[variable] = os.getenv(variable, default)
>
>
>setenv("APPLICATION_CONFIG", "development")
>
># Read configuration from the relative JSON file
>config_json_filename = os.getenv("APPLICATION_CONFIG") + ".json"
>with open(os.path.join("config", config_json_filename)) as f:
>    config = json.load(f)
>
># Convert the config into a usable Python dictionary
>config = dict((i["name"], i["value"]) for i in config)
>
>for key, value in config.items():
>    setenv(key, value)
>
>
>@click.group()
>def cli():
>    pass
>
>
>@cli.command(context_settings={"ignore_unknown_options": True})
>@click.argument("subcommand", nargs=-1, type=click.Path())
>def flask(subcommand):
>    cmdline = ["flask"] + list(subcommand)
>
>    try:
>        p = subprocess.Popen(cmdline)
>        p.wait()
>    except KeyboardInterrupt:
>        p.send_signal(signal.SIGINT)
>        p.wait()
>
>
>@cli.command(context_settings={"ignore_unknown_options": True})
>@click.argument("subcommand", nargs=-1, type=click.Path())
>def compose(subcommand):
>    cmdline = docker_compose_cmdline + list(subcommand)
>
>    try:
>        p = subprocess.Popen(cmdline)
>        p.wait()
>    except KeyboardInterrupt:
>        p.send_signal(signal.SIGINT)
>        p.wait()
>
>
>if __name__ == "__main__":
>    cli()
>```
    
You might have noticed that the two functions flask and compose are basically the same code, but I resisted the temptation to refactor them because I know that the compose command will need some changes as soon as I add a database.

The last change we need in order to make everything work properly is adding the `FLASK_CONFIG` variable to the config file

File: config/development.json

> ```
> [
>  {
>    "name": "FLASK_ENV",
>    "value": "development"
>  },
>  {
>    "name": "FLASK_CONFIG",
>    "value": "development"
>  }
> ]
> ```

Now I can run ./manage.py compose up -d and ./manage.py compose down and have the environment variables automatically passed to the system.

<span style="color:green">
For me to get this working the command I needed to write is this:
</span>

```
$ sudo PWD=$PWD python3 ./manage.py compose up -d  
```

Resources¶
- [Docker compose](https://docs.docker.com/compose/) - A tool for defining and running multi-container Docker applications
- [Docker networking](https://docs.docker.com/network/)
- [Python subprocess module](https://docs.python.org/3/library/subprocess.html)

***

## Final words¶
That's enought for this first post. We started from scratch and added some boilerplate code for a Flask project, exploring what environment variables are used by the framework, then we added a configuration system, a management script, and finally we run everything in a Docker container. In the next post I will show you how to add a persistent database to the development setup and how to use an ephemeral one for the tests. If you find my posts useful please share them with whoever you thing might be interested.

***

# Flask project setup: TDD, Docker, Postgres and more - Part 1 COMPELETED

***

Next to do: https://www.thedigitalcatonline.com/blog/2020/07/06/flask-project-setup-tdd-docker-postgres-and-more-part-2/

# Flask project setup: TDD, Docker, Postgres and more - Part 2

## Catch-up¶
In the previous post we started from an empty project and learned how to add the minimal code to run a Flask project. Then we created a static configuration file and a management script that wraps the flask and docker-compose commands to run the application with a specific configuration

In this post I will show you how to run a production-ready database alongside your code in a Docker container, both in your development setup and for the tests.

## Step 1 - Adding a database container¶
A database is an integral part of a web application, so in this step I will add my database of choice, Postgres, to the project setup. To do this I need to add a service in the docker-compose configuration file

File: docker/development.yml

> ```
> version: '3.4'
> 
> services:
>   db:
>     image: postgres
>     environment:
>       POSTGRES_DB: ${POSTGRES_DB}
>       POSTGRES_USER: ${POSTGRES_USER}
>       POSTGRES_HOSTNAME: ${POSTGRES_HOSTNAME}
>       POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
>     ports:
>       - "${POSTGRES_PORT}:5432"
>     volumes:
>       - pgdata:/var/lib/postgresql/data
>   web:
>     build:
>       context: ${PWD}
>       dockerfile: docker/Dockerfile
>     environment:
>       FLASK_ENV: ${FLASK_ENV}
>       FLASK_CONFIG: ${FLASK_CONFIG}
>     command: flask run --host 0.0.0.0
>     volumes:
>       - ${PWD}:/opt/code
>     ports:
>       - "5000:5000"
> 
> volumes:
>   pgdata:
> ```  
 
The variables starting with ```POSTGRES_``` are requested by the PostgreSQL Docker image. In particular, remember that ``POSTGRESQL_DB``` is the database that gets created by default when you create the image, and also the one that contains data on other databases as well, so for the application we usually want to use a different one.

Notice also that for the db service I'm creating a persistent volume, so that the content of the database is not lost when we tear down the container. For this service I'm using the default image, so no build step is needed.

To orchestrate this setup we need to add those variables to the JSON configuration

File: config/development.json

> ```
> [
>   {
>     "name": "FLASK_ENV",
>     "value": "development"
>   },
>   {
>     "name": "FLASK_CONFIG",
>     "value": "development"
>   },
>   {
>     "name": "POSTGRES_DB",
>     "value": "postgres"
>   },
>   {
>     "name": "POSTGRES_USER",
>     "value": "postgres"
>   },
>   {
>     "name": "POSTGRES_HOSTNAME",
>     "value": "localhost"
>   },
>   {
>     "name": "POSTGRES_PORT",
>     "value": "5432"
>   },
>   {
>     "name": "POSTGRES_PASSWORD",
>     "value": "postgres"
>   }
> ]
> ```

These are all development variables so there are no secrets. In production we will need a way to keep the secrets in a safe place and convert them into environment variables. The AWS Secret Manager for example can directly map secrets into environment variables passed to the containers, saving you from having to explicitly connect to the service with the API.

We can run the ```./manage.py compose up -d``` and ```./manage.py compose down``` here to check that the database container works properly.

<span style="color:green">
For me the command is:
</span>

```
$ sudo PWD=$PWD python3 ./manage.py compose up -d  
```


> ```
> CONTAINER ID  IMAGE       COMMAND                 ...  PORTS                   NAMES
> 9b5828dccd1c  docker_web  "flask run --host 0.…"  ...  0.0.0.0:5000->5000/tcp  > docker_web_1
> 4440a18a1527  postgres    "docker-entrypoint.s…"  ...  0.0.0.0:5432->5432/tcp  > docker_db_1
> ```

Now we need to connect the application to the database and to do this we can leverage flask-postgresql. As we will use this at every stage of the life of the application, the requirement goes among the production ones. We also need psycopg2 as it is the library used to connect to Postgres.

File: requirements/production.txt

> ```
> Flask
> flask-sqlalchemy
> psycopg2-binary
> ```

<span style="color:green">
<b>NB:</b> psycopg2 fails. I am using psycopg2-binary
</span>

Remember to run ```pip install -r requirements/development.txt``` to install the requirements locally and ```sudo PWD=$PWD python3 ./manage.py compose build web``` to rebuild the image.

<span style="color:green">
Opening the enviroment:
</span>

``` 
rm -r .env && virtualenv .env && source .env/bin/activate && pip install -r requirements/development.txt
```

At this point I need to create a connection string in the configuration of the application. The connection string parameters come from the same environment variables used to spin up the db container

File: application/config.py

> ```
> import os
> 
> basedir = os.path.abspath(os.path.dirname(__file__))
> 
> 
> class Config(object):
>     """Base configuration"""
> 
>     user = os.environ["POSTGRES_USER"]
>     password = os.environ["POSTGRES_PASSWORD"]
>     hostname = os.environ["POSTGRES_HOSTNAME"]
>     port = os.environ["POSTGRES_PORT"]
>     database = os.environ["APPLICATION_DB"]
> 
>     SQLALCHEMY_DATABASE_URI = (
>         f"postgresql+psycopg2://{user}:{password}@{hostname}:{port}/{database}"
>     )
>     SQLALCHEMY_TRACK_MODIFICATIONS = False
> 
> 
> class ProductionConfig(Config):
>     """Production configuration"""
> 
> 
> class DevelopmentConfig(Config):
>     """Development configuration"""
> 
> 
> class TestingConfig(Config):
>     """Testing configuration"""
> 
>     TESTING = True
> ```

As you can see, here I use the variable ```APPLICATION_DB``` and not ```POSTGRES_DB```, so I need to specify that as well in the config file

File: config/development.json

> ```json
> [
>   {
>     "name": "FLASK_ENV",
>     "value": "development"
>   },
>   {
>     "name": "FLASK_CONFIG",
>     "value": "development"
>   },
>   {
>     "name": "POSTGRES_DB",
>     "value": "postgres"
>   },
>   {
>     "name": "POSTGRES_USER",
>     "value": "postgres"
>   },
>   {
>     "name": "POSTGRES_HOSTNAME",
>     "value": "localhost"
>   },
>   {
>     "name": "POSTGRES_PORT",
>     "value": "5432"
>   },
>   {
>     "name": "POSTGRES_PASSWORD",
>     "value": "postgres"
>   },
>   {
>     "name": "APPLICATION_DB",
>     "value": "application"
>   }
> ]
> ```

At this point the application container needs to access some of the Postgres environment variables and the APPLICATION_DB one

File: docker/development.yml

> ```
> version: '3.4'
> 
> services:
>   db:
>     image: postgres
>     environment:
>       POSTGRES_DB: ${POSTGRES_DB}
>       POSTGRES_USER: ${POSTGRES_USER}
>       POSTGRES_HOSTNAME: ${POSTGRES_HOSTNAME}
>       POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
>     ports:
>       - "${POSTGRES_PORT}:5432"
>     volumes:
>       - pgdata:/var/lib/postgresql/data
>   web:
>     build:
>       context: ${PWD}
>       dockerfile: docker/Dockerfile
>     environment:
>       FLASK_ENV: ${FLASK_ENV}
>       FLASK_CONFIG: ${FLASK_CONFIG}
>       APPLICATION_DB: ${APPLICATION_DB}
>       POSTGRES_USER: ${POSTGRES_USER}
>       POSTGRES_HOSTNAME: ${POSTGRES_HOSTNAME}
>       POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
>       POSTGRES_PORT: ${POSTGRES_PORT}
>     command: flask run --host 0.0.0.0
>     volumes:
>       - ${PWD}:/opt/code
>     ports:
>       - "5000:5000"
> 
> volumes:
>   pgdata:
>   
> ``` 

Running compose now spins up both Flask and Postgres but the application is not properly connected to the database yet.

Resources¶
- [Postgres Docker image](https://hub.docker.com/_/postgres)
- [Flask-SQLAlchemy](https://flask-sqlalchemy.palletsprojects.com/en/2.x/) - A Flask extension to work with SQLAlchemy
- https://docs.sqlalchemy.org/en/13/dialects/postgresql.html- The Python SQL Toolkit and ORM

***

#Step 2 - Connecting the application and the database¶
To connect the Flask application with the database running in the container I need to initialise an SQLAlchemy object and add it to the application factory.

File: application/models.py

>  ```
>  from flask_sqlalchemy import SQLAlchemy
>  
>  db = SQLAlchemy()
> ```

File: application/app.py

> ```  
>  from flask import Flask
>  
>  
>  def create_app(config_name):
>  
>      app = Flask(__name__)
>  
>      config_module = f"application.config.{config_name.capitalize()}Config"
>  
>      app.config.from_object(config_module)
>  
>      from application.models import db
>  
>      db.init_app(app)
>  
>      @app.route("/")
>      def hello_world():
>          return "Hello, World!"
>  
>      return app
>  ```    
 
A pretty standard way to manage the database in Flask is to use flask-migrate, that adds some commands that allow us to create migrations and apply them.

With flask-migrate you have to create the migrations folder once and for all with flask db init and then, every time you change your models, run flask db migrate -m "Some message" and flask db upgrade. As both db init and db migrate create files in the current directory we now face a problem that every Docker-based setup has to face: file permissions.

The situation is the following: the application is running in the Docker container as root, and there is no connection between the users namespace in the container and that of the host. The result is that if the Docker container creates files in a directory that is mounted from the host (like the one that contains the application code in our example), those files will result as belonging to root. While this doesn't make impossible to work (we usually can become root on our devlopment machines), it is annoying to say the least. The solution is to run those commands from outside the container, but this requires the Flask application to be configured.

Fortunately I wrapped the flask command in the manage.py script, which loads all the required environment variables. Let's add falsk-migrate to the production requirements, together

File: requirements/production.txt

> ```
> Flask
> flask-sqlalchemy
> psycopg2
> flask-migrate
> ```

Remember to run ```pip install -r requirements/development.txt``` to install the requirements locally and ```sudo PWD=$PWD python3 ./manage.py compose build web``` to rebuild the image.

Now we can initialise a Migrate object and add it to the application factory

File: application/models.py

> ```
> from flask_sqlalchemy import SQLAlchemy
> from flask_migrate import Migrate
> 
> db = SQLAlchemy()
> migrate = Migrate()
> ```

File: application/app.py

> ```
> from flask import Flask
> 
> 
> def create_app(config_name):
> 
>     app = Flask(__name__)
> 
>     config_module = f"application.config.{config_name.capitalize()}Config"
> 
>     app.config.from_object(config_module)
> 
>     from application.models import db, migrate
> 
>     db.init_app(app)
>     migrate.init_app(app, db)
> 
>     @app.route("/")
>     def hello_world():
>         return "Hello, World!"
> 
>     return app
> ```
    
I can now run the database initialisation script

> ```
> $ PWD=$PWD ./manage.py flask db init
>   Creating directory /home/leo/devel/flask-tutorial/migrations ...  done
>   Creating directory /home/leo/devel/flask-tutorial/migrations/versions ...  done
>   Generating /home/leo/devel/flask-tutorial/migrations/env.py ...  done
>   Generating /home/leo/devel/flask-tutorial/migrations/README ...  done
>   Generating /home/leo/devel/flask-tutorial/migrations/script.py.mako ...  done
>   Generating /home/leo/devel/flask-tutorial/migrations/alembic.ini ...  done
>   Please edit configuration/connection/logging settings in > '/home/leo/devel/flask-tutorial/migrations/alembic.ini' before proceeding.
> ```  

<span style="color:green">
Feeling abit confused here, I assumed, becuase of issues with previous runs, i thought i would need to use the command..
</span>  

> ```
> sudo PWD=$PWD python3 ./manage.py flask db init
> ```
  
<span style="color:green">
but it failed. couldnt find the folder "flask".  this command worked:
</span>  

> ```
> PWD=$PWD ./manage.py flask db init
> ```


  
And, when we will start creating models we will use the commands ./manage.py flask db migrate and ./manage.py flask db upgrade. You will find a complete example at the end of this post.

Resources¶
- Flask-SQLAlchemy - A Flask extension to work with SQLAlchemy
- Flask-Migrate - A Flask extension to handle database migrations with Alembic.

***

#Step 3 - Testing setup¶
I want to use a TDD approach as much as possible when developing my applications, so I need to setup a good testing enviroment upfront, and it has to be as ephemereal as possible. It is not unusual in big projects to create (or scale up) infrastructure components explicitly to run tests, and through Docker and docker-compose we can easily do the same. Namely, I will:

spin up a test database in a container without permanent volumes
initialise it
run all the tests against it
tear down the container
This approach has one big advantage, which is that it requires no previous setup and can this be executed on infrastructure created on the fly. It also has disadvantages, however, as it can slow down the testing part of the application, which should be as fast as possible in a TDD setup. Tests that involve the databse, however, should be considered integration tests, and not run continuously in a TDD process, which is impossible (or very hard) when using a framework that merges the concept of entity and database model. If you want to know more about this read my post on the clean architecture, and the book that I wrote on the subject.

Another advantage of this setup is it that we might need other things during the test, e.g. Celery, other databases, other servers. They can all be created through the docker-compose file.

Generally speaking testing is an umbrella under which many different things can happen. As I will use pytest I can run the full suite, but I might want to select specific tests, mentioning a single file or using the powerful -k option that allows me to select tests by pattern-matching their name. For this reason I want to map the management command line to that of pytest.

Let's add pytest to the testing requirements

File: requirements/testing.txt

> ```
> -r production.txt
> 
> pytest
> coverage
> pytest-cov
> ```

As you can see I also use the coverage plugin to keep an eye on how well I cover the code with the tests. Remember to run ```pip install -r requirements/development.txt``` to install the requirements locally and ```./manage.py compose build web``` to rebuild the image.

File: manage.py

> ```
> #! /usr/bin/env python
> 
> import os
> import json
> import signal
> import subprocess
> import time
> 
> import click
> 
> 
> # Ensure an environment variable exists and has a value
> def setenv(variable, default):
>     os.environ[variable] = os.getenv(variable, default)
> 
> 
> setenv("APPLICATION_CONFIG", "development")
> 
> 
> def configure_app(config):
>     # Read configuration from the relative JSON file
>     with open(os.path.join("config", f"{config}.json")) as f:
>         config_data = json.load(f)
> 
>     # Convert the config into a usable Python dictionary
>     config_data = dict((i["name"], i["value"]) for i in config_data)
> 
>     for key, value in config_data.items():
>         setenv(key, value)
> 
> 
> @click.group()
> def cli():
>     pass
> 
> 
> @cli.command(context_settings={"ignore_unknown_options": True})
> @click.argument("subcommand", nargs=-1, type=click.Path())
> def flask(subcommand):
>     configure_app(os.getenv("APPLICATION_CONFIG"))
> 
>     cmdline = ["flask"] + list(subcommand)
> 
>     try:
>         p = subprocess.Popen(cmdline)
>         p.wait()
>     except KeyboardInterrupt:
>         p.send_signal(signal.SIGINT)
>         p.wait()
> 
> 
> def docker_compose_cmdline(config):
>     configure_app(os.getenv("APPLICATION_CONFIG"))
> 
>     docker_compose_file = os.path.join("docker", f"{config}.yml")
> 
>     if not os.path.isfile(docker_compose_file):
>         raise ValueError(f"The file {docker_compose_file} does not exist")
> 
>     return [
>         "docker-compose",
>         "-p",
>         config,
>         "-f",
>         docker_compose_file,
>     ]
> 
> 
> @cli.command(context_settings={"ignore_unknown_options": True})
> @click.argument("subcommand", nargs=-1, type=click.Path())
> def compose(subcommand):
>     cmdline = docker_compose_cmdline(os.getenv("APPLICATION_CONFIG")) + > list(subcommand)
> 
>     try:
>         p = subprocess.Popen(cmdline)
>         p.wait()
>     except KeyboardInterrupt:
>         p.send_signal(signal.SIGINT)
>         p.wait()
> 
> @cli.command()
> @click.argument("filenames", nargs=-1)
> def test(filenames):
>     os.environ["APPLICATION_CONFIG"] = "testing"
>     configure_app(os.getenv("APPLICATION_CONFIG"))
> 
>     cmdline = docker_compose_cmdline(os.getenv("APPLICATION_CONFIG")) + > ["up", "-d"]
>     subprocess.call(cmdline)
> 
>     cmdline = docker_compose_cmdline(os.getenv("APPLICATION_CONFIG")) + > ["logs", "db"]
>     logs = subprocess.check_output(cmdline)
>     while "ready to accept connections" not in logs.decode("utf-8"):
>         time.sleep(0.1)
>         logs = subprocess.check_output(cmdline)
> 
>     cmdline = ["pytest", "-svv", "--cov=application", > "--cov-report=term-missing"]
>     cmdline.extend(filenames)
>     subprocess.call(cmdline)
> 
>     cmdline = docker_compose_cmdline(os.getenv("APPLICATION_CONFIG")) + > ["down"]
>     subprocess.call(cmdline)
> 
> 
> if __name__ == "__main__":
>     cli()
>     
> ```

Notable changes are

- The environment configuration code is now in the function configure_app. This allows me to force the variable APPLICATION_CONFIG inside the script and then configure the environment, which saves me from having to call tests with APPLICATION_CONFIG=testing flask test.
- Both commands flask and compose use the development configuration. Since that is the default value of the APPLICATION_CONFIG variable they just have to call the configure_app function.
- The docker-compose command line is needed both in the compose and in the test commands, so I isolated some code into a function called docker_compose_cmdline which returns a list as needed by subprocess functions. The command line now uses also the -p (project name) option to give a prefix to the containers. This way we can run tests while running the development server.
- The test command forces APPLICATION_CONFIG to be testing, which loads the file config/testing.json, then runs docker-compose using the file docker/testing.yml (both file have not bee created yet), runs the pytest command line, and tears down the testing database container. Before running the tests the script waits for the service to be available. Postgres doesn't allow connection until the database is ready to accept them.

File: config/testing.json

> ```
> [
>   {
>     "name": "FLASK_ENV",
>     "value": "production"
>   },
>   {
>     "name": "FLASK_CONFIG",
>     "value": "testing"
>   },
>   {
>     "name": "POSTGRES_DB",
>     "value": "postgres"
>   },
>   {
>     "name": "POSTGRES_USER",
>     "value": "postgres"
>   },
>   {
>     "name": "POSTGRES_HOSTNAME",
>     "value": "localhost"
>   },
>   {
>     "name": "POSTGRES_PORT",
>     "value": "5433"
>   },
>   {
>     "name": "POSTGRES_PASSWORD",
>     "value": "postgres"
>   },
>   {
>     "name": "APPLICATION_DB",
>     "value": "test"
>   }
> ]
> 
> ```

Note that here I specified 5433 for the POSTGRES_PORT. This allows us to spin up the test database container while the development one is running, as that will use port 5432 and you can't have two different containers using the same port on the host. A more general solution could be to leave Docker pick a random host port for the container and then use that, but this requires a bit more code to be properly implemented, so I will come back to this problem when setting up the scenarios.

The last piece of setup that we need is the orchestration configuration for docker-compose

File: docker/testing.yml

> ```
> version: '3.4'
> 
> services:
>   db:
>     image: postgres
>     environment:
>       POSTGRES_DB: ${POSTGRES_DB}
>       POSTGRES_USER: ${POSTGRES_USER}
>       POSTGRES_HOSTNAME: ${POSTGRES_HOSTNAME}
>       POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
>     ports:
>       - "${POSTGRES_PORT}:5432"
>       
> ```

Now we can run ./manage.py test and get

> ```
> Creating network "testing_default" with the default driver
> Creating testing_db_1 ... done
> ========================= test session starts =========================
> platform linux -- Python 3.7.5, pytest-5.4.3, py-1.8.2, pluggy-0.13.1
> -- /home/leo/devel/flask-tutorial/venv3/bin/python3
> cachedir: .pytest_cache
> rootdir: /home/leo/devel/flask-tutorial
> plugins: cov-2.10.0
> collected 0 items
> Coverage.py warning: No data was collected. (no-data-collected)
> 
> 
> ----------- coverage: platform linux, python 3.7.5-final-0 -----------
> Name                    Stmts   Miss  Cover   Missing
> -----------------------------------------------------
> application/app.py         11     11     0%   1-21
> application/config.py      13     13     0%   1-31
> application/models.py       4      4     0%   1-5
> -----------------------------------------------------
> TOTAL                      28     28     0%
> 
> ======================= no tests ran in 0.07s =======================
> Stopping testing_db_1 ... done
> Removing testing_db_1 ... done
> Removing network testing_default
> ```

Resources¶
- [pytest](https://docs.pytest.org/en/latest/) - A full-featured Python testing framework
- [Useful pytest command line options](https://www.thedigitalcatonline.com/blog/2018/07/05/useful-pytest-command-line-options/)

***

# Step 4 - Initialise the testing database¶
When you develop a web application and then run it in production, you typically create the database once and then upgrade it through migrations. When running tests we need to create the database every time, so I need to add a way to run SQL commands on the testing database before I run pytest.

As running sql commands directly on the the database is often useful I will create a function that wraps the boilerplate for the connection. The command that creates the initial database at that point will be trivial.

File: manage.py

> ```
> #! /usr/bin/env python
> 
> import os
> import json
> import signal
> import subprocess
> import time
> 
> import click
> import psycopg2
> from psycopg2.extensions import ISOLATION_LEVEL_AUTOCOMMIT
> 
> 
> # Ensure an environment variable exists and has a value
> def setenv(variable, default):
>     os.environ[variable] = os.getenv(variable, default)
> 
> 
> setenv("APPLICATION_CONFIG", "development")
> 
> 
> def configure_app(config):
>     # Read configuration from the relative JSON file
>     with open(os.path.join("config", f"{config}.json")) as f:
>         config_data = json.load(f)
> 
>     # Convert the config into a usable Python dictionary
>     config_data = dict((i["name"], i["value"]) for i in config_data)
> 
>     for key, value in config_data.items():
>         setenv(key, value)
> 
> 
> @click.group()
> def cli():
>     pass
> 
> 
> @cli.command(context_settings={"ignore_unknown_options": True})
> @click.argument("subcommand", nargs=-1, type=click.Path())
> def flask(subcommand):
>     configure_app(os.getenv("APPLICATION_CONFIG"))
> 
>     cmdline = ["flask"] + list(subcommand)
> 
>     try:
>         p = subprocess.Popen(cmdline)
>         p.wait()
>     except KeyboardInterrupt:
>         p.send_signal(signal.SIGINT)
>         p.wait()
> 
> 
> def docker_compose_cmdline(config):
>     configure_app(os.getenv("APPLICATION_CONFIG"))
> 
>     docker_compose_file = os.path.join("docker", f"{config}.yml")
> 
>     if not os.path.isfile(docker_compose_file):
>         raise ValueError(f"The file {docker_compose_file} does not exist")
> 
>     return [
>         "docker-compose",
>         "-p",
>         config,
>         "-f",
>         docker_compose_file,
>     ]
> 
> 
> @cli.command(context_settings={"ignore_unknown_options": True})
> @click.argument("subcommand", nargs=-1, type=click.Path())
> def compose(subcommand):
>     cmdline = docker_compose_cmdline(os.getenv("APPLICATION_CONFIG")) + > list(subcommand)
> 
>     try:
>         p = subprocess.Popen(cmdline)
>         p.wait()
>     except KeyboardInterrupt:
>         p.send_signal(signal.SIGINT)
>         p.wait()
> 
> 
> def run_sql(statements):
>     conn = psycopg2.connect(
>         dbname=os.getenv("POSTGRES_DB"),
>         user=os.getenv("POSTGRES_USER"),
>         password=os.getenv("POSTGRES_PASSWORD"),
>         host=os.getenv("POSTGRES_HOSTNAME"),
>         port=os.getenv("POSTGRES_PORT"),
>     )
> 
>     conn.set_isolation_level(ISOLATION_LEVEL_AUTOCOMMIT)
>     cursor = conn.cursor()
>     for statement in statements:
>         cursor.execute(statement)
> 
>     cursor.close()
>     conn.close()
> 
> 
> @cli.command()
> def create_initial_db():
>     configure_app(os.getenv("APPLICATION_CONFIG"))
> 
>     try:
>         run_sql([f"CREATE DATABASE {os.getenv('APPLICATION_DB')}"])
>     except psycopg2.errors.DuplicateDatabase:
>         print(
>             f"The database {os.getenv('APPLICATION_DB')} already exists > and will not be recreated"
>         )
> 
> 
> @cli.command()
> @click.argument("filenames", nargs=-1)
> def test(filenames):
>     os.environ["APPLICATION_CONFIG"] = "testing"
>     configure_app(os.getenv("APPLICATION_CONFIG"))
> 
>     cmdline = docker_compose_cmdline(os.getenv("APPLICATION_CONFIG")) + > ["up", "-d"]
>     subprocess.call(cmdline)
> 
>     cmdline = docker_compose_cmdline(os.getenv("APPLICATION_CONFIG")) + > ["logs", "db"]
>     logs = subprocess.check_output(cmdline)
>     while "ready to accept connections" not in logs.decode("utf-8"):
>         time.sleep(0.1)
>         logs = subprocess.check_output(cmdline)
> 
>     run_sql([f"CREATE DATABASE {os.getenv('APPLICATION_DB')}"])
> 
>     cmdline = ["pytest", "-svv", "--cov=application", > "--cov-report=term-missing"]
>     cmdline.extend(filenames)
>     subprocess.call(cmdline)
> 
>     cmdline = docker_compose_cmdline(os.getenv("APPLICATION_CONFIG")) + > ["down"]
>     subprocess.call(cmdline)
> 
> 
> if __name__ == "__main__":
>     cli()
> ``` 
 
As you can see I took the opportunity to write the create_initial_db command as well, that just runs the very same SQL command that creates the testing database, but in any configuration I will use.

Before moving on I think it's time to refactor the manage.py file. Refactoring is not mandatory, but I feel like some parts of the script are not generic enough, and when I will add the scenarios I will definitely need my functions to be flexible.

The new script is

> ```
> #! /usr/bin/env python
> 
> import os
> import json
> import signal
> import subprocess
> import time
> 
> import click
> import psycopg2
> from psycopg2.extensions import ISOLATION_LEVEL_AUTOCOMMIT
> 
> 
> # Ensure an environment variable exists and has a value
> def setenv(variable, default):
>     os.environ[variable] = os.getenv(variable, default)
> 
> 
> setenv("APPLICATION_CONFIG", "development")
> 
> APPLICATION_CONFIG_PATH = "config"
> DOCKER_PATH = "docker"
> 
> 
> def app_config_file(config):
>     return os.path.join(APPLICATION_CONFIG_PATH, f"{config}.json")
> 
> 
> def docker_compose_file(config):
>     return os.path.join(DOCKER_PATH, f"{config}.yml")
> 
> 
> def configure_app(config):
>     # Read configuration from the relative JSON file
>     with open(app_config_file(config)) as f:
>         config_data = json.load(f)
> 
>     # Convert the config into a usable Python dictionary
>     config_data = dict((i["name"], i["value"]) for i in config_data)
> 
>     for key, value in config_data.items():
>         setenv(key, value)
> 
> 
> @click.group()
> def cli():
>     pass
> 
> 
> @cli.command(context_settings={"ignore_unknown_options": True})
> @click.argument("subcommand", nargs=-1, type=click.Path())
> def flask(subcommand):
>     configure_app(os.getenv("APPLICATION_CONFIG"))
> 
>     cmdline = ["flask"] + list(subcommand)
> 
>     try:
>         p = subprocess.Popen(cmdline)
>         p.wait()
>     except KeyboardInterrupt:
>         p.send_signal(signal.SIGINT)
>         p.wait()
> 
> 
> def docker_compose_cmdline(commands_string=None):
>     config = os.getenv("APPLICATION_CONFIG")
>     configure_app(config)
> 
>     compose_file = docker_compose_file(config)
> 
>     if not os.path.isfile(compose_file):
>         raise ValueError(f"The file {compose_file} does not exist")
> 
>     command_line = [
>         "docker-compose",
>         "-p",
>         config,
>         "-f",
>         compose_file,
>     ]
> 
>     if commands_string:
>         command_line.extend(commands_string.split(" "))
> 
>     return command_line
> 
> 
> @cli.command(context_settings={"ignore_unknown_options": True})
> @click.argument("subcommand", nargs=-1, type=click.Path())
> def compose(subcommand):
>     cmdline = docker_compose_cmdline() + list(subcommand)
> 
>     try:
>         p = subprocess.Popen(cmdline)
>         p.wait()
>     except KeyboardInterrupt:
>         p.send_signal(signal.SIGINT)
>         p.wait()
> 
> 
> def run_sql(statements):
>     conn = psycopg2.connect(
>         dbname=os.getenv("POSTGRES_DB"),
>         user=os.getenv("POSTGRES_USER"),
>         password=os.getenv("POSTGRES_PASSWORD"),
>         host=os.getenv("POSTGRES_HOSTNAME"),
>         port=os.getenv("POSTGRES_PORT"),
>     )
> 
>     conn.set_isolation_level(ISOLATION_LEVEL_AUTOCOMMIT)
>     cursor = conn.cursor()
>     for statement in statements:
>         cursor.execute(statement)
> 
>     cursor.close()
>     conn.close()
> 
> 
> def wait_for_logs(cmdline, message):
>     logs = subprocess.check_output(cmdline)
>     while message not in logs.decode("utf-8"):
>         time.sleep(0.1)
>         logs = subprocess.check_output(cmdline)
> 
> 
> @cli.command()
> def create_initial_db():
>     configure_app(os.getenv("APPLICATION_CONFIG"))
> 
>     try:
>         run_sql([f"CREATE DATABASE {os.getenv('APPLICATION_DB')}"])
>     except psycopg2.errors.DuplicateDatabase:
>         print(
>             f"The database {os.getenv('APPLICATION_DB')} already exists > and will not be recreated"
>         )
> 
> 
> @cli.command()
> @click.argument("filenames", nargs=-1)
> def test(filenames):
>     os.environ["APPLICATION_CONFIG"] = "testing"
>     configure_app(os.getenv("APPLICATION_CONFIG"))
> 
>     cmdline = docker_compose_cmdline("up -d")
>     subprocess.call(cmdline)
> 
>     cmdline = docker_compose_cmdline("logs db")
>     wait_for_logs(cmdline, "ready to accept connections")
> 
>     run_sql([f"CREATE DATABASE {os.getenv('APPLICATION_DB')}"])
> 
>     cmdline = ["pytest", "-svv", "--cov=application", > "--cov-report=term-missing"]
>     cmdline.extend(filenames)
>     subprocess.call(cmdline)
> 
>     cmdline = docker_compose_cmdline("down")
>     subprocess.call(cmdline)
> 
> 
> if __name__ == "__main__":
>     cli()
> ```

Notable changes:
- I created two new functions app_config_file and docker_compose_file that encapsulate the creation of the file paths.
- I isolated the code that waits for a message in the database container logs, creating the wait_for_logs function.
- The docker_compose_cmdline now receives a string and converts it into a list internally. This way expressing commands is more natural, as it doesn't require the ugly list syntax that subprocess works with.

## Resources¶
- Psycopg – PostgreSQL database adapter for Python

***

## Bonus step - A full TDD example¶
Before wrapping up this post, I want to give you a full example of the TDD process that I would follow given the current state of the setup, which is already complete enough to start the development of an application. Let's pretend my goal is that of adding a User model that can be created with an id (primary key) and an email fields.

First of all I write a test that creates a user in the database and then retrieves it, checking its attributes

File: tests/test_user.py

> ```
> from application.models import User
> 
> 
> def test__create_user(database):
>     email = "some.email@server.com"
>     user = User(email=email)
>     database.session.add(user)
>     database.session.commit()
> 
>     user = User.query.first()
> 
>     assert user.email == email
> ```

Running this test results in an error, because the User model does not exist

> ```
> $ ./manage.py test
> Creating network "testing_default" with the default driver
> Creating testing_db_1 ... done
> ====================================== test session starts > ======================================
> platform linux -- Python 3.7.5, pytest-5.4.3, py-1.9.0, pluggy-0.13.1 --
> /home/leo/devel/flask-tutorial/venv3/bin/python3
> cachedir: .pytest_cache
> rootdir: /home/leo/devel/flask-tutorial
> plugins: flask-1.0.0, cov-2.10.0
> collected 0 items / 1 error
> ============================================ ERRORS > =============================================
> ___________________________ ERROR collecting tests/tests/test_user.py > ___________________________
> ImportError while importing test module > '/home/leo/devel/flask-tutorial/tests/tests/test_user.py'.
> Hint: make sure your test modules/packages have valid Python names.
> Traceback:
> venv3/lib/python3.7/site-packages/_pytest/python.py:511: in > _importtestmodule
>     mod = self.fspath.pyimport(ensuresyspath=importmode)
> venv3/lib/python3.7/site-packages/py/_path/local.py:704: in pyimport
>     __import__(modname)
> venv3/lib/python3.7/site-packages/_pytest/assertion/rewrite.py:152: in > exec_module
>     exec(co, module.__dict__)
> tests/tests/test_user.py:1: in <module>
>     from application.models import User
> E   ImportError: cannot import name 'User' from 'application.models'
>     (/home/leo/devel/flask-tutorial/application/models.py)
> 
> ----------- coverage: platform linux, python 3.7.5-final-0 -----------
> Name                    Stmts   Miss  Cover   Missing
> -----------------------------------------------------
> application/app.py         11      9    18%   6-21
> application/config.py      14     14     0%   1-32
> application/models.py       4      0   100%
> -----------------------------------------------------
> TOTAL                      29     23    21%
> 
> =================================== short test summary info > ===================================
> ERROR tests/tests/test_user.py
> !!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!! Interrupted: 1 error > during collection !!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
> ====================================== 1 error in 0.20s > =======================================
> Stopping testing_db_1 ... done
> Removing testing_db_1 ... done
> Removing network testing_default
> $ 
> ```

<span style="Color:green">

Got really stuck on this getting this error when running ``` ./manage.py test```
```
    from application.app import create_app
E   ModuleNotFoundError: No module named 'application'
```
Learning outcome:  if you will be importing code form one .py into another .py  you need to touch ```__init__.py``` in every folder that has .py's you might be importing else where.

So in folders 

</span>










I won't show here all the steps of the strict TDD methodology, and implement directly the final solution, which is

File: application/models.py

> ```
> from flask_sqlalchemy import SQLAlchemy
> from flask_migrate import Migrate
> 
> db = SQLAlchemy()
> migrate = Migrate()
> 
> 
> class User(db.Model):
>     __tablename__ = "users"
>     id = db.Column(db.Integer, primary_key=True)
>     email = db.Column(db.String, unique=True, nullable=False)
> ```    
    
With this model the test passes

> ```
> $ ./manage.py test
> Creating network "testing_default" with the default driver
> Creating testing_db_1 ... done
> ================================== test session starts > ==================================
> platform linux -- Python 3.7.5, pytest-5.4.3, py-1.9.0, pluggy-0.13.1 --
> /home/leo/devel/flask-tutorial/venv3/bin/python3
> cachedir: .pytest_cache
> rootdir: /home/leo/devel/flask-tutorial
> plugins: flask-1.0.0, cov-2.10.0
> collected 1 item                                                          >                                                                 
> 
> tests/test_user.py::test__create_user PASSED
> 
> ----------- coverage: platform linux, python 3.7.5-final-0 -----------
> Name                    Stmts   Miss  Cover   Missing
> -----------------------------------------------------
> application/app.py         11      1    91%   19
> application/config.py      14      0   100%
> application/models.py       8      0   100%
> -----------------------------------------------------
> TOTAL                      33      1    97%
> 
> 
> =================================== 1 passed in 0.14s > ===================================
> Stopping testing_db_1 ... done
> Removing testing_db_1 ... done
> Removing network testing_default
> $ 
> ```

Please not that this is a very simple example and that in a real case I would add some other tests before accepting this code. In particular we should check that the email field can be empty, and maybe also test some validation on that field.

Once we are satisfied by the code we can generate the migration in the database. Spin up the development environment with

> ```
> $ ./manage.py compose up -d
> ```

If this is the first time I spin up the environment I have to create the application database and to initialise the migrations, so I run

> ```
> $ ./manage.py create-initial-db
> ```

which returns no output, and

> ```
> $ ./manage.py flask db init
>   Creating directory /home/leo/devel/flask-tutorial/migrations ...  done
>   Creating directory /home/leo/devel/flask-tutorial/migrations/versions > ...  done
>   Generating /home/leo/devel/flask-tutorial/migrations/env.py ...  done
>   Generating /home/leo/devel/flask-tutorial/migrations/README ...  done
>   Generating /home/leo/devel/flask-tutorial/migrations/script.py.mako ... >  done
>   Generating /home/leo/devel/flask-tutorial/migrations/alembic.ini ...  > done
>   Please edit configuration/connection/logging settings in > '/home/leo/devel/flask-tutorial/migrations/alembic.ini' before > proceeding.
> ```

Once this is done (or if that was already done), I can create the migration with

> ```
> $ ./manage.py flask db migrate -m "Initial user model"
> INFO  [alembic.runtime.migration] Context impl PostgresqlImpl.
> INFO  [alembic.runtime.migration] Will assume transactional DDL.
> INFO  [alembic.autogenerate.compare] Detected added table 'users'
>   Generating /home/leo/devel/flask-tutorial/migrations/versions/7a09d7f8a8> fa_initial_user_model.py ...  done
> ```  

and finally apply the migration with

> ```
> $ ./manage.py flask db upgrade
> INFO  [alembic.runtime.migration] Context impl PostgresqlImpl.
> INFO  [alembic.runtime.migration] Will assume transactional DDL.
> INFO  [alembic.runtime.migration] Running upgrade  -> 7a09d7f8a8fa, > Initial user model
> ```

After this I can safely commit my code and move on with the next requirement.

### Final words¶
I hope this post already showed you why a good setup can make the difference. The project is clean and wrapping the command in the management script plus the centralised config proved to be a good choice as it allowed me to solve the problem of migrations and testing in (what I think is) an elegant way. In the next post I'll show you how to easily create scenarios where you can test queries with only specific data in the database. If you find my posts useful please share them with whoever you thing might be interested.









***

Then: https://www.thedigitalcatonline.com/blog/2020/07/07/flask-project-setup-tdd-docker-postgres-and-more-part-3/