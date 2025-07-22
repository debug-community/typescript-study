# Item 28

## item.28 유효한 상태만 표현하는 타입을 지향하기

요약

- 유효한 상태와 무효한 상태를 둘 다 표현하는 타입은 혼란을 초래하기 쉽고 오류를 유발하게 된다.
- 유효한 상태만 표현하는 타입을 지향해야 한다. 코드가 길ㄹ어지거나 표현하기 어렵지만 결국은 시간을 절약하고 고통을 줄일 수 있다.

---

타입을 잘 설계하면 코드는 직관적으로 작성할 수 있다. 그러나 타입 설계가 엉망이라면 어떠한 기억이나 문서도 도움이 되지 못한다. 코드는 뒤죽박죽이 되고, 버그는 창궐한다.효과적으로 타입을 설계하려면, 유효한 상태만 표현할 수 있는 타입을 만들어내는 것이 가장 중요하다.

웹 애플리케이션을 만든다고 가정하자. 애플리케이션에서 페이지를 선택하면, 페이지의 내용을 로드하고 화면에 표시한다. 페이지의 상태는 다음처럼 설계했다.

```jsx
interface State {
	pageText: string;
	isLoading: boolean;
	error?: string;
}
```

페이지를 그리는 `renderPage` 함수를 작성할 때는 상태 객체의 필드를 전부 고려해서 상태 표시를 분기해야 한다.

```jsx
function renderPage(state: State) {
	if (state.error) {
		return `Error! Unable to load ${currentPage}: ${state.error}`;
	} else if (state.isLoading){
		return `Loading ${currentPage}...`;
	}
	return `<h1>${currentPage}</h1>\n${state.pageText}`
}
```

코드를 살펴보면 분기 조건이 명확히 분리되어 있지 않다는 것을 알 수 있다. **`isLoading` 이 `true` 이고 동시에 `error` 값이 존재하면 로딩 중인 상태인지 오류가 발생한 상태인지 명확히 구분할 수 없다.** 필요한 정보가 부족하기 때문이다.

한편 페이지를 전환하는 `changePage` 함수는 다음과 같다.

```jsx
async function changePage(state: State, newPage: string) {
	state.isLoading = true;
	try {
		const response = await fetch(getUrlForPage(newPage));
		if (!response.ok) {
			throw new Error(`Unable to load ${newPage}: ${response.statusText}`);
		}
		const text = await response.text();
		state.isLoading = false;
		state.pageText = text;
	} catch (e) {
		state.error = '' + e;
	}
}
```

`changePage` 에는 많은 문제점이 있다. 몇 가지 정리해보면 다음과 같다.

- 오류가 발생했을 때 `state.isLoading` 을 `false` 로 설정하는 로직이 빠져 있다.
- `state.error` 를 초기화하지 않았기 때문에, 페이지 전환 중에 로딩 메시지 대신 과거의 오류 메시지를 보여 주게 된다.
- 페이지 로딩 중에 사용자가 페이지를 바꿔 버리면 어떤 일이 벌어질지 예상하기 어렵다. 새 페이지에 오류가 뜨거나, 응답이 오는 순서에 따라 두 번째 페이지가 아닌 첫 번째 페이지로 전환될 수도 있다.

문제는 바로 상태 값의 두 가지 속성이 동시에 정보가 부족하거나(요청이 실패한 것인지 여전히 로딩 중인지 알 수 없다.), 두 가지 속성이 충돌(오류이면서 동시에 로딩 중일 수 있다)할 수 있다는 것이다.

`State` 타입은 `isLoading` 이 `true` 이면서 동시에 `error` 값이 설정되는 무효한 상태를 허용한다. 무효한 상태가 존재하면 `render()` 와 `changePage()` 둘 다 제대로 구현할 수 없게 된다.

다음은 애플리케이션의 상태를 좀 더 제대로 표현한 방법이다.

```jsx
interface RequestPending {
	state: 'pending';
}

interface RequestError {
	state: 'error';
	error: string;
}

interface RequestSuccess {
	state: 'ok';
	pageText: string;
}
type RequestState = RequestPending | RequestError | RequestSuccess;

interface State {
	currentPage: string;
	requests: {[page: string]: RequestState};
}
```

여기서는 네트워크 요청 과정 각각의 상태를 명시적으로 모델링하는 태그된 유니온(또는 구별된 유니온)이 사용되었다. 이번 예제는 상태를 나타내는 타입의 코드 길이가 서너 배 길어지긴 했지만, 무효한 상태를 허용하지 않도록 크게 개선되었다.

현재 페이지는 발생하는 모든 요청의 상태로서, 명시적으로 모델링되었다. 그 결과로 개선된 `renderPage` 와 `changePage` 함수는 쉽게 구현할 수 있다.

```jsx
function renderPage(state: State) {
	const {currentPage} = state;
	const requestState = state.requests[currentPage];
	switch (requestState.state) {
		case 'pending':
			return `Loading ${currentPage}...`;
		case 'error':
			return `Error! Unable to load ${currentPage}: ${requestState.error}`;
		case 'ok';
			return `<h1>${currentPage}</h1>\n${requestState.pageText}`;
	}
}

async function changePage(state: State, newPage: string) {
	state.requests[newPage] = {state: 'pending'};
	state.currentPage = newPage;
	try {
		const response = await fetch(getUrlForPage(newPage);
		if (!response.ok) {
			throw new Error(`Unable to load ${newPage}: ${response.statusText}`);
		}
		const pageText = await response.text();
		state.requests[newPage] = {state: 'ok', pageText};
	} catch (e) {
		state.requests[newPage] = {state: 'error', error: '' + e};
	}
}
```

이번 아이템의 처음에 등장했던 `renderPage` 와 `changePage` 의 모호함은 완전히 사라졌다. 현재 페이지가 무엇인지 명확하며, 모든 요청은 정확히 하나의 상태로 맞아 떨어진다. 요청이 진행중인 상태에서 사용자가 페이지를 변경하더라도 문제없다. 무효가 된 요청이 실행되긴 하겠지만 UI에는 영향을 미치지 않는다.

앞의 웹 애플리케이션 예제보다 간단하지만 끔찍한 예를 들어보자

2009년 6월 1일 대서양에서 추락한 에어프랑스 447 항공편이었던 에어버스 330 비행기 사례이다. 에어버스는 전자 조종식 항공기로, 조종사의 제어가 물리적으로 비행 장치에 전달되기 전에 컴퓨터 시스템을 거치게 된다. 사고 후 2년이 지나서야 추락한 비행기의 블랙박스가 복구되었다. 추락을 일으킨 많은 원인이 밝혀졌는데, 주요 원인은 바로 잘못된 상태 설계였다.

에어버스 330의 조종석에는 기장과 부기장을 위한 분리된 제어 세트가 있다. 사이드 스틱은 비행기의 전진 방향(받음각, angle of attack)을 제어한다. 뒤로 당기면 비행기가 올라가고, 앞으로 밀면 아래로 내려가는 방식이다. 에어버스 330은 두 개의 사이드 스틱이 독립적으로 움직이는 이중 입력 모드 시스템을 사용했다. 타입스크립트로 이중 입력 모드 상태를 모델링해 보면 다음과 같다.

```jsx
interface CockpitControls {
	/** 왼쪽 사이드 스틱의 각도, 0 = 중립, += 앞으로 */
	leftSideStick: number;
	/** 오른쪽 사이드 스틱의 각도, 0 = 중립, += 앞으로 */
	rightSideStick: number;
}
```

이 데이터 구조가 주어진 상태에서 현재 스틱의 설정을 계산하는 `getStickSetting` 함수를 작성한다고 가정해보자.

일단 기장(왼쪽 스틱)이 제어하고 있다고 가정하면 다음처럼 구현할 수 있다.

```jsx
function getStickSetting(controls: CockpitControls) {
	return controls.leftSideStick;
}
```

부기장(오른쪽 스틱)이 제어하고 있는 상태라면 기장의 스틱 상태는 중립일 것이다.결과적으로 기장이든 부기장이든 둘 중 하나의 스틱 값 중에서 중립이 아닌 값을 사용해야 한다.

```jsx
function getStickSetting(controls: CockPitControls) {
	const {leftSideStick, rightSideStick} = controls;
	if (leftSideStick === 0) {
		return rightSideStick;
	}
	return leftSideStick;
}
```

그러나 이 코드에는 문제가 있다. 왼쪽 스틱의 로직과 동일하게 오른쪽 스틱이 중립일 때 왼쪽 스틱 값을 사용해야 한다. 그러므로 오른쪽 스틱에 대한 체크를 코드에 추가해야 한다.

```jsx
function getStickSetting(controls: CockPitControls) {
	const {leftSideStick, rightSideStick} = controls;
	if (leftSideStick === 0) {
		return rightSideStick;
	} else if (rightSideStick === 0) {
		return leftSideStick;
	}
	// ???
}
```

두 스틱이 모두 중립이 아닌 경우를 고려해보자. 다행히 두 스틱이 비슷한 값이라면 스틱의 각도를 평균해서 계산할 수 있다.

```jsx
function getStickSetting(controls: CockPitControls) {
	const {leftSideStick, rightSideStick} = controls;
	if (leftSideStick === 0) {
		return rightSideStick;
	} else if (rightSideStick === 0) {
		return leftSideStick;
	}
	
	if (Math.abs(leftSideStick - rightSideStick) < 5) {
		return (leftSideStick + rightSideStick) / 2;
	}
	// ???
}
```

그러나 두 스틱의 각도가 매우 다른 경우는 해결하기 어렵다. 그렇다고 해결책없이 조종사에게 오류를 띄운다는 것은 현실적으로 불가능한 일이다. 비행중이기 때문에 스틱의 각도는 어떻게든 설정되어야 한다.

한편, 비행기가 폭풍에 휘말리자 부기장은 조용히 사이드 스틱을 뒤로 당겼다. 고도는 올라갔지만 결국은 속력이 떨어져서 스톨(양력을 잃고 힘없이 떨어지는) 상태가 되었고 곧이어 비행기는 추락하기 시작했다.

조종사들은 비행 중 스톨 상태에 빠지면, 스틱을 앞으로 밀어 비행기가 하강하면서 속도를 다시 높이도록 훈련받는다. 기장은 훈련대로 스틱을 앞으로 밀었다. 그러나 부기장은 여전히 스틱을 뒤로 당기고 있었다. 이때 에어버스의 계산 함수는 다음과 같은 모습이다.

```jsx
function getStickSetting(controls: CockpitControls) {
	return (controls.leftSideStick + controls.rightSideStick) / 2;
}
```

기장이 스틱을 힘껏 앞으로 밀어 봤자 부기장이 뒤로 당기고 있었기에, 평균값에는 변함이 없다. 따라서 에어버스는 아무것도 하지 않게 된다. 기장은 비행기가 하강하지 않는 이유를 알 수 없었을 것이다. 부기장이 무슨 일이 벌어지고 있는지 깨달았을 때는 이미 속력을 회복할 수 없을 정도로 고도가 너무 낮았고, 결국 바다로 추락해 228명 탑승객 전원의 목숨을 빼앗고 말았다.

이 모든 이야기의 요점은, 주어진 입력으로 `getStickSetting` 을 구현하는 제대로 된 방법이 없다는 것이다. `getStickSetting` 함수는 실패할 수밖에 없었다. 대부분의 비행기는 두 개의 스틱이 기계적으로 연결되어 있다. 부기장이 뒤로 당긴다면, 기장의 스틱도 뒤로 당겨진다. 기계적으로 연결된 스틱의 상태는 표현이 간단하다.

```jsx
interface CockpitControls {
	/** 스틱의 각도, 0 = 중립, + = 앞으로 */
	stickAngle: number;
}
```

이제는 순서도가 분명해졌다. 사실 `getStickSetting` 함수는 전혀 필요없다.

타입을 설계할 때는 어떤 값을들 포함하고 어떤 값들을 제외할지 신중하게 생각해야 한다. 유효한 상태를 표현하는 값만 허용한다면 코드를 작성하기 쉬워지고 타입 체크가 용이해진다 유효한 상태만 허용하는 것은 매우 일반적인 원칙이다. 반면 특정한 상황에서 지켜야 할 원칙들도 있는데 이후 내용에서 다루도록 한다.
