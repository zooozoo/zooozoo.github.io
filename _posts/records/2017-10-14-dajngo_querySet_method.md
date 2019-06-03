---
category: record
---
# QuerySet API

## Methods that return new QuerySets

### filter()

지정된 검색 매개 변수와 일치하는 객체가 포함 된 새 QuerySet을 반환합니다.

복잡한 쿼리문을 사용해야 할 경우 Q objects를 사용할 수 있습니다.

### exclude()

지정된 검색 매개변수와 일치하는 객체가 포함되지 않은 새 QuerySet을 반환합니다.

### annotate()

제공된 쿼리 식 목록으로 QuerySet의 각 개체에 주석을 첨부합니다.제공된 쿼리 식 목록으로 QuerySet의 각 개체에 주석을 첨부합니다. 표현식은 단순 값, 모델의 필드에 대한 참조 (또는 모든 관련 모델), 또는 개체의 개체와 관련하여 계산 된 집계 식 (평균, 합계 등)이 될 수 있습니다.

annotate ()에 대한 각 인수는 반환되는 QuerySet의 각 객체에 추가되는 주석입니다.

```
>>> from django.db.models import Count
>>> q = Blog.objects.annotate(Count('entry'))
# The name of the first blog
>>> q[0].name
'Blogasaurus'
# The number of entries on the first blog
>>> q[0].entry__count
42
```

위의 예시에서 블로그 모델은 entry__count 속성을 단독으로 정의하지 않지만 키워드 인수를 사용하여 집계 함수를 지정하면 주석의 이름을 제어 할 수 있습니다.

### order_by()

기본적으로 QuerySet에 의해 반환 된 결과는 Model's Meta의 정렬 옵션에 의해 주어진 순서 튜플에 의해 정렬됩니다. order_by 메소드를 사용하여 QuerySet 단위로이 값을 겹쳐 쓸 수 있습니다.

ex)

```
Entry.objects.filter(pub_date__year=2005).order_by('-pub_date', 'headline')
```

위 결과는 pub_date가 내림차순으로 정렬 된 다음 headline이 오름차순으로 정렬됩니다. "-pub_date"앞에있는 `-` 기호는 내림차순을 나타냅니다. 기본적으로 오름차순으로 정렬됩니다.

렌덤으로 정렬하려면 `?`를 사용합니다. (아래 예시)

```
Entry.objects.order_by('?')
```

다른 모델의 필드로 정렬하려면 모델 관계를 쿼리 할 때와 같은 구문을 사용하십시오. 즉, 필드 이름과 이중 밑줄 (__), 새 모델의 필드 이름 등이 포함되며, 원하는만큼의 모델을 추가 할 수 있습니다.

ex)

```
Entry.objects.order_by('blog__name', 'headline')
```

다른 모델과 관계가있는 필드로 정렬하려고하면 Django는 관련 모델의 기본 순서를 사용하거나 Meta.ordering이 지정되지 않은 경우 관련 모델의 기본 키순으로 정렬합니다. 

예를 들어 블로그 모델에는 기본 주문이 지정되어 있지 않으므로

```
Entry.objects.order_by('blog')
```

위의 예시는 아래의 예시와 같습니다.

```
Entry.objects.order_by('blog__id')
```

만약 Blog 테이블에 `ordering = ['name']` 이 지정되어 있다면 위의 첫번째 쿼리는 아래와 같습니다.

```
Entry.objects.order_by('blog__name')
```

### reverse()

reverse() 메서드를 사용하여 쿼리셋 결과물을 반대방향으로 정렬할 수 있습니다. reverse()를 한번더 호출하면 기존의 순서대로 다시 정렬할 수 있습니다.

다음 쿼리셋으로 끝에서 5번째까지의 항목을 불러올 수 있습니다.

```
my_queryset.reverse()[:5]
```

Django에서는 파이썬에서 사용할 수 잇는 [-1:] 과 같은 역 슬라이스를 사용하지 않습니다.   SQL에서 효율적으로 수행 할 수 없기 때문입니다. 

또한 reverse ()는 일반적으로 정의 된 순서가있는 QuerySet에서만 호출되어야합니다. 주어진 QuerySet에 대해 그러한 정렬이 정의되어 있지 않으면 reverse ()를 호출하면 아무런 효과가 없습니다.

### distinct()

SQL 쿼리에서 SELECT DISTINCT를 사용하는 새 QuerySet을 반환합니다. 이렇게하면 조회 결과에서 중복 행이 제거됩니다.

기본적으로 QuerySet은 중복 행을 제거하지 않습니다. 실제로 Blog.objects.all ()과 같은 간단한 쿼리는 결과 행이 중복 될 가능성이 있기 때문에 거의 문제가되지 않습니다. 그러나 쿼리가 여러 테이블에 걸쳐있는 경우 QuerySet을 평가할 때 중복 결과를 얻을 수 있습니다. 이러한 상황에서 distinct ()를 사용하여 중복된 항목을 제거할 수 있습니다.

### values()

iterable한 모델객체가 아닌 dictionary로 반환하는 쿼리셋입니다.

반환된 dictionary는 모델 오브젝트의 속성이름을 키로, 가지고 데이터는 value로 가집니다. 

다음의 예시는 일반적인 쿼리셋과 values()쿼리셋의 차이를 보여줍니다.

```
# This list contains a Blog object.
>>> Blog.objects.filter(name__startswith='Beatles')
<QuerySet [<Blog: Beatles Blog>]>

# This list contains a dictionary.
>>> Blog.objects.filter(name__startswith='Beatles').values()
<QuerySet [{'id': 1, 'name': 'Beatles Blog', 'tagline': 'All the latest Beatles news.'}]>
```

values()메소드는 모델이 가지고 있는 필드명을 인자로 가질 수 있습니다. 인자를 지정하지 않으면 모든 필드가 반환됩니다. (아래예시)

```
>>> Blog.objects.values()
<QuerySet [{'id': 1, 'name': 'Beatles Blog', 'tagline': 'All the latest Beatles news.'}]>
>>> Blog.objects.values('id', 'name')
<QuerySet [{'id': 1, 'name': 'Beatles Blog'}]>
```

values()메서드는 키워드인자로 annotate()를 사용할 수 있습니다.

ex)

```
>>> from django.db.models.functions import Lower
>>> Blog.objects.values(lower_name=Lower('name'))
<QuerySet [{'lower_name': 'beatles blog'}]>
```

### values_list(*fields, flat=False)

values_list()는 dictionary를 반환하는 대신 튜플로 만들어진 리스트를 반환한다는 점을 제외하면 values ()와 유사합니다. 각 튜플에는 해당 필드의 값이나 values_list () 호출로 전달 된 표현식이 들어 있으므로 첫 번째 항목이 첫 번째 입력란이됩니다. 

ex)

```
>>> Entry.objects.values_list('id', 'headline')
<QuerySet [(1, 'First entry'), ...]>
>>> from django.db.models.functions import Lower
>>> Entry.objects.values_list('id', Lower('headline'))
<QuerySet [(1, 'first entry'), ...]>
```

만약 매게변수로 단일 필드 만 전달하면 flat 매개 변수를 전달할 수도 있습니다. flat=True이면 단일 튜플 아닌 단일 값으로 이루어진 리스트 반환합니다. 

ex)

```
>>> Entry.objects.values_list('id').order_by('id')
<QuerySet[(1,), (2,), (3,), ...]>

>>> Entry.objects.values_list('id', flat=True).order_by('id')
<QuerySet [1, 2, 3, ...]>
```

하나 이상의 필드인자를 할당했는데 flat매게변수를 사용한다면 에러가 납니다.

values_list ()에 값을 전달하지 않으면 모델의 모든 필드가 선언 된 순서대로 반환됩니다.

values()와 values_list() 메서드는 모델 인스턴스를 생성하지 않고도 필드의 집합을 산출해 내는데에 최적화 되어 설계되었습니다. 관계가 형성되어있는 모델을 다룰 때에는 해당되지 않습니다. 한행에 하나의 객체가 할당되는 가정이 유지되지 않기 때문입니다. 

### dates(field, kind, order='ASC')

QuerySet의 내용 내에서 특정 종류의 모든 사용 가능한 날짜를 나타내는 datetime.date 객체의 목록으로 평가되는 QuerySet을 반환합니다.

선택한 필드는 DateField 여야 하며 종류는 "year", "month" or "day"지정되어 있어야 합니다. 산출된 결과는 해당 타입에 따라서 나눠집니다. 

```
>>> Entry.objects.dates('pub_date', 'year')
[datetime.date(2005, 1, 1)]
>>> Entry.objects.dates('pub_date', 'month')
[datetime.date(2005, 2, 1), datetime.date(2005, 3, 1)]
>>> Entry.objects.dates('pub_date', 'day')
[datetime.date(2005, 2, 20), datetime.date(2005, 3, 20)]
>>> Entry.objects.dates('pub_date', 'day', order='DESC')
[datetime.date(2005, 3, 20), datetime.date(2005, 2, 20)]
>>> Entry.objects.filter(headline__contains='Lennon').dates('pub_date', 'day')
[datetime.date(2005, 3, 20)]
```

### datetimes(field_name,  kind, order='ASC', tzinfo=None)

QuerySet의 내용 내에서 특정 종류의 사용 가능한 모든 날짜를 나타내는 datetime.datetime 객체의 목록으로 평가되는 QuerySet을 반환합니다.

선택한 필드는 DateTimeField 여야 하며 종류는 "year", "month",  "day", "hour:, "minute", "second"로지정되어 있어야 합니다. 산출된 결과는 해당 타입에 따라서 나눠집니다. 

### none()

* none ()을 호출하면 객체를 반환하지 않는 쿼리 세트가 만들어지며 결과에 액세스 할 때 쿼리가 실행되지 않습니다. qs.none () 쿼리 세트는 EmptyQuerySet의 인스턴스입니다.

### all()

* 현재의 QuerySet (또는 QuerySet 서브 클래스)의 복사본을 반환합니다. 이는 모델 매니저 또는 QuerySet을 전달하고 결과에 대해 추가 필터링을 수행하려는 상황에서 유용 할 수 있습니다. 

### union(*other_qs, all=False)

* SQL의 UNION 연산자를 사용하여 두 개 이상의 QuerySet 결과를 결합합니다.

```
>>> qs1.union(qs2, qs3)
```

* UNION 연산자는 기본적으로 고유 값만 선택합니다. 중복 값을 허용하려면 all = True 인수를 사용하십시오.

### intersection(*other_qs)

* SQL의 INTERSECT 연산자를 사용하여 두 개 이상의 QuerySets의 공통 요소를 반환합니다.

```
>>> qs1.intersection(qs2, qs3)
```

### select_related(*fields)

* 외래 키 관계가 형성되어 있는 QuerySet를 리턴 해, 조회를 실행할 때에 추가의 관련 오브젝트 데이터를 선택합니다. 이것은 하나의 복잡한 쿼리로 이어지는 성능 향상이지만 나중에 외래 키 관계를 사용하면 데이터베이스 쿼리가 필요하지 않음을 의미합니다. 다음 예제는 일반 조회와 select_related () 조회 간의 차이점을 보여줍니다. 

ex) 표준 예제

```
# Hits the database.
e = Entry.objects.get(id=5)

# Hits the database again to get the related Blog object.
b = e.blog
```

ex) select_related

```
# Hits the database.
e = Entry.objects.select_related('blog').get(id=5)

# Doesn't hit the database, because e.blog has been prepopulated
# in the previous query.
b = e.blog
```

* 한번에 관계가 이루어진 데이터베이스까지 조회해 와서 다음 쿼리시에 데이터베이스를 거치지 않고서도 원하는 정보를 산출해 낼 수 있다는 것

select_related()쿼리는 다른 쿼리 객체와도 함께 사용할 수 있습니다.

```
from django.utils import timezone

# Find all the blogs with entries scheduled to be published in the future.
blogs = set()

for e in Entry.objects.filter(pub_date__gt=timezone.now()).select_related('blog'):
    # Without select_related(), this would make a database query for each
    # loop iteration in order to fetch the related blog for each entry.
    blogs.add(e.blog)
```

filter()와 select_related()의 체이닝 순서는 중요하지 않습니다. 어떤 것이 앞에 있더라도 결과는 동일합니다.

```
Entry.objects.filter(pub_date__gt=timezone.now()).select_related('blog')
Entry.objects.select_related('blog').filter(pub_date__gt=timezone.now())
```

select_related ()를 많은 관련 객체로 호출하거나 모든 관계를 모르는 경우가 있습니다. 이러한 경우에는 인수없이 select_related ()를 호출 할 수 있습니다. 이것은 발견 할 수있는 null이 아닌 모든 외래 키를 반환합니다. null 입력 가능 외래 키가 지정되어야합니다. 대부분의 경우에는 기본 쿼리를보다 복잡하게 만들고 실제로 필요한 것보다 많은 데이터를 반환하기 때문에 권장되지 않습니다.

QuerySet에서 select_related의 이전 호출에 의해 추가 된 관련 필드 목록을 지우려면 매개 변수로 None을 전달할 수 있습니다.