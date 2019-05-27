#### Field option 중 choices

* choices 에서 앞쪽이 데이터베이스 에서 인식하는 벨류값, 뒤쪽이 클라이언트 화면에서 보이는 값

example)
```
from django.db import models

class Person(models.Model):
    SHIRT_SIZES = (
        ('S', 'Small'),
        ('M', 'Medium'),
        ('L', 'Large'),
    )
    name = models.CharField(max_length=60)
    shirt_size = models.CharField(max_length=1, choices=SHIRT_SIZES)
```


#### null값과 blank

`null`값은 데이터베이스에 어떠한 데이터도 갖고 있지 않다는 것을 말한다. (datetime 이나 integerfield에는 비어있는 시간이나 숫자라는 개념이 없기 때문에 빈값을 적용시키기 위헤서는 null=True옵션을 줘야한다.)

`blnak`값은 '비어있는 값'을 말하며 blank=True를 적용 했을 때에 데이터베이스에 '비어있는 값'을 저장하게 된다.(charfield, textfield에는 비어있는 값을 인식시키기 위헤서 blnak=True 옵션을 준다.) blank의 또다른 기능 중 하나는 admin에서 필수 입력요소 인지를 판단할 수 있게 해준다.
