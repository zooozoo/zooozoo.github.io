---
title: Spanning multi-valued relationships
tag: django
excerpt: >-
 장고의 모델 문서를 공부하다가[(Making Queries)](https://docs.djangoproject.com/ko/1.11/topics/db/queries/#spanning-multi-valued-relationships) 이전에 패캠에서 수업들을 때 한번 이해 했던 내용인데 다시보니 너무 헷갈려서 폭풍써치를 감행했고 비로소 이해를 하게 되었다. (하... 내 시간...)
category: post
---

장고의 모델 문서를 공부하다가[(Making Queries)](https://docs.djangoproject.com/ko/1.11/topics/db/queries/#spanning-multi-valued-relationships) 이전에 패캠에서 수업들을 때 한번 이해 했던 내용인데 다시보니 너무 헷갈려서 폭풍써치를 감행했고 비로소 이해를 하게 되었다. (하... 내 시간...)



해당 내용을 이미 알고 있고 스압을 느끼신다면 [stackoverflow 의 답변](https://stackoverflow.com/questions/5542874/difference-between-filter-with-multiple-arguments-and-chain-filter-in-django)으로 바로 가보시길...



해당 부분은 `Spanning multi-valued relationships` 라는 부분인데 ForeignKey나 Many-to-Many관계에있는 모델중에서 관계된 모델을 조건으로 해서 filtering 하는 내용이다.

장고 문서에서는 아래의 모델을 기준으로 설명한다.

```python
from django.db import models

class Blog(models.Model):
    name = models.CharField(max_length=100)
    tagline = models.TextField()

    def __str__(self):
        return self.name

class Author(models.Model):
    name = models.CharField(max_length=200)
    email = models.EmailField()

    def __str__(self):
        return self.name

class Entry(models.Model):
    blog = models.ForeignKey(Blog, on_delete=models.CASCADE)
    headline = models.CharField(max_length=255)
    body_text = models.TextField()
    pub_date = models.DateField()
    mod_date = models.DateField()
    authors = models.ManyToManyField(Author)
    n_comments = models.IntegerField()
    n_pingbacks = models.IntegerField()
    rating = models.IntegerField()

    def __str__(self):
        return self.headline
```

해당 모델은 Blog (일) < - > (다) Entry < - > (다)Author 의 관계를 가지고 있다.

다음은 장고 문서에서 설명하는 내용

> To handle both of these situations, Django has a consistent way of processing [`filter()`](https://docs.djangoproject.com/ko/1.11/ref/models/querysets/#django.db.models.query.QuerySet.filter) calls. Everything inside a single [`filter()`](https://docs.djangoproject.com/ko/1.11/ref/models/querysets/#django.db.models.query.QuerySet.filter) call is applied simultaneously to filter out items matching all those requirements. Successive [`filter()`](https://docs.djangoproject.com/ko/1.11/ref/models/querysets/#django.db.models.query.QuerySet.filter) calls further restrict the set of objects, but for multi-valued relations, they apply to any object linked to the primary model, not necessarily those objects that were selected by an earlier [`filter()`](https://docs.djangoproject.com/ko/1.11/ref/models/querysets/#django.db.models.query.QuerySet.filter) call.
>
> That may sound a bit confusing, so hopefully an example will clarify. To select all blogs that contain entries with both *"Lennon"* in the headline and that were published in 2008 (the same entry satisfying both conditions), we would write:
>
> ```python
Blog.objects.filter(entry__headline__contains='Lennon', entry__pub_date__year=2008)
```
>
> To select all blogs that contain an entry with *"Lennon"* in the headline **as well as** an entry that was published in 2008, we would write:
>
> ```python
 Blog.objects.filter(entry__headline__contains='Lennon').filter(entry__pub_date__year=2008)
```
>
> Suppose there is only one blog that had both entries containing *"Lennon"* and entries from 2008, but that none of the entries from 2008 contained *"Lennon"*. The first query would not return any blogs, but the second query would return that one blog.
>
> In the second example, the first filter restricts the queryset to all those blogs linked to entries with *"Lennon"* in the headline. The second filter restricts the set of blogs *further* to those that are also linked to entries that were published in 2008. The entries selected by the second filter may or may not be the same as the entries in the first filter. We are filtering the `Blog` items with each filter statement, not the `Entry` items.

요약해서 설명하자면

```python
Blog.objects.filter(entry__headline__contains='Lennon', entry__pub_date__year=2008)
```

와 같이 필터 안에 여러개의 조건을 써서 찾는 것과

```python
Blog.objects.filter(entry__headline__contains='Lennon').filter(entry__pub_date__year=2008)
```

이런식으로(chaining) 필터를 나눠서 각각 찾는것이 다른 결과를 가지며 그 이유는 결과가 Entry를 탐색하는 것이 아닌  Blog를 탐색하는 것이기 때문이라고 설명하고 있다.

두 번째 예시가 잘 이해되지 않았는데 만약 첫번째 filter로 blog 객체를 추려낸 후에 해당 결과를 가지고 두 번째 filter를 적용하면 어떻게 하든 첫 번째 예시와 같은 결과를 할 것 같다. 결국에 돌고 돌아 stackoverflow를 통해 내용을 이해할 수 있었다.

누군가 나랑 똑같은 질문을 stackoverflow에 했고 아주 상세한 설명이 있어서 덕분해 잘 이해할 수 있었다.


장고에 있는 문서보다는 [stackoverflow 에 예시](https://stackoverflow.com/questions/5542874/difference-between-filter-with-multiple-arguments-and-chain-filter-in-django)로 쓰여진 내용이 훨씬 잘 이해 된다.



아래는 해당 답변 내용이다.

> The case in which results of "multiple arguments filter-query" is different then "chained-filter-query", following:
>
> > Selecting referenced objects on the basis of referencing objects and relationship is one-to-many (or many-to-many).
> >
> > Multiple filters:
> >
> > ```python
 Referenced.filter(referencing1_a=x, referencing1_b=y)
#  same referencing model   ^^                ^^
```
> >
> > Chained filters:
> >
> > ```python
Referenced.filter(referencing1_a=x).filter(referencing1_b=y)
```
> >
> > Both queries can output different result:
> > If more then one rows in referencing-model`Referencing1`can refer to same row in referenced-model`Referenced`. This can be the case in `Referenced`: `Referencing1` have either 1:N (one to many) or N:M (many to many) relation-ship.
>
> Example:
>
> Consider my application `my_company` has two models `Employee` and `Dependent`. An employee in `my_company` can have more than dependents(in other-words a dependent can be son/daughter of a single employee, while a employee can have more than one son/daughter).
> Ehh, assuming like husband-wife both can't work in a `my_company`. I took 1:m example
>
> So, `Employee` is referenced-model that can be referenced by more then `Dependent` that is referencing-model. Now consider relation-state as follows:
>
> > ```
> > Employee:        Dependent:
> > +------+        +------+--------+-------------+--------------+
> > | name |        | name | E-name | school_mark | college_mark |
> > +------+        +------+--------+-------------+--------------+
> > | A    |        | a1   |   A    |          79 |           81 |
> > | B    |        | b1   |   B    |          80 |           60 |
> > +------+        | b2   |   B    |          68 |           86 |
> >                 +------+--------+-------------+--------------+  
> > ```
> >
> > Dependent`a1`refers to employee`A`, and dependent`b1, b2`references to employee`B`.
>
> Now my query is:
>
> Find all employees those having son/daughter has distinction marks (say >= 75%) in both college and school?
>
> ```python
> >>> Employee.objects.filter(dependent__school_mark__gte=75,
> ...                         dependent__college_mark__gte=75)
>
> [<Employee: A>]
> ```
>
> Output is 'A' dependent 'a1' has distinction marks in both college and school is dependent on employee 'A'. Note 'B' is not selected because nether of 'B''s child has distinction marks in both college and school. Relational algebra:
>
> > Employee **⋈**(school_mark >=75 AND college_mark>=75)Dependent
>
> In Second, case I need a query:
>
> Find all employees whose some of dependents has distinction marks in college and school?
>
> ```python
> >>> Employee.objects.filter(
> ...             dependent__school_mark__gte=75
> ...                ).filter(
> ...             dependent__college_mark__gte=75)
>
> [<Employee: A>, <Employee: B>]
> ```
>
> This time 'B' also selected because 'B' has two children (more than one!), one has distinction mark in school 'b1' and other is has distinction mark in college 'b2'.
> Order of filter doesn't matter we can also write above query as:
>
> ```python
> >>> Employee.objects.filter(
> ...             dependent__college_mark__gte=75
> ...                ).filter(
> ...             dependent__school_mark__gte=75)
>
> [<Employee: A>, <Employee: B>]
> ```
>
> result is same! Relational algebra can be:
>
> > (Employee **⋈**(school_mark >=75)Dependent) **⋈**(college_mark>=75)Dependent
>
> Note following:
>
> ```python
> dq1 = Dependent.objects.filter(college_mark__gte=75, school_mark__gte=75)
> dq2 = Dependent.objects.filter(college_mark__gte=75).filter(school_mark__gte=75)
> ```
>
> Outputs same result: `[<Dependent: a1>]`
>
> I check target SQL query generated by Django using `print qd1.query` and `print qd2.query`both are same(Django 1.6).
>
> But semantically both are different to *me*. first looks like simple section σ[school_mark >= 75 AND college_mark >= 75](Dependent) and second like slow nested query: σ[school_mark >= 75](σ[college_mark >= 75](Dependent)).
>
> If one need [Code @codepad](http://codepad.org/c6VODLRf)
>
> btw, it is given in documentation @[Spanning multi-valued relationships](https://docs.djangoproject.com/en/dev/topics/db/queries/#spanning-multi-valued-relationships) I have just added an example, I think it will be helpful for someone new.

스택오버플로우에 올라온 답변을 요약해 보자면

```python
>>> Employee.objects.filter(dependent__school_mark__gte=75,
...                         dependent__college_mark__gte=75)

[<Employee: A>]
```

위와 같은 필터링은 dependent에서 두개의 조건을 동시에 만족하는 Employee를 찾는 것 이므로 결과는 Employee A가 된다.

반면에

```python
>>> Employee.objects.filter(
...             dependent__school_mark__gte=75
...                ).filter(
...             dependent__college_mark__gte=75)

[<Employee: A>, <Employee: B>]
```

 위의 경우 첫번째 필터에서 걸러지는 내용은 Employee A, B 가 된다.(왜냐하면 dependent 의 a1과 b1이 조건을 만족했기 때문이다. ) 두번째 필터가 포인트 인데 두번째 필터에서 college_mark가 75점 이상인 dependent를 가지고 있는 Employee를 다시 찾게되고 그 결과값도 Employee A, B가 된다. 왜냐하면 첫번째 필터에서 걸러진 후의 결과는 Employee A, B 이고 두번째 필터는 이 중에서 다시 college_mark가 75점 이상인 경우를 찾아내는 것인데  Employee B의 경우 college_mark가 75점 이상인 dependent b2 를 가지고 있기 때문이다.

 즉 Employee B는 college_mark와 school_mark가 동시에 75점 이상인 dependent를 가지고 있지 않지만, school_mark가 75점 이상인 dependent b1과 college_mark가 75점 이상인 dependent b2 이렇게 두개의 dependent를 가지고 있기 때문에 두 번째 필터에서도 해당 조건을 만족하는 것 이다.

 언뜻 생각하기에 혼동되는 부분이 두번째 필터를 생각할 때 dependent를 기준으로 생각하기 때문이다. 첫번째 필터에서 해당되는 dependent가 a1과 b1 이기 때문에 두번 째 필터에서 dependent a1, b1 이렇게 2개를 기준으로 필터링 해야할 것 같지만 Employee 기준으로 생각하면 첫번째 필터의 결과는 해당조건을 만족하는 dependent와 연결된 Employee가 되기 때문에  필터를 하기 전 과 후는 같은 상황이 된다.

이해하고 다시 적은 내용이지만 다시 생각해 봐도 글로 설명하는 것 보다 예시를 통해 이해하는 편이 훨씬 도움이 되는 듯 하다.

잘 이해가 안된다면 stackoverflow의 답변을 차근차근 읽어내려 간다면 충분히 이해할 수 있을 것 같다.
