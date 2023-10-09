## 목적
함수형 프로그래밍을 학습하고 체화를 하기 위해서 간단한 토이프로젝트에 함수형 프로그래밍 기법을 적용하려고 시작하였다. 아무것도 없이 시작하면 시간이 너무 오래 걸리고 늘어질 것 같아서 [knex.js](https://knexjs.org/)라는 오픈소스를 참고하였다

## 이슈
### 쿼리의 상태
```js
knex.select('title', 'author', 'year')
  .from('books')
```

knex의 select는 메서드 체이닝을 통해서 동작을 하는데 select가 실행되고 기존에 있던 쿼리 상태를 가지고 있어야 한다. 현재 구현된 상황은 아래와 같다

```js
QB.prototype.select = function(...columns) {

return compose(	
	(query) => `${query} ${join(',', columns)}`
	)('SELECT')
}
```

함수형 프로그래밍에서 중요한 요소는 불변이다.  그에 맞게 쿼리가 변경될 때마다 객체를 새로 생성해야 한다. 쿼리의 요소가 추가될 때마다 전체 객체를 새로 생성하는 것은 비효율적이라는 생각이 들었다. 쿼리만 담고 있는 객체를 만들어서 변경될 때마다 전체가 아닌 쿼리 객체만 새로 만드는 게 더 좋을 것 같다

```js
const Query = function(query) {
	this.query = query
}

Query.of = function(query) {
	return new Query(query)
}

export { Query }
```

위와 같이 wrapper를 하나 만들어서 실행을 해보았다

![](https://i.imgur.com/6Pw7fx7.png)

wrapper로 만들다 보니 값이 중첩되어서  들어가는 상황이다. 중첩된 값을 하나로 평평하게 바꿔야 한다. 여기서 [[모나드(Monad)]]를 한번 적용해본다
