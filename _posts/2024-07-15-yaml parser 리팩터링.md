---
title:  "yaml parser 리팩터링"
date: 2024-07-15T22:17:48+09:00
categories: 
    - 소프트웨어 설계
    - 리팩터링
---

yaml parser는 간단하게 .yaml 파일을 nodejs 객체로 변환해주는 프로젝트이다. 코드는 아래와 같은 구조로 되어 있다.(화살표 아래 방향의 코드를 사용하고 있다는 의미다.)

![](https://i.imgur.com/WltxzwN.png)

`Parser`에서 Token 들을 nodejs에서 사용하는 타입으로 변환해주는데  yaml에서 사용하는 `array`, `|` 등등 변환하는 코드를 추가해야 하는데 그 이후에 코드가 복잡해지고 읽기 어려워져서 분리하기로 결정했다

예를 들어 `array`와 `|` 타입 모두 value들을 하나로 묶어야 한다. yaml 파일이 아래와 같이 작성되어 있다면
```yaml
array:
  - 1
  - 2
  - 3
message: |
  this 
  is
  message
```

parse 후의 결과는 아래와 같다.
```json
{
	"array": [1, 2, 3],
	"message": "this is message\n"
}
```

두 타입 모두 반복해서 요소들을 처리해야 하지만 `array`는 배열에 담아야 하고 `|`는 하나의 문자열로 합쳐야 한다. 만약 또 다른 타입이 추가된다면 더 복잡해지기 때문에 각각의 처리하는 코드를 타입으로 분리하려고 한다. 

그래서 변경된 구조는 다음과 같다
![](https://i.imgur.com/vDCT0NB.png)

코드를 살펴보는데 놓쳤던 부분을 발견하였다.
![](https://i.imgur.com/I7H02I8.png)

최종적으로 다음과 같은 구조로 변경되었다

![](https://i.imgur.com/32H9w5Y.png)

https://github.com/dimsssss/yaml-parser
