nodejs fork에 대한 예제에서 나온 코드. 검색해보니 웹 요청을 취소할 때 사용한다고 나왔는데 fork 예제에 왜 나왔는지 모르겠다
```js
if (process.argv[2] === 'child') {  
	setTimeout(() => {  
		console.log(`Hello from ${process.argv[2]}!`);  
	}, 1_000);  
} else {  
	const { fork } = require('node:child_process');  
	const controller = new AbortController();  
	const { signal } = controller;  
	const child = fork(__filename, ['child'], { signal });  
	child.on('error', (err) => {  
	// This will be called with err being an AbortError if the controller aborts 
	});  
	controller.abort(); // Stops the child process  
}
```
