# INSTALL PYTHON LATEST VERSION
```
sudo apt-get install build-essential checkinstall
sudo apt-get install libreadline-gplv2-dev libncursesw5-dev libssl-dev libsqlite3-dev tk-dev libgdbm-dev libc6-dev libbz2-dev libffi-dev zlib1g-dev
cd /opt
sudo wget https://www.python.org/ftp/python/3.8.0/Python-3.8.0.tgz
sudo tar xzf Python-3.8.0.tgz
cd Python-3.8.0
sudo ./configure --enable-optimizations
sudo make altinstall
cd /opt
sudo rm -f Python-3.8.0.tgz
```


# PYTHON DOESN'T CALL THE LATEST VERSION
.bashrc:
```
    alias python=python3.8
    alias python3=python3.8
```


# VSCODE KEYBOARD SHORTCUT
ctrl+k+s


# VSCODE PYLINT
ctrl+shift+p > settings.json:
```
    {
        ...
        "python.pythonPath": "venv/bin/python",
        "python.linting.pylintPath": "venv/bin/pylint"
    }
```


# VIRTUAL ENVIRONMENT
```
python3 -m venv myvenv
source myvenv/bin/activate
```


# ######## VIRTUAL ENVIRONMENT ########

# INSTALLING DJANGO 
```
python -m pip install --upgrade pip
```

```
djangogirls
├── myvenv
│   └── ...
└───requirements.txt
```

requirements.txt:
```
    Django~=2.2.4
```

```
pip install -r requirements.txt
```


# INSTALLING GIT
```
sudo apt install git
```


# CREATE DJANGO PROJECT
```
django-admin startproject mysite .
```

mysite/settings.py:
```
    TIME_ZONE = 'America/Sao_Paulo'
    LANGUAGE_CODE = 'pt-BR'
    STATIC_ROOT = os.path.join(BASE_DIR, 'static')
    ALLOWED_HOSTS = ['127.0.0.1', '.pythonanywhere.com']
```


# SET UP A DATABASE
```
python manage.py migrate
```


# STARTING THE WEB SERVER
```
python manage.py runserver
```


# CREATING AN APPLICATION INSIDE OUR PROJECT
```
python manage.py startapp blog
```

```
djangogirls
├── blog
│   ├── admin.py
│   ├── apps.py
│   ├── __init__.py
│   ├── migrations
│   │   └── __init__.py
│   ├── models.py
│   ├── tests.py
│   └── views.py
├── db.sqlite3
├── manage.py
├── mysite
│   ├── __init__.py
│   ├── settings.py
│   ├── urls.py
│   └── wsgi.py
├── myvenv
│   └── ...
└── requirements.txt
```

mysite/settings.py:
```
    INSTALLED_APPS = [
        ...
        'blog.apps.BlogConfig',
    ]
```


# CREATING A BLOG POST MODEL

blog/models.py:
```
    from django.conf import settings
    from django.db import models
    from django.utils import timezone


    class Post(models.Model):
        author = models.ForeignKey(settings.AUTH_USER_MODEL, on_delete=models.CASCADE)
        title = models.CharField(max_length=200)
        text = models.TextField()
        created_date = models.DateTimeField(default=timezone.now)
        published_date = models.DateTimeField(blank=True, null=True)

        def publish(self):
            self.published_date = timezone.now()
            self.save()

        def __str__(self):
            return self.title
```


# CREATE TABLES FOR MODELS IN YOUR DATABASE
```
python manage.py makemigrations blog
python manage.py migrate blog
```


# DJANGO ADMIN
blog/admin.py:
```
    from django.contrib import admin
    from .models import Post

    admin.site.register(Post)
```

```
python manage.py createsuperuser
```


# STARTING OUR GIT REPOSITORY
```
git init
git config --global user.name "..."
git config --global user.email ...

.gitignore:
    *.pyc
    *~
    /.vscode
    __pycache__
    myvenv
    db.sqlite3
    /static
    .DS_Store

git status
git add --all .
git commit -m "My Django Girls app, first commit"
```

Create GitHub repository: django-tutorial

```
git remote add origin https://github.com/marceljm/django-tutorial.git
git push -u origin master
```


# CONFIGURING OUR SITE ON PYTHONANYWHERE
Create a token: https://www.pythonanywhere.com/user/marceljm/account/#api_token

PYTHONANYWHERE > New console > $ Bash
```
    pip3.8 install --user pythonanywhere
    pa_autoconfigure_django.py --python=3.8 https://github.com/marceljm/django-tutorial.git
    python manage.py createsuperuser
```


# YOUR FIRST DJANGO URL!
mysite/urls.py:
```
    urlpatterns = [
        ...
        path('', include('blog.urls')),
    ]
```

blog/urls.py:
```
    from django.urls import path
    from . import views

    urlpatterns = [
        path('', views.post_list, name='post_list'),
    ]
```


# DJANGO VIEWS
blog/views.py:
```
    from django.shortcuts import render

    def post_list(request):
        return render(request, 'blog/post_list.html', {})
```


# YOUR FIRST TEMPLATE!
```
blog
└───templates
    └───blog
        └── post_list.html
```


# PULL YOUR NEW CODE DOWN TO PYTHONANYWHERE, AND RELOAD YOUR WEB APP
```
git pull
```
Reload: https://www.pythonanywhere.com/web_app_setup/ 


# DJANGO SHELL
```
python manage.py shell
```

Examples:
```
>>> from blog.models import Post
>>> Post.objects.all()
>>> from django.contrib.auth.models import User
>>> User.objects.all()
>>> me = User.objects.get(username='marceljm')
>>> Post.objects.create(author=me, title='Sample title', text='Test')
>>> Post.objects.all()
>>> Post.objects.filter(author=me)
>>> Post.objects.filter(title__contains='title')
>>> from django.utils import timezone
>>> Post.objects.filter(published_date__lte=timezone.now())
>>> post = Post.objects.get(title="Sample title")
>>> post.publish()
>>> [x.publish() for x in Post.objects.all()]
>>> Post.objects.order_by('created_date')
>>> Post.objects.order_by('-created_date')
>>> Post.objects.filter(published_date__lte=timezone.now()).order_by('published_date')
>>> exit()
```


# QUERYSET
blog/views.py:
```
    from django.shortcuts import render
    from django.utils import timezone
    from .models import Post

    def post_list(request):
        posts = Post.objects.filter(
            published_date__lte=timezone.now()).order_by('published_date')
        return render(request, 'blog/post_list.html', {'posts': posts})
```


# DISPLAY POST LIST TEMPLATE
blog/templates/blog/post_list.html:
```
<html>
    <head>
        <title>Django Girls blog</title>
    </head>
    <body>
        <div>
            <h1><a href="/">Django Girls Blog</a></h1>
        </div>
        
        {% for post in posts %}
            <div>
                <p>published: {{ post.published_date }}</p>
                <h2><a href="">{{ post.title }}</a></h2>
                <p>{{ post.text|linebreaksbr }}</p>
            </div>
        {% endfor %}
    </body>
</html>
```


# INSTALL BOOTSTRAP
```
<link rel="stylesheet" href="//maxcdn.bootstrapcdn.com/bootstrap/3.2.0/css/bootstrap.min.css">
<link rel="stylesheet" href="//maxcdn.bootstrapcdn.com/bootstrap/3.2.0/css/bootstrap-theme.min.css">
```


# WHERE TO PUT STATIC FILES FOR DJANGO
```
djangogirls
├── blog
│   ├── migrations
│   ├── static
│   └── templates
└── mysite
```

```
# YOUR FIRST CSS FILE!
djangogirls
└─── blog
     └─── static
          └─── css
               └─── blog.css
```


blog/templates/blog/post_list.html:
```
*   {% load static %}
    <html>

    <head>
        <title>Django Girls blog</title>
        <link rel="stylesheet" href="//maxcdn.bootstrapcdn.com/bootstrap/3.2.0/css/bootstrap.min.css">
        <link rel="stylesheet" href="//maxcdn.bootstrapcdn.com/bootstrap/3.2.0/css/bootstrap-theme.min.css">
        <link href='//fonts.googleapis.com/css?family=Lobster&subset=latin,latin-ext' rel='stylesheet' type='text/css'>
*       <link rel="stylesheet" href="{% static 'css/blog.css' %}">
    </head>

    <body>
        <div class="page-header">
            <h1><a href="/">Django Girls Blog</a></h1>
        </div>

        <div class="content container">
            <div class="row">
                <div class="col-md-8">
                    {% for post in posts %}
                    <div class="post">
                        <div class="date">
                            {{ post.published_date }}
                        </div>
                        <h2><a href="">{{ post.title }}</a></h2>
                        <p>{{ post.text|linebreaksbr }}</p>
                    </div>
                    {% endfor %}
                </div>
            </div>
        </div>
    </body>

    </html>
```


# CREATE A BASE TEMPLATE
```
blog
└───templates
    └───blog
            base.html
            post_list.html
```

blog/templates/blog/base.html:
```
    {% load static %}
    <html>

    <head>
        <title>Django Girls blog</title>
        <link rel="stylesheet" href="//maxcdn.bootstrapcdn.com/bootstrap/3.2.0/css/bootstrap.min.css">
        <link rel="stylesheet" href="//maxcdn.bootstrapcdn.com/bootstrap/3.2.0/css/bootstrap-theme.min.css">
        <link href='//fonts.googleapis.com/css?family=Lobster&subset=latin,latin-ext' rel='stylesheet' type='text/css'>
        <link rel="stylesheet" href="{% static 'css/blog.css' %}">
    </head>

    <body>
        <div class="page-header">
            <h1><a href="/">Django Girls Blog</a></h1>
        </div>
        <div class="content container">
            <div class="row">
                <div class="col-md-8">
*                   {% block content %}
*                   {% endblock %}
                </div>
            </div>
        </div>
    </body>

    </html>
```


blog/templates/blog/post_list.html:
```
    {% extends 'blog/base.html' %}

    {% block content %}
    {% for post in posts %}
    <div class="post">
        <div class="date">
            {{ post.published_date }}
        </div>
        <h2><a href="">{{ post.title }}</a></h2>
        <p>{{ post.text|linebreaksbr }}</p>
    </div>
    {% endfor %}
    {% endblock %}
```


# CREATE A TEMPLATE LINK TO A POST'S DETAIL
blog/templates/blog/post_list.html:
```
    <h2><a href="{% url 'post_detail' pk=post.pk %}">{{ post.title }}</a></h2>
```


# CREATE A URL TO A POST'S DETAIL
blog/urls.py:
```
    urlpatterns = [
        ...
        path('post/<int:pk>/', views.post_detail, name='post_detail'),
    ]
```


# ADD A POST'S DETAIL VIEW
blog/views.py:
```
    def post_detail(request, pk):
        post = get_object_or_404(Post, pk=pk)
        return render(request, 'blog/post_detail.html', {'post': post})
```


# CREATE A TEMPLATE FOR THE POST DETAILS
blog/templates/blog/post_detail.html:
```
    {% extends 'blog/base.html' %}

    {% block content %}
        <div class="post">
            {% if post.published_date %}
                <div class="date">
                    {{ post.published_date }}
                </div>
            {% endif %}
            <h2>{{ post.title }}</h2>
            <p>{{ post.text|linebreaksbr }}</p>
        </div>
    {% endblock %}
```


# UPDATING THE STATIC FILES ON THE SERVER
PythonAnywhere command-line:
```
    workon marceljm.pythonanywhere.com
    python manage.py collectstatic
```


# DJANGO FORMS
```
blog
   └── forms.py
```

blog/forms.py:
```
    from django import forms

    from .models import Post

    class PostForm(forms.ModelForm):

        class Meta:
            model = Post
            fields = ('title', 'text',)
```


# LINK TO A PAGE WITH THE FORM
blog/templates/blog/base.html:
```
    <a href="{% url 'post_new' %}" class="top-menu"><span class="glyphicon glyphicon-plus"></span></a>
```

blog/urls.py:
```
    urlpatterns = [
        ...
        path('post/new/', views.post_new, name='post_new'),
    ]
```

blog/views.py:
```
    def post_new(request):
        form = PostForm()
        return render(request, 'blog/post_edit.html', {'form': form})
```

blog/templates/blog/post_edit.html:
```
    {% extends 'blog/base.html' %}

    {% block content %}
        <h2>New post</h2>
        <form method="POST" class="post-form">{% csrf_token %}
            {{ form.as_p }}
            <button type="submit" class="save btn btn-default">Save</button>
        </form>
    {% endblock %}
```


# SAVING THE FORM
blog/views.py:
```
def post_new(request):
    if request.method == "POST":
        form = PostForm(request.POST)
        if form.is_valid():
            post = form.save(commit=False)
            post.author = request.user
            post.published_date = timezone.now()
            post.save()
            return redirect('post_detail', pk=post.pk)
    else:
        form = PostForm()
    return render(request, 'blog/post_edit.html', {'form': form})
```


# EDIT FORM
blog/templates/blog/post_detail.html:
```
    <a class="btn btn-default" href="{% url 'post_edit' pk=post.pk %}"><span class="glyphicon glyphicon-pencil"></span></a>
```

blog/urls.py:
```
    urlpatterns = [
        ...
        path('post/<int:pk>/edit/', views.post_edit, name='post_edit'),
    ]
```

blog/views.py:
```
    def post_edit(request, pk):
        post = get_object_or_404(Post, pk=pk)
        if request.method == "POST":
            form = PostForm(request.POST, instance=post)
            if form.is_valid():
                post = form.save(commit=False)
                post.author = request.user
                post.published_date = timezone.now()
                post.save()
                return redirect('post_detail', pk=post.pk)
        else:
            form = PostForm(instance=post)
        return render(request, 'blog/post_edit.html', {'form': form})
```


# SECURITY
blog/templates/blog/base.html:
```
    {% if user.is_authenticated %}
        <a href="{% url 'post_new' %}" class="top-menu"><span class="glyphicon glyphicon-plus"></span></a>
    {% endif %}
```

blog/templates/blog/post_detail.html:
```
    {% if user.is_authenticated %}
        <a class="btn btn-default" href="{% url 'post_edit' pk=post.pk %}"><span class="glyphicon glyphicon-pencil"></span></a>
    {% endif %}
```


# SAVE NEW POSTS AS DRAFTS
Remove from blog/views.py:
```
    post.published_date = timezone.now()
```

blog/templates/blog/base.html :
```
    <a href="{% url 'post_draft_list' %}" class="top-menu"><span class="glyphicon glyphicon-edit"></span></a>
```

blog/urls.py:
```
    urlpatterns = [
        ...
        path('drafts/', views.post_draft_list, name='post_draft_list'),
    ]    
```

blog/views.py:
```
    def post_draft_list(request):
        posts = Post.objects.filter(published_date__isnull=True).order_by('created_date')
        return render(request, 'blog/post_draft_list.html', {'posts': posts})
```

blog/templates/blog/post_draft_list.html:
```
    {% extends 'blog/base.html' %}

    {% block content %}
    {% for post in posts %}
    <div class="post">
        <p class="date">created: {{ post.created_date|date:'d-m-Y' }}</p>
        <h1><a href="{% url 'post_detail' pk=post.pk %}">{{ post.title }}</a></h1>
        <p>{{ post.text|truncatechars:200 }}</p>
    </div>
    {% endfor %}
    {% endblock %}
```


# ADD PUBLISH BUTTON
blog/templates/blog/post_detail.html:
```
    ...
    {% if post.published_date %}
    <div class="date">
        {{ post.published_date }}
    </div>
    {% else %}
    <a class="btn btn-default" href="{% url 'post_publish' pk=post.pk %}">Publish</a>
    {% endif %}
    ...
```

blog/urls.py:
```
    path('post/<pk>/publish/', views.post_publish, name='post_publish'),
```

blog/views.py:
```
    def post_publish(request, pk):
        post = get_object_or_404(Post, pk=pk)
        post.publish()
        return redirect('post_detail', pk=pk)
```


# DELETE POST
blog/templates/blog/post_detail.html:
```
    <a class="btn btn-default" href="{% url 'post_remove' pk=post.pk %}"><span class="glyphicon glyphicon-remove"></span></a>
```

blog/urls.py:
```
    urlpatterns = [
        ...
        path('post/<pk>/remove/', views.post_remove, name='post_remove'),
    ]
```

blog/views.py:
```
    def post_remove(request, pk):
        post = get_object_or_404(Post, pk=pk)
        post.delete()
        return redirect('post_list')
```


# AUTHORIZING ADD/EDIT OF POSTS
blog/views.py:
```
    from django.contrib.auth.decorators import login_required

    @login_required
    def post_new(request):
        [...]

    @login_required
    def post_edit(request, pk):
        [...]

    @login_required
    def post_draft_list(request):
        [...]

    @login_required
    def post_remove(request, pk):
        [...]

    @login_required
    def post_publish(request, pk):
        [...]                        
```


# LOG IN USERS
mysite/urls.py:
```
    urlpatterns = [
        [...]
        path('accounts/login/', views.LoginView.as_view(), name='login'),
        [...]
    ]
```

blog/templates/registration/login.html:
```
    {% extends "blog/base.html" %}

    {% block content %}
        {% if form.errors %}
            <p>Your username and password didn't match. Please try again.</p>
        {% endif %}

        <form method="post" action="{% url 'login' %}">
        {% csrf_token %}
            <table>
            <tr>
                <td>{{ form.username.label_tag }}</td>
                <td>{{ form.username }}</td>
            </tr>
            <tr>
                <td>{{ form.password.label_tag }}</td>
                <td>{{ form.password }}</td>
            </tr>
            </table>

            <input type="submit" value="login" />
            <input type="hidden" name="next" value="{{ next }}" />
        </form>
    {% endblock %}
```

mysite/settings.py:
```
    LOGIN_REDIRECT_URL = '/'
```


# IMPROVING THE LAYOUT
blog/templates/blog/base.html:
```
    {% if user.is_authenticated %}
        [...]
        <p class="top-menu">Hello {{ user.username }} <small>(<a href="{% url 'logout' %}">Log out</a>)</small></p>
    {% else %}
        <a href="{% url 'login' %}" class="top-menu"><span class="glyphicon glyphicon-lock"></span></a>
    {% endif %}
```

mysite/urls.py:
```
    urlpatterns = [
        [...]
        path('accounts/logout/', views.LogoutView.as_view(next_page='/'), name='logout'),
        [...]
    ]

```


# USER PERMISSIONS
http://127.0.0.1:8000/admin > add new users > give Post permissions: "blog | post | Can add/change/delete/view post"


blog/views.py:
```
    @login_required
    @permission_required('blog.add_post')
    def post_new(request):
        [...]

    @login_required
    @permission_required('blog.change_post')
    def post_edit(request, pk):
        [...]

    @login_required
    @permission_required('blog.view_post')
    def post_draft_list(request):
        [...]

    @login_required
    @permission_required('blog.change_post')
    def post_publish(request, pk):
        [...]      

    @login_required
    @permission_required('blog.delete_post')
    def post_remove(request, pk):
        [...]                    
```


# CREATING COMMENT BLOG MODEL
blog/models.py:
```
    [...]
    class Comment(models.Model):
        post = models.ForeignKey(
            'blog.Post', on_delete=models.CASCADE, related_name='comments')
        author = models.CharField(max_length=200)
        text = models.TextField()
        created_date = models.DateTimeField(default=timezone.now)
        approved_comment = models.BooleanField(default=False)

        def approve(self):
            self.approved_comment = True
            self.save()

        def __str__(self):
            return self.text
```

```
python manage.py makemigrations blog
python manage.py migrate blog
```


# REGISTER COMMENT MODEL IN ADMIN PANEL
blog/admin.py:
```
    admin.site.register(Comment)
```

# POSTGRESQL INSTALLATION
sudo apt-get install postgresql postgresql-contrib


# MAKE OUR COMMENTS VISIBLE
blog/templates/blog/post_detail.html:
```
    <hr>
    {% for comment in post.comments.all %}
    <div class="comment">
        <div class="date">{{ comment.created_date }}</div>
        <strong>{{ comment.author }}</strong>
        <p>{{ comment.text|linebreaks }}</p>
    </div>
    {% empty %}
    <p>No comments here yet :(</p>
```

static/css/blog.css:
```
    .comment {
        margin: 20px 0px 20px 20px;
    }
```

blog/templates/blog/post_list.html:
```
    <a href="{% url 'post_detail' pk=post.pk %}">Comments: {{ post.comments.count }}</a>
```


# LET YOUR READERS WRITE COMMENTS
blog/forms.py:
```
    class CommentForm(forms.ModelForm):

        class Meta:
            model = Comment
            fields = ('author', 'text',)
```

blog/templates/blog/post_detail.html:
```
    <a class="btn btn-default" href="{% url 'add_comment_to_post' pk=post.pk %}">Add comment</a>
```

blog/urls.py:
```
urlpatterns = [
    [...]
    path('post/<int:pk>/comment/', views.add_comment_to_post, name='add_comment_to_post'),
]
```

blog/views.py:
```
    def add_comment_to_post(request, pk):
        post = get_object_or_404(Post, pk=pk)
        if request.method == "POST":
            form = CommentForm(request.POST)
            if form.is_valid():
                comment = form.save(commit=False)
                comment.post = post
                comment.save()
                return redirect('post_detail', pk=post.pk)
        else:
            form = CommentForm()
        return render(request, 'blog/add_comment_to_post.html', {'form': form})    
```

blog/templates/blog/add_comment_to_post.html:
```
    {% extends 'blog/base.html' %}

    {% block content %}
        <h1>New comment</h1>
        <form method="POST" class="post-form">{% csrf_token %}
            {{ form.as_p }}
            <button type="submit" class="save btn btn-default">Send</button>
        </form>
    {% endblock %}
```


# MODERATING YOUR COMMENTS
blog/templates/blog/post_detail.html:
```
    {% if user.is_authenticated or comment.approved_comment %}
    <div class="comment">
        <div class="date">
            {{ comment.created_date }}
            {% if not comment.approved_comment %}
                <a class="btn btn-default" href="{% url 'comment_remove' pk=comment.pk %}"><span class="glyphicon glyphicon-remove"></span></a>
                <a class="btn btn-default" href="{% url 'comment_approve' pk=comment.pk %}"><span class="glyphicon glyphicon-ok"></span></a>
            {% endif %}
        </div>
        <strong>{{ comment.author }}</strong>
        <p>{{ comment.text|linebreaks }}</p>
    </div>
    {% endif %}
```

blog/urls.py:
```
    path('comment/<int:pk>/approve/', views.comment_approve, name='comment_approve'),
    path('comment/<int:pk>/remove/', views.comment_remove, name='comment_remove'),
```

blog/views.py:
```
    @login_required
    def comment_approve(request, pk):
        comment = get_object_or_404(Comment, pk=pk)
        comment.approve()
        return redirect('post_detail', pk=comment.post.pk)

    @login_required
    def comment_remove(request, pk):
        comment = get_object_or_404(Comment, pk=pk)
        comment.delete()
        return redirect('post_detail', pk=comment.post.pk)
```

blog/templates/blog/post_list.html:
```
    <a href="{% url 'post_detail' pk=post.pk %}">Comments: {{ post.approved_comments.count }}</a>
```

blog/models.py:
```
    def approved_comments(self):
        return self.comments.filter(approved_comment=True)
```


# POSTGRESQL INSTALLATION
sudo apt-get install postgresql postgresql-contrib


# CREATE DATABASE
```
sudo pg_ctlcluster 11 main start
sudo -u postgres -i
psql
    CREATE USER marcel;
    CREATE DATABASE djangogirls OWNER marcel;
```

Change password:
```
sudo -u postgres -i
psql
cd /data
psql postgres postgres
\password marcel
```

# UPDATING SETTINGS
```
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': 'djangogirls',
        'USER': 'marcel',
        'PASSWORD': '[...]',
        'HOST': 'localhost',
        'PORT': '',
    }
}
```

# INSTALLING POSTGRESQL PACKAGE FOR PYTHON
```
sudo apt-get install python-psycopg2
sudo apt install libpq-dev
pip install psycopg2
python manage.py migrate
python manage.py createsuperuser --username marcel
```


