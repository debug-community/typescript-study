# Item26

# Item.26 타입 추론에 문맥이 어떻게 사용되는지 이해하기

요약

- 타입 추론에서 문맥이 어떻게 쓰이는지 주의해서 살펴봐야 한다.
- 변수를 뽑아서 별도로 선언했을 때 오류가 발생한다면 타입 선언을 추가해야 한다.
- 변수가 정말로 상수라면 상수 단언(`as const` )을 사용해야 한다. 그러나 상수 단언을 사용하면 정의한 곳이 아니라 사용한 곳에서 오류가 발생하므로 주의해야 한다.

---

타입스크립트는 타입을 추론할 때 단순히 값만 고려하지는 않는다. 값이 존재하는 곳의 문맥까지도 살핀다. 그런데 문맥을 고려해 타입을 추론하면 가끔 이상한 결과가 나온다. 이때 타입 추론에 문맥이 어떻게 사용되는지 이해하고 있다면 제대로 대처할 수 있다.

자바스크립트는 코드의 동작과 실행 순서를 바꾸지 않으면서 표현식을 상수로 분리해 낼 수 있다. 예를 들어, 다음 두 문장은 동일하다.

```jsx
// 인라인 형태
setLanguage('JavaScript');

// 참조 형태
let launguage('JavaScript');
setLaunguage(launguage);
```

타입스크립트에서는 다음 리팩토링이 여전히 동작한다.

```jsx
function setLanguage(language: string) { /* ... */ }

setLanguage('JavaScript'); // 정상

let language = 'JavaScript';
setLanguage(language); // 정상
```

이제 문자열 타입을 더 특정해서 문자열 리터럴 타입의 유니온으로 바꾼다고 가정해보자.

```jsx
type Language = 'JavaScript' | 'TypeScript' | 'Python';
function setlanguage(language: Language) { /* ... */ }

setLanguage = 'JavaScript'; // 정상

let language = 'JavaScript'; // <- 여기서 string으로 타입 추론
setLanguage(language);
// ~~~ 'string' 형식의 인수는
// ~~~ 'Language' 형식의 매개변수에 할당될 수 없습니다.
```

인라인(inline) 형태에서 타입스크립트는 함수 선언을 통해 매개변수가 `Language` 타입이어야 한다는 것을 알고 있다. 해당 타입에 문자열 리터럴 `'JavaScript'` 는 할당 가능하므로 정상이다.

그러나 이 값을 변수로 분리해내면, 타입스크립트는 할당 시점에 타입을 추론한다. 이번 경우는 `string` 으로 추론했고, `Language` 타입으로 할당이 불가능하므로 오류가 발생한다.

이런 문제를 해결하는 두 가지 방법이 있다. 첫 번째 해법은 타입 선언에서 `language` 의 가능한 값을 제한하는 것이다.

```jsx
let language: Language = 'JavaScript'; // <- 미리 타입 선언
setLanguage(language); // 정상
```

만약 `language` 의 값에 `'Typescript'` (’S’가 아니고 ‘s’) 같은 오타가 있었다면 오류를 표시해 주는 장점도 있다.

두 번째 해법은 `language` 를 상수로 만드는 것이다.

```jsx
const language = 'JavaScript';
setLanguage(language); // 정상
```

`const` 를 사용하여 타입 체커에서 `language` 는 변경할 수 없다고 알려준다. 따라서 타입스크립트는 `language` 에 대해서 더 정확한 타입인 문자열 리터럴 `"JavaScript"`로 추론할 수 있다. `"JavaScript"` 는 `Language` 에 할당할 수 있으므로 타입 체크를 통과한다. 물론, `language` 를 재할당해야 한다면 타입 선언이 필요하다.

그런데 이 과정에서 사용되는 문맥으로부터 값을 분리했다. 문맥과 값을 분리하면 추후에 근본적인 문제를 발생시킬 수 있다. 이제부터 이러한 문맥의 소실로 인해 오류가 발생하는 몇 가지 경우와, 이를 어떻게 해결하는지 하나하나 살펴보자.

## 튜플 사용 시 주의 점

문자열 리터럴 타입과 마찬가지로 튜플 타입에서도 문제가 발생한다. 이동이 가능한 지도를 보여주는 프로그램을 작성한다고 생각해보자.

```jsx
// 매개변수는 (latitude, longitude) 쌍이다.
function panTo(where: [number, number]) { /* ... */ }

panTo([10, 20]); // 정상

const loc = [10, 20];
panTo(loc);
// ~~~ 'number[]' 형식의 인수는
// ~~~ '[number, number]' 형식의 매개변수에 할당될 수 없습니다.
```

이전 예제처럼 여기서도 문맥과 값을 분리했다. 첫 번째 경우는 `[10, 20]` 이 튜플 타입 `[number, number]` 에 할당 가능하다. 두 번째 경우는 타입스크립트가 `loc` 의 타입을 `number[]` 로 추론한다.(즉, **길이를 알 수 없는 숫자의 배열**) 많은 배열이 이와 맞지 않는 수의 요소를 가지므로 튜플 타입에 할당할 수 없다.

그러면 `any` 를 사용하지 않고 오류를 고칠 수 있는 방법을 생각해 보자. `any` 대신 `const` 로 선언하면 된다는 답이 떠오를 수도 있지만, `loc` 는 이미 `const` 로 선언한 상태이다. 그보다는 타입스크립트가 의도를 정확히 파악할 수 있도록 타입 선언을 제공하는 방법을 시도해보자.

```jsx
const loc: [number, number] = [10, 20];
panTo(loc); // 정상
```

`any` 를 사용하지 않고 오류를 고칠 수 있는 또 다른 방법은 ‘상수 문맥’을 제공하는 것이다. `const` 는 단지 값이 가리키는 참조가 변하지 않는 얕은(shallow) 상수인 반면, `as const` 는 그 값이 내부까지(deeply) 상수라는 사실을 타입스크립트에게 알려준다.

```jsx
const loc = [10, 20] as const; // -> readonly 타입
panTo(loc);
// ~~~ 'readonly [10, 20]' 형식은 'readonly' 이며
// ~~~ 변경 가능한 형식 '[number, number]'에 할당할 수 없습니다.
```

편집기에서 `loc` 에 마우스를 올려 보면, 타입은 이제 `number[]` 가 아니라 `readonly[10, 20]` 으로 추론됨을 알 수 있다. 그런데 안타깝게도 이 추론은 ‘너무 과하게’ 정확하다. `panTo` 의 타입 시그니처는 `where` 의 내용이 불변이라고 보장하지 않는다. 즉, `loc` 매개변수가 `readonly` 타입이므로 동작하지 않는다.

따라서 `any` 를 사용하지 않고 오류를 고칠 수 있는 최선의 해결책은 `panTo` 함수에 `readonly` 구문을 추가하는 것이다.

```jsx
function panTo(where: readonly [number, number]) { /* ... */}
const loc = [10, 20] as const;
panTo(loc); // 정상
```

타입 시그니처를 수정할 수 없는 경우라면 타입 구문을 사용해야 한다.

`as const` 는 문맥 손실과 관련한 문제를 깔끔하게 해결할 수 있지만, 한 가지 단점을 가지고 있다. 만약 타입 정의에 실수가 있다면(예를 들어, 튜플에 세 번째 요소를 추가한다면) 오류는 타입 정의가 아니라 호출되는 곳에서 발생한다는 것이다. 특히 여러 겹 중첩된 객체에서 오류가 발생한다면 근본적인 원인을 파악하기 어렵다.

```jsx
const loc = [10, 20, 30] as const; // 실제 오류는 여기서 발생
panTo(loc);
// ~~~ 'readonly [10, 20, 30]' 형식의 인수는
// ~~~ 'readonly [number, number]' 형식의 매개변수에 할당될 수 없습니다.
// ~~~ 'length' 속성의 형식이 호환되지 않습니다.
// ~~~ '3' 형식은 '2' 형식에 할당할 수 없습니다.
```

## 객체 사용 시 주의점

문맥에서 값을 분리하는 문제는 문자열 리터럴이나 튜플을 포함하는 큰 객체에서 상수를 뽑아낼 때도 발생한다.

```jsx
type Language = 'JavaScript' | 'TypeScript' | 'Python';
interface GovernedLanguage {
	language: Language;
	organization: string;
}

function complain(language: GovernedLanguage) { /* ... */ }
complain({language: 'TypeScript', organization: 'Microsoft' }); // 정상

const ts = {
	language: 'TypeScript',
	organization: 'Microsoft',
};
complain(ts);
// ~~ '{language: string; organization: string; }' 형식의 인수는
// 'GovernedLanguage' 형식의 매개변수에 할당될 수 없습니다.
// 'language' 속성의 형식이 호환되지 않습니다.
// 'string' 형식은 'Language' 형식에 할당할 수 없습니다.
```

ts 객체에서 `language` 의 타입은 `string` 으로 추론된다. 이 문제는 타입 선언을 추가하거나(`const ts: GovernedLanguage = ...`  ) 상수 단언(`as const` )을 사용해 해결한다.

> as const → 리터럴 타입으로 타입을 고정.
> 

## 콜백 사용 시 주의점

콜백을 다른 함수로 전달할 때, 타입스크립트는 콜백의 매개변수 타입을 추론하기 위해 문맥을 사용한다.

```jsx
function callWithRandomNumbers(fn: (n1: number, n2: number) => void {
	fn(Math.random(), Math.random());
}

callWithRandomNumber((a,b) => {
	a; // 타입이 number
	b; // 타입이 number
	console.log(a + b);
});
```

`callWithRandom` 의 타입 선언으로 인해 `a` 와 `b` 의 타입이 `number` 로 추론된다.

콜백을 상수로 뽑아내면 문맥이 소실되고 `noImplicitAny` 오류가 발생하게 된다.

```jsx
const fn = (a, b) => {
	// ~ 'a' 매개변수에는 암시적으로 'any' 형식이 포함됩니다.
	// ~ 'b' 매개변수에는 암시적으로 'any' 형식이 포함됩니다.
	console.log(a + b);
}

callWithRandomNumbers(fn);
```

이런 경우는 매개변수에 타입 구문을 추가해서 해결할 수 있다.

```jsx
const fn = (a: number, b: number) => {
	console.log(a + b);
}
callWithRandomNumbers(fn);
```

또는 가능할 경우 전체 함수 표현식에 타입 선언을 적용하는 것이다.
