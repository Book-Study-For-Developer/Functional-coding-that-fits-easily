## 소프트웨어 설계란 무엇입니까?

소프트웨어 설계

코드를 만들고, 테스트하고, 유지보수하기 쉬운 프로그래밍 방법을 선택하기 위해 미적 감각을 사용하는 것

## 계층형 설계란 무엇인가요?

소프트웨어를 계층으로 구성하는 기술

## 설계 감각을 키우기

- 입력
  - 함수 본문
    - 길이
    - 복잡성
    - 구체화 단계
    - 함수 호출
    - 프로그래밍 언어의 기능 사용
  - 계층 구조
    - 화살표 길이
    - 응집도
    - 구체화 단계
  - 함수 시그니처
    - 함수명
    - 인자 이름
    - 인잣값
    - 리턴값
- 출력
  - 조직화
    - 새로운 함수를 어디에 놓을 지 결정
    - 함수를 다른 곳으로 이동
  - 구현
    - 구현 바꾸기
    - 함수 호출하기
    - 데이터 구조 바꾸기
  - 변경
    - 새 코드를 작성할 곳 선택하기
    - 적절한 수준의 구체화 단계 결정하기

## 계층형 설계 패턴

- **패턴1. 직접 구현**
  직접 구현된 함수를 읽을 때, 함수 시그니처가 나나태고 있는 문제를 함수 본문에서 적절한 구체화 수준에서 해결
- **패턴2. 추상화 벽**
  계층형 설계를 할 때 어떤 계층에서 세부 구현을 감추고 인터페이스를 제공
- **패턴3. 작은 인터페이스**
  비즈니스 개념을 나타내는 중요한 인터페이스는 작고 강력한 동작으로 구성하는 것이 좋음
- **패턴4. 편리한 계층**
  소프트웨어를 더 빠르고 고품질로 제공하는데 도움이 되는 계층을 추가

## 패턴1. 직접 구현

넥타이 한개를 사면 넥타이 클립 한개를 무료로 주는 코드

```jsx
function freeTieClip(cart) {
  let hasTie = false;
  let hasTieClip = false;
  for (let i = 0; i < cart.length; i++) {
    let item = cart[i];
    if (item.name === "tie") hasTie = true;
    if (item.name === "tie clip") hasTieClip = true;
  }
  if (hasTie && !hasTieClip) {
    var tieClip = make_item("tie clip", 0);
    return add_item(cart, tieClip);
  }
  return cart;
}
```

한 함수안에 많은 기능이 있다. 장바구니를 돌면서 항목을 체크하고 무엇인가를 결정하고..

첫 번째 계층형 설계 패턴인 **직접 구현**을 따르지 않고 있다

freeTieClip함수가 알 필요가 없는 구체적인 내용을 담고 있음

### 장바구니 설계 개선

반복문 추출해 새로운 함수 만들기

```jsx
function freeTieClip(cart: Cart) {
  const hasTie = isInCart(cart, "tie");
  const hasTieClip = isInCart(cart, "tie clip");
  if (hasTie && !hasTieClip) {
    var tieClip = make_item("tie clip", 0);
    return add_item(cart, tieClip);
  }
  return cart;
}

// 제품이 있는지 확인
function isInCart(cart: Cart, name: string) {
  return cart.some((item) => item.name === name);
}
```

저수준의 반복문을 추출해야할 가능성이 높음!

**직접 구현 패턴을 사용하여 비슷한 추상화 계층에 있는 함수를 호출함**

`freeTieClip()`이 사용하는 모든 함수는 장바구니가 배열인지 몰라도 됨

**호출 그래프를 만들어 함수 호출을 시각화하기**

개선 전 다이어그램

![image](https://drek4537l1klr.cloudfront.net/normand/Figures/f0176-01.jpg)

개선 후 다이어그램

![image](https://drek4537l1klr.cloudfront.net/normand/Figures/f0176-02.jpg)

**같은 계층에 있는 함수는 같은 목적을 가져야 함**

![image](https://drek4537l1klr.cloudfront.net/normand/Figures/f0185-01.jpg)

어떤 계층에 있는 함수를 읽거나 고칠 때 낮은 수준의 구체적인 내용은 신경 쓰지 않아도 됨

## 3단계 줌 레벨

1. 전역 줌 레벨

   계층 사이의 상호 관계를 포함해서 모든 문제 영역을 살펴볼 수 있음

2. 계층 줌 레벨

   한 계층과 연결된 바로 아래 계층을 볼 수 있는 줌 레벨

3. 함수 줌 레벨

   함수와 바로 아래 연결된 함수들을 볼 수 있음

직접 구현 패턴을 사용하면 모든 화살표가 같은 길이를 가져야 하지만, 위 다이어 그램은 다양한 길이의 화살표를 가지고 있다

_🤔 어떻게 하면 화살표를 같은 길이로 만들 수 있을까?_

중간에 함수를 두는 것!

언어 기능을 사용하는 긴 화살표를 줄이고, 반복문을 빼내어 새로운 함수를 만든다

```jsx
// 개선 전
function remove_item_by_name(cart: Cart, name: string) {
  let idx = null;
  for (let i = 0; i < cart.length; i++) {
    if (cart[i].name === name) idx = i;
  }
  if (idx !== null) return removeItems(cart, idx, 1);
  return cart;
}

// 개선 후
function remove_item_by_name(cart: Cart, name: string) {
  let idx = indexOfItem(cart, name);
  if (idx !== null) return removeItems(cart, idx, 1);
  return cart;
}

function indexOfItem(cart: Cart, name: string) {
  for (let i = 0; i < cart.length; i++) {
    if (cart[i].name === name) return i;
  }
  return null;
}
```

![image](https://drek4537l1klr.cloudfront.net/normand/Figures/f0189-02.png)

읽기 쉬워졌고, 다이어그램에서도 보기 쉬워짐

## 직접 구현 패턴 리뷰

- 직접 구현한 코드는 한 단계의 구체화 수준에 관한 문제만 해결
- 계층형 설계는 특정 구체화 단계에 집중할 수 있게 도와줌
- 호출 그래프는 구체화 단계에 대한 풍부한 단서를 보여줌
- 함수를 추출하면 더 일반적인 함수로 만들 수 있음
- 일반적인 함수가 많을 수록 재사용하기 좋음
- 복잡성을 감추지 않음

계층형 설계에서 모든 계층은 바로 아래 계층에 의존해야한다!

## 요점 정리

- 계층형 설계는 코드를 추상화 계층으로 구성함. 각 계층을 볼 때 다른 계층에 구체적인 내용을 몰라도 됨
- 문제 해결을 위한 함수를 구현할 때 어떤 구체화 단계로 쓸 지 결정하는 것이 중요
- 함수 이름, 본문, 호출 그래프 등의 요소로 함수가 어떤 계층에 속할지 알 수 있음
- 함수 이름은 의도를 알려줌. 비슷한 목적의 이름을 가진 함수를 함께 묶을 수 있음
- 함수 본문은 중요한 세부 사항을 알려줌. 함수 본문은 함수가 어떤 계층 구조에 있어야 하는지 알려줌
- 호출 그래프로 구현이 직접적이지 않다는 것을 알 수 있음. 함수를 호출하는 화살표가 다양한 길이를 가지고 있다면 직접 구현되어 있지 않다는 신호
- 직접 구현 패턴은 함수를 명확하고 아름답게 구현해 계층을 구성할 수 있도록 알려줌

💬 연습 문제 부분은 이렇게 까지 해야 하는건가 싶다..! 저수준의 반복문은 추출하여 함수로 빼내고, 한 함수안에 비슷한 수준의 추상화 계층으로 구성하는 것은 좋은 것 같다. 평소에 코드 짤 때 흐릿하게 느끼고 있는 것들을 다이어그램을 통해 구체적으로 볼 수 있어서 좋았다. 이러한 설계 패턴 생각하면서 짜봐야지..
