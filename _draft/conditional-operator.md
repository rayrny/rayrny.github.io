두리뭉실하게 알고는 있었던 조건 연산자의 동작 방식에 대해서 자세히 알아보자.

예제 1)

```jsx
var promise1 = () => new Promise(resolve => {
    console.log('promise1');
    resolve(true);
});

var promise2 = () => new Promise(resolve => {
    console.log('promise2');
    resolve(false);
});

var results = false || await promise1() || await promise2();
```

결과는? 처음에는 promise1과 promise2가 모두 찍히고, 결과는 true가 될 것이라고 가볍게 예상했었는데 결과가 그렇지 않아서 당황했다. 이후에는 비동기와 관련된건가? 싶어서 열심히 비동기쪽을 찾아보았지만 이유는 따로 있었다.

![스크린샷 2022-02-21 오후 2 01 47](https://user-images.githubusercontent.com/48341341/155140316-97f910ff-3afb-4972-91a5-af6a18abd794.png)

![스크린샷 2022-02-21 오후 2 02 17](https://user-images.githubusercontent.com/48341341/155140327-f3cd4605-f2b8-45af-ba27-d191ad18c7b9.png)

||, && 연산자와 관련한 문제였고, `||는 true`, `&&는 false` 즉, 판단할 수 있는 명확한 근거가 발생한 순간 뒤의 연산을 하지 않는다고 한다.

그런데, 여기서 한 가지 더 살펴볼 수 있는 것은, 바로 비동기 함수의 실행 여부이다.

예제 2)

```jsx
var promise1 = () => new Promise(resolve => {
    console.log('promise1');
    resolve(true);
});

var promise2 = () => new Promise(resolve => {
    console.log('promise2');
    resolve(false);
});

var promise1Results = await promise1();
var promise2Results = await promise2();

var results = false || promise1Results || promise2Results;
```

위의 코드는 실행 결과가 어떻게 될까? 차이점은 `await promise1()` 를 조건 연산자에서 바로 사용하는지 아니면 변수로 그 결과를 담아서 사용하는지이다. 

결과는 콘솔에 `promise1, promise2`가 모두 찍히게 된다. 아까와 매우 유사한 코드이나 실행 결과가 다르다. 

논리적으로 찬찬히 생각해보면 맞는 결과이다. 

예제 1의 경우 조건을 판단하는 당시에 promise1()이 실행되며 promise1이 콘솔에 찍히고, 곧바로 이어서 true를 반환한다. results는 이 true를 만남과 동시에 뒤에 있는 promise2()를 실행하지 않고 끝나게 된다.

예제 2는 변수에 각각 함수의 실행 결과를 담기 때문에 비동기 함수 두개 모두가 일단 실행된다. 

코드를 짤 때 생각해 볼 수 있는 문제로, 만약 비동기 함수에서 엄청나게 많은 연산을 수행하게 될 경우에는 예제 1의 경우를 참고할 수 있을 것 같다. 그러나 그 외의 경우에는 읽기 쉬운 코드를 위해 연산을 수행하더라도 코드를 읽는 사람으로 하여금 더 이해가 잘 되도록 짜는 것도 나쁘지 않을 것 같다.