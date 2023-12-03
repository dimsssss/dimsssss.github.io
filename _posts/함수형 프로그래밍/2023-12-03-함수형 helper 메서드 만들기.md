## helper 메서드를 만들게 된 이유
개발을 하다가 사용하고 있는 라이브러리에 없는 기능이 필요해서 직접 만들게 되었다. 팀에서는 [ramdajs](https://ramdajs.com/)를 사용하고 있는데  그 라이브러리에 내가 찾는 기능과 비슷하게 동작하는  [reject](https://ramdajs.com/docs/#reject) 메서드가 있다.

```js
const isOdd = (n) => n % 2 !== 0; 
// 1
R.reject(isOdd, [1, 2, 3, 4]); //=> [2, 4] 
// 2
R.reject(isOdd, {a: 1, b: 2, c: 3, d: 4}); //=> {b: 2, d: 4}
```

약간 아쉬웠던 것은 위의 두 번째 예제에서 객체를 대상으로 할 때 a, b 같은 key를 대상으로 할 수는 없었다. 즉 isOdd가 1, 2, 3, 4라는 value를 대상으로만 동작하였다. 나는 key와 value 전부를 대상으로 할 수 있는 유틸을 만들었다. 내가 만든 유틸은 아래와 같다
```js
import {compose, filter} from 'ramda'

const rejectForObject = compose(
	Object.fromEntries
	filter(fn),
	Object.entries,
)(fn, o)

```

## 아쉬운 점
내가 만들 메서드에도 아쉬운 점이 있었는데 list에는 동작하지 않는다는 점이다. array가 아닌 object만 대상으로 동작을 해서 유틸이라고 부르긴 애매하고 helper 정도로 사용을 해야겠다. 
