# Item 20

## item.20 다른 타입에는 다른 변수 사용하기

요약

- 변수의 값은 바뀔 수 있지만 타입은 일반적으로 바뀌지 않는다.
- 혼란을 막기 위해 타입이 다른 값을 다룰 때에는 변수를 재사용하지 않도록 한다.

---

자바스크립트에서는 한 변수를 다른 목적을 가지는 다른 타입으로 재사용해도 된다.

```jsx
let id = "12-34-56";
fetchProduct(id); // string으로 사용
id = 123456;
fetchProductBySerialNumber(id); // number로 사용
```

반면 타입스크립트에서는 두 가지 오류가 발생한다.

```jsx
let id = "12-34-56";
fetchProduct(id);

id = 123456;
// ~~ '123456' 형식은 'string' 형식에 할당할 수 없습니다.
fetchProductBySerialNumber(id);
// ~~ 'string' 형식의 인수는
// ~~ 'number' 형식의 매개변수에 할당될 수 없습니다.
```

편집기에서 첫 번째 `id` 위에 마우스 커서를 올려 놓으면 무엇이 문제인지 알 수 있다. 타입스크립트는 `"12-34-56"` 이라는 값을 보고, `id` 의 타입을 `string` 으로 추론했다. `string` 타입에는 `number` 타입을 할당할 수 없기 때문에 오류가 발생한다.

여기서 “변수의 값은 바뀔 수 있지만 그 타입은 보통 바뀌지 않는다.” 는 중요한 관점을 알 수 있다. 타입을 바꿀 수 있는 한 가지 방법은 범위를 좁히는 것인데(타입 좁히기), 새로운 변수값을 포함하도록 확장하는 것이 아니라 타입을 더 작게 제한하는 것이다. 이 관점에 반하는 타입 지정 방법이 있는데, 이 방법은 어디까지나 예외이지 규칙은 아니다.

이제 오류가 발생한 앞의 예제를 고쳐보자. `id` 의 타입을 바꾸지 않으려면, `string` 과 `number` 를 모두 포함할 수 있도록 타입을 확장하면 된다. `string | number` 로 표현하며, 유니온(union) 타입이라고 한다.

```jsx
let id: string | number = "12-34-56";
fetchProduct(id);
id = 123456; // 정상
fetchProductBySerialNumber(id); // 정상
```

타입스크립트는 첫 번째 함수 호출에서 `id` 는 `string` 으로, 두 번째 호출에서는 `number` 라고 제대로 판단한다. 할당문에서 유니온 타입으로 범위가 좁혀졌기 때문이다.

유니온 타입으로 코드가 동작하기는 하겠지만 더 많은 문제가 생길 수 있다. `id` 를 사용할 때마다 값이 어떤 타입인지 확인해야 하기 때문에 유니온 타입은 `string` 이나 `number` 같은 간단한 타입에 비해 다루기 더 어렵다.

차라리 별도의 변수를 도입하는 것이 낫다.

```jsx
const id = "12-34-56";
fetchProduct(id);

const serial = 123456; // 정상
fetchProductBySerialNumber(serial); // 정상
```

앞의 예제에서 첫 번째 `id` 와 재사용한 두 번째 `id` 는 서로 관련이 없었다. 그냥 변수를 재사용했을 뿐이다. 변수를 무분별하게 재사용하면 타입 체커와 사람 모두에게 혼란을 줄 뿐이다.

다른 타입에는 별도의 변수를 사용하는 게 바람직한 이유는 다음과 같다.

- 서로 관련이 없는 두 개의 값을 분리한다. (`id` 와 `serial` )
- 변수명을 더 구체적으로 지을 수 있다.
- 타입 추론을 향상시키며, 타입 구문이 불필요해진다.
- 타입이 좀 더 간결해진다. (`string | number` 대신 `string` 과 `number` 사용)
- `let` 대신 `const` 로 변수를 선언하게 된다. `const` 로 변수를 선언하면 코드가 간결해지고, 타입 체커가 타입을 추론하기에도 좋다.

타입이 바뀌는 변수는 되도록 피해야 하며, 목적이 다른 곳에는 별도의 변수명을 사용해야 한다.

그런데 지금까지 이야기한 재사용되는 변수와, 다음 예제에 나오는 ‘가려지는(shadowed)’ 변수를 혼동해선 안된다.

```jsx
const id = "12-34-56";
fetchProduct(id);

{
	const id = 123456; // 정상
	fetchProductBySerialNumber(id); // 정상
}
```

여기서 두 `id` 는 이름이 같지만 실제로는 서로 아무런 관계가 없다. 그러므로 각기 다른 타입으로 사용되어도 문제없다. 그런데 동일한 변수명에 타입이 다르다면, 타입스크립트 코드는 잘 동작할지 몰라도 다른 사람에게 혼란을 줄 수 있다.

목적이 다른 곳에는 별도의 변수명을 사용하는 것이 권장된다. 많은 개발팀이 린터 규칙을 통해 ‘가려지는’ 변수를 사용하지 못하도록 하고 있다.
