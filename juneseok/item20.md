# 요약

- **하나의 변수에 여러 타입을 넣지 마세요.**
- 변수의 값은 바뀔 수 있지만, 일반적으로는 바뀌지 않습니다.
- 서로 목적이 다른 값은 **별도의 변수**로 분리하는 것이 훨씬 명확하고 안정적입니다.

# 왜 하나의 변수에 여러 타입을 쓰면 안 될까?

```tsx
let id = "12-34-56";        // string 타입
fetchProduct(id);

id = 123456;                // ❌ 오류: string 변수에 number는 안 됨
fetchProductBySerialNumber(id);
```

- 유니온 타입으로 해결하려면

```tsx
let id: string | number = "12-34-56";
id = 123456;
```

# 그래서 더 나은 선택: 변수 분리 + const 사용

> 별도의 변수명 덕분에 관련 없는 값은 분리
타입 추론이 더 정확해지며 cosnt로 쓰면 변경 불가능한 값이 되어 안전해진다.
> 

```tsx
const id = "12-34-56";
fetchProduct(id);

const serial = 123456;
fetchProductBySerialNumber(serial);
```

| 항목 | 설명 |
| --- | --- |
| 코드 가독성↑ | 변수의 목적이 명확 |
| 타입 안정성↑ | `string |
| 추론력 강화 | 타입 추론이 더 정밀 |
| `const` 사용 가능 | 변경 불가능하게 선언 가능 |

# 예시

```tsx
interface Product {
  id: string;
  serial: number;
}

function buyProduct(id: string) { /* ... */ }
function registerSerial(serial: number) { /* ... */ }

const id = "ABC-123";           // product ID
buyProduct(id);

const serial = 987654;          // serial number
registerSerial(serial);
```