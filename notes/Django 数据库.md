---
attachments: [Clipboard_2020-03-10-15-11-14.png, Clipboard_2020-03-10-15-11-17.png]
tags: [Django]
title: Django 数据库
created: '2020-03-10T07:09:34.745Z'
modified: '2020-03-27T08:01:19.681Z'
---

# Django 数据库

- 更新`models.py`至数据库
```python
python manage.py makemigrations
python manage.py migrate
```

- 从现有数据库中生成`new-models.py`
```python
python manage.py inspectdb > new-models.py
```


