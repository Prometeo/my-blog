+++
title = "N+1 query problem"
date = 2020-06-02T17:31:21-05:00
tags = ["django", "sql", "orm", "python"]
featured_image = ""
description = "Resolviendo problemas de performance relacionados al N+1 query problem"
author= "Xairi Valdivia"
+++

El **N+1 query problem** es un problema de performance comunmente presente en los
ORM([Object Relational Mapping](https://en.wikipedia.org/wiki/Object-relational_mapping)).
Cualquier código que recorra un arreglo y por cada elemento de este arreglo ejecute un
query adicional tiene este problema.

> El término "N+1 query problem" o "N+1 select problem" surgió a principios de los 2000
> con el uso del ORM Hibernate. La primera referencia a este problema puede ser
> la que se hace en el libro [Java Persistence with Hibernate](https://www.manning.com/books/java-persistence-with-hibernate)

Si bien este problema es muy común, actualmente los orm nos ofrecen herramientas para poder evitarlo.

Analicemos el siguiente ejemplo para poder entender mejor como es que podemos introducir este problema
en nuestro código sin darnos cuenta.

Primero crearemos estos modelos dos modelos obtenidos del tutorial de la página web de django.

```python
class Question(models.Model):
    question_text = models.CharField(max_length=200)
    pub_date = models.DateTimeField('date published')

class Choice(models.Model):
    question = models.ForeignKey(Question, on_delete=models.CASCADE)
    choice_text = models.CharField(max_length=200)
    votes = models.IntegerField(default=0)
```

A continuación creamos la siguiente vista:

```python
def choices_detail(request):
    choices = Choice.objects.all()
    context = {'choices': choices}
    return render(request, 'polls/choices_detail.html', context)
```
El **QuerySet** de nuestra vista es el que va a causar problemas en nuestro template, pero ningún código SQL
es ejecutado aún porque los QuerySets son [lazy](https://docs.djangoproject.com/en/2.2/topics/db/queries/#querysets-are-lazy).

Finalmente agregaremos esta porción de código a nuestro template para que obtenga los datos
de la vista creada anteriormente:

```python
      {% for choice in choices %}
        <p>Question: {{ choice.question.question_text }}</p>
        <p>Choice: {{ choice.choice_text }}</p>
        <p>Votes: {{ choice.votes }}</p>
      {% endfor %}
```
La línea de código que generará un query adicional por cada choice es: {{ choice.question.question_text }}.

## ¿Por qué ocurre este problema?
Como vemos en el ejemplo anterior, nosotros esperamos ejecutar un solo query para obtener
todos los registros de la tabla Choice e imprimir los valores, pero choice.question representa
una llave foránea entre Choice y Question. El comportamiento del ORM de django por defecto es
el de no incluir data de las relaciones en un Queryset a menos que sea requerido explícitamente,
por lo tanto, en este caso question no estará disponible cuando el loop ejecute la línea
que pinta **choice.question.question_text**, por lo tanto django va a tener que ejecutar otro query
para poder obtener los datos relacionados al elemento actual.

## Usando el método select_related
Nuestra mejor herramienta contra el **N+1 problem** es el método [select_related](https://docs.djangoproject.com/en/2.2/ref/models/querysets/#select-related).

El método select_related toma una lista de strings como parámetros, Cada string debe coincidir
con el nombre del campo que representa una llave foránea en el modelo, en este caso "question".

Para solucionar el problema de performance descrito anteriormente modificaremos nuestra vista
y haremos uso del método select_related.

```python
def choices_detail(request):
    choices = Choice.objects.all().select_related('question')
    context = {'choices': choices}
    return render(request, 'polls/choices_detail.html', context)
```

Con esta modificación ya no se generará un nuevo query por cada iteración de choices, ya que
el select_related habrá hecho un join con "question" y habrá obtenido los datos de la tabla
relacionada a ese campo.

A continuación el query generado por el select_related de nuestra vista:

```sql
SELECT "polls_choice"."id", "polls_choice"."question_id",
"polls_choice"."choice_text", "polls_choice"."votes",
"polls_question"."id", "polls_question"."question_text",
"polls_question"."pub_date" FROM "polls_choice"
INNER JOIN "polls_question" ON ("polls_choice"."question_id" = "polls_question"."id")
```

## Método prefetch_related
Mientras que el método "select_related" hace uso del **JOIN** para incluir data relacionada en un query,
**prefetch_related** hace uso de otra técnica de optimización: crea una query adicional para obtener
la data relacionada y entonces la combina con el queryset después de que las queries sean evaluadas.
A continuación mostraremos como es que funciona este método.

Primero creamos los modelos:

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

Ahora ejecutaremos una consulta sin hacer uso del método prefetch_related.

```python
for article in Article.objects.all():
    print(article.publications.all())
```

Este es el query generado por el una iteración del bucle:

```sql
SELECT "polls_publication"."id", "polls_publication"."title"
FROM "polls_publication"
INNER JOIN "polls_article_publications"
ON ("polls_publication"."id" = "polls_article_publications"."publication_id")
WHERE "polls_article_publications"."article_id" = 3
ORDER BY "polls_publication"."title"
```

Si tuviésemos 5000 artículos, generaríamos 1 query para obtenerlos todos y después generaríamos
otras 5000 queries para obtener cada publicación y esto es precisamente lo que queremos evitar.

Ahora ejecutamos al misma consulta pero con el método **prefetch_related**

```python
for article in Article.objects.all().prefetch_related('events'):
    print(article.publications.all())
```

Este es el query generado por el código anterior:

```sql
SELECT ("polls_article_publications"."article_id") AS "_prefetch_related_val_article_id",
"polls_publication"."id", "polls_publication"."title" FROM "polls_publication"
INNER JOIN "polls_article_publications"
ON ("polls_publication"."id" = "polls_article_publications"."publication_id")
WHERE "polls_article_publications"."article_id" IN (3, 1, 2)
ORDER BY "polls_publication"."title" ASC;
```

Como vemos, el método **prefetch_related** evita que generemos un query por cada iteración, por lo tanto
nuestro código es mas eficiente.

## ¿Cómo sabemos cuando usar prefetch_related o select_related?

prefetch_related trabaja con relaciones **many-to-many** y **many-to-one**
mientras que **select_related** solo trabaja con con relaciones single-value como **foreign key**.
