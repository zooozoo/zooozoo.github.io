---
category: record
---
# Making Queries

[장고문서](https://docs.djangoproject.com/ko/1.11/topics/db/queries/#retrieving-specific-objects-with-filters) 를 공부하며 기록한 내용입니다. 



```python
>>> from blog.models import Blog
>>> b = Blog(name='Beatles Blog', tagline='All the latest Beatles news.')
>>> b.save()
```

> This performs an `INSERT` SQL statement behind the scenes. Django doesn't hit the database until you explicitly call [`save()`](https://docs.djangoproject.com/ko/1.11/ref/models/instances/#django.db.models.Model.save).

안쪽에서는 sql 문이 작동되는데, 장고는 save() method가 실행되기 전까지는 database를 치지 않는다. 



### Creating objects

> save() takes a number of advanced options not described here. See the documentation for save() for complete details.
>
> To create and save an object in a single step, use the create() method.

save() method에는 다양한 옵션이 존재하고, 한번에 save()까지 처리하려면 create() mothod를 사용한다.



### Saving changes to objects

> To save changes to an object that's already in the database, use save().

save() method는 이미 저장된 내용을 변경하는데에도 사용된다. 



#### Saving `ForeignKey` and `ManyToManyField` fields

> Updating a ForeignKey field works exactly the same way as saving a normal field -- simply assign an object of the right type to the field in question. This example updates the blog attribute of an Entry instance entry, assuming appropriate instances of Entry and Blog are already saved to the database (so we can retrieve them below):

ForeignKey field의 경우 일반 필드와 같이 업데이트 할 수 있습니다.



> Updating a ManyToManyField works a little differently -- use the add() method on the field to add a record to the relation. This example adds the Author instance joe to the entry object:

```python
>>> from blog.models import Author
>>> joe = Author.objects.create(name="Joe")
>>> entry.authors.add(joe)
```

ManyToManyField의 경우 약간 다르게 작동합니다. add()라는 method를 사용합니다. 예시의 경우 joe라는 Author객체를 만들고 entry와 author의 관계를 add() method를 생성합니다. 



> To add multiple records to a ManyToManyField in one go, include multiple arguments in the call to add(), like this:

```python
>>> john = Author.objects.create(name="John")
>>> paul = Author.objects.create(name="Paul")
>>> george = Author.objects.create(name="George")
>>> ringo = Author.objects.create(name="Ringo")
>>> entry.authors.add(john, paul, george, ringo)
```

한번에 여러개의 데이터를 ManyToMany 관계에 추가하려고 할 때에, 한번에 속성을 여러개 적용할 수 있습니다.



### Retrieving objects

> To retrieve objects from your database, construct a QuerySet via a Manager on your model class.
>
> A QuerySet represents a collection of objects from your database. It can have zero, one or many filters. Filters narrow down the query results based on the given parameters. In SQL terms, a QuerySet equates to a SELECT statement, and a filter is a limiting clause such as WHERE or LIMIT.
>
> You get a QuerySet by using your model's Manager. Each model has at least one Manager, and it's called objects by default. Access it directly via the model class, like so:

```python
>>> Blog.objects
<django.db.models.manager.Manager object at ...>
>>> b = Blog(name='Foo', tagline='Bar')
>>> b.objects
Traceback:
    ...
AttributeError: "Manager isn't accessible via Blog instances."
```

데이터베이스에서 객체를 꺼내려 할 때 당신의 모델 클래스의 매니저를 통해 쿼리셋을 구성해야 합니다.

하나의 쿼리셋은 당신의 데이터베스로부터 나온 객체의 모음을 의미합니다.  해당 쿼리셋은 0개 혹은 다수의 filter를 포함할 수 있습니다. filter는 주어진 매개변수에 기반하여 쿼리 결과물의 범위를 좁게할 수 있습니다. SQL문 에서 쿼리셋은 SELECT문과 같고, filter는 WHERE 또는 LIMIT와 같이 조건절 입니다.

당신은 모델매니저를 사용함으로써 쿼리셋을 만들 수 있습니다. 각각의 모델은 적어도 하나 이상의 Manager를 가지며, 기본값으로 objects로 주어집니다. 예시와 같이 모델클래스를 통해서 바로 불러올 수 있습니다.



#### Retrieving specific objects with filters

> For example, to get a QuerySet of blog entries from the year 2006, use filter() like so:

```python
Entry.objects.filter(pub_date__year=2006)
```

> With the default manager class, it is the same as:

```python
Entry.objects.all().filter(pub_date__year=2006)
```

filter() 사용에 있어서 예시로 나온 2가지는 같은 방식이다.



##### Chaining filters

> The result of refining a QuerySet is itself a QuerySet, so it's possible to chain refinements together. For example:

```python
>>> Entry.objects.filter(
...     headline__startswith='What'
... ).exclude(
...     pub_date__gte=datetime.date.today()
... ).filter(
...     pub_date__gte=datetime.date(2005, 1, 30)
... )
```

> This takes the initial QuerySet of all entries in the database, adds a filter, then an exclusion, then another filter. The final result is a QuerySet containing all entries with a headline that starts with "What", that were published between January 30, 2005, and the current day.

쿼리셋을 걸러낸 결과는 여전히 쿼리셋이므로 해당 쿼리셋을 다시 여러번 연결하여 걸러낼 수 있다. 

예시에서 나온 쿼리셋은 데이터베이스에서 모든 엔트리를 가져온다. 그리고나서 filter를 추가하고 난후에 exclusion을 적용하고, 마지막으로 다시 filter를 적용한다. 결과적으로 쿼리셋은 'What'으로 시작하고, pub date가 2005년 1월 30일 부터 오늘미만의 객체를 가지고 있게 된다. 



##### Filtered QuerySets are unique

> Each time you refine a QuerySet, you get a brand-new QuerySet that is in no way bound to the previous QuerySet. Each refinement creates a separate and distinct QuerySet that can be stored, used and reused.

쿼리셋을 걸러낼 때 마다 당시는 새로운 쿼리셋을 얻게 되며 그 쿼리셋은 이전 쿼리셋에 영향을 주지 않는다. 각각의 걸러내는 과정은 저장될 수 있고 재사용 될 수 있는 분리된 고유의 쿼리셋이다. 



##### QuerySets are lazy

> QuerySets are lazy -- the act of creating a QuerySet doesn't involve any database activity. You can stack filters together all day long, and Django won't actually run the query until the QuerySet is evaluated. Take a look at this example:

```python
>>> q = Entry.objects.filter(headline__startswith="What")
>>> q = q.filter(pub_date__lte=datetime.date.today())
>>> q = q.exclude(body_text__icontains="food")
>>> print(q)
```

> Though this looks like three database hits, in fact it hits the database only once, at the last line (print(q)). In general, the results of a QuerySet aren't fetched from the database until you "ask" for them. When you do, the QuerySet is evaluated by accessing the database. For more details on exactly when evaluation takes place, see When QuerySets are evaluated.

쿼리셋은 자주 작동 되지 않습니다 --쿼리셋을 생성하는 것이 데이터베이스의 작동을 포함하지 않는다는 말 입니다. 당신은 filter를 하루종일 쌓아나갈 수 있으며 장고는 쿼리가 평가되기 전까지는 데이터베이스를 작동시키지 않을 것 입니다. 

예시에서는 마치 데이터베이스를 3번 친 것 처럼 보이지만, 사실 마지막 라인에서 한번 쳤으며(print(q)). 일반적으로 쿼리셋의 결과는 당신이 요청하기 전까지 데이터베이스에서 데이터를 가져오지 않습니다.  당신이 요청을 할때 쿼리셋은 평가되며 데이터베이스에 접근하게 됩니다. 

아래의 예시와 같이 쿼리가 작동했는지 여부를 확인할 수 있다. [출처 stackoverflow](https://stackoverflow.com/questions/20171736/when-is-a-django-queryset-evaluated)

```python
>>> from django.contrib.auth.models import *
>>> from django.db import connection
>>> connection.queries
[]
>>> query = User.objects.filter(pk=1)
>>> connection.queries
[]
>>> query.exists()
True
>>> connection.queries
[{u'time': u'0.000', u'sql': u'SELECT (1) AS `a` FROM `auth_user` WHERE `auth_user`.`id` = 1  LIMIT 1'}]
>>> query.get()
<User: root>
>>> connection.queries
[{u'time': u'0.000', u'sql': u'SELECT (1) AS `a` FROM `auth_user` WHERE `auth_user`.`id` = 1  LIMIT 1'}, 
 {u'time': u'0.000', u'sql': u'SELECT `auth_user`.`id`, `auth_user`.`password`, `auth_user`.`last_login`, `auth_user`.`is_superuser`, `auth_user`.`username`, `auth_user`.`first_name`, `auth_user`.`last_name`, `auth_user`.`email`, `auth_user`.`is_staff`, `auth_user`.`is_active`, `auth_user`.`date_joined` FROM `auth_user` WHERE `auth_user`.`id` = 1 '}]
>>> 
```

구글에서 찾은 효과적인 쿼리셋 사용법 블로그글 [참조: Using Django querysets effectively](http://blog.etianen.com/blog/2013/06/08/django-querysets/)



#### Limiting QuerySets

> Use a subset of Python's array-slicing syntax to limit your [`QuerySet`](https://docs.djangoproject.com/ko/1.11/ref/models/querysets/#django.db.models.query.QuerySet) to a certain number of results. This is the equivalent of SQL's `LIMIT` and `OFFSET` clauses.

파이썬 slicing 문법의 subset을 활용하여 당신의 쿼리셋의 결과의 갯수를 제한 할 수 있다. 

```python
>>> Entry.objects.all()[:5]
```

위의 예시는 처음부터 5개의 객체만 리턴한다.

```python
>>> Entry.objects.all()[5:10]
```

위의 예시는 6번째 객체부터 10번째 까지 리턴한다.

> Negative indexing (i.e. `Entry.objects.all()[-1]`) is not supported.

마이너스(-) 슬라이스는 지원하지 않는다.

> Generally, slicing a [`QuerySet`](https://docs.djangoproject.com/ko/1.11/ref/models/querysets/#django.db.models.query.QuerySet) returns a new [`QuerySet`](https://docs.djangoproject.com/ko/1.11/ref/models/querysets/#django.db.models.query.QuerySet) -- it doesn't evaluate the query. An exception is if you use the "step" parameter of Python slice syntax. For example, this would actually execute the query in order to return a list of every *second* object of the first 10:
>
> ```python
> >>> Entry.objects.all()[:10:2]
> ```

일반적으로 쿼리셋을 슬라이싱 하면 새로운 쿼리셋을 반환한다. 이것은 쿼리를 평가하지 않는다. 만약 당신이 파이썬 슬라이싱문법의 'step'파라미터를 사용할 경우 에러가 발생한다. 



#### Field lookups

> Field lookups are how you specify the meat of an SQL `WHERE` clause. They're specified as keyword arguments to the [`QuerySet`](https://docs.djangoproject.com/ko/1.11/ref/models/querysets/#django.db.models.query.QuerySet) methods [`filter()`](https://docs.djangoproject.com/ko/1.11/ref/models/querysets/#django.db.models.query.QuerySet.filter), [`exclude()`](https://docs.djangoproject.com/ko/1.11/ref/models/querysets/#django.db.models.query.QuerySet.exclude) and [`get()`](https://docs.djangoproject.com/ko/1.11/ref/models/querysets/#django.db.models.query.QuerySet.get).
>
> Basic lookups keyword arguments take the form `field__lookuptype=value`. (That's a double-underscore). For example:
>
> ```python
> >>> Entry.objects.filter(pub_date__lte='2006-01-01')
> ```
>
> translates (roughly) into the following SQL:
>
> ```python
> SELECT * FROM blog_entry WHERE pub_date <= '2006-01-01';
> ```

Field lookup은 SQL문의 WHERE절을의 가장 중요한 부분을 정의 하는 방법입니다. 이것들은 filter, exclude, get 과같은 쿼리셋 메소드의 특정 키워드의 인수로 지정되어 있습니다.

> The field specified in a lookup has to be the name of a model field. There's one exception though, in case of a [`ForeignKey`](https://docs.djangoproject.com/ko/1.11/ref/models/fields/#django.db.models.ForeignKey) you can specify the field name suffixed with `_id`. In this case, the value parameter is expected to contain the raw value of the foreign model's primary key. For example:
>
> ```python
> >>> Entry.objects.filter(blog_id=4)
> ```

검색에 사용되는 필드는 반드시 모델의 필드이름으로 지정되어야 합니다. 하지만 예외가 한가지 있는데 `ForeignKey`의 경우 입니다. 필드네임 뒤에 `_id`를 붙여서 사용해야 합니다. 파라미터 값은 foreign 모델의 프라이머리 키값이 주어져야만 합니다. 

exact

> If you don't provide a lookup type -- that is, if your keyword argument doesn't contain a double underscore -- the lookup type is assumed to be `exact`.
>
> For example, the following two statements are equivalent:
>
> ```python
> >>> Blog.objects.get(id__exact=14)  # Explicit form
> >>> Blog.objects.get(id=14)         # __exact is implied
> ```

만약 검색 타입을 지정하지 않았다면 (그것은 당신의 키워드 인자가 더블언더스코어를 가지지 않았다는 말입니다.) 검색타입은 exact로 가정하고 작동합니다.

iexact

> A case-insensitive match. So, the query:
>
> ```python
> >>> Blog.objects.get(name__iexact="beatles blog")
> ```
>
> Would match a `Blog` titled `"Beatles Blog"`, `"beatles blog"`, or even `"BeAtlES blOG"`.

contains

> Case-sensitive containment test. For example:
>
> ```python
> Entry.objects.get(headline__contains='Lennon')
> ```
>
> Roughly translates to this SQL:
>
> ```python
> SELECT ... WHERE headline LIKE '%Lennon%';
> ```
>
> Note this will match the headline `'Today Lennon honored'` but not `'today lennonhonored'`.
>
> There's also a case-insensitive version, [`icontains`](https://docs.djangoproject.com/ko/1.11/ref/models/querysets/#std:fieldlookup-icontains).



#### Lookups that span relationships

> Django offers a powerful and intuitive way to "follow" relationships in lookups, taking care of the SQL `JOIN`s for you automatically, behind the scenes. To span a relationship, just use the field name of related fields across models, separated by double underscores, until you get to the field you want.
>
> This example retrieves all `Entry` objects with a `Blog` whose `name` is `'Beatles Blog'`:
>
> ```python
> >>> Entry.objects.filter(blog__name='Beatles Blog')
> ```

장고는 강력하고 직관적인 방법으로 관계간의 쿼리검색을 제공합니다. 내부적으로는 SQL JOIN문을 자동으로 작동시킵니다. 연관된 관계를 검색하기 위해서 두 개의 언더스코어로 구분된 해당 모델과 연결된 필드의 이름을 사용하면 됩니다. 

예시에서는 연결된 블로그 모델의 name 필드가 'Beatles Blog' 인 Entry 쿼리셋을 반환합니다.

> This spanning can be as deep as you'd like.
>
> It works backwards, too. To refer to a "reverse" relationship, just use the lowercase name of the model.
>
> This example retrieves all `Blog` objects which have at least one `Entry` whose `headline` contains `'Lennon'`:
>
> ```python
> >>> Blog.objects.filter(entry__headline__contains='Lennon')
> ```

확장된 관계 검색은 당신이 원하는 단계가 깊어질 수 있습니다. 

역참조 또한 가능합니다. 단지 역관계로 이어진 해당 모델의 소문자를 사용하면 됩니다. 

> If you are filtering across multiple relationships and one of the intermediate models doesn't have a value that meets the filter condition, Django will treat it as if there is an empty (all values are `NULL`), but valid, object there. All this means is that no error will be raised. For example, in this filter:
>
> ```python
> Blog.objects.filter(entry__authors__name='Lennon')
> ```
>
> (if there was a related `Author` model), if there was no `author` associated with an entry, it would be treated as if there was also no `name` attached, rather than raising an error because of the missing `author`. Usually this is exactly what you want to have happen. The only case where it might be confusing is if you are using [`isnull`](https://docs.djangoproject.com/ko/1.11/ref/models/querysets/#std:fieldlookup-isnull). 

만약 여러관계로 이어진 모델에서 필터링 할 경우, 그리고 중간모델 중 하나가 필터에서 지정한 조건을 만족하지 못하는 경우 장고는 유효한 NULL값으로 간주한다. 이것은 여러관계를 걸쳐서 검색을 할 경우에 조건을 만족시키는 값이 없다 하더라도 장고는 에러를 일으키지 않는다는 것을 의미한다. 

만약 entry와 관계된 author가 없다면, 그것은 해당 이름이 없다는 것으로 간주되며 일반적으로 이해되는 상황이다. 헷갈릴 수 있는 한가지 경우는 isnull을 사용할 때다. 

> ```python
> Blog.objects.filter(entry__authors__name__isnull=True)
> ```
>
> will return `Blog` objects that have an empty `name` on the `author` and also those which have an empty `author` on the `entry`. If you don't want those latter objects, you could write:
>
> ```python
> Blog.objects.filter(entry__authors__isnull=False, entry__authors__name__isnull=True)
> ```

예시에서는 연결된 author의 name이 빈 값인 entry와 연결된 blog를 반환한다.만약 이러한 결과를 원하지 않는 것이라면 아래와 같이 사용하면 된다.



##### Spanning multi-valued relationships

[posting]({% post_url /posts/2018-01-24-Django_Spanning multi-valued relationships %}) 내용으로 대체



#### Filters can reference fields on the model

한 모델안에 필드들 간의 비교가 필요할 때 `F expressions`을 통해서 할 수 있다.

> For example, to find a list of all blog entries that have had more comments than pingbacks, we construct an `F()`object to reference the pingback count, and use that `F()` object in the query:

예를 들어 pingback 보다 더 많은 comments를 가지고 있는 blog 리스트를 찾기 위해서 우리는 F()객체를 만들고 pingback을 참조하는 방법으로 목표를 달성할 수 있다.

```python
>>> from django.db.models import F
>>> Entry.objects.filter(n_comments__gt=F('n_pingbacks'))
```

(참고로 필드 이름은 각각 `n_comments`, `n_pingbacks`로 만들어 놓음)

> Django supports the use of addition, subtraction, multiplication, division, modulo, and power arithmetic with `F()`objects, both with constants and with other `F()` objects. To find all the blog entries with more than *twice* as many comments as pingbacks, we modify the query:
>
> ```python
> >>>Entry.objects.filter(n_comments__gt=F('n_pingbacks') * 2)
> ```

장고는 상수와의 더하기, 빼기, 곱하기, 나누기 모듈을 F()과 함께 지원한다. 예시에서는 pingback의 2배 이상의 comment를 가지고 있는 Entry 의 리스트를 찾아내는 filter

> To find all the entries where the rating of the entry is less than the sum of the pingback count and comment count, we would issue the query:
>
> ```python
> >>> Entry.objects.filter(rating__lt=F('n_comments') + F('n_pingbacks'))
> ```

comments와 pingbacks의 수를 합친 것보다 작은 rating을 가지고 있는 Entry를 찾아내기 위하여 예시와 같은 쿼리셋을 만들수도 있다. 

> You can also use the double underscore notation to span relationships in an `F()` object. An `F()` object with a double underscore will introduce any joins needed to access the related object. For example, to retrieve all the entries where the author's name is the same as the blog name, we could issue the query:
>
> ```python
> >>> Entry.objects.filter(authors__name=F('blog__name'))
> ```

또한 F()안에서 더블언더스코어 표현법을 사용하여 관계를 확장하여 필터링 하는 기능 또한 사용할 수 있다. 예시는 author의 이름과 blog의 이름이 같은 Entry의 리스트를 찾는 쿼리다.

> For date and date/time fields, you can add or subtract a [`timedelta`](https://docs.python.org/3/library/datetime.html#datetime.timedelta) object. The following would return all entries that were modified more than 3 days after they were published:
>
> ```python
> >>> from datetime import timedelta
> >>> Entry.objects.filter(mod_date__gt=F('pub_date') + timedelta(days=3))
> ```

date/time 필드에서는 timedelta 객체를 더하거나 뺄 수 있다. 예시에서는 발행된지 3일 이후에 수정된 Entry의 리스트를 찾는다. 

> The `F()` objects support bitwise operations by `.bitand()`, `.bitor()`, `.bitrightshift()`, and `.bitleftshift()`. For example:
>
> ```python
> >>> F('somefield').bitand(16)
> ```

F() 객체는 바이트단위 기능도 제공한다. 아래는 예시



#### The pk lookup shortcut

쿼리에서의 pk값의 다양한 사용이 가능하다. 

```python
# Get blogs entries with id 1, 4 and 7
>>> Blog.objects.filter(pk__in=[1,4,7])

# Get all blog entries with id > 14
>>> Blog.objects.filter(pk__gt=14)
```

pk값 조회는 관계의 단계를 넘어서도 가능하다. 다음은 동일한 예시 3문장

```python
>>> Entry.objects.filter(blog__id__exact=3) # Explicit form
>>> Entry.objects.filter(blog__id=3)        # __exact is implied
>>> Entry.objects.filter(blog__pk=3)        # __pk implies __id__exact
```



#### Escaping percent signs and underscores in LIKE statements

장고에서 필드를 찾아보는 쿼리를 사용할 때 퍼센트 마크와(`%`) 언더스코어는(`_`)자동으로 escape 문자가 적용된다. 왜냐하면 SQL의 like문 에서는 `%`는 multiple character wildcard , `_`는 single character wildcard를 의미하기 때문이다. 



#### Caching and QuerySets

> Each [`QuerySet`](https://docs.djangoproject.com/ko/1.11/ref/models/querysets/#django.db.models.query.QuerySet) contains a cache to minimize database access. Understanding how it works will allow you to write the most efficient code.

각각의 쿼리셋은 데이터베이스 접근을 최소화 하기 위하여 캐시값을 보유하고 있다. 

> In a newly created [`QuerySet`](https://docs.djangoproject.com/ko/1.11/ref/models/querysets/#django.db.models.query.QuerySet), the cache is empty. The first time a [`QuerySet`](https://docs.djangoproject.com/ko/1.11/ref/models/querysets/#django.db.models.query.QuerySet) is evaluated -- and, hence, a database query happens -- Django saves the query results in the [`QuerySet`](https://docs.djangoproject.com/ko/1.11/ref/models/querysets/#django.db.models.query.QuerySet)’s cache and returns the results that have been explicitly requested (e.g., the next element, if the [`QuerySet`](https://docs.djangoproject.com/ko/1.11/ref/models/querysets/#django.db.models.query.QuerySet) is being iterated over). Subsequent evaluations of the [`QuerySet`](https://docs.djangoproject.com/ko/1.11/ref/models/querysets/#django.db.models.query.QuerySet) reuse the cached results.

새로생성되 쿼리의 캐시값은 비어있습니다. 쿼리가 처음 평가되고 이에따라 데이터베이스 쿼리가 발생하면 장고는 쿼리의 결과를 쿼리케시에 저장합니다. 그리고 그것이 명시적으로 요청되었을 때 캐시에 가지고 있는 결과값을 리턴합니다. 뒤에 따라오는 쿼리셋의 평가는 캐시값의 결과를 재사용 합니다. 

> Keep this caching behavior in mind, because it may bite you if you don't use your [`QuerySet`](https://docs.djangoproject.com/ko/1.11/ref/models/querysets/#django.db.models.query.QuerySet)s correctly. For example, the following will create two [`QuerySet`](https://docs.djangoproject.com/ko/1.11/ref/models/querysets/#django.db.models.query.QuerySet)s, evaluate them, and throw them away:
>
> ```python
> >>> print([e.headline for e in Entry.objects.all()])
> >>> print([e.pub_date for e in Entry.objects.all()])
> ```

캐시가 이런식으로 움직인다는 것을 명심해야 합니다. 왜냐하면 이러한 사실은 당신이 쿼리셋을 올바르게 사용하지 않도록 할 수 있기 때문입니다. 예를들면 예시에서는 두개의 쿼리를 생성할 것입니다. 그리고 평가한 후에 버려질 것 입니다. 

> That means the same database query will be executed twice, effectively doubling your database load. Also, there's a possibility the two lists may not include the same database records, because an `Entry` may have been added or deleted in the split second between the two requests.
>
> To avoid this problem, simply save the [`QuerySet`](https://docs.djangoproject.com/ko/1.11/ref/models/querysets/#django.db.models.query.QuerySet) and reuse it:
>
> ```python
> >>> queryset = Entry.objects.all()
> >>> print([p.headline for p in queryset]) # Evaluate the query set.
> >>> print([p.pub_date for p in queryset]) # Re-use the cache from the evaluation.
> ```

이것은 동일한 데이터베이스 쿼리가 두번 실행 되리라는 것을 의미합니다. 실제로 당신의 데이터베이스는 두번 로드됩니다. 또한 두개의 리스트는 동일한 데이터베이스의 기록을 공유하고 있지 않을 가능성도 있습니다. 왜냐하면 두개의 쿼리가 동작하는 짧은 시간 사이에 Entry가 추가되거나 지워질 수도 있기 때문입니다. 

이러한 문제를 피하기 위해서 우리는 간단하게 쿼리셋을 저장하고 다시 재사용할 수 있습니다. 

(그러니까 첫번째와 두 번째 쿼리가 작동할 때 각각 데이터베이스를 치게 되는데, 아래쪽의 예시에서는 두번째 쿼리에서 첫번 째 쿼리의 캐시값을 재사용 하기 때문에 더 효율적이라는 이야기, 더 자세한 쿼리 최적화 관련해서는 다른 페이지의 관련 문서를 읽어봐야 할 것 같다. [[Database access optimization]](https://docs.djangoproject.com/en/2.0/topics/db/optimization/)

##### When QuerySets are not cached

> Querysets do not always cache their results. When evaluating only *part* of the queryset, the cache is checked, but if it is not populated then the items returned by the subsequent query are not cached. Specifically, this means that[limiting the queryset](https://docs.djangoproject.com/ko/1.11/topics/db/queries/#limiting-querysets) using an array slice or an index will not populate the cache.

쿼리셋이 언제나 그 결과를 캐시에 저장하는 것은 아닙니다. 쿼리셋의 일부만 평가 할 때에는 캐시값을 확인하지만, 만약 뒤에오는 쿼리에 의해 반환되는 아이템이 없을 경우에 쿼리는 캐시값을 저장하지 않습니다. 특히 이것은 쿼리값을 캐시로 저장하지 않는 슬라이스나 인덱스를 이용한 쿼리 사용을 제한한다는 것을 의미합니다. 

> For example, repeatedly getting a certain index in a queryset object will query the database each time:
>
> ```python
> >>> queryset = Entry.objects.all()
> >>> print(queryset[5]) # Queries the database
> >>> print(queryset[5]) # Queries the database again
> ```

예를 들어 쿼리 객체에서 특정 인덱스를 반복해서 호출하는 것은 매번 데이터베이스에 쿼리를 날릴 것 입니다. 

>However, if the entire queryset has already been evaluated, the cache will be checked instead:
>
>```python
>>>> queryset = Entry.objects.all()
>>>> [entry for entry in queryset] # Queries the database
>>>> print(queryset[5]) # Uses cache
>>>> print(queryset[5]) # Uses cache
>```

그러나 전체 쿼리가 이미 평가되었다면 캐시값은 저장 될 것입니다.

> Here are some examples of other actions that will result in the entire queryset being evaluated and therefore populate the cache:
>
> ```python
> >>> [entry for entry in queryset]
> >>> bool(queryset)
> >>> entry in queryset
> >>> list(queryset)
> ```

이번 예문은 전체쿼리가 평가되고 그럼으로써 캐시값을 가지게 됩니다. 

(간단히 말해서 슬라이스나, 인덱스를 이용해서 쿼리를 날리는 경우 캐시되지 않기 때문에 주의하라는 내용)



### Complex lookups with Q objects

> keyword argument queries -- in [`filter()`](https://docs.djangoproject.com/ko/1.11/ref/models/querysets/#django.db.models.query.QuerySet.filter), etc. -- are "AND"ed together. If you need to execute more complex queries (for example, queries with `OR` statements), you can use [`Q objects`](https://docs.djangoproject.com/ko/1.11/ref/models/querysets/#django.db.models.Q).
>
> A [`Q object`](https://docs.djangoproject.com/ko/1.11/ref/models/querysets/#django.db.models.Q) (`django.db.models.Q`) is an object used to encapsulate a collection of keyword arguments. These keyword arguments are specified as in "Field lookups" above.
>
> For example, this `Q` object encapsulates a single `LIKE` query:
>
> ```python
> from django.db.models import Q
> Q(question__startswith='What')
> ```

filter()와 같은 쿼리의 키워드는 AND의 형태로 묶여서 사용됩니다. 만약 더 복잡한 쿼리동작이 필요하다면 (예를 들면 OR조건문과 같이),  Q objects를 사용할 수 있습니다. 

Q object는 키워드인자모음을 캡슐화 하기 위해서 사용되는 객체입니다. 이런 키워드인자는 "Field lookups"의 인자로 지정됩니다. 

예시에서는 Q object가 하나의 LIKE쿼리를 캡슐화 합니다.

> `Q` objects can be combined using the `&` and `|` operators. When an operator is used on two `Q` objects, it yields a new `Q`object.
>
> For example, this statement yields a single `Q` object that represents the "OR" of two `"question__startswith"`queries:
>
> ```python
> Q(question__startswith='Who') | Q(question__startswith='What')
> ```

Q objects는 `&`와 `|`를 연산자를 사용하여 묶을 수 있습니다. 두개의 Q objects 에서 하나의 연산자가 사용되면, 새로운 Q object는 무시됩니다.

예시에서는 하나의 Q object가 무시되는데 이것은 두개의 `"question__startswith"`쿼리가 `OR의 관계임을 나타냅니다.

>This is equivalent to the following SQL `WHERE` clause:
>
>```python
>WHERE question LIKE 'Who%' OR question LIKE 'What%'
>```

이것은 이전 예문과 동일한 SQL WHERE절 입니다.

> You can compose statements of arbitrary complexity by combining `Q` objects with the `&` and `|` operators and use parenthetical grouping. Also, `Q` objects can be negated using the `~` operator, allowing for combined lookups that combine both a normal query and a negated (`NOT`) query:
>
> ```python
> Q(question__startswith='Who') | ~Q(pub_date__year=2005)
> ```
>
> 

당시는 `&`와 `|` 그리고 괄호를 사용하여 Q objects를 조합해서 임의의 복잡한 문장을 만들어낼 수 있습니다. 또한 Q objects는 `~`를 사용하여 부정의 의미로 사용할 수 있으며, 이것은 일반적인 커리와 부정의 의미인 쿼리를 묶어서 사용할 수 있게 해줍니다.  

> Each lookup function that takes keyword-arguments (e.g. [`filter()`](https://docs.djangoproject.com/ko/1.11/ref/models/querysets/#django.db.models.query.QuerySet.filter), [`exclude()`](https://docs.djangoproject.com/ko/1.11/ref/models/querysets/#django.db.models.query.QuerySet.exclude), [`get()`](https://docs.djangoproject.com/ko/1.11/ref/models/querysets/#django.db.models.query.QuerySet.get)) can also be passed one or more `Q` objects as positional (not-named) arguments. If you provide multiple `Q` object arguments to a lookup function, the arguments will be "AND"ed together. For example:
>
> ```python
> Poll.objects.get(
>     Q(question__startswith='Who'),
>     Q(pub_date=date(2005, 5, 2)) | Q(pub_date=date(2005, 5, 6))
> )
> ```
>
> ... roughly translates into the SQL:
>
> ```python
> SELECT * from polls WHERE question LIKE 'Who%'
>     AND (pub_date = '2005-05-02' OR pub_date = '2005-05-06')
> ```

filter(), exclude(), get()과 같이 키워드 인자를 가지는 검색 function은 하나 이상의 Q objects를 위치 인자로 가질 수 있습니다. 만약 복수의 Q object 인자를 검색function에 전달 할 경우, 전달 된 인자들은 "AND로 엮이게 됩니다.

( `|` 로 붂인것은 "OR"로 인식되고, `,` 구분된 인자로 된 것들 끼리는 서로 "AND"로 묶인다는 이야기)

> Lookup functions can mix the use of `Q` objects and keyword arguments. All arguments provided to a lookup function (be they keyword arguments or `Q` objects) are "AND"ed together. However, if a `Q` object is provided, it must precede the definition of any keyword arguments. For example:
>
> ```python
> Poll.objects.get(
>     Q(pub_date=date(2005, 5, 2)) | Q(pub_date=date(2005, 5, 6)),
>     question__startswith='Who',
> )
> ```
>
> ... would be a valid query, equivalent to the previous example; but:
>
> ```python
> # INVALID QUERY
> Poll.objects.get(
>     question__startswith='Who',
>     Q(pub_date=date(2005, 5, 2)) | Q(pub_date=date(2005, 5, 6))
> )
> ```
>
> ... would not be valid.

검색 function은 Q objects와 키워드 인자를 함께 사용하여 구성할 수 있다. 전달된 모든 인자는 "AND"로 엮이게 된다. 하지만, 만약 Q object를 함께 사용하는 경우에 키워드 인자의 순서를 명확하게 해주어야만 합니다다.

(python은 function을 사용할 때 위치인자를 먼저 앞에두고 키워드 인자는 뒷쪽에 두는 순서로 사용해야만 한다는 이야기)



### Deleting objects

> The delete method, conveniently, is named [`delete()`](https://docs.djangoproject.com/ko/1.11/ref/models/instances/#django.db.models.Model.delete). This method immediately deletes the object and returns the number of objects deleted and a dictionary with the number of deletions per object type. Example:
>
> ```python
> >>> e.delete()
> (1, {'weblog.Entry': 1})
> ```
>

삭제 메소드는 `delete()`로 이름지어져 있다. 이 메소드는 즉시 해당 객체를 지우고 삭제된 객체의 갯수와, 삭제된 각 객체의 타입을 담은 딕셔너리를 반환합니다.

>You can also delete objects in bulk. Every [`QuerySet`](https://docs.djangoproject.com/ko/1.11/ref/models/querysets/#django.db.models.query.QuerySet) has a [`delete()`](https://docs.djangoproject.com/ko/1.11/ref/models/querysets/#django.db.models.query.QuerySet.delete) method, which deletes all members of that [`QuerySet`](https://docs.djangoproject.com/ko/1.11/ref/models/querysets/#django.db.models.query.QuerySet).
>
>For example, this deletes all `Entry` objects with a `pub_date` year of 2005:
>
>```python
>>>> Entry.objects.filter(pub_date__year=2005).delete()
>(5, {'webapp.Entry': 5})
>```

당신은 많은 야으이 객체를 한번에 삭제할 수도 있다. 모든 쿼리셋은 `delete()`메서드를 가지며 쿼리셋으로 가져오는 모든 객체를 삭제한다. 예시에서는 `pub_date`가 2005년인 모든 Entry를 삭제합니다.

> Keep in mind that this will, whenever possible, be executed purely in SQL, and so the `delete()` methods of individual object instances will not necessarily be called during the process. If you've provided a custom `delete()` method on a model class and want to ensure that it is called, you will need to "manually" delete instances of that model (e.g., by iterating over a [`QuerySet`](https://docs.djangoproject.com/ko/1.11/ref/models/querysets/#django.db.models.query.QuerySet) and calling `delete()` on each object individually) rather than using the bulk [`delete()`](https://docs.djangoproject.com/ko/1.11/ref/models/querysets/#django.db.models.query.QuerySet.delete) method of a [`QuerySet`](https://docs.djangoproject.com/ko/1.11/ref/models/querysets/#django.db.models.query.QuerySet).

가능할 때 마다 이것이 순전히 SQL에서 작동된다는 점을 명심하십시요, 그럼으로써 각 객체의  `delete()`메소드가 지우는 과정에 있어서 필요하지 않을 것입니다. 만약 모델 클래스에 커스텀 `delete()`메소드를 만들고, 그것이 사용되어지기를 확실하게 하기 위해서는 한꺼번에 `delete()`메서드를 사용하기 보다는 일일이 모델 인스턴스를 지워야 할 것입니다.

> When Django deletes an object, by default it emulates the behavior of the SQL constraint `ON DELETECASCADE` -- in other words, any objects which had foreign keys pointing at the object to be deleted will be deleted along with it. For example:
>
> ```python
> b = Blog.objects.get(pk=1)
> # This will delete the Blog and all of its Entry objects.
> b.delete()
> ```
> This cascade behavior is customizable via the on_delete argument to the ForeignKey.

장고가 객체를 삭제할 때에는, 기본적으로 SQL에 강제되어 있는 `ON DELETECASCADE`를 모방합니다. 다르게 말하자면, 지워지는 객체를 foreign key로 지정하고 있는 객체는 함께 지워진다는 말 입니다.

cascade 동작은 `on_delete`인자를 통해서 ForeignKey에서 원하는 대로 조정할 수 있습니다.

> Note that [`delete()`](https://docs.djangoproject.com/ko/1.11/ref/models/querysets/#django.db.models.query.QuerySet.delete) is the only [`QuerySet`](https://docs.djangoproject.com/ko/1.11/ref/models/querysets/#django.db.models.query.QuerySet) method that is not exposed on a [`Manager`](https://docs.djangoproject.com/ko/1.11/topics/db/managers/#django.db.models.Manager) itself. This is a safety mechanism to prevent you from accidentally requesting `Entry.objects.delete()`, and deleting *all* the entries. If you *do* want to delete all the objects, then you have to explicitly request a complete query set:
>
> ```python
> Entry.objects.all().delete()
> ```

`delete()`메서드는 Manager에서 파생되지 않은 유일한 쿼리셋 메서드라는 점을 기억하세요. 이거는 당신이 우발적으로 `Entry.objects.delete()`와 같이 요청해서 모든 엔트리를 지우게끔 하는 것을 방지하기 위한 장치입니다. 만약 모든 객체를 지우고 싶다면 전체 request 쿼리셋 문장을 명시해야 합니다.



### Copying model instances

> Although there is no built-in method for copying model instances, it is possible to easily create new instance with all fields' values copied. In the simplest case, you can just set `pk` to `None`. Using our blog example:
>
> ```python
> blog = Blog(name='My blog', tagline='Blogging is easy')
> blog.save() # blog.pk == 1
>
> blog.pk = None
> blog.save() # blog.pk == 2
> ```

모델인스턴스를 복사사하는 내장 메서드가 따로 있는건 아니지만 간단하게 인스턴스의 전체 필드 내용을 복사하는 것이 가능합니다. 가장 간단한 캐이스로, `pk`값을 `None` 으로 지정하는 것 이다. 

(pk 필드는 자동으로 생성되니까 None으로 설정하고 새로 저장하면 같은 내용의 새로운 인스턴스가 생성되는 것)

> Things get more complicated if you use inheritance. Consider a subclass of `Blog`:
>
> ```python
> class ThemeBlog(Blog):
>     theme = models.CharField(max_length=200)
>
> django_blog = ThemeBlog(name='Django', tagline='Django is easy', theme='python')
> django_blog.save() # django_blog.pk == 3
>
> ```
>
> Due to how inheritance works, you have to set both `pk` and `id` to None:
>
> ```python
> django_blog.pk = None
> django_blog.id = None
> django_blog.save() # django_blog.pk == 4
> ```

더 복잡한 경우는 상속을 사용하는 경우다. 이런경우는 `pk`와 `id` 모두 `None`을 할당한 후에 save()해야 복사된다.

> This process doesn't copy relations that aren't part of the model's database table. For example, `Entry` has a `ManyToManyField` to `Author`. After duplicating an entry, you must set the many-to-many relations for the new entry:
>
> ```python
> entry = Entry.objects.all()[0] # some previous entry
> old_authors = entry.authors.all()
> entry.pk = None
> entry.save()
> entry.authors.set(old_authors)
> ```

복제기능은 해당모델의 데이이터베이스 테이블의 일부분이 아닌 관계모델에 대해서는 지원하지 않습니다. 예를들면 `Entry`를 복제하게 될 경우 save()를 한 후에 mtm관계에 있는 `Author`는 따로 지정해해 주어야 합니다.

> For a `OneToOneField`, you must duplicate the related object and assign it to the new object's field to avoid violating the one-to-one unique constraint. For example, assuming `entry` is already duplicated as above:
>
> ```python
> detail = EntryDetail.objects.all()[0]
> detail.pk = None
> detail.entry = entry
> detail.save()
> ```

oto 필드의 경우, save() 전에 관계모델을 지정해 주어야하는데 oto필드의 연결된 모델간의 1대1 연결방식을 유지하기 위함입니다. 



### Updating multiple objects at once

> Sometimes you want to set a field to a particular value for all the objects in a [`QuerySet`](https://docs.djangoproject.com/ko/1.11/ref/models/querysets/#django.db.models.query.QuerySet). You can do this with the [`update()`](https://docs.djangoproject.com/ko/1.11/ref/models/querysets/#django.db.models.query.QuerySet.update) method. For example:
>
> ```python
> # Update all the headlines with pub_date in 2007.
> Entry.objects.filter(pub_date__year=2007).update(headline='Everything is the same')
> ```

당신의 하나의 쿼리문을 통해서 특정 변수를 필드개체에 입력할 수 있습니다. `update()`문을 활용하면 되는데 다음 예시와 같이 활용할 수 있다.

> You can only set non-relation fields and [`ForeignKey`](https://docs.djangoproject.com/ko/1.11/ref/models/fields/#django.db.models.ForeignKey) fields using this method. To update a non-relation field, provide the new value as a constant. To update [`ForeignKey`](https://docs.djangoproject.com/ko/1.11/ref/models/fields/#django.db.models.ForeignKey) fields, set the new value to be the new model instance you want to point to. For example:
>
> ```python
> >>> b = Blog.objects.get(pk=1)
>
> # Change every Entry so that it belongs to this Blog.
> >>> Entry.objects.all().update(blog=b)
> ```

다른 모델과 관계 설정이 없는 일반 필드와, ForeignKey 필드에 `update()`메소드를 사용할 수 있습니다. 다른 모델과 관계 설정이 없는 필드의 경우 새로운 변수를 제공하면 됩니다. ForeignKey 필드의 경우 지정하고자 하는 새로운 모델 인스턴스에 새 변수를 지정할 수 있습니다. 

> The `update()` method is applied instantly and returns the number of rows matched by the query (which may not be equal to the number of rows updated if some rows already have the new value). The only restriction on the [`QuerySet`](https://docs.djangoproject.com/ko/1.11/ref/models/querysets/#django.db.models.query.QuerySet) being updated is that it can only access one database table: the model's main table. You can filter based on related fields, but you can only update columns in the model's main table. Example:
>
> ```python
> >>> b = Blog.objects.get(pk=1)
>
> # Update all the headlines belonging to this Blog.
> >>> Entry.objects.select_related().filter(blog=b).update(headline='Everything is the same')
> ```

`update()`메소드는 즉시 적용되며 쿼리문에 의하여 매칭되는 행들의 숫자를 반환합니다(만약 다른 행에서 업데이트 하는 새로운 변수를 이미 가지고 있다면, 리턴하는 행의 숫자는 업데이트된 행의 숫자와 같지 않을 수 있습니다.). 업데이트 되는 쿼리문에 대한 유일한 제약사항은 하나의 데이터베이스 테이블에만 접근 가능하다는 것 입니다: 그 테이블은 해당 모델의 메인 테이블 입니다. 당신은 관계된 필드들을 기반으로 필터를 사용할 수 있습니다. 하지만 모델의 메인 테이블의 열만 새로 추가할 수 있습니다. 

> Be aware that the `update()` method is converted directly to an SQL statement. It is a bulk operation for direct updates. It doesn't run any [`save()`](https://docs.djangoproject.com/ko/1.11/ref/models/instances/#django.db.models.Model.save) methods on your models, or emit the `pre_save` or `post_save` signals (which are a consequence of calling [`save()`](https://docs.djangoproject.com/ko/1.11/ref/models/instances/#django.db.models.Model.save)), or honor the [`auto_now`](https://docs.djangoproject.com/ko/1.11/ref/models/fields/#django.db.models.DateField.auto_now) field option. If you want to save every item in a [`QuerySet`](https://docs.djangoproject.com/ko/1.11/ref/models/querysets/#django.db.models.query.QuerySet) and make sure that the [`save()`](https://docs.djangoproject.com/ko/1.11/ref/models/instances/#django.db.models.Model.save) method is called on each instance, you don't need any special function to handle that. Just loop over them and call [`save()`](https://docs.djangoproject.com/ko/1.11/ref/models/instances/#django.db.models.Model.save):
>
> ```python
> for item in my_queryset:
>     item.save()
> ```

`update()`메소드가 직접적인 방식을 통해 SQL문으로 전환된다는 것을 명심하십시오. 이것은 바로 업데이트를 하기 위한 큰 규모의 작동입니다.  이것은 당신의 모델에서  `save()`메소드를 사용하지 않으며, `pre_save`나 `post_save`와 같은 시그널을 보내거나(둘다 `save()`메서드를 호출하기 위한 순서이다), `auto_now`필드의 혜택을 받지 않습니다. 만약 모든 아이템들을 하나의 쿼리셋으로 처리하고 싶고, `save()`메서드를 각각의 인스턴스마다 확실하게 호출 하고 싶다면, 다른 특별한 함수가 필요하지 않습니다. 그냥 루프를 돌면서 `save()`메서드를 호출하면 됩니다. 

> Calls to update can also use [`F expressions`](https://docs.djangoproject.com/ko/1.11/ref/models/expressions/#django.db.models.F) to update one field based on the value of another field in the model. This is especially useful for incrementing counters based upon their current value. For example, to increment the pingback count for every entry in the blog:
>
> ```python
> >>> Entry.objects.all().update(n_pingbacks=F('n_pingbacks') + 1)
> ```

다른 필드의 값에 기반하여 특정 필드의 인스턴스 값을 업데이트 하는 작업을  update를 호출과 `F_expressions`함께 사용해서 완성할 수 있습니다. 이것은 특별히 이미 존재하는 값을기반으로 하여 숫자를 증가시키는 데에 유용합니다.  



### Related objects

> When you define a relationship in a model (i.e., a [`ForeignKey`](https://docs.djangoproject.com/ko/1.11/ref/models/fields/#django.db.models.ForeignKey), [`OneToOneField`](https://docs.djangoproject.com/ko/1.11/ref/models/fields/#django.db.models.OneToOneField), or[`ManyToManyField`](https://docs.djangoproject.com/ko/1.11/ref/models/fields/#django.db.models.ManyToManyField)), instances of that model will have a convenient API to access the related object(s).
>
> Using the models at the top of this page, for example, an `Entry` object `e` can get its associated `Blog`object by accessing the `blog` attribute: `e.blog`.
>
> (Behind the scenes, this functionality is implemented by Python [descriptors](https://docs.python.org/3/howto/descriptor.html). This shouldn't really matter to you, but we point it out here for the curious.)
>
> Django also creates API accessors for the "other" side of the relationship -- the link from the related model to the model that defines the relationship. For example, a `Blog` object `b` has access to a list of all related `Entry` objects via the `entry_set` attribute: `b.entry_set.all()`.
>
> All examples in this section use the sample `Blog`, `Author` and `Entry` models defined at the top of this page.

만약 특정모델에 관계를 정의 했다면(ForeignKey, OneToOneField, 혹은 ManyToManyField), 해당 모델의 인스턴스는 관계된 모델에 접근하기 위한 편리한 API를 가질 것 입니다. 

예를 들어 이페이지 꼭대기에 있는 모델을 사용할 경우, 하나의 Entry object 인 e는 blog라는 속성에 접근하는 것을 통하여 e와 연관된 Blog 오브젝트를 가질 수 있습니다. 

(이러한 장면의 뒤에서, 해당 기능은  Python의 descriptors의 기능이 적용되었습니다. 이것은 크게 중요하지는 않지만 궁금한 사람들을 위해 남겨두었습니다.)

장고역시 관계의 다른쪽 방향(관계설정의 대상이 되는 모델에서 관계설정이 정의된 모델로의 방향, 역참조)의 접근을 위한 API를 생성합니다. 예를 들자면 Blog 오브젝트인 b는 `entry_set`속성을 통해서(b.entry_set.all()) 관계설정이 된 모든 Entry오브젝트의 리스트에 접근한다. 

이번섹션의 모든 예문은 페이지 가장 위에 있는 샘플인 Blog, Author 그리고 Entry 모델을 사용한다.

#### One-to-many relationships

##### Forward

> If a model has a [`ForeignKey`](https://docs.djangoproject.com/ko/1.11/ref/models/fields/#django.db.models.ForeignKey), instances of that model will have access to the related (foreign) object via a simple attribute of the model.
>
> Example:
>
> ```python
> >>> e = Entry.objects.get(id=2)
> >>> e.blog # Returns the related Blog object.
> ```

만약 모델이 ForeignKey를 가졌다면, 해당 모델의 인스턴스는 간단한 모델 속성을 통하여 관계된(foreign)오브젝트로의 접근이 가능하다.

> You can get and set via a foreign-key attribute. As you may expect, changes to the foreign key aren't saved to the database until you call [`save()`](https://docs.djangoproject.com/ko/1.11/ref/models/instances/#django.db.models.Model.save). Example:
>
> ```python
> >>> e = Entry.objects.get(id=2)
> >>> e.blog = some_blog
> >>> e.save()
> ```

foreign-key 속성을 통해 값을 직접 값을 설정하거나 가져올 수 있습니다. 이미 예상하고 있듯이 foreign key를 바꾸는 것은 `save()`메서드를 호출하기 전까지는 데이터베이스에 저장되지 않습니다.

