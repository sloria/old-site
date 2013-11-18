---
layout: post
title: "The M's, V's, and C's: Keep 'em light and fluffy"
description: ""
category: software design
tags: [python, rest, design]
---
{% include JB/setup %}

The MVC design pattern gives us general guidelines on where various components of an application should go. Web frameworks attempt to enforce MVC by giving us cookiecutter classes and interfaces for each of the three layers.

But as an application gets larger, it becomes more difficult to decide where certain components belong. Even if we adhere to the "fat models, skinny controllers" advice, we can end up with massive, god-like model classes with many coupled responsibilities.

Here I use the example of data formatting to demonstrate how one might separate responsibilities between classes in a web application and introduce a library called [marshmallow](http://marshmallow.readthedocs.org) which makes the process easier.

## First, let's clear up some terminology.

The terms "Model", "View", and "Controller" can mean many different things depending on the context of the conversation. For example, the Django docs give a [sensible reason](https://docs.djangoproject.com/en/dev/faq/general/#django-appears-to-be-a-mvc-framework-but-you-call-the-controller-the-view-and-the-view-the-template-how-come-you-don-t-use-the-standard-names) for calling controller functions "views"[^1].

In this article, I use the abstract "layers" definition of MVC. **This means that there may be many different components that make up any of the M, V, or C**. The model layer, for example, aren't just subclasses of `django.models.Model` or `ActiveRecord`. It may include many other domain objects that pertain to a single subset of an functionality, e.g. a PaymentModel or a LoginModel.

- **Model** layer: Business data, rules, and logic. As pointed out above, may be more than just model classes.
- **View** layer: Presentation of data. May include [ViewModels](https://en.wikipedia.org/wiki/Model_View_ViewModel) that convert and format model data for output to a user.
- **Controller** layer: Accepts user input and updates model state and associated view accordingly. Includes URL routing, controller functions, and helpers that orchestrate data between models.

Any of the components that comprise these layers may not fit neatly into the classes prescribed by an MVC framework, e.g. an Observer on a model will not be a model class; a ViewModel is not a template.

**When any one component starts to get "fat"--yes, even your models--, you must consider if you can break up the component into smaller subclasses or sublayers**.

## The Problem: Outputting Data

Let's take the example of determining output data for a REST API. Where does the logic for outputting the model data as JSON belong?

- **In the model (Good, but not best)**. Adding a serialization method to a model class is a fine solution that is reuseable and testable. However, it adds unnecessary heft to the models, which introduces problems that we'll see shortly.
- **In the controller (Bad)**. Leads to duplicated code and is difficult to test.
- **In the view (Ugly)**. Just because your templating engine offers a ``to_json`` template helper doesn't mean you should use it. Doing this is even less reusable and testable than formatting data in the controller.

### Can we do better?

Putting the serialization logic in the model is a far superior solution than putting it in the controller or view. However, there are still flaws with this method:

- **It violates the [Single Responsibility Principle](http://www.oodesign.com/single-responsibility-principle.html)** (SRP), which states: "A class should have one, and only one, reason to change." For example:

        class User(Model):
            username = CharField()
            password = CharField()
            ...
            def to_json(self):
                return {...}

This class violates SRP because there are at least two reasons to change the `User` class:

1. The schema changes.
2. The JSON output format changes.

We should separate these entities so that they can be changed independently of each other.

- **Can't serialize arbitrary objects.** What if you need to serialize an object from a third-party library, such as a `session` object? Or a form? **You would need to write a custom serializer class that is passed the object and outputs the formatted data, which, conveniently, is a good idea for *any* object**.

## Slimming down

The best solution here is to put the serialization logic into a separate class, responsible only for formatting the output data. 

You could write these from scratch, though there are a number of libraries that make this easier. One of these is [marshmallow](http://marshmallow.readthedocs.org), written for Python (full disclosure: I'm the author).

<a href="http://marshmallow.readthedocs.org">
<img src="http://marshmallow.readthedocs.org/en/latest/_static/marshmallow-logo.png" height="200" alt="marshmallow docs">
</a>

## Why marshmallow?

- **Separates business logic from the presentation.** See above.
- **Reusability.** Serializers classes are reusable, are nestable, and allow for inheritance.
- **Testability.** Serializer classes are easier to test than views and controllers.
- **Agnostic.** ORM, ODM, Flask, Django--doesn't matter. Marshmallow serializes all objects in the same way.
- **Familiar syntax**. Serializers look very much like [WTForms](https://wtforms.readthedocs.org/en/latest/) or a model definition in any of the popular ORMs.

## First steps with marshmallow

Let's take some simple models.

    class User(Model):
        email = EmailField()
        password = CharField()
        joined_on = DateTimeField(auto_now_add=True)

    class Todo(Model):
        task = CharField()
        done = BooleanField(default=False)
        user = ForeignKey(User)        

Then we'll declare our serializers. 

    from marshmallow import Serializer, fields

    class UserSerializer(Serializer):
        email = fields.Email()  # Validates email
        joined_on = fields.DateTime()  # Formats datetimes as RFC-822 string

    class TodoSerializer(Serializer):
        task = fields.String()
        done = fields.Boolean()
        user = fields.Nested(UserSerializer)

Example usage:

    user = User(email="monty@python.org", password="secret")
    todo = Todo("Refactor everything")   
    serialized = TodoSerializer(todo)
    serialized.is_valid()
    # True
    serialized.data
    # {"task": "Refactor everything", "done": false, 
    # "user": {"joined_on": "Mon, 18 Nov 2013 00:42:19 -0000", 
    #          "email": "monty@python.org"}
    # }

We can make the serializer more concise by using the `fields` option on the `Meta` options class.

    class TodoSerializer(Serializer):
        user = fields.Nested(UserSerializer)
        class Meta:
            fields = ('task', 'done', 'user')

    TodoSerializer(todo).data  # Same as above

Check out marshmallow's documentation--with examples in Flask, SQL-Alchemy, and Peewee--at [http://marshmallow.rtfd.org](http://marshmallow.rtfd.org).

## Conclusion

Whatever tools you choose, keeping your classes lightweight (always remind yourself of the SRP) will force you to design with your application's evolution in mind.

[^1]: The Django docs distinguish between "what data are presented" (the view) and "how the data look" (the template). They therefore call Django a Model-Template-View framework, which is probably a more accurate description of many other popular frameworks such as Rails and Flask.

