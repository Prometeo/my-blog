+++
title = "N+1 query problem"
date = 2020-05-17T17:31:21-05:00
tags = ["django", "sql", "orm", "python"]
featured_image = ""
description = "Resolviendo problemas de performance relacionados al N+1 query problem"
+++

El "N+1 query problem** es un problema de performance muy común en los ORM; este problema ocurre cuando se crea un query con el ORM que inadvertidamente genera queries adicionales para obtener registros adicionales que no fueron cargados por el query original.

Para ilustralo trabajaremos con estos dos modelos sacados del tutorial de la web oficial de django

## select_related

```python
class Question(models.Model):
    question_text = models.CharField(max_length=200)
    pub_date = models.DateTimeField('date published')

class Choice(models.Model):
    question = models.ForeignKey(Question, on_delete=models.CASCADE)
    choice_text = models.CharField(max_length=200)
    votes = models.IntegerField(default=0)
```

Creamos la siguiente vista:

```python
def choices_detail(request):
    choices = Choice.objects.all()
    context = {'choices': choices}
    return render(request, 'polls/choices_detail.html', context)
```

Creamos el sguiente template:

```python
      {% for choice in choices %}
        <p>Question: {{ choice.question.question_text }}</p>
        <p>Choice: {{ choice.choice_text }}</p>
        <p>Votes: {{ choice.votes }}</p>
      {% endfor %}
```
{{ choice.question.question_text }} -> esta línea generará una consulta adicional por cada "choice"

para solucionarlo modificamos la vista:

```
def choices_detail(request):
    choices = Choice.objects.all().select_related('question')
    context = {'choices': choices}
    return render(request, 'polls/choices_detail.html', context)
```

genera una sola consulta
```sql
SELECT "polls_choice"."id", "polls_choice"."question_id",
"polls_choice"."choice_text", "polls_choice"."votes",
"polls_question"."id", "polls_question"."question_text",
"polls_question"."pub_date" FROM "polls_choice"
INNER JOIN "polls_question" ON ("polls_choice"."question_id" = "polls_question"."id")
```

## prefetch related

modelos:
```python
class Publication(models.Model):
    title = models.CharField(max_length=30)

    class Meta:
        ordering = ['title']

    def __str__(self):
        return self.title


class Article(models.Model):
    headline = models.CharField(max_length=100)
    publications = models.ManyToManyField(Publication)

    class Meta:
        ordering = ['headline']

    def __str__(self):
        return self.headline

```

without prefetch_related
```python
for article in Article.objects.all():
    print(article.publications.all())
```
If you tried to use User.objects.all().select_related('events')[:10] with this code, you would
get the following error:
```
FieldError: Invalid field name(s) given in select_related: 'events'. Choices are: profile.
```
This is because only profile , which uses a ForeignKey relationship to User , is available to
select_related

example sql generated for each article
```sql
SELECT "polls_publication"."id", "polls_publication"."title"
FROM "polls_publication"
INNER JOIN "polls_article_publications"
ON ("polls_publication"."id" = "polls_article_publications"."publication_id")
WHERE "polls_article_publications"."article_id" = 3
ORDER BY "polls_publication"."title"
```

with prefetch_related

```python
for article in Article.objects.all().prefetch_related('events'):
    print(article.publications.all())
```

generated sql
```sql
SELECT ("polls_article_publications"."article_id") AS "_prefetch_related_val_article_id",
"polls_publication"."id", "polls_publication"."title" FROM "polls_publication"
INNER JOIN "polls_article_publications"
ON ("polls_publication"."id" = "polls_article_publications"."publication_id")
WHERE "polls_article_publications"."article_id" IN (3, 1, 2)
ORDER BY "polls_publication"."title" ASC;
```
