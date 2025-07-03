# 요약

- 타입스크립트는 **타입 추론** 덕분에 코드가 짧고 깔끔해진다.
- **불필요한 타입 선언**은 오히려 방해가 된다.
- 이상적인 경우 함수/메서드의 시그니처에는 타입구문이 있지만, 함수 내 지역변수에는 타입 구문이 없다.
- 타입 추론을 믿고 (단, 예외 상황은 명확히 구분) 짧고 명확한 코드(유지보수, 생산성 모두 챙긴다)를 작성하자

# 타입 추론?

> 타입스크립트가 변수, 함수 등의 타입을 코드만 보고 자동으로 추론해주는 기능
> 

```tsx
let axis = 'x';   // axis: string
let axis2 = 'y' as const; // axis2: 'y'
```

# 명시적 타입 선언 vs 타입 추론

```tsx
// (1) 명시적 타입 선언 - 불필요하게 장황
let person1: {
  name: string;
  born: { where: string; when: string };
  died: { where: string; when: string };
} = {
  name: 'Sojourner Truth',
  born: { where: 'Swartekill, NY', when: 'c.1797' },
  died: { where: 'Battle Creek, MI', when: 'Nov. 26, 1883' }
};

// (2) 타입 추론 활용 - 더 간결하고 리팩토링에 유리
let person2 = {
  name: 'Sojourner Truth',
  born: { where: 'Swartekill, NY', when: 'c.1797' },
  died: { where: 'Battle Creek, MI', when: 'Nov. 26, 1883' }
};

```

# 타입 추론을 활용하면 좋은점

- 코드가 짧아지고 가독성↑
- 리팩토링할 때 한 곳만 수정하면 됨
- 타입 선언 실수로 인한 에러 감소
- 불필요한 타입 구문 줄여서 생산성↑

# 타입을 명시적으로 선언 해야 하는 경우

- **객체 리터럴 할당**
    
    → 타입을 명시하면 “잉여 속성 체크”가 동작해서 잘못된 값 할당을 막아줌
    
- **함수 반환 타입**
    
    → 함수의 시그니처를 명확하게 하고, 호출부로 오류 전파 방지
    
    → 명명된 타입을 사용할 수 있음
    
- **타입 추론이 불가능한 상황**
    
    → 정보가 부족할 때는 명시 필요
    

# 예시: interface 리팩토링

```tsx
interface Product {
    id: number;
    name: string;
    price: number;
}

function logProduct(product: Product) {
    // 굳이 각 변수에 타입 선언 필요 없음!
    const id = product.id;
    const name = product.name;
    const price = product.price;
    console.log(id, name, price);
}

```