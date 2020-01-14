# 1 Insetall Python and flask
## 1.1 Install Python
pass
## 1.2 Install flask
We use pipenv tool to manage package during the tutorial

```bash
# mkdir microblog && cd microblog
# pipenv install --python 3.7
# pipenv install flask
```
In order to improve the package downloading speed, edit the Pipfile and set the url as following:
```bash
url = "https://pypi.douban.com/simple"
```
>NOTE: the direcory notes under flask_tutorial is just for purpose of storing the notes. Eerything for the project is under the microblog.

Assuming the current work directory is `microblog` , and let us create a "Hello World" application

The applicatoi will exist in a package . In Python, a sub-direcotry that includes a __init__.py file is 
considered a package, and can be imported. When you import a package, the __init__.py
executes and defines what symbols the exposes to the outside world.

Let's create a package called `app` , that will host the application. Make sure you are in the `microblog` directory and then run the following command:
```bash
$ mkdir app && cd app
$ touch __init__.py
```
the `__init__.py` for the `app` package is going to contain the the following code:

```python
from flask import Flask

app = Flask(__name__)

from app import routes
```

The script above simley creates the application object as an instance of class `Flask` imported from the flask package. The `__name__` variable passed to the `Flask` class
is a Python **predefined variable** , which si set to the name of the module in which it is used. **Flask uses the location of the module paased here as a starting point when it needs to load associated resources such as template files. For all practical purposes, passing `__name__` is almost always going to configure Flask in the correct way** . The application then imports the `routes` module , which doesn't exist yet.

One aspect that may seem confusing at first is that there are two entities named `app` . The `app` package is defined by the *app* directory and the `__init__.py` script, and is referenced in the `from app import routes` statement. The **app** variable is defined as instance of class `Flask` int he `__init__.py` script, **which make it a memner of the `app` package** .

Another peculiarity is that the `routes` module is imported at the bottom and not at the top of the script as it always done. The bottom import is a workaround to *circular* imports, a common problem with Flask applications. You are going to see that **the `routes` module needs to import the `app` variable defined in this script ,so putting one of the reciprocal imports at the bottom avoids the error that results from the mutual references between these two files**.

So what goes in the `routes` module? The routes are the different URLs that the application implements. **In Flask, handlers for the application routes are written as Python functions, called *view functions* , View functions are mapped to one or more route URLs so that Flask knows what logic to execute when a client requests a given URL** .

Here is your first view function , which you need to write in the new module named `app/routes.py` :

```python
from app import app


@app.route('/')
@app.route('/index')
def index():
    return "Hello, World!"
```

The two strange `@app.route` lines above the function are decorators, a unique feature of the Python language. A decorator modifies the function that follows it. **A common pattern with decorators is to use them to register functions as callbacks for certain events**. In this case, the `@app.route` decorator creates an association between the URL given as an argument and the function. In this example there are two decorators, which associate the URLs `/` and `/index` to this function. This means that when a web browser requests either of these two URLs, Flask is going to invoke this function and pass the return value of it back to the browser as a response. 

**To complete the application, you need to have a Python script at the top-level that defines the Flask application instance**. Let's call this script microblog.py, and define it as a single line that imports the application instance:
`microblog/microblog.py`

```python
from app import app
```

**The Flask application instance is called `app` and is a member of the *app* package**. The `from app import app` statement imports the app variable that is a member of the app package. 

Just to make sure that you are doing everything correctly, below you can see a diagram of the project structure so far:
```bash
$ tree microblog/
microblog/
├── app
│   ├── __init__.py
│   └── routes.py
├── microblog.py
├── Pipfile
└── Pipfile.lock
```

Before running it, though, Flask needs to be told how to import it, by setting the `FLASK_APP` environment variable:

```bash
$ pwd
/path/to/microblog
$ export FLASK_APP=microblog.py
```

You can run your first web application, with the following command:

```bash
$ pwd
/path/to/microblog
$ pipenv shell
$ flask run
* Serving Flask app "microblog.py"
* Environment: production
  WARNING: This is a development server. Do not use it in a production deployment.
  Use a production WSGI server instead.
* Debug mode: off
* Running on http://127.0.0.1:5000/ (Press CTRL+C to quit)
```

Network servers listen for connections on a specific port number. Applications deployed on production web servers typically listen on port 443, or sometimes 80 if they do not implement encryption, but access to these ports require administration rights. Since this application is running in a development environment, Flask uses the freely available port 5000. 

Before I end this chapter, I want to mention one more thing. **Since environment variables aren't remembered across terminal sessions, you may find tedious to always have to set the `FLASK_APP` environment variable when you open a new terminal window**. Starting with version 1.0, Flask allows you to register environment variables that you want to be automatically imported when you run the flask command. To use this option you have to install the `python-dotenv` package:

```bash
$ pwd
/path/to/microblog
$ pipenv install python-dotenv
```
Then you can just write the environment variable name and value in a `.flaskenv` file **in the top-level directory of the project**:

```bash
FLASK_APP=microblog.py
```
Doing this is optional. If you prefer to set the environment variable manually, that is perfectly fine, as long as you always remember to do it.