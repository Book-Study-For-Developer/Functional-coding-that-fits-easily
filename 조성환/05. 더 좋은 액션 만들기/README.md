# 더 좋은 액션 만들기

## 이번 장에서 살펴볼 내용

- 암묵적 입력과 출력을 제거해서 재사용하기 좋은 코드를 만드는 방법을 알아봅니다.
- 복잡하게 엉킨 코드를 풀어 더 좋은 구조로 만드는 법을 배웁니다.

## 비즈니스 요구 사항과 설계,함수를 맞추기

**비즈니스 요구 사항**  
주문 결과가 무료 배송인지 확인할 수 있어야 됨

**현재 설계**  
장바구니에 담긴 제품의 가격과 어떤 아이템의 가격을 받아서 무료 배송인지 체크

```ts
const getsFreeShipping = (total: number, itemPrice: number) => {
  return itemPrice + total >= 20;
};
```

**개선할 설계**  
주문하는 제품을 넘기면 무료배송인지 확인

```ts
const getsFreeShipping = (cart: Cart) => {
  // 주문하는 제품이 담긴 카트를 넘김
  return calcTotal(cart) >= 20; // 카트에서 값을 모두 더해주는 calcTotal을 사용
};

const calcTotal = (cart: Cart) => {
  // 현준님 코드 좀 쓰겠습니다 😆
  const total = cart.reduce((accumulator, currentValue) => {
    return accumulator + currentValue.price;
  }, 0);
  return total;
};
```

기존에 **장바구니에 담긴 상품 가격이랑 주문할 상품의 가격 합으로 무료배송 여부**를 보여주던 함수에서 **카트 넘기면 무료배송 여부를 반환**하도록 수정

**버튼에 무료배송 마크 달아주던 코드 수정**

```ts
// 기존코드
const updateShippingIcons = () => {
  const buttons = getBuyButtonsDom();
  buttons.forEach((button) => {
    const item = button.item;
    if (getsFreeShipping(shoppingCartTotal, item.price)) {
      button.showFreeShippingIcon();
    } else {
      button.hideFreeshippingIcon();
    }
  });
};

// 카트를 전달하도록 바꾼 코드
const updateShippingIcons = () => {
  const buttons = getBuyButtonsDom();
  buttons.forEach((button) => {
    const item = button.item;
    const newCart = addItem(shoppingCart, item.name, item.price);
    if (getsFreeShipping(newCart)) {
      button.showFreeShippingIcon();
    } else {
      button.hideFreeshippingIcon();
    }
  });
};
```

기존 장바구니 카트와 상품을 합친 Cart데이터를 만들어서 전달

> [!NOTE]
> 우리가 구현해야 되는 비즈니스 규칙이 무엇인지 명확히 아는 것이 중요할 것 같음

## 암묵적 입력과 출력 줄이기

```ts
const updateShippingIcons = (cart: Cart) => {
  const buttons = getBuyButtonsDom();
  buttons.forEach((button) => {
    const item = button.item;
    const newCart = addItem(cart, item.name, item.price);
    if (getsFreeShipping(newCart)) {
      button.showFreeShippingIcon();
    } else {
      button.hideFreeshippingIcon();
    }
  });
};

const calcCartTotal = () => {
  shoppingCartTotal = calcTotal(shoppingCart);
  setCartTotalDom();
  updateShippingIcons(shoppingCart);
  updateTaxDom();
};
```

cart 라는 명시적 인자를 받고  
전역 변수를 사용하던 곳에 cart를 전달  
updateShippingIcons에 cart를 전달

## 코드 다시 살펴보기

비즈니스 규칙에 맞게 수정하고, 암묵적 입출력을 없애면서 보이게 된 과한 코드나, 사용하지 않는 코드를 정리

- 굳이 함수로 추출하지 않아도 될 것
- 이제 더 이상 사용하지 않는 코드 제거

```ts
// 원래 코드
const addItemToCart = (name: string, price: number) => {
  shoppingCart = addItem(shoppingCart, name, price);
  calcCartTotal(shoppingCart); // 굳이 함수로 추출한 의미가 없음
}

const calcCartTotal = (cart: Cart) => {
  const total = calcTotal(cart);
  setCartTotalDom(total);
  updateShippingIcons(total);
  updateTaxDom(total);
  shoppingCartTotal = total; // 주문 제품을 전달하면 무료배송 유무인지 확인하도록 바꿔서 사용하지 않는 값
}

// 코드를 다시 살펴보아 개선한 코드
const addItemToCart = (name, price) => {
  shoppingCart = addItem(shoppingCart, name, price);
  const total = calcTotal(shoppingCart);
  const cartTotalDom(total);
  updateShippingIcons(shoppingCart);
  updateTaxDom(total);
}
```

## 설계는 엉켜있는 코드를 푸는 것이다

함수를 사용하면 관심사를 자연스럽게 분리할 수 있음  
분리된 것은 언제든 쉽게 조합할 수 있음

분리하게 되면 함수가 작아지면서  
**재사용하기 쉬워짐**  
**유지보수하기 쉬움**  
**테스트하기 쉬움**

### addItem 함수를 분리해 더 좋은 설계 만들기

분리할 내용 찾기

```ts
const addItem = (cart: Cart, name: string, price: number) => {
  const newCart = cart.slice(); // 1. 배열을 복사
  newCart.push({
    // 2. item 객체를 생성, 3. 복사본에 item 추가
    name,
    price,
  });
  return newCart; // 4. 복사본을 리턴
};

addItem(shoppingCart, "shoes", 3.45);
```

addItem 함수는 cart 와 item 구조를 모두 알고 있기 때문에 item에 관한 코드를 별도의 함수로 분리

```ts
const makeCartItem = (name: string, price: number) => {
  // item 구조만 알고 있는 함수
  return {
    name,
    price,
  };
};

const addItem = (cart: Cart, item: Item) => {
  // cart 구조만 알고 있는 함수
  const newCart = cart.slice();
  newCart.push(item);
  return newCart;
};

addItem(shoppingCart, makeCartItem("shoes", 3.45));
```

## 카피-온-라이트 패턴을 빼내기

addItem 함수는 단순히 복사본을 만들고, 그 복사본을 수정하고, 리턴해주는 역할을 함  
이런건 좀 더 일반적인 이름으로 변경 가능  
`addItem` -> `addElementLast`

```ts
const addElementLast = <T>(array: T[], elem: T) => {
  const newArray = array.slice();
  newArray.push(elem);
  return newArray;
};
```

## 최종 코드

```ts
const addElementLast = <T>(array: T[], elem: T) => {
  const newArray = array.slice();
  newArray.push(elem);
  return newArray;
};

const addItem = (cart: Cart, item: Item) => {
  return addElementLast<Item>(cart, item);
};

const makeCartItem = (name: string, price: number) => {
  return {
    name,
    price,
  };
};

const calcTotal = (cart: Cart) => {
  const total = cart.reduce((accumulator, currentValue) => {
    return accumulator + currentValue.price;
  }, 0);
  return total;
};

const getsFreeShipping = (cart: Cart) => {
  return calcTotal(cart) >= 20;
};

const calcTax = (amount: number) => {
  return amount * 0.1;
};
```

원래 addItem 함수를 세개로 나눠 Cart, Item, Array 에 관한 함수로 분리  
분리하는 이유는 나중에 다룰 **설계 기술**을 미리 보여주는 것으로, 최종적으로는 **구분된 그룹과 분리된 계층으로 구성**할 것

> ❓ 질문
> addItem 함수를 삭제하고 바로 addElementLast를 사용하지 않은 이유가 있었던걸까? 싶은 궁금증이 있네요.  
> 뭔가 카트에 아이템을 추가하는걸 명시하기 위해서 addItem함수를 만들고 그 안에 범용적인 함수를 부르는 건지, 아니면 그냥 책에서 그렇게 정리를 한거고 실무에서는 addElementLast를 사용해도 되는건지...

## 요점정리

- 일반적으로 암묵적 입력과 출력은 인자와 리턴값으로 바꿔 없애는 것이 좋음
- 설계는 엉켜있는 것을 푸는 것으로 풀려 있는건 언제든 다시 합칠수 있음
- 엉켜 있는 것을 풀어 각 함수가 하나의 일만 하도록 하면 개념을 중심으로 쉽게 구성이 가능
