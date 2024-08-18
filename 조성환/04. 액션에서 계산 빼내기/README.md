# 액션에서 계산 빼내기

## 이번장에서 살펴볼 내용

- 어떻게 함수로 정보가 들어가고 나오는지 살펴보기
- 테스트하기 쉽고 재사용성이 좋은 코드를 만들기 위한 함수형 기술에 대해 알아보기
- 액션에서 계산 빼내는 방법 배우기

## 메가마트 코드로 생각해보기

mega 마트는 온리인 쇼핑몰이며,
가지고 있는 기능은 
- 장바구니에 담겨있는 제품의 금액 합계를 볼 수 있는 기능
- 장바구니에 넣으면 합계가 20달러가 넘는 제품의 구매 버튼 옆에 무료 배송 아이콘을 표시
- 장바구니 합계가 바뀔 때마다 세금을 다시 계산

### 기존 코드

```js
let shopping_cart = [];
let shopping_cart_total = 0;

const add_item_to_cart = (name, price) => { // 장바구니에 담는 함수
  shopping_cart.push({
    name,
    price,
  })
  calc_cart_total();
}

const calc_cart_total = () => { // 장바구니 합계 볼수 있게 만든 함수
  shopping_cart_total = 0;
  shopping_cart.forEach((item) => {
    shopping_cart_total += item.price;
  })
  set_cart_total_dom();
  update_shipping_icons();
  update_tax_dom();
}

const update_shipping_icons = () => { // 무료배송 아이콘 표시하기 위해 만든 함수
  const buy_buttons = get_buy_buttons_dom();
  buy_buttons.forEach((button) => {
    const item = button.item;
    if(item.price + shoping_cart_total >= 20){
      button.show_free_shipping_icon();
    } else {
      button.hide_free_shipping_icon();
    }
  })
}

const update_tax_dom = () => { // 장바구니 총합의 세금 계산하기 위해 만든 함수
  set_tax_dom(shopping_cart_total * 0.10);
}
```

### 테스트하기 쉽게 만들기

지금 코드는 테스트 하기가 어려움   
코드가 바뀔때 마다 아래와 같은 테스트를 만들어야됨

1. 브라우저 설정하기
2. 페이지 로드하기
3. 장바구니에 제품 담기 버튼 클릭
4. DOM이 업데이트될 때까지 기다리기
5. DOM에서 값 가져오기
6. 가져온 문자열 값을 숫자로 바꾸기
7. 예상하는 값과 비교하기

테스트 개선을 위한 방법으로

- DOM 업데이트와 비즈니스 규칙은 분리되어야 한다.
- 전역변수가 없어야 한다.

의 방법이 있음

### 재사용하기 쉽게 만들기

결제팀과 배송팀에서 작성한 코드를 재사용하려고 했지만 아래와 같은 이유로 재사용이 불가능   
(ex 장바구니 총액이 얼마인지, 배송비가 무료인지)

- 장바구니 정보를 전역 변수에서 읽어오지만, 결제팀과 배송팀은 데이터베이스에서 장바구니 정보를 읽어와야 함
- 결과를 보여주기 위해 DOM을 직접 바꾸고 있지만, 결제팀은 영수증을, 배송팀은 운송장을 출력해야 함 

```js
const update_shipping_icons = () => {
  const buy_buttons = get_buy_buttons_dom();
  buy_buttons.forEach((button) => {
    const item = button.item;
    if(item.price + shoping_cart_total >= 20){ // 배송비가 무료인지 알고 싶지만 DOM이 업데이트 되는걸로 알수 있음
      button.show_free_shipping_icon();
    } else {
      button.hide_free_shipping_icon();
    }
  })
}
```

재사용성을 개선하기 위한 방법으로

- 전역변수에 의존하지 않아야 함
- DOM을 사용할 수 있는 곳에서 실행된다고 가정하면 안됨
- 함수가 결괏값을 리턴해야 함

와 같은 방법이 있음

> [!NOTE]
> 테스트 개선을 위한 방법
> - DOM 업데이트와 비즈니스 규칙은 분리되어야 한다.
> - 전역변수가 없어야 한다.
>
> 재사용성을 개선하기 위한 방법으로
> - 전역변수에 의존하지 않아야 함
> - DOM을 사용할 수 있는 곳에서 실행된다고 가정하면 안됨
> - 함수가 결괏값을 리턴해야 함


### 코드를 개선하기 위해 먼저 할 일은 액션과 계산, 데이터를 구분하는 것

```js
let shopping_cart = []; // 액션
let shopping_cart_total = 0; // 액션
const add_item_to_cart = (name, price) => {...} // 액션
const calc_cart_total = () => {...} // 액션
const update_shipping_icons = () => {...} // 액션
const update_tax_dom = () => {...} // 액션
```

**함수에는 입력과 출력이 있음**   
```js
let total = 0;
const add_to_total = (amount) => { // 인자는 입력
  total = total + amount; // 전역값을 읽어오는건 입력, 전역값을 바꾸는것은 출력
  return total // 리턴값은 출력
}
```

**암묵적 입력과 출력이 있으면 액션**   
```js
let total = 0;
const add_to_total = (amount) => { // 인자는 명시적 입력
  total = total + amount; // 전역값을 읽어오는건 암묵적 입력, 전역값을 바꾸는것은 암묵적 출력
  return total // 리턴값은 명시적 출력
}
```

