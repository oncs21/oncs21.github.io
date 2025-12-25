---
title: "Integrating rich text editor with Django"
date: 2022-01-07
---

Creating blogs or article-based tutorials is one of the main aims of every web developer after successful deployment of a website. While it is not easy to create a text editor from scratch using JavaScript, it is certainly possible to make use of well-developed open source text editors. So, in this blog we will be integrating "ck editor" with Django.

Before we move on, I am assuming that all of the following pre-requisites are satisfied:
- A thorough understanding of model forms in Django
- Form rendering in template
- Data models
- Python and Django already installed in system

Alright! So, with the basics covered, let us quickly learn how to integrate "ck editor" with Django. For the sake of simplicity, I have presented the steps with lucrative code examples.  

## Step 1
In order to use the features of "ck editor", we first must need to install it. The installation is simple: Just execute the below command in the terminal of your Operating System.

```bash
pip install django-ckeditor
```

In a span of 2-3 minutes, the "ck-editor" would be installed. If it does not, don't worry! Try once again.

## Step 2
It's time to create a new Django project or to open an existing Django project. Once, you are inside the project look for the setting.py file inside the project folder. Inside the file, again, look for a list named "INSTALLED_APPS". Just like adding a new app in the list, add "ckeditor" to the list.

```python
INSTALLED_APPS = [
  'shareledge',
  'django.contrib.admin',
  'django.contrib.auth',
  'django.contrib.contenttypes',
  'django.contrib.sessions',
  'django.contrib.messages',
  'django.contrib.staticfiles',
  'ckeditor'
]
```

## Step 3
Now navigate to the `views.py` file inside the app folder where you wish to include "ck-editor". Inside the `views.py` file, create a model form linked to one of the models in the `models.py` file. For instance, I have created a model form named "ArticleForm".

```python
class ArticleForm(forms.ModelForm):
    content = RichTextFormField(config_name="default")

    class Meta:
        model = Article
        fields = ["title", "content", "category", "contentImage"]
        widgets = {
            "title": forms.TextInput(
                attrs={
                    "class": "form-control",
                    "placeholder": "Title",
                }
            ),
            "category": forms.TextInput(
                attrs={
                    "class": "form-control",
                    "placeholder": "Category",
                }
            ),
            "contentImage": forms.TextInput(
                attrs={
                    "class": "form-control",
                    "placeholder": "Image",
                }
            ),
        }
```

## Step 4
In the same `views.py` file, inside one of the views where you wish to display the "ck editor", use the model form which you created in the previous step and pass the form to the template through a variable using the render method.

```python
form = ArticleForm()
return render(request, 'shareledge/create.html', {
  'articles': articles,
  'myform': form,
  'toEdit': False
})
```

Using the `form()` method, I have created a new ArticleForm and then assigned it to a variable simply named "form". At last, I have passed the form to the template `create.html` as myForm.

## Step 5
Now, we have to decide which field of the form has to be given the "ck editor" features. To do this, open the `models.py` file inside the app folder in which you wish to add the "ck editor" features. Locate the model and hence, the required field. To this field, assign the RichTextField() attribute. Make sure to import RichTextField().

```python
from ckeditor.fields import RichTextField
class Article(models.Model):
    title = models.CharField(max_length=300)
    user = models.ForeignKey(User, on_delete=models.CASCADE, related_name="creator")
    datePosted = models.CharField(max_length=20, default="")
    timePosted = models.CharField(max_length=20, default="")
    category = models.CharField(max_length=300)
    content = RichTextField(null=True, blank=True, 
    config_name="special", external_plugin_resources=[(
    'youtube', '/static/shareledge/ckeditor-plugins/youtube/youtube/', 'plugin.js',
    )])
```

In my case, I have used the field content. At this point of time, the use of RichTextField(null=True, blank=True) would suffice.

## Step 6
This is the final step. Head over to the template where the form has to be rendered. We know that any form can be rendered in the template using `form`. Since, in my case the form is passed as `myForm`, I will just render it using `{{myForm}}`. In order to add "ck editor" to the template, I will just have to add {{ myForm.media }} to the code:

```python
<form>
  {{ myForm.media }}
  {{ myForm }}
</form>
```

Of course, I will have to customize each field of the form according to my styling preferences. Part of this is shown below.

```python
<form id="article-create-form"
      method="POST"
      action="{% url 'shareledge:createPage' %}"
      enctype="multipart/form-data">

    {% csrf_token %}

    {{ myform.media }}

    <div class="form-group">
        <label class="label">Title</label>
        {{ myform.title }}
    </div>

    <br>

    <div class="form-group">
        <label class="label">Category</label>
        {{ myform.category }}
    </div>

    <br>

    <div class="form-group">
        <label class="label">Content</label>
        {{ myform.content }}
    </div>

    <br>

    <div class="form-group">
        <label class="label">Image</label>
        {{ myform.contentImage }}
    </div>

    <br>

    <button type="submit" class="btn btn-primary">
        Create Article
    </button>
</form>
```

That's it! We are done. If we open the webpage, we would be able to see the field rendered as "ck editor".

<p align="center">
<img width="800" height="499" alt="image" src="https://github.com/user-attachments/assets/3a4e387d-4cb5-4e77-8229-f77e1a6a16d9" />
</p>

This field now inherits tons of features such as Bold, Italic, Line spacing, Image insertion, Font colour and much more. Additionally, if we look at the admin interface the same field will use the "ck editor" features there itself too. So, that's it about this blog. I hope you find the information in this blog useful.
