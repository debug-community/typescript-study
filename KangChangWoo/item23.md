# Item 23

## item.23 한꺼번에 객체 생성하기

요약

- 속성을 제각각 추가하지 말고 한꺼번에 객체로 만들어야 한다. 안전한 타입으로 속성을 추가하려면 객체 전개`({...a, ...b})` 를 사용하면 된다.
- 객체에 조건부로 속성을 추가하는 방법을 익혀야 한다.

---

변수의 값은 변경될 수 있지만, 타입스크립트의 타입은 일반적으로 변경되지 않는다. 이러한 특성 덕분에 일부 자바스크립트 패턴을 타입스크립트로 모델링하는 게 쉬워진다. 즉, 객체를 생성할 때는 속성을 하나씩 추가하기보다는 여러 속성을 포함해서 한꺼번에 생성해야 타입 추론에 유리하다.

다음은 자바스크립트에서 2차원 점을 표현하는 객체를 생성하는 방법이다.

```jsx
const pt = {};
pt.x = 3;
// ~ '{}' 형식에 'x' 속성이 없습니다.
pt.y = 4;
// ~ '{}' 형식에 'y' 속성이 없습니다.
```

왜냐하면 첫 번째 줄의 `pt` 타입은 `{}` 값을 기준으로 추론되기 때문이다. 존재하지 않는 속성을 추가할 수는 없다.

```jsx
interface Point { x: number; y: number; }
const pt: Point = {};
	// ~~~ '{}' 형식에 'Point' 형식의 x, y 속성이 없습니다.
pt.x = 3;
pt.y = 4;
```

이 문제들은 객체를 한번에 정의하면 해결할 수 있다.

```jsx
const pt = {
	x: 3,
	y: 4,
} // 정상
```

객체를 반드시 제각각 나눠서 만들어야 한다면, 타입 단언문(`as` )을 사용해 타입 체커를 통과하게 할 수 있다.

```jsx
const pt = {} as Point;
pt.x = 3;
pt.y = 4; // 정상
```

물론 이 경우에도 선언할 때 객체를 한꺼번에 만드는게 더 낫다.

```jsx
const pt: Point = {
	x: 3,
	y: 4,
};
```

작은 객체들을 조합해서 큰 객체를 만들어야 하는 경우에도 여러 단계를 거치는 것은 좋지 않은 생각이다.

```jsx
const pt = {x: 3, y: 4};
const id = {name: 'Pythagoras'};
const namedPoint = {};
Object.assign(namedPoint, pt, id);
namedPoint.name;
	// ~~~ '{}' 형식에 'name' 속성이 없습니다.
```

다음과 같이 ‘객체 전개 연산자’ `...` 를 사용하면 큰 객체를 한꺼번에 만들어 낼 수 있다.

```jsx
const namedPoint = {...pt, ...id};
namedPoint.name; // 정상, 타입이 string
```

객체 전개 연산자를 사용하면 타입 걱정 없이 필드 단위로 객체를 생성할 수도 있다. 이때 모든 업데이트마다 새 변수를 사용하여 각각 새로운 타입을 얻도록 하는 게 중요하다.

```jsx
const pt0 = {};
const pt1 = {...pt0, x: 3};
const pt: Point = {...pt1, y: 4}; // 정상
```

이 방법은 간단한 객체를 만들기 위해 우회하기는 했지만, 객체에 속성을 추가하고 타입스크립트가 새로운 타입을 추론할 수 있게 해 유용하다.

타입에 안전한 방식으로 조건부 속성을 추가하려면, 속성을 추가하지 않는 `null` 또는 `{}` 으로 객체 전개를 사용하면 된다.

```jsx
declare let hasMiddle: boolean;
const firstLast = {first: 'Harry', last: 'Truman'};
const president = {...firstLast, ...(hasMiddle ? {middle: 'S'} : {})};
```

편집기에서 `president` 심벌에 마우스를 올려 보면, 타입이 선택적 속성을 가진 것으로 추론된다는 것을 확인할 수 있다.

```jsx
const president {
	middle?: string;
	first: string;
	last: string;
}
```

전개 연산자로 한꺼번에 여러 속성을 추가할 수도 있다.

```jsx
declare let hasDates: boolean;
const nameTitle = {name: 'Khufu', title: 'Pharaoh'};
const pharaoh = {
	...nameTitle,
	...(hasDates ? {start: -2589, end: -2566} : {})
};
```

편집기에서 `pharaoh` 심벌에 마우스를 올려 보면, 이제는 타입이 유니온으로 추론된다.

```jsx
const pharaoh: {
	start: number;
	end: number;
	name: string;
	title: string;
} | {
	name: string;
	title: string;
}
```

`start` 와 `end` 가 선택적 필드이기를 원했다면 이런 결과가 당황스러울 수 있다. 이 타입에서는 `start` 를 읽을 수 없다.

```jsx
pharaoh.start
// ~~~ '{name: string; title: string;}' 형식에
// ~~~ 'start' 속성이 없습니다.
```

이 경우는 `start` 와 `end` 가 항상 함께 정의된다. 이 점을 고려하면 유니온을 사용하는게 가능한 값의 집합을 더 정확히 표현할 수 있다.

그런데 유니온보다는 선택적 필드가 다루기에는 더 쉬울 수 있다. 선택적 필드 방식으로 표현하려면 다음처럼 헬퍼 함수를 사용하면 된다.

```jsx
function addOptional<T extends object, U extends object>(
	a: T, b: U | null
): T & Partial<U> {
	return {...a, ...b};
}

const pharaoh = addOptional(
	nameTitle,
	hasDates ? { start: -2589, end: -2566 } : null
);
pharaoh.start // 정상, 타입이 number | undefined
```

가끔 객체나 배열을 변환해서 새로운 객체나 배열을 생성하고 싶을 수 있다. 이런 경우 루프 대신 내장된 함수형 기법 또는 로대시 같은 유틸리티 라이브러리를 사용하는 것이 ‘한꺼번에 객체 생서하기’ 관점에서 보면 옳다.
