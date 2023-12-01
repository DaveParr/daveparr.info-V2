+++
title = "Building my first Django project with CSS and Static Files"
description = "Even more notes from Django for Beginners by William S. Vincent"
date = 2020-06-25
[taxonomies]
tags = ["python", "django", "class-based views", "templates"]
+++

I'm working through [Django for Beginners by William S. Vincent](https://djangoforbeginners.com/). Until now we've had some pretty barebones Times New Roman style UI, however that's all about to change! I think.

I'm really appreciating this incremental iteration of concepts in each new Chapter. Each time I start a new app for the chapter it's engraining the muscle memory and concept recall into me. I'm able to run through the first 5 lines of code at the CLI nearly from memory. It's also a great mechanism to help you if you get stuck on a wierd error and don't know what to do. You know the next chapter will have a clean slate. Great idea William.

The repetition isn't just the set-up though.

> “Now we can add the functionality for individual blog pages. How do we do that? We need to create a new view, url, and template. I hope you’re noticing a pattern in development with Django now!”

I sure am. 

I'm glad that the book also doesn't go into non-django areas, but still points readers to places to look for more info. The last chapter pointed me towards some resources to understand more about databases, and this one does similar with CSS.

It was interesting to see that the approach used to identify a blog post for navigation in the URL patterns looked _kinda_ regex-ish:

```py
urlpatterns = [
    path('post/<int:pk>/', BlogDetailView.as_view(), name='post_detail'),
    path('', BlogListView.as_view(), name='home'),
]
```

`<int:pk>` is the part that identifies the blog post required for the url to link from the ListView to the DetailView. It was interesting to find out there was something like a unique identifier for each post baked into it in the background. I was wondering about how this primary key is treated in bigger projects. Is it common to keep this approach of integer ordered primary keys, or do they get replaced in larger, more complex projects with hashes or other unique ids?