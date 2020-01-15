In this chapter we will continue working on the same application at the end of chapter 1, and in particular, we are going to learn how to generate more elaborate web pages that have a complex structure and many dynamic components.

# What are Tmeplattes?
I want the home page of my microblog application to have a heading that welcomes the user. For the moment, I'm going to ignore the fact that the application does not have the concept of users yet, as this is going to come later. Instead , I'm going yo use a *mock* user, which i'm going yo implement as a Python dictionary , as follows:
```python
user = {'username': 'Miguel'}
```
Creating mock objects is a useful technique that allows you to concentrate on one part of the application without having to worry about other parts of the system that don't exist yet. 

The view function in the application returns a simple string . What I want to do now is expand that returned string into a complete HTML page, maybe something like this:
```python
# app/routes.py

from app import app

@app.route('/')
@app.route('/index')
def index():
    user = {'username': 'Miguel'}
    return '''
    <html>
        <head>
            <title>Home Page - Mircoblog</title>
        </head>
        <body>
            <h1>Hello, ''' + user['username'] + '''!</h1>
        </body>
    </html>'''
```

I hope you are agree with that the solution used above to deliver HTML to the browser is not good. 

If you could keep the logic of your application separate from the layout or presentation of your web pages , then things would be much better organized. You could even hire a web designer to create a killer web site while you code the application logic in Python.

Templates help achieve this separation between prensentation and business logic. In Flask, templates are written as separate files , stored in a *template* folder that is inside the application package. So after making sure that you are in the `microblog` directory, create the directory where templates will be stored:

```bash
$ pwd
/path/to/microblog
$ mkdir app/templates
```

Below you can see your first temlate, which is similar in functionality to the HTML page returned by the `index()` view function above. Write this file in `app/templates/index.html` :

```html
<html>
    <head>
        <title>{{ title }} - Microblog</title>
    </head>
    <body>
        <h1>Hello, {{ user.name }}</h1>
    </body>
</html>
```
This is a mostly standard , very simply HTML page. The only interesting thing in this page is that there are a couple of palceholders for the dynamic content, enclosed in {{ ... }} sections. The paceholders represent the parts of the page that are variable and will only be known at runtime.

Now that the presentation of the page was offloaded to the HTML template, the view function can be simplified:

```python
# app/routes.py

from flask import render_template
from app import app

@app.route('/')
@app.route('/index')
def index():
    user = {"username": "Miguel"}
    return render_template('index.html', title='Home', user=user)
```

The operation that converts a template into a complete HTML page is called *rendering* . To render the template I had to import a function that comes with the Flask framework called `render_template()` . This function takes a template filename and a variable list of template arguments and returns the same template , but with all the placeholders in it replaced with actual values.

The `render_template()` function invokes the Jinja2 tempalte engine that comes buldled with the Flask framework. Jinja2 substitutes `{{ ... }}` blocks with the corresponding values, given by the arguments provided in the `render_template()` call.

## Template Inheritance
Most web applications these days have a navigation bar at the top of the page with a few frequently used links, such as a link to edit your profile, to login, logout, etc. I can easi;y add a navigation bar to the `index.html` template with some more HTML, but as the application grows I will be needing this same navigation bar in other pages. I don't really want to have to maintain several copies of the navigation bar in many HTML templates, it is a good practice to not repeat yourself if that is possible.

Jinja2 has template inheritance feature that specifically addresses this problem. In essence, what you can do is move the parts of the page layout that are common to all templates to a base template, from which all other templates are derived.

So what I'm going to do now is define a base template called `base.html` that includes a simple navigation bar and also the title logic I implemented earlier. You need to write the following template in file `app/templates/base.html` :

```html
<html>
    <head>
        {% if title %}
        <title>{{ title }} - Microblog</title>
        {% else %}
        <title>Welcome to Microblog</title>
        {% endif %}
    </head>
    <body>
        <div>Microblog: <a href="/index">Home</a></div>
        <hr>
        {% block content %}{% enblock %}
    </body>
</html>
```

In this template I used the `block` control statement to define the place where the derived tempaltes can insert themselves. Blocks are given a unique name, which derived templates can reference when they provide theire content.

With the base template in place, I can now simplify `index.html` by making it inherit from `base.html` :

```html
{% extends "base.html" %}

{% block content %}
    <h1>Hi, {{ user.name }}!</h1>
    {% for post in posts %}
        <div><p>{{ post.author.username }} says: <b>{{ post.body }}</b></p></div>
    {% endfor %}
{% endblock %}
```

Since the `base.html` template will now take care of the general page structure , I have removed all those elements from `index.html` and left only the content part. The `extends` statement establishes the inheritance link between the two templates, so that Jinja2 knows that when it is asked to render `index.html` it need to embed it inside `base.html` . The two templates have matching `block` statements with name `content` , and this is how Jinja2 knows how to combine the two templates into one. Now if I need to create additional pages for the application, I can create them as derived templates from the same `base.html` templates, and that is how I can have all the pages of the application sharing the same look and feel without duplication.

