# 신뢰할 수 없는 코드를 쓰면서 불변성 지키기

## 이번 장에서 살펴볼 내용

- 레거시 코드나 신뢰할 수 없는 코드로부터 내 코드를 보호하기 위해 방어적 복사를 만듭니다.
- 얕은 복사와 깊은 복사를 비교합니다.
- 카피-온-라이트와 방어적 복사를 언제 사용하면 좋은지 알 수 있습니다.

바꿀 수 없는 라이브러리나 레거시 코드들과 같은 카피-온-라이트를 적용할 수 없는 코드를 함께 사용해야 할 때도 있다.   
이런 코드에 불변 데이터를 전달하는 방식으로 **데이터를 변경하는 코드를 함께 사용하면서 불변성을 지키는 방법**을 알아본다.


## 레거시 코드와 불변성

MegaMart는 블랙 프라이데이 세일을 준비하는데, 블랙 프라이데이 세일을 구현한 함수는 불변성을 지키지 않는 레거시 코드이다.   
블랙 프라이데이 행사 코드는 많은 곳에서 장바구니 데이터를 변경하기 때문에, 불변성을 유지할 수 있는 안전한 인터페이스가 필요하다.   

```ts
const addItemToCart = (name:string, price:number) => {
  const item = makeCartItem(name, price);
  shoppingCart = addItem(shoppingCart, item);
  const total = calcTotal(shoppingCart);
  setCartTotalDom(total);
  updateShippingIcons(shoppingCart);
  updateTaxDom(total);
  blackFridayPromotion(shoppingCart); // 이 코드를 추가해서 블랙 프라이데이 행사를 진행
}
```

카피-온-라이트 원틱을 지키면서 안전하게 함수를 사용할 수 있는 `방어적 복사`를 사용해 데이터를 바꾸는 코드와 데이터를 주고받아야 함   

<br />

우리가 만든 모든 코드는 불변성이 지켜지는 **안전지대**에 있음   
블랙 프라이데이 행사 코드는 안전지대 밖에 있지만 요구 사항을 구현하려면 우리가 만든 코드에서 안전하지 않은 함수를 써야 하는 상황   

**여기서 문제가 되는 사항은**    
안전지대 밖으로 나가는 데이터는 신뢰할 수 없는 데이터가 데이터를 바꿀 수 있기 때문에 잠재적으로 바뀔 수 있고,   
신뢰할 수 없는 코드에서 안전지대로 들어오는 데이터 역시 잠재적으로 바뀔 수 있다.   
이유는 신뢰할 수 없는 코드가 계속 데이터 참조를 가지고 있기 때문에 언제든 바뀔 수 있는 것.   

<br />

그래서 불변성을 지키면서 데이터를 주고 받는 방법을 찾아야 한다는 것   
카피-온-라이트 패턴은 데이터를 바꾸기 전에 복사하고, 데이터를 변경하고, 리턴하는데   
무엇이 바뀌는지 알기 때문에 무엇을 복사해야 할지 예상 할 수 있음

<br />

하지만 레거시 코드에서 무엇을 바꾸는지, 어떤 일이 일어날지 아무도 모르기 때문에 데이터가 바뀌는 것을 완벽히 막아주는 원칙인 **방어적 복사**가 필요

## 방어적 복사가 동작하는 방법

바뀔 수 도 있는 데이터가 신뢰할 수 없는 코드에서 안전지대로 들어오면, 들어온 데이터로 깊은 복사본을 만들고 변경 가능한 원본은 버립니다.   
**신뢰할 수 있는 코드만 복사본을 쓰기 때문에 데이터는 바뀌지 않는데**, 이런 방법으로 들어오는 데이터를 보호 할 수 있음

<br />

안전지대에서 나가는 코드도 바뀌면 안되는데, 안전지대 밖으로 나간 데이터를 신뢰할 수 없는 코드가 값을 변경 할 수 있기 때문   
그래서 **나가는 데이터도 깊은 복사본을 만들어 내보내어 변경할 수 없도록 만들어야** 한다. 이렇게 나가는 데이터를 보호 할 수 있다.

<br />

안전지대에 불변성을 유지하고, 바뀔 수도 있는 데이터가 안전지대로 들어오지 못하도록 하는 것이 **방어적 복사의 목적**

## 방어적 복사 구현하기

장바구니 값을 넘기기 전에 깊은 복사를 해서 함수에 넘기면 인자로 넘긴 원본이 바뀌지 않음

```ts
// 원래 코드
const addItemToCart = (name:string, price:number) => {
  const item = makeCartItem(name, price);
  shoppingCart = addItem(shoppingCart, item);
  const total = calcTotal(shoppingCart);
  setCartTotalDom(total);
  updateShippingIcons(shoppingCart);
  updateTaxDom(total);
  blackFridayPromotion(shoppingCart); // 이 코드를 추가해서 블랙 프라이데이 행사를 진행
}

// 데이터를 전달하기 전에 복사
const addItemToCart = (name:string, price:number) => {
  const item = makeCartItem(name, price);
  shoppingCart = addItem(shoppingCart, item);
  const total = calcTotal(shoppingCart);
  setCartTotalDom(total);
  updateShippingIcons(shoppingCart);
  updateTaxDom(total);
  const cartCopy = deepCopy(shoppingCart); // 깊은 복사를 해서 새로운 참조값을 만듬
  blackFridayPromotion(cartCopy); // 그걸 전달하면 기존 shoppingCart의 참조값을 건드릴 수 없음
}
```

블랙 프라이데이 행사 함수는 인자로 받은 데이터의 원본을 변경하기 때문에 함수 실행 후의 cartCopy가 결과값이다.   
이 cartCopy의 참조값을 그대로 사용하게 되면 블랙 프라이데이 함수가 값을 변경해 버그로 발견 될 수 있음   
그래서 데이터 전달후에도 깊은복사를 해 방어적 복사를 적용해야 함

```ts
// 데이터를 전달하기 전에 복사
const addItemToCart = (name:string, price:number) => {
  const item = makeCartItem(name, price);
  shoppingCart = addItem(shoppingCart, item);
  const total = calcTotal(shoppingCart);
  setCartTotalDom(total);
  updateShippingIcons(shoppingCart);
  updateTaxDom(total);
  const cartCopy = deepCopy(shoppingCart); // 깊은 복사를 해서 새로운 참조값을 만듬
  blackFridayPromotion(cartCopy); // 그걸 전달하면 기존 shoppingCart의 참조값을 건드릴 수 없음
}

// 데이터를 전달한 후에도 복사
const addItemToCart = (name:string, price:number) => {
  const item = makeCartItem(name, price);
  shoppingCart = addItem(shoppingCart, item);
  const total = calcTotal(shoppingCart);
  setCartTotalDom(total);
  updateShippingIcons(shoppingCart);
  updateTaxDom(total);
  const cartCopy = deepCopy(shoppingCart);
  blackFridayPromotion(cartCopy);
  shoppingCart = deepCopy(cartCopy); // 블랙프라이데이 함수가 아무리 데이터를 바꿔도 shoppingCart의 참조값에는 영향을 줄수 없음
}
```

## 방어적 복사 규칙

- 데이터가 안전한 코드에서 나갈 때 복사하기

  1. 불변성 데이터를 위한 깊은 복사본을 만든다.
  2. 신뢰할 수 없는 코드로 복사본을 전달한다.

- 안전한 코드로 데이터가 들어올 때 복사하기

  1. 변경 될 수도 있는 데이터가 들어오면 바로 깊은 복사본을 만들어 안전한 코드로 전달한다.
  2. 복사본을 안전한 코드에서 사용한다.

<br />

첫 번째 규칙과 두 번째 규칙은 순서에 관계없이 사용할 수 있다.   
어떤 경우는 먼저 데이터가 나가고 나중에 들어올 수도 있고,(신뢰할 수 없는 라이브러리 함수를 사용)   
데이터가 들어오고 나중에 나갈 수도 있음(공유라이브러리를 만들때)   
그래서 규칙은 어떤 순서로 적용해도 됨

## 신뢰할 수 없는 코드 감싸기

addItemToCart 함수만 봤을때 왜 코드를 복사하고 전달하고 다시 복사하는지 모를수 있음, 그리고 블랙 프라이데이 함수를 다른곳에서 사용할 수도 있음   
그렇기 때문에 방어적 복사 코드를 분리해 새로운 함수로 만들어 주면 좋을 것 같음

```ts
// 기존 코드
const addItemToCart = (name:string, price:number) => {
  const item = makeCartItem(name, price);
  shoppingCart = addItem(shoppingCart, item);
  const total = calcTotal(shoppingCart);
  setCartTotalDom(total);
  updateShippingIcons(shoppingCart);
  updateTaxDom(total);
  const cartCopy = deepCopy(shoppingCart);
  blackFridayPromotion(cartCopy);
  shoppingCart = deepCopy(cartCopy);
}

// 방어적 복사 코드를 분리
const addItemToCart = (name:string, price:number) => {
  const item = makeCartItem(name, price);
  shoppingCart = addItem(shoppingCart, item);
  const total = calcTotal(shoppingCart);
  setCartTotalDom(total);
  updateShippingIcons(shoppingCart);
  updateTaxDom(total);
  shoppingCart = blackFridayPromotionSafe(shoppingCart); // 분리한 함수 사용
}

const blackFridayPromotionSafe = (cart: Cart) => { // 방어적 복사에 대한 코드 분리
  const cartCopy = deepCopy(cart);
  blackFridayPromotion(cartCopy);
  return deepCopy(cartCopy);
}
```

## 익숙한 방어적 복사

웹 API속에 방어적 복사가 있음   
JSON 데이터가 API에 요청으로 들어왔다고 했을 때, 클라이언트는 데이터를 인더넷을 통해 API로 보내려고 직렬화를 한다.   
이때 JSON 데이터는 깊으 복사본이고, 응답으로 오는 JSON데이터도 깊은 복사본이다.   
이 처럼 웹 API는 방어적 복사를 한다.

## 카피-온-라이트와 방어적 복사는 둘 다 필요한가?

안전지대에서도 방어적 복사로 불변성을 유지할 수 있음   
하지만, 방어적 복사는 깊은 복사를 하기 때문에 위에서 아래로 모든 계층의 중첩된 데이터를 복사하기 때문에 얕은 복사보다 많은 비용이 듬   
그렇기 때문에 많은 복사본 때문에 연산과 메모리를 낭비하는것을 막기 위해 가능한 안전지대에서 카피-온-라이트를 사용해야 함

## 자바스크립트에서 깊은복사를 구현하는 것은 어렵습니다.

깊은 복사는 중첩된 데이터를 모두 재생성해서 복사를 해야 하기 때문에 재귀로 계속 복사를 할 수 있도록 구현해야 됨   
많은 예외 사항이 있기 때문에 많은 사람들이 사용하고 검증된, `Lodash 라이브러리를 사용`하는 것을 추천

## 요점 정리

- 방어적 복사는 불변성을 구현하는 원칙으로 데이터가 들어오고 나갈때 복사본을 만든다.
- 방어적 복사는 깊은 복사를 한다. 그래서 카피-온-라이드보다 비용이 더 든다.
- 카피-온-라이트와 다르게 방어적 복사는 불변성 원칙을 구현하지 않은 코드로부터 데이터를 보호 해 준다.
- 복사본이 많이 필요하지 않기 때문에 카피-온-라이트를 더 많이 사용한다. 방어적 복사는 신뢰 할 수 없는 코드와 함께 사용할 때만 사용한다.
- 깊은 복사는 위에서 아래로 중첩된 데이터 전체를 복사, 얕은 복사는 필요한 부분만 최소한으로 복사한다.