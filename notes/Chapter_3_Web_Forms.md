# Web Forms
In Chapter 2 I created a simple template for the home page of the application, and used fake objects as placeholders for things I don't have yet, like users or blog posts. In this chapter I'm going to address one of the many holes I still have in this application, specifically how to accept input from users through web forms.

Web forms are one of the most basic building blocks in any web application. I will be using forms to allow users to submit blog posts, and also for logging in to the application.

## Introduction to Flask-WTF

To handle the web forms in this application I'm going to use the `Flask-WTF` extension, which is a thin wrapper around the `WTForms` package that nicely integrates it with Flask. Extensions are a very important part of the Flask ecosystem, as they provide solutions to problems that Flask is intentionally not opinionated about.

Flask extensions are regular Python packages that are installed with pip . You can go ahead and install Flask-WTF in you virtual environment:

```bash
$ pwd
/path/to/microblog
$ pipenv install flask-wtf
```
## Configuration
So far the application is very simple, and for that reason I did not need to wory about its *configuration* . But for any applications except  the simplest ones, you are going to find that Flask (and possibly also the Flask extensions that you use) offer some amount of freedom in how to do things , and **you need to make some decisions, which you pass to the framework as a list of configuration variables**.

There are several formats for the application to specify configuration options. The most basic solution is to define your variales as keys in `app.config` , which uses a dictionary style to work with variables. For example, you could do something like this:

```python
app = Flask(__name__)
app.config['SECRET_KEY'] = 'you-will-never-guess'
# ... and more variables here s needed
```
While the above syntax is sufficient to create cofiguration options for Flask, **I like to enforce the principle of seperation of cocerns** , so instead of puting my configuration in the same palce where I create my application I will use a slightly **more elaborate structure that allows me to keep my configuration in a separate file**.

A format that I really like because it is very extensible , is to **use a class to store configuration variables**. To keep things nicely organied, I'm going to **create the configuration class in a separate Python module**. Below you can see the new configuration class for this application, stored in a `config.py` module in the **top-level directory** .

```python
# config.py : Secret key configuration

import os


class Config(object):
    SECRET_KEY = os.environ.get('SECRET_KEY') or 'you-will-never-guess'
```

**he configuration settings are defined as class variables inside the `Config` class. As the application needs more configuration items, they can be added to this class, and later if I find that I need to have more than one configuration set, I can create subclasses of it**.

The `SECRET_KEY` configuration variable that I added as the only configuration item is an important part in most Flask applications. **Flask and some of its extentsions use the value of the secret key as a cryptographic key, useful to generate signatures or tokens**. The Flask-WTF extension uses it to protect web forms against a nasty attack called *Cross-Site Request Forgery* or CSRF (pronounced "seafurf"). As its name implies, **the secret key is supposed to be secret, as the strength of the tokens and signatures generated with it depends on no person outside of the trusted maintainers of the application knowing it**.

**The value of the secret key is set as an expression with two terms, joined by the `or` operator. The first item looks for the value of an environment variable, also called `SECRET_KEY` . The second term, is just a hardcoded string**. This is a pattern that you will see me repeat often for configuration variables. The idea is that a value sourced from an  environment variable is preferred, but if the environment doest not define the variable, then the hardcodeed string is used instead. When you are developing this application , the security requirements are low, so you can just ignore this setting and let the hardcoded string be used. **But when this application is deployed on a production server, I will be setting a unique and difficult to guess value in the environment, so that the server has a secure key that nobody else knows**.

Now that I have a config file , I need to tell Flask to read it and apply it. That can be done right after the Flask application instance is created using the `app.config.from_object()` method:

```python
# app/__init__.py : Flask configuration

from flask import Flask
from config import Config

app = Flask(__name__)
app.config.from_object(Config)

from app import routes
```

The way I'm importing the `Config` class may seem confusing at first, but if you look at how the `Flask` class (uppercase "F") is imported from the `flask` package(lowercase "f") you'll notice that I'm doing the same with the configuration. **The lowercase "config" is the name of the Python module `config.py` , and pobviously the one with the uppercase "C" is the actual class.

As I mentioned above, the configuratioin items can be accessed with a dictionary syntax from `app.config` . Here you can see a quick session with the Python interpreter where I check what is the value of the secret key:
```python
>>> from microblog import app
>>> app.config['SECRET_KEY']
'you-will-never-guess'
```

## User Login Form
**The Flask-WTF extension uses Python classes to represent web forms. A form class simply defines the fields of the form as class cariables**.

**Once again having separation of concerns in mind, I'm going to use a new `app/forms.py` module to store my web form classes**. To begin, let's define a user login form, which asks the user to enter a username and a password. The form will also include a "remember me" check box, and a submit button:
```python
# app/forms.py : Login form

from flask_wtf import FlaskForm
from wtforms import StringField, PasswordField, BooleanField, SubmitField
from wtforms.validators import DataRequired


class LoginForm(FlaskForm):
    username = StringField('Username', validators=[DataRequired()])
    password = PasswordField('Password', validators=[DataRequired()])
    remember_me = BooleanField('Remember Me')
    submit = SubmitField('Sign In')
```
Most Flask extensions use a `flask_<name>` naming convention for their top-level import symbol. In this case, Flask-WTF has all its symbols under `flask_wtf` . This is where the `FlaskForm` base class is imported from at the top of `app/forms.py` .

The four classes that represent the field types that I'm using for this form are imported directly from the WTForms package, since the Flask-WTF extension does not provide customized versions. For each field , an object is created as a class variable in the `LoginForm` class. Each field is given a description or label as a first argument.

The optional `validators` argument that you see in some of the fields is used to attach validation behaviors to fields. The `DataRequired` validator simply checks that the fields is not sunmitted empty. There are many more validators available, some of which will be used in other forms.
## Form Templates
The next step is to add the form to an HTML template so that it can be rendered on a web page. The good news is that the fields that are defined in the `LoginForm` class know how to render themselves as HTML, so this task is fairly simple. Below you can see the login template, which I'm going to store in file `app/templates/login.html` :

```html
{% extends "base.html" %}

{% block content %}
    <h1>Sing In</h1>
   <form action="", method="POST" novalidate>
       {{ form.hidden_tag() }}
       <p>
           {{ form.username.label }}<br />
           {{ form.username(size=32)}}
       </p>
       <p>
           {{ form.password.label }}<br />
           {{form.password(size=32)}}
       </p>
       <p>{{ form.remember_me() }} {{ form.remember_me.label }}</p>
       <p>{{ form.submit() }}</p>
   </form> 
{% endblock %}
```
For this template I'm reusing one more time the `base.html` template as shown in Chapter 2, through the `extends` template inheritance statement. I will actually do this with all the templates, to ensure a consistent of layout that includes a top navigation bar across all the pages of the application.

This template expects a form object instantiated from the `LoginForm` class to be given as an argument, which you can see referenced as `form` . This argument will be sent by the login view function, which I still haven't written.