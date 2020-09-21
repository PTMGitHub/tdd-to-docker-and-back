# tdd-to-docker-and-back

This is based on the below tutorial.

I did this one becuse unlike previous one that use these tools, it also includes TDD.

**Tutorial Source:** 
Flask project setup: TDD, Docker, Postgres and more - Part 1
https://www.thedigitalcatonline.com/blog/2020/07/05/flask-project-setup-tdd-docker-postgres-and-more-part-1/

**Local Folder:** ApprenticeAcademy/api_flask_docker/tdd-to-docker-and-back

**GitHub:** git@github.com:PTMGitHub/tdd-to-docker-and-back.git

***
## Log

|     |     |
| --- | --- |
| **Date** | **Description** |
| 14 Sep 2020 | Start project, inital commit |
| 15 Sep 2020 | Commit at the end of step 3. Its doesnt quite work i still need to use FLASK_CONFIG="development". Going to go through it with Steve|
| 15 Sep 2020 | 2nd Commit of the day - End of Step 3 - Working. Discussed with Steve and both concluded FLASK_CONFIG should be set in the development.json|
| 21 Sep 2020 | Completed the first tutorial.|

***
## Tutorial Notes
**NB:** <span style="color:green">Text in green are additional notes I have added</span>

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
$ virtualenv .venv
```
To activate the environment: 	
```
$ source .venv/bin/activate
```
This one installs any old venv creates a news one and installs all the requirements in one
```
$ rm -r .venv && virtualenv .venv && source .env/bin/activate && pip install -r requirements.txt
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

Then: https://www.thedigitalcatonline.com/blog/2020/07/07/flask-project-setup-tdd-docker-postgres-and-more-part-3/
