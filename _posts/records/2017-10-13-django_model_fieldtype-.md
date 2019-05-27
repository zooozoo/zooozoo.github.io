# Model field reference

## Field types

### AutoField(**options)

* 추가하는 인스턴스마다 자동으로 사용가능한 ID값을 할당하는 IntegerField. 기본적으로 primary key field가 자동으로 추가해 주기 때문에 필요한 경우에 사용하면 됩니다.

### BigAutoField(**options)

* 사용 할 수 있는 숫자가 1부터 9223372036854775807까지 보장되는 AutoField와 유사한 필드입니다.

### BigIntegerField(**options)

* 9223372036854775807까지의 숫자를 맞출 수 있다는 것을 제외하고는 IntegerField와 매우 흡사 한 64 비트 정수입니다.이 필드의 기본 양식 위젯은 TextInput입니다.

### BinaryField(**options)

* 이진 데이터를 저장하는 필드입니다. 바이트 할당 만 지원합니다. 이 입력란에는 기능이 제한되어 있습니다. 예를 들어, BinaryField 값에 대한 쿼리 집합을 필터링 할 수 없습니다. 또한 BinaryField를 ModelForm에 포함시키는 것도 불가능합니다.

### BooleanField(**options)

* Ture/False 값을 가지는 필드입니다. 
* 필드에 대한 기본 위젯은 CheckboxInput입니다.
* null값을 받아야만 한다면 NullBooleanField를 대신 사용하면 됩니다.
* default값이 정의되어 있지 않으면 BooleanField의 기본값은 None입니다. 

### Charfield(max_length=None, **options)

* string field로 길거나 짧은 문자열 보두 지원합니다.
* 대량의 텍스트를 사용해야 한다면 Textfield를 사용하면 됩니다.
* 기본 위젯은 TextInput입니다.
* CharField는 필수인자인 `max_length`를 반드시 입력해줘야 합니다. 

### DateField(auto_now=False, auto_now_add=False, **options)

* python에서 datetime.date 인스턴스로 표현되는 날짜입니다. 추가로 몇 가지 선택적 인수가 있습니다.
* DateField.auto_now : 데이터가 save될 때 마다 현재시각을 자동으로 부여합니다.
* DateField.auto_now_add : 데이터를 처음 만들 때 마다 자동으로 현재시각을 부여합니다.
* 기본적으로 현재시각을 부여하지만 기본값으로 `default=date.today`나 `default=timezone.now`와 같은 옵션을 설정할 수 있습니다. 
* 기본 위젯은 TextInput입니다. admin페이지에는 기본적으로 Today를 설정할 수 있는 단축키가 만들어 집니다.
* auto_now_add, auto_now 및 default 옵션은 상호 배타적입니다. 이러한 옵션을 함께 사용하면 오류가 발생합니다.
* auto_now 또는 auto_now_add를 True로 설정하면 해당 필드는 editable = False 및 blank = True로 설정됩니다.

### DateTimeField(auto_now=False, auto_now_add=False, **options)

* Python에서 datetime.datetime 인스턴스로 표현되는 날짜와 시간입니다. 
* DateField와 동일한 인수를 사용합니다.
* 기본 위젯으로 한개의 TextInput을 가지며, admin페이지 JavaScript 바로 가기가있는 두 개의 별도 TextInput 위젯을 사용합니다.

### DecimalField(max_digits=None, decimal_places=None, **options)

* 고정 소수점 이하의 십진수로 파이썬에서 Decimal 인스턴스로 나타냅니다. 필수 인자로 `max_digits(최대 자릿수)`와 `decimal_places(소숫점)`가 있습니다.

### Durationfield(**options)

* 특정 기간을 담을 수 있는 필드입니다. (python의 timedelta)

### EmailField(max_length=254, **options)

* email주소 유효성을 확인할 수 있는 CharField입니다. EmailCalidator를 이용하여 유효성을 검사합니다.

### FileField(upolad_to=None, max_length=100, **options)

* 해당 필드에 primary_key는 사용할 수 없습니다. 
* 이 속성은 업로드 디렉토리와 파일 이름을 설정하는 방법을 제공하며 두 가지 방법으로 설정할 수 있습니다.
* upload_to : 이 속성은 업로드 디렉토리와 파일 이름을 설정하는 방법을 제공하며 두 가지 방법으로 설정할 수 있습니다. 두 경우 모두 값은 Storage.save () 메서드에 전달됩니다. 

```
class MyModel(models.Model):
    # file will be uploaded to MEDIA_ROOT/uploads
    upload = models.FileField(upload_to='uploads/')
    # or...
    # file will be saved to MEDIA_ROOT/uploads/2015/01/30
    upload = models.FileField(upload_to='uploads/%Y/%m/%d/')
```

* 기본 FileSystemStorage를 사용하는 경우 문자열 값이 MEDIA_ROOT 경로에 추가되어 업로드 된 파일이 저장 될 로컬 파일 시스템의 위치가 형성됩니다.
* upload_to는 함수와 같이 호출 할 수 있습니다. 이것은 파일 이름을 포함하여 업로드 경로를 얻기 위해 호출됩니다. 호출 할 때에는 FileField가 선언된 model의 instance와 filename인자가 필요합니다.
* storage  : 파일의 저장 및 검색을 처리하는 저장 할 수 있는 속성입니다.  
* Filefield와 ImageField를 사용함에 있어서 몇가지 순서가 있습니다.

    1. 설정 파일에서 Django가 업로드 된 파일을 저장할 디렉토리의 전체 경로로 MEDIA_ROOT을 정의해야합니다. MEDIA_URL을 해당 디렉토리의 기본 공개 URL로 정의하십시오. 
    2. 모델에 FileField 또는 ImageField를 추가하고 upload_to 옵션을 정의하여 업로드 된 파일이 저장될 MEDIA_ROOT의 하위 디렉토리를 지정합니다.
    3. 데이터베이스에 저장되는 것은 파일 경로입니다. 장고가 제공하는 url속성도 사용할 수 있습니다. 예를 들어 mug_shot 이라는 ImageField를 만들었다고 가정하면 `{{ object.mug_shot.url }}`과 같은 형식으로 파일의 경로를 가져올 수 있습니다.

* 업로드된 파일의 URL을 url속성을 사용하여 사용할 수 있습니다.

    #### FieldFile

* 모델에서 FileField에 액세스하면 FieldFile 인스턴스가 기본 파일에 액세스하기위한 프록시로 제공됩니다.

```
* FieldFile.name - 연결된 FileField의 저장소 루트로부터의 상대 경로를 포함하는 파일의 이름입니다.
* FieldFile.size - 기본 Storage.size () 메서드의 결과입니다.
* FieldFile.url - 기본 Storage 클래스의 url () 메서드를 호출하여 파일의 상대 URL에 액세스하는 읽기 전용 속성입니다.
* FieldFile.open(mode='rb') - 지정된 모드에서이 인스턴스와 관련된 파일을 열거 나 다시 엽니 다. 표준 파이썬 open () 메소드와는 달리, 파일 디스크립터를 리턴하지 않습니다.
* FieldFile.close -  표준 파이썬 file.close () 메소드와 유사하게 동작하고이 인스턴스와 관련된 파일을 닫습니다.
* FieldFile.save(name, content, save=True) - 이 메서드는 파일 이름과 파일 내용을 가져 와서 필드의 저장소 클래스에 전달한 다음 저장된 파일을 모델 필드와 연결합니다. 모델의 FileField 인스턴스에 파일 데이터를 수동으로 연결하려면 save () 메서드를 사용하여 해당 파일 데이터를 유지합니다.
* FieldFile.delete(save=True) - 이 인스턴스와 관련된 파일을 삭제하고 필드의 모든 특성을 지 웁니다. 이 메소드는 delete ()가 호출 될 때 열려있을 경우 파일을 닫습니다.모델을 삭제하면 관련 파일이 삭제되지 않습니다. 고아 파일을 정리해야하는 경우 직접 처리해야합니다.
```

### FilePathField(path=None, match=None, recursive=False, max_length=100, **options)

* CharField는 파일 시스템의 특정 디렉토리에있는 제한된 파일 이름을 선택합니다. 3개의 인자를 받으며 그중 첫번째는 필수인자 입니다.

```
* FilePathField.path - 필수인자로 해당 필드가 선택해야하는 파일의 절대경로를 입력해줍니다.
* FilePathField.match - 정규표현식을 통해 해당 필드가 파일네임을 필터링 할 수 있게 합니다.
* FilePathField.recursive - path의 모든 하위 디렉토리가 포함되어야하는지 여부를 지정합니다.
* FilePathField.allow_files - 지정된 위치의 파일을 포함할지 여부를 지정합니다.
* FilePathField.allow_folders - 지정된 위치의 폴더를 포함할지 여부를 지정합니다.
```

### FloatField(**options)

* float 인스턴스로 파이썬에서 표현 된 부동 소수점 숫자입니다.

### ImageField(upload_to=None, height_field=None, width_field=None, max_length=100, **options)

* FileField의 모든 특성 및 메서드를 상속하지만 업로드 된 개체가 유효한 이미지인지 확인합니다.
* FileField에서 사용할 수있는 특수 특성 외에도 ImageField에는 height 및 width 특성이 있습니다. 이러한 속성에 대한 쿼리를 용이하게하기 위해 ImageField에는 두 개의 추가 선택적 인수가 있습니다.

```
* ImageField.height_field - 모델 인스턴스가 저장 될 때마다 자동으로 지정된 높이를 갖게하는모델 필드의 이름입니다.
* ImageField.width_field - 모델 인스턴스가 저장 될 때마다 자동으로 지정된 너비를 갖게하는모델 필드의 이름입니다.
```

* Imagefield를 사용하기 위해서는 Pillow 라이브러리를 설치해야만 한다.

### IntegerField(**options)

* 정수를 받을 수 있는 필드입니다. 정수. -2147483648에서 2147483647까지의 값은 장고가 지원하는 모든 데이터베이스에서 안전합니다. 이 필드의 기본 양식 위젯은 localize가 False 일 때 NumberInput이고 그렇지 않으면 TextInput입니다.

### GenericIPAddressField(protocol='both', unpack_ipv4=False, **options)

* 문자열 형식의 IPv4 또는 IPv6 주소를 받을 수 있는 필드입니다. 

### NullBooleanField(**options)

* BooleanField와 비슷하지만 NULL을 옵션 중 하나로 허용합니다. BooleanField 대신 null = True를 사용할 수도 있습니다. 

### PositiveIntegerField(**options)

* IntegerField와 같으나 양수 또는 0이어야합니다. 0에서 2147483647 사이의 값은 장고가 지원하는 모든 데이터베이스에서 안전합니다. 이전 버전과의 호환성을 위해 0 값이 허용됩니다.

### PositiveSmallIntegerField(**options)

* PositiveIntegerField와 같지만 특정 (데이터베이스에 따라 다릅니다.) 지점에서만 값을 허용합니다. 0에서 32767 사이의 값은 장고가 지원하는 모든 데이터베이스에서 안전합니다.

### SlugField(max_length=50, **options)

* Slug는 신문 용어입니다. Slug는 글자, 숫자, 밑줄 또는 하이픈 만 포함하는 짧은 레이블입니다. 일반적으로 URL에 사용됩니다.

### SmallIntegerField(**options)

* IntegerField와 같지만 특정 (데이터베이스에 따라) 지점에서만 값을 허용합니다. -32768에서 32767 사이의 값은 장고가 지원하는 모든 데이터베이스에서 안전합니다.

### TextField(**options)

* 큰 텍스트 필드. 이 필드의 기본 양식 위젯은 Textarea입니다. max_length 속성을 지정하면 자동 생성 양식 필드의 Textarea 위젯에 반영됩니다. 그러나 모델 또는 데이터베이스 수준에서는 적용되지 않습니다. CharField를 사용하십시오.

### TimeField(auto_now=False, auto_now_add=False, **options)

* Python에서 datetime.time 인스턴스로 표현되는 시간입니다. DateField와 동일한 자동 채우기 옵션을 적용합니다.

### URLField(max_length=200, **options)

* url을 위한 CharField입니다. 

### UUIDField(**options)

* 보편적으로 유일한 식별자를 저장하기위한 필드입니다. 파이썬의 UUID 클래스를 사용합니다. PostgreSQL에서 사용될 때 이것은 uuid 데이터 유형에 저장되고, 그렇지 않으면 char (32)에 저장됩니다.