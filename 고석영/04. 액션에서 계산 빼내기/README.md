### MegaMart.com에 오신 것을 환영합니다​

```js
var shopping_cart = []
var shopping_cart_total = 0

// 장바구니에 새로운 제품을 추가하는 함수
function add_item_to_cart(name, price) {
  shopping_cart.push({
    name: name,
    price: price,
  })
  calc_cart_total() // 장바구니 제품 변경에 따른 금액 합계 업데이트
}

// 장바구니에 담긴 제품의 합계 금액을 구하고 반영하는 함수
function calc_cart_total() {
  shopping_cart_total = 0
  for (var i = 0; i < shopping_cart.length; i++) {
    var item = shopping_cart[i]
    shopping_cart_total += item.price
  }
  set_cart_total_dom() // 금액 합계를 반영하기 위한 DOM 업데이트
}
```

### 요구사항 1: 무료 배송비 계산하기

- 구매 합계가 20달러 이상이면 무료 배송
- 장바구니에 넣으면 합계가 20달러가 넘는 구매 버튼 옆에 무료 배송 아이콘 표시

```js
// 절차적인 방법으로 구현하기
// 무료 배송이 가능 여부에 따라 무료 배송 아이콘을 표시하는 함수
function update_shipping_icons() {
  var buy_buttons = get_buy_buttons_dom()
  for (var i = 0; i < buy_buttons.length; i++) {
    var button = buy_buttons[i]
    var item = button.item
    if (item.price + shopping_cart_total >= 20) button.show_free_shipping_icon()
    else button.hide_free_shipping_icon()
  }
}

function calc_cart_total() {
  shopping_cart_total = 0
  for (var i = 0; i < shopping_cart.length; i++) {
    var item = shopping_cart[i]
    shopping_cart_total += item.price
  }
  set_cart_total_dom()
  update_shipping_icons() // 합계 금액이 변경될 때마다 모든 아이콘 업데이트
}
```

### 요구 사항 2: 세금 계산하기

- 장바구니 금액 합계가 바뀔 때마다 세금 다시 계산

```js
function update_tax_dom() {
  set_tax_dom(shopping_cart_total * 0.1)
}

function calc_cart_total() {
  shopping_cart_total = 0
  for (var i = 0; i < shopping_cart.length; i++) {
    var item = shopping_cart[i]
    shopping_cart_total += item.price
  }
  set_cart_total_dom()
  update_shipping_icons()
  update_tax_dom() // 합계 금액이 변경될 때마다 세금을 다시 계산
}
```

### 테스트하기 쉽게 만들기

#### 현재 코드가 비즈니스 규칙을 테스트하기 어려운 이유

코드가 변경될 때마다 아래와 같은 테스트를 만들어야 한다.

1. 브라우저 설정하기
2. 페이지 로드하기
3. 장바구니에 제품 담기 버튼 클릭
4. DOM이 업데이트될 때까지 기다리기
5. DOM에서 값 가져오기
6. 가져온 문자열 값을 숫자로 바꾸기
7. 예상하는 값과 비교하기

#### 테스트 해야하는 비즈니스 규칙 확인하기

```ts
const update_tax_dom = () => {
  set_tax_dom(shopping_cart_total * 0.1)
}
```

- `set_tax_dom()`: 결과 값을 얻을 방법은 DOM에서 값을 가져오는 방법밖에 없음
- `shopping_cart_total`: 테스트하기 전 전역 변수 값을 설정해야 함
- `total * 0.1`: 최종적으로 테스트 해야하는 비즈니스 규칙

#### 테스트 개선 방안

- DOM 업데이트와 비즈니스 규칙 분리하기
- 전역변수 없애기

### 재사용하기 쉽게 만들기

앞에서 요구 사항을 적용한 코드는 테스트하기 어려울 뿐만 아니라 다른 팀에서 재사용하기에도 어렵다.

- 장바구니 정보를 전역변수에서 읽어오고 있지만, 다른 팀은 데이터베이스에서 읽어와야 한다.
- 결과를 보여주기 위해 DOM을 직접 바꾸고 있지만, 다른 팀은 영수증과 운송장을 출력해야 한다.

#### 재사용할 비즈니스 규칙과 이를 방해하는 요인을 구분하기

```js
function update_shipping_icons() {
  var buy_buttons = get_buy_buttons_dom()
  for (var i = 0; i < buy_buttons.length; i++) {
    var button = buy_buttons[i]
    var item = button.item
    if (item.price + shopping_cart_total >= 20) button.show_free_shipping_icon()
    else button.hide_free_shipping_icon()
  }
}
```

- `>= 20`: 재사용할 비즈니스 규칙
- `shopping_cart_total`: 함수가 전역 변수 값이 있어야 실행할 수 있음
- `button.show_free_shipping_dom()`: DOM이 있어야 실행할 수 있음
- `button.hide_free_shipping_dom()`: DOM이 있어야 실행할 수 있음
- `update_shipping_icons`: 리턴값이 없어 결과를 받을 방벙이 없음

#### 재사용성을 높이기 위한 개선 방안

- 전역변수에 의존하지 않아야 함
- DOM을 사용할 수 있는 곳에서만 실행된다고 가정하지 말아야 함
- 함수가 결과 값을 리턴해야 함

### 액션과 계산, 데이터 구분하기

| 코드                      | A / C / D |                       |
| ------------------------- | :-------: | --------------------- |
| `shopping_cart`           |    `A`    | 변경 가능한 전역 변수 |
| `shopping_cart_total`     |    `A`    | 변경 가능한 전역 변수 |
| `add_item_to_cart()`      |    `A`    | 전역 변수 변경        |
| `update_shipping_icons()` |    `A`    | DOM 읽기              |
| `update_tax_dom()`        |    `A`    | DOM 변경              |
| `calc_cart_total()`       |    `A`    | 전역 변수 변경        |

모든 코드가 액션이기 때문에 액션에서 계산으로 빼낼 수 있는 부분을 빼내야 한다.

### 함수에는 입력과 출력이 있습니다

```js
let total = 0

function add_to_cart(amount) {
  console.log('Old total: ' + total)
  total += amount
  return total
}
```

- 입력과 출력은 명시적이거나 암묵적일 수 있다.
- 함수에서 암묵적 입력과 출력이 있으면 액션이 된다.

|        | 입력           | 출력                      |
| ------ | -------------- | ------------------------- |
| 명시적 | 인자           | 리턴 값                   |
| 암묵적 | 전역 변수 읽기 | 전역 변수 변경, 콘솔 출력 |

### 테스트와 재사용성은 입출력과 관련 있습니다

- DOM 업데이트와 비즈니스 규칙은 분리되어야 한다.
- 전역변수가 없어야 한다.
- 전역변수에 의존하지 않아야 한다.
- DOM을 사용할 수 있는 곳에서만 실행된다고 가정하면 안 된다.
- 함수가 결과 값을 리턴해야 한다.

### 액션에서 계산 빼내기

- 서브 루틴 추출하기(extract subroutine) -> `calc_total()`
- 암묵적 입출력을 없애기 -> `shopping_cart_total = calc_total(shopping_cart)`

#### `calc_cart_total()`에서 `calc_total()` 빼내기

```ts
type Item = {
  name: string
  price: number
}

const calc_total = (cart: Item[]): number => cart.reduce((total, item) => total + item.price, 0)

const calc_cart_total = () => {
  shopping_cart_total = calc_total(shopping_cart)
  set_cart_total_dom()
  update_shipping_icons()
  update_tax_dom()
}
```

#### `add_item_to_cart()`에서 `add_item()` 빼내기

```ts
const add_item = (cart: Item[], item: Item): Item[] => [...cart, item]

const add_item_to_cart = (item: Item) => {
  add_item(item)
  calc_cart_total()
}
```

#### `update_tax_dom()`에서 `calc_tax()` 빼내기

```ts
const calc_tax = (amount: number): number => amount * 0.1

const update_tax_dom = () => {
  set_tax_dom(calc_tax(shopping_cart_total))
}
```

#### `update_shipping_icons()`에서 `gets_free_shipping()` 빼내기

```ts
const gets_free_shipping = (total: number, item_price: number): boolean => item_price + total >= 20

const update_shipping_icons = () => {
  const buy_buttons = get_buy_buttons_dom()
  buy_buttons.forEach(buy_button => {
    const { item } = buy_button
    const is_free_shipping_available = gets_free_shipping(shopping_cart_total, item.price)

    if (is_free_shipping_available) buy_button.show_free_shipping_icon()
    else buy_button.hide_free_shipping_icon()
  })
}
```

### 계산 추출을 단계별로 알아보기

1. 계산 코드를 찾아 빼내 새로운 함수로 만든다.
2. 새 함수에 암묵적 입력과 출력을 찾는다.
3. 암묵적 입력은 인자로, 암묵적 출력은 리턴값으로 바꾼다.
4. 명시적 입력과 출력만을 갖는 함수로 만든 뒤 원래 함수에서 호출한다.

### 전체 코드

```ts
type Item = {
  name: string
  price: number
}

let shopping_cart: Item[] = []
let shopping_cart_total = 0

// 계산
const calc_total = (cart: Item[]): number => cart.reduce((total, item) => total + item.price, 0)

const calc_tax = (amount: number): number => amount * 0.1

const gets_free_shipping = (total: number, item_price: number): boolean => item_price + total >= 20

// 액션
const update_shipping_icons = () => {
  const buy_buttons = get_buy_buttons_dom()
  buy_buttons.forEach(buy_button => {
    const { item } = buy_button
    const is_free_shipping_available = gets_free_shipping(shopping_cart_total, item.price)

    if (is_free_shipping_available) buy_button.show_free_shipping_icon()
    else buy_button.hide_free_shipping_icon()
  })
}

const update_tax_dom = () => {
  set_tax_dom(calc_tax(shopping_cart_total))
}

const calc_cart_total = () => {
  shopping_cart_total = calc_total(shopping_cart)
  set_cart_total_dom()
  update_shipping_icons()
  update_tax_dom()
}

// 계산
const add_item = (cart: Item[], item: Item): Item[] => [...cart, item]

// 액션
const add_item_to_cart = (item: Item) => {
  shopping_cart = add_item(shopping_cart, item)
  calc_cart_total()
}
```
