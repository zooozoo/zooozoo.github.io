---
title: celery와 aws sqs사용할 때 pycurl설치문제
tag: [sqs, celery]
excerpt: >-
 celery와 sqs를 연결하여 사용할 때 pycurl을 설치할 때 발생한 문제와 해결책에 대한 내용입니다.
category: post
---

celery 를 sqs와 함께 사용하기 위한 전반적인 내용은 아래 url에 내용이 설명되어 있다.
[celery 공식문서](http://docs.celeryproject.org/en/latest/getting-started/brokers/sqs.html)

celery와 sqs를 연결하면서 해결했던 몇가지 문제들을 기록해 둔다.(비록 간단한 문제들이지만)

1. broker url 만들기
  처음 broker url을 만들 때, json파일로 만들어져 있는 `access id`와 `secret key`를
  아래와 같이 가져와 broker url을 만들고 celery를 실행했는데 `ValueError: invalid literal for int() with base 10: '2kDlXtkl9pGy2vxgJOcrLhsGMU4dKzpAzw'`라는 에러가 났다.
  ```
  AWS_ACCESS_KEY_ID = config_secret_common['aws']['AWS_ACCESS_KEY_ID']
  AWS_SECRET_ACCESS_KEY = config_secret_common['aws']['AWS_SECRET_ACCESS_KEY']

  CELERY_BROKER_URL = f"sqs://{AWS_ACCESS_KEY_ID}:{AWS_SECRET_ACCESS_KEY}@"
  ```
  에러내용을 바탕으로 해결책을 찾았는데 다해결하고 난 다음에야 들어온 문서내용

  > If you specify AWS credentials in the broker URL, then please keep in mind that
  > the secret access key may contain unsafe characters that need to be URL encoded.

  사실 먼저 읽었던 내용이었는데 그때는 이게 무슨말인지 잘 몰랐었다. 어쨌건 검색을 통해 아래와 같이
  broker url을 구성해 문제를 해결 할 수 있었다.
  (stackoverflow에서 찾았는데 url을 잃어버렸다... 에러내용을 그대로 복붙해서 찾아냈다.)
  ```python
  AWS_ACCESS_KEY_ID = config_secret_common['aws']['AWS_ACCESS_KEY_ID']
  AWS_SECRET_ACCESS_KEY = config_secret_common['aws']['AWS_SECRET_ACCESS_KEY']
  aws_access_key_id = urllib.parse.quote(f'{AWS_ACCESS_KEY_ID}', safe='')
  aws_secret_access_key = urllib.parse.quote(f'{AWS_SECRET_ACCESS_KEY}', safe='')

  CELERY_BROKER_URL = f"sqs://{aws_access_key_id}:{aws_secret_access_key}@"
  ```
  url용 문자로 인코딩 하기 위하여 모듈을 활용해 인코딩을 새로 해주는 것으로 이해
  [URL Quoting, python 문서](https://docs.python.org/3/library/urllib.parse.html#url-quoting)

2. `pycurl`설치하기
  1번을 마친 후 다시 실행하면 `Unrecoverable error: ImportError('The curl client requires the pycurl library.',)`이런 에러가 나온다.
  그래서 `pycurl`을 `pip install pycurl`명령어를 실행하면 `__main__.ConfigurationError: Could not run curl-config: [Errno 2] No such file or directory`와 같은 에러가 난다.

  google신 검색결과 [pycurl 설치관련 stackoverflow 답변](https://stackoverflow.com/questions/23937933/could-not-run-curl-config-errno-2-no-such-file-or-directory-when-installing)을 얻었다.

  결론은 `sudo apt-get install libcurl4-openssl-dev` 명령어로 `libcurl4-openssl-dev`를 설치해 주고 `pip install pycurl`로 `pycurl`을 설치하면 된다.
