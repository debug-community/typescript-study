# Item 27

## item.27 함수형 기법과 라이브러리로 타입 흐름 유지하기

요약

- 타입 흐름을 개선하고, 가독성을 높이고, 명시적인 타입 구문의 필요성을 줄이기 위해 직접 구현하기보다는 내장된 함수형 기법과 로대시 같은 유틸리티 라이브러리를 사용하는 것이 좋다.

---

파이썬, C, 자바 등에서 볼 수 있는 표준 라이브러리가 자바스크립트에는 포함되어 있지 않다. 수년간 많은 라이브러리들은 표준 라이브러리의 역할을 대신하기 위해 노력해왔다.

제이쿼리(jQuery)는 DOM과의 상호작용뿐만 아니라 객체와 배열을 순회하고 매핑하는 기능을 제공했다. 언더스코어(Underscore)는 주로 일반적인 유틸리티 함수를 제공하는 데 초점을 맞추었고, 이러한 노력을 바탕으로 로대시(Lodash)가 만들어졌다. 람다(Ramda) 같은 최근의 라이브러리는 함수형 프로그래밍의 개념을 자바스크립트 세계에 도입하고 있다.

이러한 라이브러리들의 일부 기능(`map, flatMap, filter, reduce` 등)은 순수 자바스크립트로 구현되어 있다. 이러한 기법과 로대시의 제공 함수들은 루프를 대체할 수 있기 때문에 자바스크립트에서 유용하게 사용되는데, 타입스크립트와 조합하여 사용하면 더욱 빛을 발한다.

그 이유는 타입 정보가 그대로 유지되면서 타입 흐름(flow)이 계속 전달되도록 하기 때문이다. 반면에 직접 루프를 구현하면 타입 체크에 대한 관리도 직접 해야 한다.

예를 들어, 어떤 CSV 데이터를 파싱한다고 생각해보자. 순수 자바스크립트에서는 절차형(imperative) 프로그래밍 형태로 구현할 수 있다.

```jsx
const csvData = " ... ";
const rawRows = csvData.split('\n');
const headers = rawRows[0].split(',');

const rows = rawRows.slice(1).map(rowStr => {
	const row = {};
	rowStr.split(',').forEach((val, j) => {
		row[headers[j]] = val;
	});
	return row;
})
```

함수형 마인드를 조금이라도 가진 자바스크립트 개발자라면 `reduce` 를 사용해서 행 객체를 만드는 방법을 선호할 수도 있다.

```jsx
const rows = rawRows.slice(1)
	.map(rowStr => rowStr.split(',').reduce(
		(row, val, i) => (row[headers[i]] = val, row),
		{})
	);
```

이 코드는 절차형 코드에 비해 세 줄(약 20개 글자)을 절약했지만 보는 사람에 따라 더 복잡하게 느껴질 수도 있다. 키와 값 배열로 취합(zipping)해서 객체로 만들어주는, 로대시의 `zipObject` 함수를 이용하면 코드를 더욱 짧게 만들 수 있다.

```jsx
import _ from 'lodash';
const rows = rawRows.slice(1)
	.map(rowStr => _.zipObject(headers, rowStr.split(',')));
```

코드가 매우 짧아졌다. 그런데 자바스크립트에서는 프로젝트에 서드파티 라이브러리 종속성을 추가할 때 신중해야 한다. 만약 서드파티 라이브러리 기반으로 코드를 짧게 줄이는 데 시간이 많이 든다면, 서드파티 라이브러리를 사용하지 않는게 낫기 때문이다.

그러나 같은 코드를 타입스크립트로 작성하면 서드파티 라이브러리를 사용하는 것이 무조건 유리하다 타입 정보를 참고하며 작업할 수 있기 때문에 서드파티 라이브러리르 기반으로 바꾸는 데 시간이 훨씬 단축된다.

한편, CSV 파서의 절차형 버전과 함수형 버전 모두 같은 오류를 발생시킨다.

```jsx
const rowsA = rawRows.slice(1).map(rowStr => {
	const row = {};
	rowStr.split(',').forEach((val, j) => {
		row[headers[j]] = val;
		// ~~~ '{}' 형식에서 'string' 형식의 매개변수가 포함된
		// ~~~ 인덱스 시그니처를 찾을 수 없습니다.
	});
	return row;
});

const rowsB = rawRows.slice(1)
	.map(rowStr => rowStr.split(',').reduce(
		(row, val, i) => (row[headers[i]] = val, row),
			// ~~~ '{}' 형식에서 'string' 형식의 매개변수가
			// ~~~ 포함된 인덱스 시그니처를 찾을 수 없습니다.
		{}));
```

두 버전 모두 `{}` 의 타입으로 `{[column: string]: string}` 또는 `Record<string, string>` 을 제공하면 오류가 해결된다.

반면 로대시 버전은 별도의 수정 없이도 타입 체커를 통과한다.

```jsx
const rows = rawRows.slice(1)
	.map(rowStr => _.zipObject(headers, rowStr.split(',')));
	// 타입이 _.Dictionary<string>[]
```

`Dictionary` 는 로대시의 타입 별칭이다. `Dictionary<string>` 은 `{[key: string]: string}` 또는 `Record<string, string>` 과 동일하다. 여기서 중요한 점은 타입 구문이 없어도 `rows` 의 타입이 정확하다는 것이다.

데이터의 가공(munging)이 정교해질수록 이러한 장점은 더욱 분명해진다. 예를 들어, 모든 NBA팀의 선수 명단을 가지고 있다고 가정해보자.

```jsx
interface BasketballPlayer {
	name: string;
	team: string;
	salary: number;
}
declare const rosters: {[team: string]: BasketballPlayer[]};
```

루프를 사용해 단순(flat) 목록을 만드려면 배열에 `concat` 을 사용해야 한다.

다음 코드는 동작이 되지만 타입 체크는 되지 않는다.

```jsx
let allPlayers = [];
// ~~~ 'allPlayers' 변수는 형식을 확인할 수 없는 경우
// 일부 위치에서 암시적으로 'any[]' 형식이다.

for (const players of Object.values(rosters)) {
	allPlayers = allPlayers.concat(players);
		// ~~~ 'allPlayers' 변수에는 암시적으로
		// 'any[]' 형식이 포함됩니다.
}
```

이 오류를 고치려면 `allPlayers` 에 타입 구문을 추가해야 한다.

```jsx
let allPlayers: BasketballPlayers[] = [];
for (const players of Object.values(rosters)) {
	allPlayers = allPlayers.concoat(players); // 정상
}
```

그러나 더 나은 해법은 `Array.prototype.flat` 을 사용하는 것이다.

```jsx
const allPlayers = Object.values(rosters).flat();
// 정상, 타입이 BasketballPlayer[]
```

`flat` 메서드는 다차원 배열을 평탄화해(flatten) 준다. 타입 시그니처는 `T[][] => T[]` 같은 형태이다. 이 버전이 가장 간결하고 타입 구문도 필요없다. 또한 `allPlayers` 변수가 향후에 변겨오디지 않도록 `let` 대신 `const` 를 사용할 수 있다.

`allPlayers` 를 가지고 각 팀별로 연봉 순으로 정렬해서 최고 연봉 선수의 명단을 만든다고 가정해보자.

로대시 없는 방법은 다음과 같다. 함수형 기법을 쓰지 않은 부분은 타입 구문이 필요하다.

```jsx
const teamToPlayers: {[team: string]: BasketballPlayer[]} = [];
for (const player of allPlayers) {
	const {team} = player;
	teamToPlayers[team] = teamToPlayers[team] || [];
	teamToPlayers[team].push(player);
}

for (const players of Object.values(teamToPlayers => players[0]);
bestPaid.sort((playerA, PlayerB) => playerB.salary - playerA.salary);
console.log(bestPaid);
```

결과는 다음과 같다.

```jsx
[
	{team: 'GSW', salary: 37457154, name: 'Stephen Curry' },
	{team: 'HOU', salary: 35654150, name: 'Chris Paul' },
	{team: 'LAL', salary: 35654150, name: 'LeBron James' },
	{team: 'OKC', salary: 35654150, name: 'Russell Westbrook' },
	{team: 'DET', salary: 32088932, name: 'Black Griffin' },
	...
]
```

로대시를 사용해서 동일한 작업을 하는 코드를 구현하면 다음과 같다.

```jsx
const bestPaid = _(allPlayers)
	.groupBy(player => player.team)
	.mapValues(players => _.maxBy(players, p => p.salary)!)
	.values()
	.sortBy(p => -p.salary)
	.value() // 타입이 BasketballPlayer[]
```

길이가 절반으로 줄었고, 보기에도 깔끔하며 `null` 아님 단언문(타입 체커는 `_.maxBy` 로 전달된 `players` 배열이 비어 있지 않은지 알 수 없다.)을 딱 한번만 사용했다. 또한 로대시와 언더스코어의 개념인 ‘체인’을 사용했기 때문에, 더 자연스러운 순서로 일련의 연산을 작성할 수 있었다. 만약 체인을 사용하지 않는다면 다음 에제처럼 뒤에서부터 연산이 수행된다.

```jsx
_.c(_.b(_.a(v)))
```

체인을 사용하면 다음처럼 연산자의 등장 순서와 실행 순서가 동일하게 된다.

```jsx
_(v).a().b().c().value()
```

`_(v)` 는 값을 ‘래핑(wrap)’하고, `.value()` 는 ‘언래핑(unwrap)’한다.

래핑된 값의 타입을 보기 위해 체인의 각 함수 호출을 조사할 수 있고, 결과는 항상 정확하다.

로대시의 어떤 기발한 단축 기법이라도 타입스크립트로 정확하게 모델링될 수 있다. 그런데 내장된 `Array.prototype.map` 대신 `_.map` 을 사용하려는 이유가 무엇일까?

한 가지 이유는 콜백을 전달하는 대신 속성의 이름을 전달할 수 있기 때문이다.

예를 들어 다음 세 가지 종류의 호출은 모두 같은 결과를 낸다.

```jsx
// 1. 타입이 string[]
const namesA = allPlayers.map(player => player.name);

// 2. 타입이 string[]
const namesB = _.map(allPlayers, player => player.name)

// 3. 타입이 string[]
const namesC = _.map(allPlayers, 'name');
```

타입스크립트 타입 시스템이 정교하기 때문에 앞의 예제처럼 다양한 동작을 정확히 모델링할 수 있다. 사실 함수 내부적으로는 문자열 리터럴 타입과 인덱스  타입의 조합으로만 이루어져 있기 때문에 타입이 자연스럽게 도출된다.

만약 C++ 또는 자바에 익숙하다면 이런 종류의 타입 추론이 마법처럼 느껴질 수 있다.

```jsx
const salaries = _.map(allPlayers, 'salary'); // 타입이 number[]
const teams = _.map(allPlayers, 'team'); // 타입이 string[]
const mix = _.map(allPlayers, Math.random() < 0.5 ? 'name' : 'salary');
// 타입이 (string | number) []
```

내장된 함수형 기법들과 로대시 같은 라이브러리에 타입 정보가 잘 유지되는 것은 우연이 아니다. 함수 호출 시 전달된 매개변수 값을 건디리지 않고 매번 새로운 값을 반환함으로써, 새로운 타입으로 안전하게 반환할 수 있다.

넓게 보면, 타입스크립트의 많은 부분이 자바스크립트 라이브러리의 동작을 정확히 모델링하기 위해서 개발되었다. 그러므로 라이브러리를 사용할 때 타입 정보가 잘 유지되는 점을 십분 활용해야 타입스크립트의 원래 목적을 달성할 수 있다.
