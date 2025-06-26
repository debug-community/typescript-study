# Item 17

## item.17 변경 관련된 오류 방지를 위해 readonly 사용하기

*요약*

- 만약 함수가 매개변수를 수정하지 않는다면 `readonly` 로 선언하는 것이 좋다. `readonly` 매개변수는 인터페이스를 명확하게 하며, 매개변수가 변경되는 것을 방지한다.
- `readonly` 를 사용하면 변경하면서 발생하는 오류를 방지할 수 있고, 변경이 발생하는 코드도 쉽게 찾을 수 있다.
- `const` 와 `readonly` 의 차이를 이해해야 한다.
- `readonly` 는 얕게 동작한다는 것을 명심해야 한다.

---

아래는 삼각수(triangular number, 1, 1+2, 1+2+3 …)를 출력하는 코드이다.

```jsx
function printTriangles(n: number) {
	const nums = [];
	for (let i = 0; i < n; i++) {
		nums.push(i);
		console.log(arraySym(nums));
	}
}
```

코드는 간단하다. 그러나 실행해보면 문제가 발생한다.

```jsx
> printTriangles(5)
0
1
2
3
4
```

`arraySum` 이 `nums` 을 변경하지 않는다고 간주해서 문제가 발생했다.

이 문제는 다음 코드와 같이 해결할 수 있다.

```jsx
function arraySum(arr: number[]) {
	let sum = 0, num;
	while ((num = arr.pop()) !== undefined) {
		sum += num;
	}
	return sum;
}
```

이 함수는 배열 안의 숫자들을 모두 합친다. 그런데 계산이 끝나면 원래 배열이 전부 비게 된다. 자바스크립트 배열은 내용을 변경할 수 있기 때문에, 타입스크립트에서도 역시 오류 없이 통과하게 된다.

오류의 범위를 좁히기 위해 `arraySum` 이 배열을 변경하지 않는다는 선언을 해보자. `readonly` 접근 제어자를 사용하면 된다.

```jsx
function arraySum(arr: readonly number[]) {
	let sum = 0, num;
	while ((num = arr.pop()) !== undefined) {
		// ~~ 'readonly number[]' 형식에 'pop' 속성이 없습니다.
		sum += num;
	}
	return sum;
}
```

이 오류 메시지를 자세히 살펴보자. `readonly number[]` 는 타입이고, `number[]` 와 구분되는 몇 가지 특징이 있다.

- 배열의 요소를 읽을 수 있지만, 쓸 수는 없다.
- `length` 를 읽을 수 있지만, 바꿀 수는 없다. (배열을 변경)
- 배열을 변경하는 `pop`을 비롯한 다른 메서드를 호출할 수 없다.

`number[]` 는 `readonly number[]` 보다 기능이 많기 때문에, `readonly number[]` 의 서브타입이 된다. 따라서 변경 가능한 배열을 `readonly` 배열에 할당할 수 있다. 하지만 그 반대는 불가능하다.

```jsx
const a: number[] = [1, 2, 3];
const b: readonly number[] = a;
const c: number[] = b;
	// ~ 'readonly number[]' 타입은 'readonly'이므로
	// 변경 가능한 'number[]' 타입에 할당될 수 없습니다.
```

타입 단언문 없이 `readonly` 접근 제어자를 제거할 수 있다면 `readonly` 는 쓸모없을 것이므로 여기서 오류가 발생하는 것이 이치에 맞다.

매겨변수를 `readonly` 로 선언하면 다음과 같은 일이 생긴다.

- 타입스크립트는 매개변수가 함수 내에서 변경이 일어나는지 체크한다.
- 호출하는 쪽에서는 함수가 매개변수를 변경하지 않는다는 보장을 받게 된다.
- 호출하는 쪽에서 함수에 `readonly` 배열을 매개변수로 넣을 수도 있다.

**자바스크립트에서는(타입스크립트에서도 마찬가지) 명시적으로 언급하지 않는 한, 함수가 매개변수를 변경하지 않는다고 가정한다. 그러나 이러한 암묵적인 방법은 타입 체크에 문제를 일으킬 수 있다.**

명시적인 방법을 사용하는 것이 컴파일러와 사람 모두에게 좋다.

앞 예제의 `arraySum` 을 고치는 방법은 간단하다. 배열을 변경하지 않으면 된다.

```jsx
function arraySum(arr: readonly number[]) {
	let sum = 0;
	for (const num of arr) {
		sum += num;
	}
	return sum;
}
```

이제 `printTriangles` 이 제대로 동작한다.

```jsx
> printTriangles(5)
0
1
3
6
10
```

만약 함수가 매개변수를 변경하지 않는다면, `readonly` 로 선언해야 한다. 더 넓은 타입으로 호출할 수 있고, 의도치 않은 변경은 방지될 것이다. 이로 인한 단점은 상대적으로 적다.

굳이 단점을 찾아보자면 매개변수가 `readonly` 로 선언되지 않은 함수를 호출해야 할 경우도 있다는 것이다. 만약 함수가 매개변수를 변경하지 않고도 제어가 가능하다면 `readonly` 로 선언하면 된다. 그런데 어떤 함수를 `readonly` 로 만들면, 그 함수는 호출하는 다른 함수도 `readonly` 로 만들어야 한다. 그러면 인터페이스를 명확히 하고 타입 안전성을 높일 수 있기 때문에 꼭 단점이라고 볼 순 없다. 그러나 다른 라이브러리에 있는 함수를 호출하는 경우라면, 타입 선언을 바꿀 수 없으므로 타입 단언문(`as number[]` )을 사용해야 한다.

`readonly` 를 사용하면 지역 변수와 관련된 모든 종류의 변경 오류를 방지할 수 있다. 예를 들어 소설(novel)에 다양한 처리를 하는 프로그램을 만든다고 가정해보자. 연속된 행을 가져와서 빈 줄을 기준으로 구분되는 단락으로 나누는 기능을 하는 프로그램이다.

```jsx
function parseTaggedText(lines: string[]): string[][] {
	const paragraphs: string[][] = [];
	const currPara: string[] = [];
	
	const addParagraph = () => 
		if (currPara.length) {
			paragraphs.push(currPara);
			currPara.length = 0; // 배열을 비움
		}
	};
	
	for (const line of lines) {
		if (!line) {
			addParagraph();
		} else {
			currPara.push(line);
		}
	}
	addParagraph();
	return paragraphs;
}
```

어떤 소설을 입력으로 넣고 실행하면, 다음처럼 출력된다.

```jsx
[ [], [], [] ]
```

완전히 잘못되었다. 이 코드의 문제점은 별칭과 변경을 동시에 사용해 발생했다. 별칭은 다음 행에서 발생한다.

```jsx
paragraphs.push(currPara);
```

`currPara` 의 내용이 삽입되지 않고 배열의 참조가 삽입되어있다. `currPara` 에 새 값을 채우거나 지운다면 동일한 객체를 참조하고 있는 `paragraphs` 요소에도 변경이 반영된다.

결국 다음 코드가 문제의 핵심이다.

```jsx
paragraphs.push(currPara);
currPara.length = 0;
```

이 코드는 새 단락을 `paragraphs` 에 삽입하고 바로 지워 버린다.

문제는 `currPara.length` 를 수정하고 `currPara.push` 를 호출하면 둘 다 `currPara` 배열을 변경한다는 점이다. `currPara` 를 `readonly` 로 선언하여 이런 동작을 방지할 수 있다. 선언을 바꾸는 즉시 코드 내에서 몇 가지 오류가 발생하게 된다.

…

`paragraphs` 에 대한 오류를 바로잡는 방법은 세가지이다.

첫 번째, `currPara` 의 복사본을 만드는 방법.

```jsx
paragraphs.push([...currPara]);
```

`currPara` 는 `readonly` 로 유지되지만, 복사본은 원하는 대로 변경이 가능하기 때문에 오류는 사라진다.

두 번째, `paragraphs` (그리고 함수의 반환 타입)를 `readonly string[]` 의 배열로 변경하는 방법이다.

```jsx
const paragraphs: (readonly string[])[] = [];
```

(여기서 괄호가 중요한데, `readonly string[][]` 은 `readonly` 배열의 변경 가능한 배열이 아니라 변경 가능한 배열의 `readonly`  배열이기 때문이다.)

앞의 코드는 동작하지만 `parseTaggedText` 의 사용자에게는 조금 불친절하게 느껴질 것이다. 이미 함수가 반환한 값에 대해 영향을 끼치는 것이 맞는 방법인지 고민해 봐야 한다.

세 번째, 배열의 `readonly` 속성을 제거하기 위해 단언문을 쓰는 방법이다.

```jsx
paragraphs.push(currPara as string[]);
```

바로 다음 문장에서 `currPara` 를 새 배열에 할당하므로, 매우 공격적인 단언문처럼 보이지는 않는다.

`readonly` 는 얕게(shallow) 동작한다는 것에 유의하며 사용해야 한다. 앞에서 이미 `readonly string[][]` 을 봤다. 만약 객체의 `readonly` 배열이 있다면, 그 객체 자체는 `readonly` 가 아니다.

```jsx
const date: readonly Date[] = [new Date()];
dates.push(new Date());
	// ~~~ 'readonly Date[]' 형식에 'push' 속성이 없습니다.
dates[0].setFullYear(2037); // 정상
```

비슷한 경우가 `readonly` 의 사촌 격이자 객체에 사용되는 `Readonly` 제너릭에도 해당된다.

```jsx
interface Outer {
	inner: {
		x: number;
	}
}

const o: Readonly<Outer> = { inner : {x: 0} };
o.inner = { x: 1 };
// ~~ 읽기 전영 속성이기 때문에 'inner'에 할당할 수 없습니다.
o.inner.x = 1; // 정상
```

타입 별칭을 만든 다음에 정확히 무슨 일이 일어나는지 편집기에서 살펴볼 수 있다.

```jsx
type T = Readonly<Outer>;
// Type T = {
//    readonly inner: {
//    x: number;
//    }
// }
```

중요한 점은 `readonly` 접근제어자는 `inner` 에 적용되는 것이지 `x` 는 아니라는 것이다. 현재 시점에는 깊은 `readonly` 타입이 기본으로 지원되지 않지만, 제너릭을 만들면 깊은 `readonly` 타입을 사용할 수 있다. 그러나 제너릭은 만들기 까다롭기 때문에 라이브러리를 사용하는 게 낫다. 예를 들어 `ts-essentials` 에 있는 `DeepReadonly` 제너릭을 사용하면 된다.

인덱스 시그니처에도 `readonly` 를 쓸 수 있따. 읽기는 허용하되 쓰기를 방지하는 효과가 있다.

```jsx
let obj: {readonly [k: string]: number} = {};
// 또는 Readonly<{[k: string]: number}
obj.hi = 45;
// ~~ ... 형식의 인덱스 시그니처는 읽기만 허용됩니다.
obj = {...obj, hi: 12}; // 정상
obj = {...obj, bye: 34}; // 정상
```

이 코드처럼 인덱스 시그니처에 `readonly` 를 사용하면 객체의 속성이 변경되는 것을 방지할 수 있다.
