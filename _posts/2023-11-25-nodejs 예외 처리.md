## 문제점
코드 리뷰 시간에 내가 작성한 Promise 코드의 잠재적인 문제점을 알게 되었다. 아래는 비슷하게 작성한 코드다
```js
async function badPromise() {
	try {
		const uncaughtException = thenPromise()
		.catch(() => {
			throw new Error('uncaught error')
		})
		console.log('not catch error')
	} catch (error) {
		console.error('console.error(error)')
	}
}

  

function thenPromise() {
	return new Promise((resolve, reject) => {
		reject(2)
	})
}

badPromise()

```

내가 작성한 코드와 별개로 호출하는 코드에 try catch가 있다면 모든 예외나 에러가 잡히는 줄 알았다. 그러나 위의 코드는 not catch error가 콘솔에 찍힌다

## 원인
위의 코드는 비동기 작업이어서 Promise가 끝날 때까지 코드가 멈추지 않는다. 에러가 throw되기 전에 console.log('not catch error')가 실행되므로 catch 문에 걸리기 전에 badPromise가 종료된다.

## 해결책
### await
비동기 동작이 원인이었기 때문에 promise에 await을 붙여준다
```js
async function badPromise() {
	try {
		const uncaughtException = await thenPromise()
		.catch(() => {
			throw new Error('uncaught error')
		})
		console.log('not catch error')
	} catch (error) {
		console.error('console.error(error)')
	}
}

  

function thenPromise() {
	return new Promise((resolve, reject) => {
		reject(2)
	})
}

badPromise()
```

thenPromise의 동작이 완료될 때까지 다음 코드가 지연되기 때문에 결과적으로 throw new Error를 잡을 수 있다.
### uncaughtException
nodejs 프로세스는 event loop에서 잡지 못한 예외를 eventemitter에 등록된 'uncaughtException'를 발생시켜서 처리한다. 일반적으로[운영체제에서 프로세스가 종료되면 0을 반환](https://ko.wikipedia.org/wiki/%EC%A2%85%EB%A3%8C_%EC%83%81%ED%83%9C)하는데 [nodejs는 1로 덮어 씌워서 출력한다](https://nodejs.org/api/process.html#event-uncaughtexception)

결과적으로 위의 이벤트를 등록해서 처리할 수 있다
```js
process.on('uncaughtException', (err) => {
	console.log('uncaughtException ', err)
})

async function badPromise() {
	try {
		const uncaughtException = thenPromise()
		.catch(() => {
			throw new Error('uncaught error')
		})
		console.log('not catch error')
	} catch (error) {
		console.error('console.error(error)')
	}
}

  

function thenPromise() {
	return new Promise((resolve, reject) => {
		reject(2)
	})
}

badPromise()

```

나는 언제든지 실수 할 수 있기 때문에  어플리케이션이 시작되는 entrypont에 위의 코드를 추가하는 것으로 좀 더 안심하고 코드를 작성할 수 있을 것 같다
