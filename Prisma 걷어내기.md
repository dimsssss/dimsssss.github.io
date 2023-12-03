## 이 일을 하게 된 이유
팀은 github actions를 이용해서 스테이징 서버에 배포를 진행한다. 예전에 prisma에 반복적으로 하는 작업을 자동화 스크립트로 작성했는데 github action에서 배포 스크립트를 실행할 때 prisma 자동화 부분이 문제를 일으켜서 배포를 할 수 없는 상황이 되었다 그래서 prisma를 전부 걷어내고 raw Query로 바꾸는 작업을 하고 있다.
## 리팩터링
ORM을 걷어낸 자리에  sql 쿼리가 가득 차니 코드를 읽기가 힘들었다. 현재 코드는 router 영역, 자바 스프링에서는 presentation, controller에 모든 코드가 다 들어가 있다.
이렇게 된 이유는 비지니스 로직이라고 할만한 코드가 별로 없었기 때문이다. 데이터베이스 접근 코드는 따로 분리를 했지만 시니어의 권유로 모든 코드를 controller에 집중했다

기존 코드 + 늘어난 sql 쿼리 때문에 코드를 보는 것이 어지러웠다. 그래서 분리를 하기로 했다. 어떻게 분리 해야 할까? 먼저 기존 코드는 아래와 같고 가독성이 떨어지는 것과 실질적인 문제가 있다.

```js
const body = {
	name: 'banana',
	price: 500
}
await connectionPool.transaction().query(`
	INSERT INTO products(name, price) VALUES ('${body.name}', '${body.price}')
`)
// ... 생략

```

name과 price가 순서가 바뀌면 아래 코드는 실패한다. 순서에 상관 없이 컬럼에 매치되는 값이 있다면 쿼리는 성공을 해야한다
