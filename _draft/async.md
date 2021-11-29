feat. [캡틴판교](https://joshua1988.github.io/web-development/javascript/promise-for-beginners/)

자바스크립트의 비동기 처리란, '특정 코드 실행이 완료될 때 까지 기다리지 않고 다음 코드를 먼저 수행하는 것'을 말한다. 한마디로 데이터를 가져오는 등 시간이 걸리는 일을 수행할 때, 이 작업이 완료되길 기다리지 않고 다음 코드를 마저 수행 후 데이터를 가져왔을 때 해당 작업을 마무리하는 것이다. 

비동기 처리 로직을 위해 callback 함수를 사용하기도 하지만, 콜백 지옥을 야기한다. 때문에 등장한 방법이 바로 `Promise` 와 `Async await`이다. 

- **Promise**
    - Pending(대기) : 비동기 처리 로직이 아직 완료되지 않은 상태
    - Fulfilled(이행) : 비동기 처리가 완료되어 프로미스가 결과 값을 반환해준 상태
    - Rejected(실패) : 비동기 처리가 실패하거나 오류가 발생한 상태
    
- **Async await**
    
    async 함수는 promise를 리턴하고, 모든 await 함수는 일반적으로 promise가 된다.
    
    따라서 async 함수의 리턴 값은 `get()` 을 사용할 수 있다.(Promise의 문법)
    
<hr />

위의 설명을 통해 비동기 처리의 개념에 대해 알아보았다. 그렇다면 예제 코드를 통해 이해해보자.

```jsx
    async function resultFunction() {
    return 5;
    }
    const value = await resultFunction();
    
    // 아래 두 블록은 같은 역할을 한다.
    axios.get().then(res => {
    console.log(res);
    })
    
    const res = axios.get();
    console.log(res);
```
    

```jsx
// promise
const intervalFunction = () => {
	console.log('start');
	return new Promise((*resolve*, *reject*) => {
		setTimeout(() => {
			resolve(5);
		}, 3000);
	}).then((data) => {
	console.log(data);
	return data;
	});
}

// async await
const intervalFunction = () => {
	return new Promise((*resolve*, *reject*) => {
		setTimeout(() => {
			resolve(5);
		}, 3000);
	});
}

async function resultFunction() {
	const data = await intervalFunction();
	return data;
}
```

일반적으로 A 함수가 다 실행 된 뒤에 B 함수를 실행하고 싶다면

```jsx
await A(); // 여기서 A, B 함수는 모두 async 함수이다.
B(); // B 함수에 await를 붙이지 않는 이유는? 
// -> B 함수를 실행하고 뒤에 처리할 로직이 더 없기 때문에 굳이 붙이지 않는다.
// await을 붙인다고 다 좋은 것이 아니다!
```

처음에는 callback으로 실행하도록 생각했는데 굳이 그럴 필요는 없을 것 같다.

왜냐하면! callback 피하려고 나온 게 async await 니까!

```jsx
A().then(
	B();
)
```

![example-code](https://user-images.githubusercontent.com/48341341/143876055-d0bf901f-e129-4502-bdd0-8b23f03fd5f8.png)