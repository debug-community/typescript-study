# Item 22

## item.22 타입 좁히기

요약

- 분기문 외에도 여러 종류의 제어 흐름을 살펴보며 타입스크립트가 타입을 좁히는 과정을 이해해야 한다.
- 태그된/구별된 유니온과 사용자 정의 타입 가드를 사용하여 타입 좁히기 과정을 원활하게 만들 수 있다.

---

타입 넓히기의 반대는 타입 좁히기이다. 타입 좁히기는 타입스크립트가 넓은 타입으로부터 좁은 타입으로 진행하는 과정을 말한다. 아마도 가장 일반적인 예시는 `null` 체크일 것이다.

```jsx
// 타입이 HTMLElement | null
const el = document.getElementById('foo');

if (el) {
	el // 타입이 HTMLElement
	el.innerHTML = 'Party Time'.blink();
} else {
	el // 타입이 null
	alert('No element #foo');
}
```

만약 `el` 이 `null` 이라면, 분기문의 첫 번째 블록이 실행되지 않는다. 즉, 첫 번째 블록에서 `HTMLElement | null` 타입의 `null` 을 제외하므로, 더 좁은 타입이 되어 작업이 훨씬 쉬워진다. 타입 체커는 일반적으로 이러한 조건문에서 타입 좁히기를 잘 해내지만, 타입 별칭이 존재한다면 그러지 못할 수도 있다.

분기문에서 예외를 던지거나 함수를 반환하여 블록의 나머지 부분에서 변수의 타입을 좁힐 수도 있다. 예를 들어보자.

```jsx
const el = document.getElementById('foo'); // 타입이 HTMLElement | null
if (!el) throw new Error('Unable to find #foo');
el; // 이제 타입은 HTMLElemet
el.innerHTML = 'Party Time'.blink();
```

이 외에도 타입을 좁히는 방법은 많이 있다. 다음은 `instanceof` 를 사용해서 타입을 좁히는 예제이다.

```jsx
function contains(text: string, search: string | RegExp) {
	if (search instanceof RegExp) {
		search // 타입이 RegExp
		return !!search.exec(text);
	}
	
	search // 타입이 string
	return text.includes(search);
}
```

속성 체크로도 타입을 좁힐 수 있다.

```jsx
interface A { a: number }
interface B { b: number }

function pickAB(ab: A | B) {
	if ('a' in ab) {
		ab // 타입이 A
	} else {
		ab // 타입이 B
	}
	ab // 타입이 A | B
}
```

`Array.isArray` 같은 일부 내장 함수로도 타입을 좁힐 수 있다.

```jsx
function contains(text: string, terms: string | string[]) {
	const termList = Array.isArray(terms) ? terms : [terms];
	termList // 타입이 string[]
	// ...
}
```

타입스크립트는 일반적으로 조건문에서 타입을 좁히는 데 매우 능숙하다.

그러나 타입을 섣불리 판단하는 실수를 저지르기 쉬우므로 다시 한번 꼼꼼히 따져 봐야 한다. 예를 들어, 다음 예제는 유니온 타입에서 `null` 을 제외하기 위해 잘못된 방법을 사용했다.

```jsx
const el = document.getElementById('foo'); // 타입이 HTMLElement | null
if (typeof el === 'object') {
	el; // 타입이 HTMLElement | null
}
```

자바스크립트에서 `typeof null` 이 `"object"` 이기 때문에, `if` 구문에서 `null` 이 제외되지 않았다. 또한 기본형 값이 잘못되어도 비슷한 사례가 발생한다.

```jsx
function foo(x?: number | string | null) {
	if (!x) {
		x; // 타입이 string | number | null | undefined
	}
}
```

빈 문자열 `''` 과 `0` 모두 `false` 가 되기 때문에, 타입은 전혀 좁혀지지 않았고 `x` 는 여전히 블록 내에서 `string` 또는 `number` 가 된다.

타입을 좁히는 또 다른 일반적인 방법은 명시적 ‘태그’ 를 붙이는 방법이다.

```jsx
interface UploadEvent {
	type: 'upload';
	filename: string;
	contents: string;
}

interface DownloadEvent {
	type: 'download';
	filename: string;
}

type AppEvent = UploadEvent | DownloadEvent;
function handleEvent(e: AppEvent) {
	switch (e.type) {
		case 'download':
			e // 타입이 DownloadEvent
			break;
		case 'upload':
			e // 타입이 UploadEvent
			break;
	}
}
```

이 패턴은 ‘태그된 유니온(tagged union)’ 또는 ‘구별된 유니온(discriminated union)’ 이라고 불리며, 타입스크립트 어디에서나 찾아볼 수 있다.

만약 타입스크립트가 타입을 식별하지 못한다면, 식별을 돕기 위해 커스텀 함수를 도입할 수 있다.

```jsx
function isInputElement(el: HTMLElement): el is HTMLInputElement {
	return 'value' in el;
}

function getElementContent(el: HTMLElement) {
	if (isInputElement(el)) {
		el; // 타입이 HTMLInputElement
		return el.value;
	}
	
	el; // 타입이 HTMLElement
	return el.textContent;
}
```

이러한 기법을 ‘사용자 정의 타입 가드’ 라고 한다. 반환 타입의 `el is HTMLInputElement` 는 함수의 반환이 `true` 인 경우, 타입 체커에게 매개변수의 타입을 좁힐 수 있다고 알려준다.

어떤 함수들은 타입 가드를 사용해 배열과 객체의 타입 좁히기를 할 수 있다. 예를 들어, 배열에서 어떤 탐색을 수행할 때 `undefined` 가 될 수 있는 타입을 사용할 수 있다.

```jsx
const jackson5 = ['Jackie', 'Tito', 'Jermaine', 'Marlon', 'Michael'];
const members = ['Janet', 'Michael'].map(
	who => jackson5.find(n => n === who)
); // 타입이 (string | undefined)[]
```

이럴 때 타입 가드를 사용하면 타입을 좁힐 수 있다.

```jsx
function isDefined<T>(x: T | undefined): x is T {
	return x !== undefined;
}

const members = ['Janet', 'Michael'].map(
	who => jackson5.find( n => n === who)
).filter(isDefined); // 타입이 string[]
```

편집기에서 타입을 조사하는 습관을 가지면 타입 좁히기가 어떻게 동작하는지 자연스레 익힐 수 있다. 타입스크립트에서 타입이 어떻게 좁혀지는지 이해한다면 타입 추론에 대한 개념을 잡을 수 있고, 오류 발생의 원인을 알 수 있으며, 타입 체커를 더 효율적으로 이용할 수 있다.
