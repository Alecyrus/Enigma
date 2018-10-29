# Allegro

[![Join the chat at https://gitter.im/Alecyrus/Lobby](https://badges.gitter.im/Alecyrus/Lobby.svg)](https://gitter.im/Alecyrus/Lobby?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)
[![Build Status](https://travis-ci.org/Alecyrus/Allegro.svg?branch=master)](https://travis-ci.org/Alecyrus/Allegro)
[![PyPI version](https://img.shields.io/pypi/pyversions/allegro.svg)](https://pypi.python.org/pypi/Allegro)
[![PyPI](https://img.shields.io/pypi/v/allegro.svg)](https://pypi.python.org/pypi/Allegro)

Allegro is a python backend integration framework, which provides a simple way to make the task of building any kind of RESTFul system easy and efficient. The RESTFul service is based on [sanic](https://github.com/channelcat/sanic), and [Celery](http://www.celeryproject.org/) provides tasks scheduler service. 

With the framework, the API can be defined through a config file, and we just focus on the processing logic. What's more, it' very efficient and easy for frond-end to build a mock server.

**Important:** This is **not** library for integration with Allegro.pl REST API

## Installation
> `pip3 install allegro`

It only supports Python3.5 or higher.

## Get Started
### start.py
```python
from allegro import Allegro

app = Allegro("test_project")
app.initialize("test.ini")
app.start()
```

### stop.py
```python
from allegro import Allegro

app = Allegro("test_project")
app.initialize("test.ini")
app.stop()
```

### task1.py
```python
from celery import Celery

app = Celery('tasks', backend='redis://localhost:6379/0', broker = 'redis://localhost:6379/0')


@app.task
def get(message):
    print(message)
    return {"app":"Get Got"}

@app.task
def post(message):
    print(message)
    return {"app":"Post Got"}

```
### settings.ini
```ini
[basic]
;The ip address of the host you runs the RESTFul API service
bind_host = 0.0.0.0
;The port
bind_port = 8000
;The number of the RESTFul API service process
api_worker = 3
;The directory where the consumer modules you defined can be found 
root_path = /home/luze/Code/Allegro/examples
;The file that contains the pids of all the process
pid_path = /tmp/my_project_pid_file
;Timeout
timeout = 60

[service]
;The services your projects provided. If you want to define more than
;one service, separate each service's name by a comma. And then you must 
;define every service's specific information with a section named after
;the service'name.
keys = Test1Service


[Test1Service]
;The uri of the service. And the api can be accessed by the url;
;(http://bind_host:bind_port/uri)
uri = /test1
;The HTTP method the service's API provided.
method = get,post
;The module where the handler class is located
module = task1
;The number of the service workers. If the value of eventlet_enabled is True, the item will be of no effect
workers = 3
processes_pool = 10
;use eventlet  
eventlet_enabled = True
eventlet_pool = 1000

file_upload_enabled = False
```


#### Run the command:
> `# python3 start.py`

#### Call the API(example: http://localhost:8000/test1 (POST))

#### Respose:
>`{"app":"Get Got"}`

#### Run `python3 stop.py` to terminate the program.

## License
Allegro is open source and released under the MIT Licence.
