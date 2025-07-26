# Item 33

## item.33 string 타입보다 더 구체적인 타입 사용하기

요약

- ‘문자열을 남발하여 선언된’ 코드를 피하자. 모든 문자열을 할당할 수 있는 `string` 타입보다는 더 구체적인 타입을 사용하는 것이 좋다.
- 변수의 범위를 보다 정확하게 표현하고 싶다면 `string` 타입보다는 문자열 리터럴 타입의 유니온을 사용하면 된다. 타입 체크를 더 엄격히 할 수 있고 생산성을 향상시킬 수 있다.
- 객체의 속성 이름을 함수 매개변수로 받을 때는 `string` 보다 `keyof T` 를 사용하는 것이 좋다.

---

`string` 타입의 범위는 매우 넓다. `"x"` 나 `"y"` 같은 한 글자도, ‘모비 딕’ 의 전체 내용도 `string` 타입이다. `string` 타입으로 변수를 선언하려 한다면, 혹시 그보다 더 좁은 타입이 적절하지는 않을지 검토해 보아야 한다.

음악 컬렉션을 만들기 위해 앨범의 타입을 정의한다고 가정해보자.

```jsx
interface Album {
	artist: string;
	title: string;
	releaseDate: string; // YYYY-MM-DD
	recordingType: string; // 예를 들어, "live" 또는 "studio"
}
```

`string` 타입이 남발된 모습이다. 게다가 주석에 타입 정보를 적어 둔 걸 보면, 현재 인터페이스가 잘못되었다는 것을 알 수 있다. 다음 예시처럼 `Album` 타입에 엉뚱한 값을 설정할 수 있다.

```jsx
const kindOfBlue: Album = {
	artist: 'Miles Davis',
	title: 'Kind of Blue',
	releaseDate: 'August 17th, 1959', // 날짜 형식이 다릅니다
	recordingType: 'Studio', // 오타(대문자 S)
}; // 정상
```

`releaseDate` 필드의 값은 주석에 설명된 형식과 다르며, recordingType 필드의 값 `"Studio"` 는 소문자 대신 대문자가 쓰였다. 그러나 이 두 값 모두 문자열이고, 해당 객체는 `Album` 타입에 할당 가능하며 타입 체커를 통과한다.

또한 `string` 타입의 범위가 넓기 때문에 제대로 된 `Album` 객체를 사용하더라도 매개변수 순서가 잘못된 것이 오류로 드러나지 않는다.

```jsx
function recordRelease(title: string, data: string) { /* ... */ }
recordRelease(kindOfBlue.releaseData, kindOfBlue.title); // 오류여야 하지만 정상
```

`recordRelease` 함수의 호출에서 매개변수들의 순서가 바뀌었지만, 둘 다 문자열이기 때문에 타입 체커가 정상으로 인식한다. 앞의 예제처럼 `string` 타입이 남용된 코드를 “문자열을 남발하여 선언되었다(stringly typed)” 고 표현하기도 한다.

앞의 오류를 방지하기 위해 타입의 범위를 좁히는 방법을 생각해보자. 억지스러운 예시일 수 있지만, 가수 이름이나 앨범 제목으로 ‘모비 딕’의 전체 텍스트가 쓰일 수도 있다. 그러므로 `artist` 나 `title` 같은 필드에는 `string` 타입이 적절하다. 그러나 `releaseData` 필드는 `Date` 객체를 사용해서 날짜 형식으로만 제한하는 것이 좋다.

`recordingType` 필드는 `"live"` 와 `"studio"` , 단 두 개의 값으로 유니온 타입을 정의할 수 있다.(`enum` 을 사용할 수도 있지만 일반적으로는 추천하지 않는다.)

```jsx
type RecordingType = 'studio' | 'live';

interface Album {
	artist: string;
	title: string;
	releaseData: Data;
	recordingType: RecordingType;
}
```

앞의 코드처럼 바꾸면 타입스크립트는 오류를 더 세밀하게 체크한다.

```jsx
const kindOfBlue: Album = {
	artist: 'Miles Davis',
	title: 'Kind of Blue',
	releaseData: new Date('1959-08-17'),
	recordingType: 'Studio'
	// ~~~~~~~~~~ '"Studio"' 형식은 'RecordingType' 형식에 할당할 수 없습니다.
}
```

이러한 방식에는 세 가지 장점이 더 있다.

첫 번째, 타입을 명시적으로 정의함으로써 다른 곳으로 값이 전달되어도 타입 정보가 유지된다. 예를 들어, 특정 레코딩 타입의 앨범을 찾는 함수를 작성한다면 다음처럼 정의할 수 있다.

```jsx
function getAlbumsOfType(recordingType: string): Album[] {
	// ...
}
```

`getAlbumsOfType` 함수를 호출하는 곳에서 `recordingType` 의 값이 `string` 타입이어야 한다는 것 외에는 다른 정보가 없다. 주석으로 써놓은 `"studio"` 또는 `"live"` 는 `Album` 의 정의에 숨어 있고, 함수를 사용하는 사람은 `recordingType` 이 `"studio"` 또는 `"live"` 여야 한다는 것을 알 수 없다.

두 번째, 타입을 명시적으로 정의하고 해당 타입의 의미를 설명하는 주석을 붙여 넣을 수 있다.

```jsx
/** 이 녹음은 어떤 환경에서 이루어졌는지? */
type RecordingType = 'live' | 'studio';
```

`getAlbumsOfType` 이 받는 매개변수를 `string` 대신 `RecordingType` 으로 바꾸면, 함수를 사용하는 곳에서 `RecordingType` 의 설명을 볼 수 있다. 편집기에서 `RecordingType` 에 마우스 포인터를 올려보면 `string` 대신 명명된 타입을 확인할 수 있고, 주석도 확인할 수 있다.

세 번째, `keyof` 연산자로 더욱 세밀하게 객체의 속성 체크가 가능해진다.

함수의 매개변수에 `string` 을 잘못 사용하는 일은 흔하다. 어떤 배열에서 한 필드의 값만 추출하는 함수를 작성한다고 생각해보자. 실제로 언더스코어(Underscore) 라이브러리에는 `pluck` 이라는 함수가 있다.

```jsx
function pluck(records: any[], key: string): any[] {
	return records.map(r => r[key]);
}
```

타입 체크가 되긴 하지만 `any` 타입이 있어서 정밀하지 못하다. 특히 반환 값에 `any` 를 사용하는 것은 매우 좋지 않은 설계이다. 먼저, 타입 시그니처를 개선하는 첫 단계로 제너릭 타입을 도입해보자

```jsx
function plcuk<T>(records: T[], key: string): any[] {
	return records.map(r => r[key]);
											// ~~~~~~~ '{}' 형식에 인덱스 시그니처가 없으므로
											// ~~~~~~~ 요소에 암시적으로 'any' 형식이 있습니다.
}
```

이제 타입스크립트는 `key` 의 타입이 `string` 이기 때문에 범위가 너무 넓다는 오류를 발생시킨다. `Album` 의 배열을 매개변수로 전달하면 기존의 `string` 타입의 넓은 범위와 반대로, `key` 는 단 네 개의 값(`"artist", "title", "releaseData", recordingType"` )만이 유효하다.

다음 예시는 `keyof Album` 타입으로 얻게 되는 결과이다.

```jsx
type K = keyof Album;
// 타입이 "artist" | "title" | "releaseData" | "recordingType"
```

그러므로 `string` 을 `keyof T` 로 바꾸면 된다.

```jsx
function pluck<T>(records: T[], key: keyof T) {
	return records.map(r => r[key]);
}
```

이 코드는 타입 체커를 통과한다. 또한 타입스크립트가 반환 타입을 추론할 수 있게 해 준다. `pluck` 함수에 마우스를 올려 보면, 추론된 타입을 알 수 있다.

```jsx
function pluck<T>(records: T[], key: keyof T): T[keyof T][]
```

`T[keyof T]` 는 `T` 객체 내의 가능한 모든 값의 타입이다.

그런데 `key` 의 값으로 하나의 문자열을 넣게 되면, 그 범위가 너무 넓어서 적절한 타입이라고 보기 어렵다. 예를 들어보자.

```jsx
// 타입이 (string | Date)[]
const releaseDates = pluck(albums, 'releaseData');
```

`releaseDates` 의 타입은 `(string | Date)[]` 가 아니라 `Date[]` 이어야 한다. `keyof T` 는 `string` 에 비하면 훨씬 범위가 좁기는 하지만 그래도 여전히 넓다. 따라서 범위를 더 좁히기 위해서, `keyof T` 의 부분 집합(아마도 단일 값)으로 두 번째 제너릭 매개변수를 도입해야 한다.

```jsx
function pluck<T, K extends keyof T>(records: T[], key: K): T[k][] {
	return records.map(r => r[key]);
}
```

이제 타입 시그니처가 완벽해졌다. `pluck` 을 여러 가지 방법으로 호출하면서 제대로 반환 타입이 추론되는지, 무효한 매개변수를 방지할 수 있는지 확인해 볼 수 있다.

```jsx
pluck(albums, 'releaseData'); // 타입이 Date[]
pluck(albums, 'artist'); // 타입이 string[]
pluck(albums, 'recordingType'); // 타입이 RecordingType[]
pluck(albums, 'recordingDate');
						// ~~~~~~~~~~~~~~ '"recordingDate"' 형식의 인수는
						// ... 형식의 매개변수에 할당될 수 없습니다.
```

매개변수 타입이 정밀해진 덕분에 언어 서비스는 `Album` 의 키에 자동완성 기능을 제공할 수 있게 해준다.

`string` 은 `any` 와 비슷한 문제를 가지고 있다. 따라서 잘못 사용하게 되면 무효한 값을 허용하고 타입 간의 관계도 감추어 버린다. 이러한 문제점은 타입 체커를 방해하고 실제 버그를 찾지 못하게 만든다. 타입스크립트에서 `string` 의 부분 집합을 정의할 수 있는 기능은 자바스크립트 코드에 타입 안정성을 코게 높인다. 보다 정확한 타입을 사용하면 오류를 방지하고 코드의 가독성도 향상시킬 수 있다.
