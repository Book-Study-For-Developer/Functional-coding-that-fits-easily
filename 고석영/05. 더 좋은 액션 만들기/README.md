## 비즈니스 요구 사항과 설계를 맞추기

### 요구 사항에 맞춰 더 나은 추상화 단계 선택하기

#### 요구사항에 맞지 않는 코드

- 요구 사항: 장바구니에 담긴 제품을 주문할 때 무료 배송인지 확인하는 것
- 개선점: 장바구니로 무료 배송을 확인하지 않고 제품의 합계와 가격으로 확인
- 코드 개선 사항: 비즈니스 로직에 맞게 장바구니를 받는 함수로 변경해야 함

```ts
const gets_free_shipping = (total: number, item_price: number): boolean => item_price + total >= 20
```

#### 코드의 냄새: 중복된 코드

- 개선점: 합계에 제품의 가격을 더하는 코드가 중복되고 있음
- 코드 개선 사항: 중복되는 코드를 제거해야 함

```ts
const calc_total = (cart: Item[]): number => cart.reduce((total, item) => total + item.price, 0)
```

### 코드 개선 사항 반영하기

- 비즈니스 로직에 맞게 `get_free_shipping()`의 인자를 `total, item_price`에서 `cart`로 변경
- `calc_total()`을 재사용하여 중복되지 않게 변경

```ts
const gets_free_shipping = (cart: Item[]): boolean => calc_total(cart) >= 20
```

## 비즈니스 요구 사항과 함수를 맞추기

- 비즈니스 요구 사항에 맞는 엔티티 타입인 장바구니 데이터를 사용한 함수로 변경
- 변경된 함수 시그니처에 맞춰 함수 호출 부분 변경

```ts
const update_shipping_icons = () => {
  const buy_buttons = get_buy_buttons_dom()
  buy_buttons.forEach(buy_button => {
    const { item } = buy_button
    const new_cart = add_item(shopping_cart, item)
    const is_free_shipping_available = gets_free_shipping(new_cart)

    if (is_free_shipping_available) buy_button.show_free_shipping_icon()
    else buy_button.hide_free_shipping_icon()
  })
}
```

## 원칙 1: 암묵적 입력과 출력은 적을수록 좋습니다

- 암묵적 입출력이 있는 함수 -> 다른 컴포넌트와 강하게 연결된 컴포넌트
- 다른 곳에서 사용할 수 없기 때문에 모듈 x -> 명시적 입출력으로 바꿔 모듈화된 컴포넌트로 변경 가능

> **납땜하는 대신 쉽게 떼었다 붙일 수 있는 커넥터로 연결된 형태**

### 암묵적 입력과 출력 줄이기

- 암묵적 입력을 명시적 입력인 인자로 변경
- 함수 시그니처가 변경됐다면 함수 호출 부분도 변경

```ts
const update_shipping_icons = (cart: Item[]) => {
  const buy_buttons = get_buy_buttons_dom()
  buy_buttons.forEach(buy_button => {
    const { item } = buy_button
    const new_cart = add_item(cart, item)
    const is_free_shipping_available = gets_free_shipping(new_cart)

    if (is_free_shipping_available) buy_button.show_free_shipping_icon()
    else buy_button.hide_free_shipping_icon()
  })
}

const calc_cart_total = () => {
  shopping_cart_total = calc_total(shopping_cart)
  set_cart_total_dom()
  update_shipping_icons(shopping_cart)
  update_tax_dom()
}
```

> [!NOTE]
>
> - 함수가 더 좋아졌나?
>   - 암묵적 입출력이 없애 외부 요인 의존도를 낮추고 비즈니스 로직에 맞게 인자를 변경해 함수의 역할이 명확해졌다.
> - 다양한 환경에서 재사용할 수 있나?
>   - 암묵적인 입출력이 없으므로 전보다는 재사용하기 용이해졌지만 다양한 환경에서 사용하기는 아직이지 않은지...
> - 테스트하기 쉬워졌나?
>   - 입출력이 명시적이므로 기존보다 테스트하기 쉽다.

### 코드 다시 살펴보기

- `shopping_cart_total` 전역변수와 `calc_cart_total()` 함수는 없애고
- `add_item_to_cart()`에 없앤 `calc_cart_total()` 로직을 옮김

```ts
const add_item_to_cart = (item: Item) => {
  shopping_cart = add_item(shopping_cart, item)

  const total = calc_total(shopping_cart)
  set_cart_total_dom(total)
  update_shipping_icons(shopping_cart)
  update_tax_dom(total)
}
```

### 계산 분류하기

| 동작             | 표시 방법 |
| ---------------- | :-------: |
| cart에 대한 동작 |    `C`    |
| item에 대한 동작 |    `I`    |
| 비즈니스 규칙    |    `B`    |

```ts
// C, I
const add_item = (cart: Item[], item: Item): Item[] => [...cart, item]

// C, I, B
const calc_total = (cart: Item[]): number => cart.reduce((total, item) => total + item.price, 0)

// B
const gets_free_shipping = (cart: Item[]): boolean => calc_total(cart) >= 20

// B
const calc_tax = (amount: number): number => amount * 0.1
```

## 원칙 2: 설계는 엉켜있는 코드를 푸는 것이다

> [!NOTE]
> 함수를 사용하면 관심사를 자연스럽게 분리할 수 있고, 또 이렇게 분리된 것을 조합해 설계한다.

- 재사용하기 쉽다.
- 유지보수하기 쉽다.
- 테스트하기 쉽다.

### `add_item()`을 분리해 더 좋은 설계 만들기

```ts
const add_item = (cart: Item[], item: Item): Item[] => [...cart, item]
```

1. 배열을 복사한다.
2. item 객체를 만든다.
3. 복사본에 item을 추가한다.
4. 복사본을 리턴한다.

```ts
// item의 구조만 알고 있는 함수
const make_cart_item = (name: string, price: number) => {
  return { name, price }
}

// cart의 구조만 알고 있는 함수
const add_item = (cart: Item[], item: Item) => [...cart, item]

// 호출
add_item(shopping_cart, make_cart_item('shoes', 3.45))
```

#### 카피-온-라이트 패턴을 빼내기

- `add_item()`: 카피-온-라이트를 사용해 배열에 항목을 추가하는 함수
- 일반적인 이름으로 변경하면 장바구니와 제품에만 쓸 수 있는 함수가 아닌 어떤 배열이나 항목에도 쓸 수 있는 유틸리티 함수로 사용 가능

```ts
const add_element_last = <T>(array: T[], element: T) => [...array, element]
```

#### `add_item()` 사용하기

```ts
const add_item = (cart: Item[], item: Item) => add_element_last(cart, item)

const add_item_to_cart = ({ name, price }: Item) => {
  const item = make_cart_item(name, price)
  shopping_cart = add_item(shopping_cart, item)

  const total = calc_total(shopping_cart)
  set_cart_total_dom(total)
  update_shipping_icons(shopping_cart)
  update_tax_dom(total)
}
```

### 계산 분류하기

| 동작             | 표시 방법 |
| ---------------- | :-------: |
| cart에 대한 동작 |    `C`    |
| item에 대한 동작 |    `I`    |
| 비즈니스 로직    |    `B`    |
| 배열 유틸리티    |    `A`    |

```ts
// A
const add_element_last = <T>(array: T[], element: T) => [...array, element]

// C
const add_item = (cart: Item[], item: Item) => add_element_last(cart, item)

// I
const make_cart_item = (name: string, price: number) => {
  return { name, price }
}

// C, I, B
const calc_total = (cart: Item[]): number => cart.reduce((total, item) => total + item.price, 0)

// B
const gets_free_shipping = (cart: Item[]): boolean => calc_total(cart) >= 20

// B
const calc_tax = (amount: number): number => amount * 0.1
```

## 작은 함수와 많은 계산

### 액션

```ts
type Item = {
  name: string
  price: number
}

let shopping_cart: Item[] = []

const add_item_to_cart = ({ name, price }: Item) => {
  const item = make_cart_item(name, price)
  shopping_cart = add_item(shopping_cart, item)
  const total = calc_total(shopping_cart)
  set_cart_total_dom(total)
  update_shipping_icons(shopping_cart)
  update_tax_dom(total)
}

const update_shipping_icons = (cart: Item[]) => {
  const buy_buttons = get_buy_buttons_dom()
  buy_buttons.forEach(buy_button => {
    const { item } = buy_button
    const new_cart = add_item(cart, item)
    const is_free_shipping_available = gets_free_shipping(new_cart)

    if (is_free_shipping_available) buy_button.show_free_shipping_icon()
    else buy_button.hide_free_shipping_icon()
  })
}

const update_tax_dom = (total: number) => {
  set_tax_dom(calc_tax(total))
}
```

### 계산

```ts
const add_element_last = <T>(array: T[], element: T) => [...array, element]

const add_item = (cart: Item[], item: Item) => add_element_last(cart, item)

const make_cart_item = (name: string, price: number) => {
  return { name, price }
}

const calc_total = (cart: Item[]) => cart.reduce((total, item) => total + item.price, 0)

const calc_tax = (amount: number) => amount * 0.1

const gets_free_shipping = (cart: Item[]) => calc_total(cart) >= 20
```
