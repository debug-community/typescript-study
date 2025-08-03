# Item 39

## item.39 any를 구체적으로 변형해서 사용하기

요약

- `any` 를 사용할 때는 정말로 모든 값이 허용되어야만 하는지 면밀히 검토해야 한다.
- `any` 보다 더 정확하게 모델링할 수 있도록 `any[]` 또는 `{[id: string]: any}` 또는 `() => any` 처럼 구체적인 형태를 사용해야 한다.

---

`any` 는 자바스크립트에서 표현할 수 있는 모든 값을 아우르는 매우 큰 범위의 타입이다. `any` 타입에는 모든 숫자, 문자열, 배열, 객체, 정규식, 함수, 클래스, DOM 엘리멘트는 물론 `null` 과 `undefined` 까지도 포함된다. 반대로 말하면, 일반적인 상황에서는 `any` 보다 더 구체적으로 표현할 수 있는 타입이 존재할 가능성이 높기 때문에 더 구체적인 타입을 찾아 타입 안정성을 높이도록 해야 한다.

예를 들어, `any` 타입의 값을 그대로 정규식이나 함수에 넣는 것은 권장되지 않는다.

```jsx
function getLengthBad(array: any) { // 이렇게 하지 맙시다!
	return array.length;
}

function getLength(array: any[]) {
	return array.length;
}
```

앞의 예제에서 `any` 를 사용하는 `getLengthBad` 보다는 `any[]` 를 사용하는 `getLength` 가 더 좋은 함수이다. 그 이유는 세 가지이다.

- 함수 내의 `array.length` 타입이 체크된다.
- 함수의 반환 타입이 `any` 대신 `number` 로 추론된다.
- 함수 호출될 때 매개변수가 배열인지 체크한다.

배열이 아닌 값을 넣어서 실행해 보면, `getLength` 는 제대로 오류를 표시하지만 `getLengthBad` 는 오류를 잡아내지 못하는 것을 볼 수 있다.

```jsx
getLengthBad(/123/); // 오류 없음, undefined를 반환한다.
getLength(/123/);
				// ~~~~ 'RegExp' 형식의 인수는
				// ~~~~ 'any[]' 형식의 매개변수에 할당될 수 없습니다.
```

함수의 매개변수를 구체화할 때, (요소의 타입에 관계없이) 배열의 배열 형태라면 `any[][]` 처럼 선언하면 된다. 그리고 함수의 매개변수가 객체이긴 하지만 값을 알 수 없다면 `{[key: string]: any}` 처럼 선언하면 된다.

```jsx
function hasTwelveLetterKey(o: {[key: string]: any}) {
	for (const key in o) {
		if (key.length === 12) {
			return true;
		}
	}
	return false;
}
```

앞의 예제처럼 함수의 매개변수가 객체지만 값을 알 수 없다면 `{[key: string]: any}` 대신 모든 비기본형(non-primitive) 타입을 포함하는 `object` 타입을 사용할 수도 있다. `object` 타입은 객체의 키를 열거할 수는 있지만 속성에 접근할 수 없다는 점에서 `{[key: string]: any}` 와 약간 다르다.

```jsx
function hasTwelveLetterKey(o: object) {
	for (const key in o) {
		if (key.length === 12) {
			console.log(key, o[key]);
										// ~~~~~~ '{}' 형식에 인덱스 시그니처가 없으므로
										// ~~~~~~ 요소에 암시적으로 'any' 형식이 있습니다.
			return true;
		}
	}
	return false;
}
```

객체지만 속성에 접근할 수 없어야 한다면 `unknown` 타입이 필요한 상황일 수 있다.

함수의 타입에도 단순히 `any` 를 사용해서는 안된다. 최소한으로나마 구체화할 수 있는 세 가지 방법이 있다.

```jsx
type Fn0 = () => any; // 매개변수 없이 호출 가능한 모든 함수
type Fn1 = (arg: any) => any; // 매개변수 1개
type FnN = (...args: any[]) => any; // 모든 개수의 매개변수
																		// "Function" 타입과 동일하다.
```

앞의 예제 등장한 세 가지 함수 타입 모두 `any` 보다는 구체적이다. 마지막 줄을 잘 살펴보면 `...args` 의 타입을 `any[]` 로 선언했다. `any` 로 선언해도 동작하지만 `any[]` 로 선언하면 배열 형태라는 것을 알 수 있어 더 구체적이다.

```jsx
const numArgsBad = (...args: any) => args.length; // any를 반환합니다.
const numArgsGood = (...args: any[]) => args.length // number를 반환합니다.
```

이 예제가 `any[]` 타입을 사용하는 가장 일반적인 경우이다.
