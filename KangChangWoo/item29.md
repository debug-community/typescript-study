# Item 29

## item.29 사용할 때는 너그럽게, 생성할 때는 엄격하게

요약

- 보통 매개변수 타입은 반환 타입에 비해 범위가 넓은 경향이 있다. 선택적 속성과 유니온 타입은 반환 타입보다 매개변수 타입에 더 일반적이다.
- 매개변수와 반환 타입의 재사용을 위해서 기본 형태(반환 타입)와 느슨한 형태(매개변수 타입)를 도입하는 것이 좋다.

---

> TCP 구현체는 견고성의 일반적 원칙을 따라야 한다. 당신의 작업은 엄격하게 하고, 다른 사람의 작업은 너그럽게 받아들여야 한다.
> 

함수의 시그니처에도 비슷한 규칙을 적용해야 한다. 함수의 매개변수는 타입의 범위가 넓어도 되지만, 결과를 반환할 때는 일반적으로 타입의 범위가 더 구체적이어야 한다.

예를 들어 3D 매핑 API는 카메라의 위치를 지정하고 경계 박스의 뷰포트를 계산하는 방법을 제공한다.

```jsx
declare function setCamera(camera: CameraOptions): void;
declare function viewportForBounds(bounds: LngLatBounds): CameraOptions;
```

카메라의 위치를 잡기 위해 `viewportForBounds` 의 결과가 `setCamera` 로 바로 전달될 수 있다면 편리할 것이다.

`CameraOptions` 와 `LngLat` 타입의 정의를 살펴보자

```jsx
interface CameraOptions {
	center?: LngLat;
	zoom?: number;
	bearing?: number;
	pitch?: number;
}

type LngLat =
{ lng: number; lat: number; } |
{ lon: number; lat: number; } |
[number, number];
```

일부 값은 건드리지 않으면서 동시에 다른 값을 설정할 수 있어야 하므로 `CameraOptions` 의 필드는 모두 선택적이다. 유사하게 `LngLat` 타입도 `setCamera` 의 매개변수 범위를 넓혀 준다. 매개변수로 `{lng, lat}` 객체, `{lon, lat}` 객체, 또는 순서만 맞다면 `[lng, lat]` 쌍을 넣을 수도 있다. 이러한 편의성을 제공하여 함수 호출을 쉽게 할 수 있다.

`viewportForBounds` 함수는 또 다른 자유로운 타입을 매개변수로 받는다.

```jsx
type LngLatBounds = 
{ northeast: LngLat, southwest: LngLat } |
[ LngLat, LngLat ] |
[ number, number, number, number ];
```

이름이 주어진 모서리, 위도/경도 쌍, 또는 순서만 맞다면 4-튜플을 사용하여 경계를 지정할 수 있다. `LngLat` 는 세 가지 형태를 받을 수 있기 때문에, `LngLatBounds` 의 가능한 형태는 19가지 이상으로 매우 자유로운 타입이다.

이제 `GeoJSON` 기능을 지원하도록 뷰포트를 조절하고, 새 뷰포트를 URL에 저장하는 함수를 작성해보자.

```jsx
function focusOnFeature(f: Feature) {
	const bounds = calculatedBoundingBox(f);
	const camera = viewportForBounds(bounds);
	setCamera(camera);
	const { center: {lat, lng}, zoom } = camera;
	// ~~~ ... 형식에 'lat' 속성이 없습니다.
	// ~~~ ... 형식에 'lng' 속성이 없습니다.
	zoom; // 타입이 number | undefined
	window.location.search = `?v=@${lat},${lng}z${zoom}`;
}
```

이 예제의 오류는 `lat` 과 `lng` 속성이 없고 `zoom` 속성만 존재하기 때문에 발생했지만, 타입이 `number | undefined` 로 추론되는 것 역시 문제이다. 근본적인 문제는 `viewportForBounds` 의 타입 선언이 사용될 때뿐만 아니라 만들어질 때에도 너무 자유롭다는 것이다.

`camera` 값을 안전한 타입으로 사용하는 유일한 방법은 유니온 타입의 각 요소별로 코드를 분기하는 것이다.

수 많은 선택적 속성을 가지는 반환 타입과 유니온 타입은 `viewportForBounds` 를 사용하기 어렵게 만든다. 매개변수 타입의 범위가 넓으면 사용하기 편리하지만, 반환 타입의 범위가 넓으면 불편하다. 즉, 사용하기 편리한 API일수록 반환 타입이 엄격하다.

유니온 타입의 요소별 분기를 위한 한 가지 방법은, 좌표를 위한 기본 형식을 구분하는 것이다. 배열과 배열 같은 것(array-like)의 구분을 위해 자바스크립트의 관례에 따라, `LngLat` 와 `LngLatLike` 를 구분할 수 있다. 또한 `setCamera` 함수가 매개변수로 받을 수 있도록, 완전하게 정의된 `Camera` 타입과 `Camera` 타입이 부분적으로 정의된 버전을 구분할 수도 있다.

```jsx
interface LngLat = { lng: number; lat: number; };
type LngLatLike = LngLat | { lon: number; lat: number; } |
									[number, number];
									
interface Camera {
	center: LngLat;
	zoom: number;
	bearing: number;
	pitch: number;
}

interface CameraOptions extends Omit<Partial<Camera>, 'center'> {
	center?: LngLatLike;
}

type LngLatBounds = 
	{ northease: LngLatLike, southwest: LngLatLike } |
	[LngLatLike, LngLatLike] |
	[number, number, number, number];
	
declare function setCamera(camera: CameraOptions): void;
declare function viewportForBounds(bounds: LngLatBounds): Camera;
```

`Camera` 가 너무 엄격하므로 조건을 완하하여 느슨한 `CameraOptions` 타입으로 만들었다.

`setCamera` 매개변수 타입의 `center` 속성에 `LngLatLike` 객체를 허용해야 하기 때문에 `Partial<Camera>` 를 사용하면 코드가 동작하지 않는다. 그리고 `LngLatLike` 가 `LngLat` 의 부분 집합이 아니라 상위 집합이기 때문에 `CameraOptions extends Partial<Camera>` 를 사용할 수 없다. 너무 복잡해 보인다면 약간의 반복 작업을 해야겠지만 명시적으로 타입을 추출해서 다음처럼 작성할 수도 있다.

```jsx
interface CameraOptions {
	center?: LngLatLike;
	zoom?: number;
	bearing?: number;
	pitch?: number;
}
```

앞에서 설명한 `CameraOptions` 를 선언하는 두 가지 방식 모두 `focusOnFeature` 함수가 타입 체커를 통과할 수 있게 한다.

```jsx
function focusOnFeature(f: Feature) {
	const bounds: calculateBoundingBox(f);
	const camera = viewportForBounds(bounds);
	setCamera(camera);
	const {center: {lat, lng}, zoom} = camera; // 정상
	zoom; // 타입이 number
	window.location.search = `?v=@${lat},${lng}z${zoom}`;
}
```

이번에는 `zoom` 의 타입이 `number | undefined` 가 아니라 `number`  이다. 이제 `viewportForBounds` 함수를 사용하기 훨씬 쉬워졌다. 그리고 `bounds` 를 생성하는 다른 함수가 있다면 `LngLatBounds` 와 `LngLatBoundsLike` 를 구분할 수 있도록 새로운 기본 형식을 도입해야 한다.

앞에 등장한 것처럼 경계 박스의 형태를 19가지나 허용하는 것은 좋은 설계가 아니다. 그러나 다양한 타입을 허용해야만 하는 라이브러리의 타입 선언을 작성하고 있다면, 어쩔 수 없이 다양한 타입을 허용해야 하는 경우가 생긴다. 하지만 그때도 19가지 반환 타입이 나쁜 설계라는 것을 잊지 말자.
