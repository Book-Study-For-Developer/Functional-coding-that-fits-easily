## 패턴2. 추상화 벽

팀간 책임을 명확하게 나누는 것

### 추상화 벽으로 구현을 감춥니다

추상화 벽: 세부 구현을 감춘 함수로 이루어진 계층

추상화 벽에 있는 함수를 사용할 때는 구현을 전혀 몰라도 함수를 사용할 수 있다

추상화 벽 아래에서 일하는 사람들ㅇ느 추상화 벽에 있는 함수를 어떻게 사용하는지 신경쓰지 않아도 된다

### 세부적인 것을 감추는 것은 대칭적입니다

추상화 벽을 만든 개발팀은 추상화 벽에 있는 함수를 사용하는 마케팅 관련 코드를 신경쓰지 않아도 되고

마케팅 팀도 세부 구현을 신경쓰지 않아도 된다

흔히 사용하는 라이브러리나 api와 비슷

책임을 명확하게 나눠줌!

### 장바구니 데이터 구조 바꾸기

장바구니를 객체로 다시 만들면 더 효율적이고 첫번째 패턴인 직접 구현 패턴에 더 가까움

- 배열 대신 객체로 관리하는 것이 더 효율적인 이유 ?
  1. 빠른 접근 속도

     배열은 인덱스를 통해 데이터를 접근하지만 객체는 키를 통해 직접 접근 가능하여 객체에서 특정 상품을 찾는 시간이 빠르다

  2. 데이터 업데이트가 용이

     특정 키에 해당하는 데이터를 바로 수정할 수 있다

  3. 메모리 관리

     배열은 순서가 중요한 경우에 적합하고, 순서가 필요하지 않은 경우 객체로 관리하면 메모리와 성능 측면에서 더 효율적일 수 있다

```tsx
// 배열로 만든 장바구니
function add_item(cart, item) {
  return add_element_last(cart, item);
}

function calc_total(cart) {
  var total = 0;
  for (var i = 0; i < cart.length; i++) {
    var item = cart[i];
    total += item.price;
  }
  return total;
}

function setPriceByName(cart, name, price) {
  var cartCopy = cart.slice();
  for (var i = 0; i < cartCopy.length; i++) {
    if (cartCopy[i].name === name) cartCopy[i] = setPrice(cartCopy[i], price);
  }
  return cartCopy;
}

function remove_item_by_name(cart, name) {
  var idx = indexOfItem(cart, name);
  if (idx !== null) return splice(cart, idx, 1);
  return cart;
}

function indexOfItem(cart, name) {
  for (var i = 0; i < cart.length; i++) {
    if (cart[i].name === name) return i;
  }
  return null;
}

function isInCart(cart, name) {
  return indexOfItem(cart, name) !== null;
}

// 객체로 만든 장바구니
function add_item(cart, item) {
  return objectSet(cart, item.name, item);
}

function calc_total(cart) {
  var total = 0;
  var names = Object.keys(cart);
  for (var i = 0; i < names.length; i++) {
    var item = cart[names[i]];
    total += item.price;
  }
  return total;
}

function setPriceByName(cart, name, price) {
  if (isInCart(cart, name)) {
    var item = cart[name];
    var copy = setPrice(item, price);
    return objectSet(cart, name, copy);
  } else {
    var item = make_item(name, price);
    return objectSet(cart, name, item);
  }
}

function remove_item_by_name(cart, name) {
  return objectDelete(cart, name);
}

function isInCart(cart, name) {
  return cart.hasOwnProperty(name);
}
```

추상화 벽이 의미하는 것은 추상화 벽 위에 있는 함수가 데이터 구조를 몰라도 된다는 것을 말한다

즉, 추상화 벽에 있는 함수만 사용하면 되고 장바구니 구현에 대해서는 신경쓰지 않아도 된다는 것이다

### 추상화 벽은 언제 사용하면 좋을까요?

- 쉽게 구현을 바꾸기 위해
- 코드를 읽고 쓰기 쉽게 만들기 위해
- 팀 간에 조율해야 할 것을 줄이기 위해
- 주어진 문제에 집중하기 위해

**패턴2: 추상화 벽 리뷰**

- 추상화 벽으로 추상화 벽 **아래에 있는 코드와 위에 있는 코드의 의존성을 없앨** 수 있다
- 추상화 벽 위에 있는 코드는 데이터 구조와 같은 구체적인 내용을 신경쓰지 않아도 된다
- 추상화 벽 아래에 있는 코드는 높은 수준의 계층에서 함수가 어떻게 사용되는지 몰라도 된다

## 패턴 3: 작은 인터페이스

장바구니에 제품을 많이 담은 사람이 시계를 구입하면 10% 할인 해주는 코드

방법1: 추상화 벽 만들기

- 해시 맵 데이터 구조로 되어있는 장바구니에 접근할 수 있음
- 같은 계층에 있는 하수를 사용할 수 없음

```tsx
function getWatchDiscount(cart: Cart) {
  let total = 0;
  let names = Object.keys(cart);
  for (let i = 0; i < names.length; i++) {
    let item = cart[name[i]];
    total += item.price;
  }
}
```

방법2: 추상화 벽 위에 만들기

- 해시 데이터 구조를 직접 접근할 수 없음
- 추상화 벽에 있는 함수를 사용해서 장바구니에 접근

```tsx
function getWatchDiscount(cart: Cart) {
  const total = calcTotal(cart);
  const hasWatch = isInCart("watch");
  return total > 100 && hasWatch;
}
```

_🤔 어떤 방법으로 구현하는 것이 더 좋을까?_

방법1로 구현한 코드를 보면 반복문 같은 구체적인 구현이 되어있고 이부분을 개발팀에서 관리하게 되면 마케팅 팀에서 코드를 바꾸고 싶을 때 개발팀에게 이야기하고 용어도 맞추고 해야함. 의존성이 생기고 시간이 많이 들어 추상화 벽의 장점을 약화 시킨다

새로운 기능을 만들 때 하위 계층에 기능을 추가하거나 고치기 보다 상위 계층에 만드는 것이 **작은 인터페이스 패턴**이라고 할 수 있다

새로운 요구사항: 장바구니를 담을 때 로그를 남기기

```tsx
function add_item(cart: Cart, item: Item) {
  logAddToCart(global_user_id, item);
  return objectSet(cart, item.name, item);
}
```

_🤔 설계의 관점에서 생각해보면 여기서 로그를 남기는 것이 괜찮을까?_

`logAddToCart` 함수는 액션이기 때문에 계산이었던 `add_item` 함수도 액션이 되어버리고, `add_item` 를 호출하는 모든 함수가 액션이 되어버려 액션이 전체로 퍼져 테스트 하기가 어려워짐

```tsx
function add_item_to_cart(name: string, price: number) {
  const item = make_cart_item(name, price);
  shopping_cart = add_item(shopping_cart, item);
  const total = calc_total(shopping_cart);
  set_cart_total_dom(total);
  update_shipping_icons(shopping_cart);
  update_tax_dom(total);

  // 장바구니를 담을 때 로그를 남기는 함수
  logAddToCart();
}
```

**패턴3: 작은 인터페이스 리뷰**

- 추상화 벽을 작게 만들어야하는 이유
  - 구현이 변경되었을 때 고쳐야 할 것이 적다
  - 추상화 벽에 있는 코드는 낮은 수준의 코드이기 때문에 버그가 많을 수 있으며 이해하기 어렵다
  - 추상화 벽에 코드가 많을수록 팀간 조율해야할 것도 많아지고, 알아야할 것도 많아 사용하기 어렵다

상위 계층에 어떤 함수를 만들 때 가능한 현재 계층에 있는 함수로 구현하는 것이 작은 인터페이스를 실천하는 방법

**함수 목적에 맞는 계층이 어디있는지** 찾는 감각을 기르는 것이 중요!

## 패턴 4: 편리한 계층

언제 패턴을 적용하고 또 언제 멈춰야 하는지 실용적인 방법을 알려주는 패턴

작업하는 코드가 편리하지 않다고 느끼거나, 구체적인 것을 너무 많이 알아야 하거나, 코드가 지저분하다고 느껴진다면 다시 패턴을 적용해보는 것

## 그래프로 알 수 있는 코드에 대한 정보는 무엇이 있을까요?

호출 그래프로 알 수 있는 비기능적 요구사항

- 유지보수성: 요구사항이 변경되었을 때 가장 쉽게 고칠 수 있는 코드는 무엇인가요?
  - 위로 연결된 것이 적은 함수가 바꾸기 쉽다
  - 자주 바뀌는 코드는 가능한 위쪽에 있어야 한다
- 테스트성: 어떤 것을 테스트하는 것이 가장 중요한가요?
  - 위쪽으로 많이 연결된 함수를 테스트하는 것이 더 가치있다
  - 아래쪽에 있는 함수를 테스트하는 것이 위쪽에 있는 하수를 테스트하는 것보다 가치있다
- 재사용성: 어떤 함수가 재사용하기 좋나요?
  - 아래쪽에 함수가 적을수록 더 재사용하기 좋다
  - 낮은 수준의 단계로 함수를 빼내면 재사용성이 더 높아진다

### 그래프의 가장 위에 있는 코드가 고치기 가장 쉽습니다

그래프에서 가장 아래에 있는 코드는 해당 코드 위에 너무 많은 코드를 만들었기 때문에 고치기 힘들고, 가장 위에 있는 코드는 어디에서도 호출하지 않기 때문에 고치기 쉽다

### 아래에 있는 코드는 테스트가 중요합니다

제대로 만들었다면 가장 아래있는 코드보다 가장 위에 있는 코드가 더 자주 바뀔 것이고, 많은 코드가 가장 아래에서 잘 동작하는 코드에 의존하기 때문에 **아래에 있는 함수**를 테스트 하는 것이 더 가치가 있다

아래 있는 함수를 테스트 하면 위에 있는 함수를 더 믿고 쓸 수 있음

### 아래에 있는 코드가 재사용하기 더 좋습니다

낮은 계층으로 함수를 추출하면 재사용할 가능성이 더 높다

## 요점정리

- 추상화 벽 패턴을 사용하면 세부적인 것을 완벽히 감출 수 있기 때문에 더 높은 차원에서 생각할 수 있다
- 작은 인터페이스 패턴을 사용하면 완성된 인터페이스에 가깝게 계층을 만들 수 있다. 중요한 비즈니스 개념을 표현하는 인터페이스는 한번 잘 만들어 놓고 바뀌거나 늘어나지 않아야 함
- 편리한 계층 패턴을 사용하면 다른 패턴을 요구사항에 맞게 사용할 수 있다
- 호출 그래프 구조에서 어떤 코드를 테스트 하는 것이 가장 좋은지, 유지보수나 재사용하기 좋은 코드는 어디에 있는지 알 수 있다
