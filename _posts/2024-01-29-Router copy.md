---
title:  "참조 투명성"
date: 2024-01-29T01:20:46+09:00
categories: 
    - 함수형 프로그래밍
tags:
    - 함수형
---

인증 프로세스를 수정하면서 여러 가지 생각들이 동시에 났다. 최근에 [함수형 프로그래밍에서 참조 투명성](https://github.com/MostlyAdequate/mostly-adequate-guide-kr/blob/master/ch03-kr.md#%ED%95%A9%EB%A6%AC%EC%84%B1) 이란 내용을 읽으면서 예전에 봤었던 객체지향과 함수형의 차이에 대한 [문장](https://twitter.com/mfeathers/status/29581296216)이 생각났다(이거 찾는데 오래 걸렸다) 요약하면 아래와 같다

>객체지향은 동작을 캡슐화하고 함수형은 동작을 최소화한다

처음 봤을 때는 이해를 못했고 위의 참조 투명성을 읽고 나서야 왜 그런 말이 나왔는지 알았다. 

또 하나의 기억은 예전에 시니어 개발자(타입스크립트 혐오자, 함수형 프로그래밍 선호)가 내 코드를 리뷰 할 때 함수를 감싸는 것을 지양하라고 했었는데 어떤 맥락에서 그랬는지 지금 이해한다.

인증 프로세스를 수정하면서 아래 코드를 '리팩터링 해야할까? 둬야할까?' 라는 생각이 들었다 .
```ts
async signIn(email, pass) : Promise<Auth> {
  try {
    // ... 중략
    const [accessToken, refreshToken] = await Promise.all([
      this.jwtService.signAsync({ userId: user.id, username: user.firstname}, { expiresIn : '1h', secret: this.configService.get('ACCESS_SECRET')}),
      this.jwtService.signAsync({ userId: user.id, username: user.firstname}, { expiresIn : '30d', secret: this.configService.get('REFRESH_SECRET')})
    ])
    return new Auth(accessToken, refreshToken, user.id, user.lastname)
} catch(err) {
  throw err
  }
} 

async refreshToken(prevRefreshToken) : Promise<Auth> {
  try {
    const payload = await this.jwtService.verifyAsync(prevRefreshToken,{secret: this.configService.get('REFRESH_SECRET')});

    const [newAccessToken, newRefreshToken] = await Promise.all([
      this.jwtService.signAsync({ userId: payload.userId, username: payload.username}, { expiresIn : '1h', secret: this.configService.get('ACCESS_SECRET')}),
      this.jwtService.signAsync({ userId: payload.userId, username: payload.username}, { expiresIn : '30d', secret: this.configService.get('REFRESH_SECRET')})
    ])

    return new Auth(newAccessToken, newRefreshToken, payload.userId, payload.username)

} catch(error) {
  console.error(error)
  throw error
  }
}
```

예전 같으면 토큰을 쌍으로 발급하는 코드가 중복이라서 바로 메서드로 분리했을 것이다. 그러나 지금은 그대로 두는 게 더 낫다는 생각을 하였다. 왜냐하면 참조 투명성과 별개로 분리하는 순간 의존성이 생긴다. 타입스크립트라서 private로 내부에 분리하면 별 문제가 되지 않을까라는 생각도 잠시 들었지만 예전에 중복된 코드를 분리해서 두 곳에서 참조를 하게 되었는데 수정하다가 귀찮았던 기억이 생각났기 때문이다. 새로운 기능을 추가할 때 분리한 메서드를 사용할 수 있다

더 정확히 말하면 '애초에 문제의 소지를 만들지 말자'가 적절한 것 같다. 클래스 내부라도 무작정 분리하지 말자.