# Item 32

## item.32 유니온의 인터페이스보다는 인터페이시의 유니온을 사용하기

*요약*

- 유니온 타입의 속성을 여러 개가 가지는 인터페이스에서는 속성 간의 관계가 분명하지 않기 때문에 실수가 자주 발생하므로 주의해야 한다.
- 유니온의 인터페이스보다 인터페이스의 유니온이 더 정확하고 타입스크립트가 이해하기도 좋다.
- 타입스크립트가 제어 흐름을 분석할 수 있도록 타입에 태그를 넣는 것을 고려해야 한다. 태그된 유니온은 타입스크립트와 매우 잘 맞기 때문에 자주 볼 수 있는 패턴이다.

---

**유니온 타입의 속성을 가지는 인터페이스를 작성 중이라면, 혹시 인터페이스의 유니온 타입을 사용하는게 더 알맞지는 않을지 검토해 봐야 한다.**

벡터를 그리는 프로그램을 작성 중이고, 특정한 기하학적(geometry) 타입을 가지는 계층의 인터페이스를 정의한다고 가정해보자.

```jsx
interface Layer {
	layout: FillLayout | LineLayout | PointLayout;
	paint: FillPaint | LinePaint | PointPaint;
}
```

`layout` 속성은 모양이 그려지는 방법과 위치(둥근 모서리, 직선)를 제어하는 반면, `paint` 속성은 스타일(파란선, 굵은선, 얇은선, 점선)을 제어한다.

`layout` 이 `LineLayout` 타입이면서 `paint` 속성이 `FillPaint` 타입인 것은 말이 되지 않는다. 이런 조합을 허용한다면 라이브러리에서는 오류가 발생하기 쉽상이고 인터페이스를 다루기도 어려울 것이다.

더 나은 방법으로 모델링하려면 각각 타입의 계층을 분리된 인터페이스로 둬야 한다.

```jsx
interface FillLayer {
	layout: FillLayout;
	paint: FilPaint;
}

interface LineLayer {
	layout: LineLayout;
	paint: LinePaint;
}

interface PointLayer {
	layout: PointLayout;
	paint: PointPaint;
}
type Layer = FillLayer | LineLayer | PointLayer;
```

이런 형태로 `Layer` 를 정의하면 `layout` 과 `paint` 속성이 잘못된 조합으로 섞이는 경우를 방지할 수 있다. 이 코드에서는 유효한 상태만을 표현하도록 타입을 정의했다.

이러한 패턴의 가장 일반적인 예시는 태그된 유니온(또는 구분된 유니온)이다. `Layer` 의 경우 속성 중의 하나는 문자열 리터럴 타입의 유니온이 된다.

```jsx
interface Layer {
	type: 'fill' | 'line' | 'point';
	layout: FillLayout | LineLayout | PointLayout;
	paint: FillPaint | LinePaint | PointPaint;
}
```

`type: 'fill'` 과 함께 `LineLayout` 과 `PointPaint` 타입이 쓰이는 것은 말이 되지 않는다. 이러한 경우를 방지하기 위해 `Layer` 를 인터페이스의 유니온으로 변환해보자.

```jsx
interface FillLayer {
	type: 'fill';
	layout: FillLayout;
	paint: FillPaint;
}

interface LineLayer {
	type: 'line';
	layout: LineLayout;
	paint: LinePaint;
}

interface PointLayer {
	type: 'paint';
	layout: PointLayout;
	paint: PointPaint;
}
type Layer = FillLayer | LineLayer | PointLayer;
```

`type` 속성은 ‘태그’이며 런타임에 어떤 타입의 `Layer` 가 사용되는지 판단하는 데 쓰인다. 타입스크립트는 태그를 참고해 `Layer` 의 타입의 범위를 좁힐 수도 있다.

```jsx
function drawLayer(layer: Layer) {
	if (layer.type === 'fill') {
		const {paint} = layer; // 타입이 FillPaint
		const {layout} = layer; // 타입이 FillLayout
	} else if (layer.type === 'line') {
		const {paint} = layer; // 타입이 LinePaint
		const {layout} = layer; // 타입이 LineLayout
	} else {
		const {paint} = layer; // 타입이 PointPaint
		const {layout} = layer; // 타입이 PointLayout	
	}
}
```

각 타입의 속성들 간의 관계를 제대로 모델링하면, 타입스크립트가 코드의 정확성을 체크하는 데 도움이 된다. 다만 타입 분기 후 `layer` 가 포함된 동일한 코드가 반복되는 것이 어수선해 보인다.

태그된 유니온은 타입스크립트 타입 체커와 잘 맞기 때문에 타입스크립트 코드 어디에서나 찾을 수 있다. 이 패턴을 잘 기억해서 필요할 때 적용할 수 있도록 해야 한다. 어떤 데이터 타입을 태그된 유니온으로 표현할 수 있다면 보통은 그렇게 하는 것이 좋다. 또는 여러 개의 선택적 필드가 동시에 값이 있거나 동시에 `undefined` 인 경우도 태그된 유니온 패턴이 잘 맞는다.

다음 코드의 타입을 보자

```jsx
interface Person {
	name: string;
	// 다음운 둘 다 동시에 있거나 없습니다.
	placeOfBirth?: string;
	dateOfBirth?: Date;
}
```

타입 정보를 담고 있는 주석은 문제가 될 소지가 매우 높다.

`placeOfBirth` 와 `dateOfBirth` 필드는 실제로 관련되어 있지만, 타입 정보에는 어떠한 관계도 표현되지 않았다.

두 개의 속성을 하나의 객체로 모으는 것이 더 나은 설계이다. 이 방법은 `null` 값을 경계로 두는 방법과 비슷하다.

```jsx
interface Person {
	name: string;
	birth?: {
		place: string;
		date: Date;
	}
}
```

이제 `place` 만 있고 `date` 가 없는 경우에는 오류가 발생한다.

```jsx
const alanT: Person = {
	name: 'Alan Turing',
	birth: {
		// ~~~ 'date' 속성이 '{ place: string; }' 형식에 없지만
		// ~~~ '{place: string; date: Date; }' 형식에서 필수입니다.
		place: 'London'
	}
}
```

`Person` 객체를 매개변수로 받는 함수는 `birth` 하나만 체크하면 된다.

```jsx
function eulogize(p: Person) {
	console.log(p.name);
	const {birth} = p;
	if (birth) {
		console.log(`was born on ${birth.date} in ${birth.place}.`);
	}
}
```

타입의 구조를 손 댈 수 없는 상황(예를 들어 API의 결과)이면, 앞서 다룬 인터페이스의 유니온을 사용해서 속성 사이의 관계를 모델링할 수 있다.

```jsx
interface Name {
	name: string;
}

interface PersonWithBirth extens Name {
	placeOfBirth: string;
	dateOfBirth: Date;
}

type Person = Name | PersonWithBirth;
```

이제 중첩된 객체어서도 동일한 효과를 볼 수 있다.

```jsx
function eulogize(p: Person) {
	if ('placeOfBirth' in p) {
		p // 타입이 PersonWithBrith
		const {dateOfBirth} = p // 정상, 타입이 Date
	}
}
```

앞의 두 경우 모두 타입 정의를 통해 속성 간의 관계를 더 명확하게 만들 수 있다.
