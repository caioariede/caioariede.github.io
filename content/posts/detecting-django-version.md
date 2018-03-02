---
title: "Detecting which version of Django a website is running"
date: 2018-03-02T15:57:37-03:00

---

A few days ago, before doing a job interview I was studying the current tech
stack of the company and started wondering how hard it would be to check which
version of Django they were using.

I first thought it wouldn't be trivial but I quickly figured out an easy way of
doing it. The approach is really stupid and will not work on every case, the
detection is only possible if the following requirements are met:

1. The project is using `django.contrib.admin`
2. You managed to find the path to static files (usually, /static/)

The idea is to verify the presence or absence of some files that are included
or removed between two versions. In some cases, no files were included or
removed, so in these cases I just checked for changes. I've ended up with two
basic checks:

* check if url exists
* check if url exists AND contains a certain text

[The code](https://github.com/caioariede/detect-django-version) is something
like this:

```
if url_exists(url):
    return VERSION_1
elif url_contains(url):
    return VERSION_2
...
```

With only these two checks, I was able to cover Django versions from 2.0 to 1.3.
I've done some tests myself and I'm frankly surprised how effective this
approach was for my tests. I was also surprised to see some websites still
using Django 1.3, damn.

I used the following script to perform my tests:

```
from detect import detect

URLS = [
    'http://example.com/static/admin',
    ...
]

for url in urls:
    print(url, detect(url))
```

Which gave me the following output:

```
http://www.*****.com/static/admin 1.11
http://app.******.com/static/admin 1.11
http://***.********.com/static/admin 1.11
http://support.cdn.*******.net/static/admin 1.8
http://dashboard.*******.com/static/admin 1.8
http://www.*************.com/static/admin 1.4
http://********.fr/static/admin 1.4
https://*****.com/assets/admin 1.9
https://login.*****.com/site_media/admin 1.10
https://******.******.com/static/admin 1.8
http://www.*************.com/media/static/admin 1.4
http://www.*************.com/media/admin 1.3
http://********.org.uk/adminmedia 1.3
http://******.*****.it/static/admin 1.8
http://www.******.com/media/admin 1.4
http://www.*********.com/static/admin 1.6
http://www.*******************.fr/static/admin 1.4
https://www.*******.es/static/admin 1.4
https://www.****************.com/static/admin 1.10
http://*********************.de/static/admin 1.6
```

You can find the code in [this repository](https://github.com/caioariede/detect-django-version).
If you want to propose an improvement, I'll be happy to see a pull request there. :)
