## 계층형 설계 패턴

1. 직접 구현: 적절한 구체화 수준을 통한 계층형 설계
2. 추상화 벽: 중요한 세부 구현을 감추고 인터페이스로 제공
3. 작은 인터페이스: 비즈니스 개념을 나타내는 중요한 인터페이스를 작고 강력한 동작으로 구성
4. 편리한 계층: 소프트웨어를 고품질로 제공하는 데 도움이 되는 계층에 시간을 투자

## 패턴 2: 추상화 벽

> [!NOTE]
> 추상화 벽은 여러 가지 문제를 해결합니다. 그 중 하나는 **팀 간 책임을 명확하게 나누는 것**입니다.

- 세부 구현을 감춘 함수로 이루어진 계층
- 추상화 벽에 있는 함수를 사용할 때는 데이터 구조, 구현을 전혀 몰라도 사용 가능
- 세부적인 것을 감추는 것은 대칭적
  - 만들고 쓰는 대상 간 구현하는 코드를 신경쓰지 않아도 됨
  - 두 대상이 독립적으로 일할 수 있음
  - 라이브러리나 API를 만들고 사용하는 사람 간의 세부적인 구현 형태를 신경쓰지 않아도 됨

### 장바구니 데이터 구조 바꾸기

#### 원래 코드

```tsx
const add_item = (cart: Cart, item: Item) =>
	add_element_last(cart, item)

const calc_total = (cart: Cart) =>
	cart.reduce((total, item) => total + item.price, 0)

const setPriceByName = (cart: Cart, name: string, price: number) =>
	cart.map(item => item.name === name ? setPrice(item, price) : item

const indexOfItem = (cart: Cart, name: string) => {
  const index = cart.findIndex(item => item.name === 'name')
  return index !== -1 ? index : null
}

const remove_item_by_name = (cart: Cart, name: string) => {
	const indexOfItem(cart, name)

	if (idx !== null) removeItems(cart, idx, 1)
}

const isInCart = (cart: Cart, name: string) =>
	indexOfItem(cart, name) !== null
```

- 배열 → 객체(해시맵)
- 장바구니를 자바스크립트 객체로 만드는 게 더 효율적이고 직접 구현 패턴에도 더 가까움

#### 장바구니를 객체로 다시 만들기

```tsx
const add_item = (cart: Cart, item: Item) => objectSet(cart, item.name, item)

const calc_total = (cart: Cart) => {
  const total = Object.values(cart).reduce((total, item) => total + item.price, 0)

  return total
}

const setPriceByName = (cart: Cart, name: string, price: number) => {
  if (isInCart(cart, name)) {
    return objectSet(cart, name, { ...cart[name], price })
  } else {
    const newItem = make_item(name, price)
    return objectSet(cart, name, newItem)
  }
}

const remove_item_by_name = (cart: Cart, name: string) => objectDelete(cart, name)

const isInCart = (cart: Cart, name: string) => Object.hasOwn(cart, name) // ES2022
```

### 추상화 벽이 있으면 구체적인 것을 신경쓰지 않아도 됩니다

> **장바구니 동작을 쓰는 함수를 전부 바꾸지 않고 어떻게 데이터 구조를 바꿀 수 있었나요?**

- 바꾼 함수가 추상화 벽에 있는 함수이기 때문
- 추상화 벽 → “어떤 것을 신경쓰지 않아도 되지?”
- 계층 구조에서 어떤 계층에 있는 함수들이 장바구니와 같이 공통된 개념을 신경쓰지 않아도 된다면 그 계층을 추상화 벽이라고 할 수 있음
- 추상화 벽 위에 있는 함수가 데이터 구조를 몰라도 된다는 것
- 즉, 추상화 벽에 있는 함수만 사용하면 되고 장바구니 구현에 대해서는 신경쓰지 않아도 됨
- 완전하지 않은 추상화 벽을 완전한 추상화 벽으로 만드는 방법은 추상화 벽에 새러운 함수를 만드는 것

> **추상화 벽은 언제 사용하면 좋을까요?**

- 쉽게 구현을 바꾸기 위해
- 코드를 읽고 쓰기 쉽게 만들기 위해
- 팀 간에 조율해야 할 것을 줄이기 위해
- 주어진 문제에 집중하기 위해

### 추상화 벽 리뷰

> **신경쓰지 않아도 되는 것을 다루는 것이 추상화 벽의 핵심이다.**

- 코드의 의존성을 없앨 수 있는 패턴
- 추상화 벽 위에 있는 코드는 구체적인 내용을 신경 쓰지 않아도 됨
- 추상화 단계의 상위에 있는 코드와 하위에 있는 코드는 서로 의존하지 않게 정의
- 추상화 벽으로 추상화를 강력하고 명시적으로 만들 수 있다.
- 팀 간의 커뮤니케이션 비용을 줄일 수 있으며,
- 일반적으로 여러 구체화 수준이 섞일 일이 없기 때문에 좋은 코드

## 패턴 3: 작은 인터페이스

### 추가 비즈니스 코드를 위한 두 가지 방법

#### 추상화 벽에 구현하는 방법

- 추상화 벽 계층에 있으면 해시 맵 데이터 구조로 되어 있는 장바구니에 접근할 수 있음
- 같은 계층에 있는 함수를 사용할 수 없음 (`calc_total()`, `isInCart()` 등)

```jsx
const getsWatchDiscount = (cart: Cart) {
	const names = Object.keys(cart);
	const total = names.reduce((total, item) => total + item.price, 0)

	return total > 100 && Object.hasOwn(cart, "watch");
}
```

#### 추상화 벽 위에 있는 계층에 구현하는 방법

- 추상화 벽 위에 만들면 해시 데이터 구조를 직접 접근할 수 없음
- 추상화 벽에 있는 함수를 사용해서 장바구니에 접근해야 함

```jsx
const getsWatchDiscount = (cart: Cart) {
	const total = calcTotal(cart)
	const hasWatch = isInCart("watch")

	return total > 100 && hasWatch
}
```

- 첫 번째 방법 → 시스템 하위 계층 코드가 늘어나기 때문에 좋지 않음
- 두 번째 방법 → 직접 구현에 가깝기 때문

> 첫 번째 방법을 사용해도 추상화 벽을 유지하지 못하는 것은 아니지만, 해당 방법은 마케팅팀이 구체적인 구현에 신경 쓸 수밖에 없게 만들고 있다.
> 추상화 벽에 만드는 함수는 개발팀과 마케팅팀 사이에 계약이라고 할 수 있는데, 추상화 벽에 새로운 함수가 생긴다면 이건 계약이 늘어나는 것으로 볼 수 있다.

### 장바구니에 제품을 담을 때 로그 추가하기

#### 데이터베이스에 로그 기록하기

```tsx
logAddToCart(user_id, item) // 액션
```

**`add_item()` 함수에서 호출하기 (추상화 벽)**

```tsx
const add_item = (cart: Cart, item: Item) => {
  logAddToCart(global_user_id, item)
  return objectSet(cart, item.name, item)
}
```

- 액션을 부르기 때문에 기껏 만든 계산 함수가 액션이 됨……..
- 액션이 되면서 테스트하기도 어려워짐
- `update_shipping_icons()` 함수 같이 꼭 장바구니에 제품을 담지 않아도 `add_item()` 함수를 사용함
  - 불필요한 로그가 남게 됨

#### `add_item_to_cart()` 함수에서 호출하기 (추상화 벽 위)

```tsx
const add_item_to_cart = ({ name, price }: Item) => {
  const item = make_cart_item(name, price)
  shopping_cart = add_item(shopping_cart, item)
  const total = calc_total(shopping_cart)
  set_cart_total_dom(total)
  update_shipping_icons(shopping_cart)
  update_tax_dom(total)

  logAddToCart() // 추가
}
```

- 사용자가 장바구니에 제품을 담을 때 해야 할 모든 일을 담는 핸들러 함수
  - 사용자가 장바구니에 제품을 담는 의도를 정확히 반영하는 위치!

### 작은 인터페이스 리뷰

> 함수의 목적에 맞는 계층이 어디인지 찾는 감각을 기르는 것이 가장 중요하다.

#### 추상화 벽을 작게 만들어야 하는 이유

- 추상화 벽에 코드가 많을수록 구현이 변경되었을 때 고쳐야 할 것이 많음
- 추상화 벽에 있는 코드는 낮은 수준의 코드이기 때문에 더 많은 버그가 있을 수 있음
- 낮은 수준의 코드는 이해하기 더 어려움
- 추상화 벽에 코드가 많을수록 팀 간 조율해야 할 것도 많아짐
- 추상화 벽에 인터페이스가 많으면 알아야 할 것이 많아 사용하기 어려움

## 패턴 4: 편리한 계층

- 편리한 계층 패턴은 언제 패턴을 적용하고 또 언제 멈춰야 하는지 실용적인 방법을 알려준다.
- 만약 작업하는 코드가 편리하다고 느낀다면 설계는 조금 멈추자. 구체적인 것을 너무 많이 알아야 하거나, 코드가 지저분하다고 느껴진다면(코드에서 나는 냄새!) 그때 다시 패턴을 적용하면 된다.
- 어떤 코드도 이상적인 모습에 도달하기는 어렵고, 언제나 설계와 새로운 기능의 필요성 사이 어느 지점에 머물게 된다. (🥹)

## 호출 그래프로 알 수 있는 비기능적 요구사항

### 유지보수성

- 위로 연결된 것이 적은 함수가 바꾸기 쉬움
- 자주 바뀌는 코드는 가능한 위쪽에 있어야 함

### 테스트성

- 위쪽으로 많이 연결된 함수를 테스트하는 것이 더 가치 있음
- 아래쪽에 있는 함수를 테스트하는 것이 위쪽에 있는 함수를 테스트하는 것보다 가치 있음

### 재사용성

- 아래쪽에 있는 함수가 적을수록 더 재사용하기 좋음
- 낮은 수준의 단계로 함수를 빼내면 재사용성이 더 높아짐
