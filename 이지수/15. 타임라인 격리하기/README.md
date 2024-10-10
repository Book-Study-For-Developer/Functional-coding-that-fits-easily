## 버그

```tsx
function add_item_to_cart(name, price, quantity) {
  cart = add_item(cart, name, price, quantity);
  calc_cart_total();
}

function calc_cart_total() {
  total = 0;
  cost_ajax(cart, function (cost) {
    total += cost;
    shipping_ajax(cart, function (shipping) {
      total += shipping;
      update_total_dom(total);
    });
  });
}
```

장바구니에 상품 담기를 빠르게 두번 클릭했을 때 합계 금액이 잘못 계산되는 버그가 있음

## 타임라인 다이어그램은 시간에 따라 어떤 일이 일어나는지 보여줍니다

타임 라인 다이어그램은 시간에 따른 액션 순서를 시각적으로 표시한 것

각 액션이 서로 어떻게 상호작용하고 간섭하는지 볼 수 있다

## 타임라인 다이어그램 기본 규칙

- 액션은 순서대로 실행되거나 동시에 실행됨
- 순서대로 실행되는 액션은 같은 타임라인에서 하나가 끝나면 다른 하나가 실행됨
- 동시에 실행되는 액션은 여러 타임라인에서 나란히 실행됨

1. 두 액션이 순서대로 나타나면 같은 타임라인에 넣는다
2. 두 액션이 동시에 실행되거나 순서를 예상할 수 없다면 분리된 타임라인에 넣는다

## 자세히 보면 놓칠 수 있는 액션 순서에 관한 두가지 사실

1. ++와 +=는 사실 세단계임

   ```tsx
   total++;

   // 숨겨진 단계
   let temp = total; // 읽기(액션);
   temp = temp + 1; // 더하기;
   total = temp; // 쓰기(액션);
   ```

2. 인자는 함수를 부르기 전에 실행함

   전역변수를 읽는 것이어서 표시해야함

   ```tsx
   console.log(total);

   let temp = total;
   console.log(temp);
   ```

## add-to-cart 타임라인 그리기

1. 액션을 확인한다
2. 순서대로 실행되거나 동시에 실행되는 액션을 그린다
3. 플랫폼에 특화된 지식을 사용해 다이어그램을 단순하게 만든다

```tsx
function add_item_to_cart(name, price, quantity) {
  cart = add_item(cart, name, price, quantity);
  calc_cart_total();
}

function calc_cart_total() {
  total = 0;
  cost_ajax(cart, function (cost) {
    total += cost;
    shipping_ajax(cart, function (shipping) {
      total += shipping;
      update_total_dom(total);
    });
  });
}
```

위 코드에 13가지의 액션이 있다

비동기 호출도 두개가 있다 (const_ajax()에 전달되는 콜백, shipping_ajax()에 전달되는 콜백)

비동기 호출은 새로운 타임라인으로 그린다

### 타임라인 다이어그램으로 순서대로 실행되는 코드에도 두가지 종류가 있다

- 순서가 섞일 수 있는 코드
  - 두 액션 사이에 시간은 얼마나 걸릴지 알 수 없다
- 순서가 섞이지 않는 코드
  - 두 액션이 차례로 실행되는데 그 사이에 다른 작업이 끼어들지 못함

짧은 타임라인은 관리하기 쉽고, 박스가 적은 것이 좋다

(액션은 박스, 액션 사이에 걸리는 시간은 선)

### 타임라인 다이어그램으로 동시에 실행되는 코드는 순서를 예측할 수 없다는 것을 알 수 있다

동시에 실행되는 코드는 순서를 확신할 수 없다

!https://drek4537l1klr.cloudfront.net/normand/Figures/f0408-02.jpg

다 다르게 생겼지만 모두 같은 의미!

## 좋은 타임라인의 원칙

1. 타임라인은 적을수록 이해하기 쉽다

   타임라인이 하나라면 모든 액션은 앞의 액션 다음에 실행

2. 타임라인은 짧을수록 이해하기 쉽다

   타임라인의 단계를 줄여 실행 가능 순서의 수도 줄인다

3. 공유하는 자원이 적을수록 이해하기 쉽다

   서로 자원을 공유하는 액션을 주의 깊게 봐야함

4. 자원을 공유한다면 서로 조율해야함

   올바른 순서대로 자원을 쓰고 돌려주는 안전한 공유를 해야함

5. 시간을 일급으로 다룬다

   타임라인 다루는 재사용 가능한 객체를 만들면 타이밍 문제를 쉽게 만들 수 있다

### 타임라인 단순화하기

1. 하나의 타임라인에 있는 모든 액션을 하나로 통합한다

   !https://drek4537l1klr.cloudfront.net/normand/Figures/f0415-01.jpg

2. 타임라인이 끝나는 곳에서 새로운 타임라인이 하나만 생긴다면 통합한다

   !https://drek4537l1klr.cloudfront.net/normand/Figures/f0415-02.jpg

### 완성된 타임라인 읽기

타임라인 다이어그램으로 가능한 액션의 순서를 알 수 있다

실행 순서를 이해하면 코드가 제대로 동작하는지 알 수 있고,

잘 동작하지 않는 실행 순서를 알면 버그를 찾을 수 있다

## 버그 해결하기

전역변수를 지역변수로 바꾸기

전역변수를 인자로 바꾸기

```tsx
function add_item_to_cart(name, price, quantity) {
  cart = add_item(cart, name, price, quantity);
  calc_cart_total(cart);
}

function calc_cart_total(cart) {
  let total = 0;
  cost_ajax(cart, function (cost) {
    total += cost;
    shipping_ajax(cart, function (shipping) {
      total += shipping;
      update_total_dom(total);
    });
  });
}
```

## 재사용하기 더 좋은 코드로 만들기

비동기 호출을 사용한다면 출력을 콜백으로 바꾼다

```tsx
function calc_cart_total(cart, callback) {
  var total = 0;
  cost_ajax(cart, function (cost) {
    total += cost;
    shipping_ajax(cart, function (shipping) {
      total += shipping;
      callback(total);
    });
  });
}

function add_item_to_cart(name, price, quant) {
  cart = add_item(cart, name, price, quant);
  calc_cart_total(cart, update_total_dom);
}
```

### 원칙: 비동기 호출에서 명시적인 출력을 위해 리턴값 대신 콜백을 사용할 수 있다

비동기 호출에서 결과를 받을 수 있는 방법은 콜백을 사용하는 것

동기화된 함수에서 액션을 빼낼 때 액션을 호출하는 대신 액션에 넘기는 인자값을 리턴하고 호출하는 곳에서 리턴값을 받아 액션을 호출한다

비동기함수에서는 리턴값 대신 콜백을 사용한다
