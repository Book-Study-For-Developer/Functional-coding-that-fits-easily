```tsx
function add_item_to_cart(name, price, quantity) {
  cart = add_item(cart, name, price, quantity)
  calc_cart_total()
}

function calc_cart_total() {
  total = 0
  cost_ajax(cart, function (cart) {
    total += cart
    shipping_ajax(cart, function (shipping) {
      total += shipping
      update_total_dom(total)
    })
  })
}
```

- 구매 버튼을 빠르게 두 번 클릭했을 때 버그 발생
- 버그가 클릭 속도와 관련이 있는 것 같음 → 순서에 의해 발생되는 문제는 아닐까? → 타임라인을 그려보자

## 타임라인 다이어그램

- 타임라인: 액션을 순서대로 나열한 것
- 타임라인 다이어그램: 시간에 따른 액션 순서를 시각적으로 표시한 것
- 다이어그램 옆으로 타임라인을 하나씩 추가하면 각 액션이 서로 어떻게 상호작용하고 간섭하는지 알 수 있음

### 타임라인 다이어그램 기본 규칙

- 두 액션이 순서대로 나타나면 같은 타임라인
- 두 액션이 동시에 실행되거나 순서를 예상할 수 없다면 타임라인 분리

#### 액션의 실행 순서

- 액션 → 순서대로 실행되거나 동시에 실행
  - 순서대로 실행되는 액션 → 같은 타임라인에서 하나가 끝나면 다른 하나가 실행
  - 동시에 실행되는 액션 → 여러 타임라인에서 나란히 실행

### 액션 순서에 관한 두 가지 사실

#### `++`와 `+=`는 사실 세 단계로 이루어져 있습니다

```tsx
total++
```

- 자바스크립트의 증감 연산자는 **읽기+더하기+쓰기**가 압축된 형태

```tsx
let temp = total
temp = temp + 1
total = temp
```

#### 인자는 함수를 부르기 전에 실행합니다

```tsx
console.log(total)
```

- `console.log()`를 실행 전에 total을 읽고 있음

```tsx
let temp = total
console.log(total)
```

## add-to-cart 타임라인 그리기

1. 액션 확인하기
2. 순서에 따라 액션 그리기(순서대로 실행되는 액션 / 동시에 실행되는 액션)
3. 플랫폼에 특화된 지식을 활용하여 다이어그램을 단순화하기

### 1단계: 액션 확인하기

![image.png](https://drek4537l1klr.cloudfront.net/normand/Figures/figure_14-3.png)

### 비동기 호출은 새로운 타임라인으로 그리기

> `cost_ajax()` 와 `shipping_ajax()` 는 비동기 콜백

```tsx
saveUserAjax(user, function () {
  setUserLoadingDOM(false)
})
setUserLoadingDOM(true)
saveDocumentAjax(document, function () {
  setDocLoadingDOM(false)
})
setDocLoadingDOM(true)
```

- `setUserAjax()` → 타임라인이 없으므로 새로운 타임라인 만들기
- `setUserLoadingDOM(false)` → 비동기 콜백이므로 새로운 타임라인 필요 → 실행 순서를 표시하기 위해 점선 뒤에 표시
- `setUserLoadingDOM(true)` → 콜백이 아니므로 원래 타임라인에서 실행 → `saveUserAjax()` 다음에 표시
- `saveDocumentAjax()` → 콜백이 아니므로 원래 타임라인에서 실행 → `setUserLoadingDOM(true)` 다음에 표시
- `setDocLoadingDOM(false)` → 비동기 콜백이므로 새로운 타임라인 필요 → 실행 순서를 표시하기 위해 점선 뒤에 표시
- `setDocLoadingDOM(true)` → 콜백이 아니므로 원래 타임라인에서 실행 → `saveDocumentAjax()` 다음에 표시

![image.png](https://drek4537l1klr.cloudfront.net/normand/Figures/f0402-01.jpg)

### 2단계: 순서에 따라 액션 그리기

> `const_ajax()`와 `shipping_ajax()`는 ajax 요청이기 때문에 콜백은 새로운 타임라인에 그린다.

![image.png](https://drek4537l1klr.cloudfront.net/normand/Figures/f0406-01.jpg)

#### 타임라인 다이어그램 확인하기

- 확인된 모든 액션(13개)이 다이어그램에 있나요? ✅
- 각 비동기 콜백(`const_ajax()` / `shipping_ajax()`)이 다른 타임라인에 있나요? ✅

#### 타임라인 다이어그램으로 순서대로 실행되는 코드에도 두 가지 종류가 있다는 것을 알 수 있습니다

| 순서가 섞일 수 있는 코드                                                        | 순서가 섞이지 않는 코드                                                         |
| ------------------------------------------------------------------------------- | ------------------------------------------------------------------------------- |
| 두 액션 사이에 시간은 얼마나 걸릴지 알 수 없음                                  | 두 액션이 차례로 실행되는데 그 사이에 다른 작업이 끼어들 수 없음                |
| 액션은 박스로, 액션 사이에 걸리는 시간은 선으로 표시                            | 액션을 같은 상자에 표시                                                         |
| ![imgae.png](https://drek4537l1klr.cloudfront.net/normand/Figures/f0407-01.jpg) | ![imgae.png](https://drek4537l1klr.cloudfront.net/normand/Figures/f0407-02.jpg) |

> 짧은 타임라인이 관리하기 쉬우므로 박스가 적은 것이 좋음!

#### 타임라인 다이어그램으로 동시에 실행되는 코드는 순서를 예측할 수 없다는 것을 알 수 있습니다

![image.png](https://drek4537l1klr.cloudfront.net/normand/Figures/f0408-01.jpg)

- 타임라인에서 나란히 표현된 액션은 정확한 실행 순서를 예측할 수 없음
- 타임라인이 길어지거나 더 많아지면 가능한 실행 순서는 빠르게 늘어나게 됨
- 순서가 섞이는 것과 마찬가지로 실행 가능한 순서는 사용하는 플랫폼에서 지원하는 스레드 모델마다 다름

> [!NOTE]
>
> #### 좋은 타임라인 원칙
>
> - 타임라인은 적을수록 이해하기 쉽습니다.
> - 타임라인은 짧을수록 이해하기 쉽습니다.
> - 공유하는 자원이 적을수록 이해하기 쉽습니다.
> - 자원을 공유한다면 서로 조율해야 합니다.
> - 시간을 일급으로 다룹니다.

### 3단계: 타임라인 단순화하기

- 플랫폼의 스레드 모델에 따라 다이어그램을 단순하게 만드는 일

#### 자바스크립트에서 단순화하는 단계

- 하나의 타임라인에 있는 모든 액션을 하나로 통합하기
- 타임라인이 끝나는 곳에서 새로운 타임라인이 하나 생긴다면 통합하기

![image.png](https://drek4537l1klr.cloudfront.net/normand/Figures/f0415-02.jpg)

#### `add-to-cart` 타임라인에 적용하기

| 1. 하나의 타임라인에 있는 모든 액션을 하나로 통합하기                           | 2. 타임라인이 끝나는 곳에서 새로운 타임라인이 하나 생긴다면 통합하기            |
| ------------------------------------------------------------------------------- | ------------------------------------------------------------------------------- |
| ![image.png](https://drek4537l1klr.cloudfront.net/normand/Figures/f0423-01.jpg) | ![image.png](https://drek4537l1klr.cloudfront.net/normand/Figures/f0423-02.jpg) |

## 요약: 타임라인 다이어그램 그리기

- 액션을 확인하기
- 액션을 그리기
- 타임라인을 단순화하기
- 타임라인 읽기

## 공유하는 자원을 없애 문제를 해결하기

| 두 번 천천히 클릭한 경우                                                        | 두 번 빠르게 클릭한 경우                                                        |
| ------------------------------------------------------------------------------- | ------------------------------------------------------------------------------- |
| ![image.png](https://drek4537l1klr.cloudfront.net/normand/Figures/f0428-01.jpg) | ![image.png](https://drek4537l1klr.cloudfront.net/normand/Figures/f0429-01.jpg) |

> [!NOTE]
>
> #### 문제는 공유하는 자원!
>
> 두 타임라인 모두 같은 전역변수를 사용하고 있기 때문에 실행 순서가 섞인 상태에서도 각 타임라인이 전역변수에 접근하고 있음

### 전역 변수를 지역변수로 바꾸기

> `total`은 공유할 필요가 없으니 전역변수 대신 지역변수를 사용하자

#### 1. 지역변수로 바꿀 수 있는 전역변수를 찾기

```tsx
function calc_cart_total() {
  total = 0
  cost_ajax(cart, function (cost) {
    total += cost
    shipping_ajax(cart, function (shipping) {
      total += shipping
      update_total_dom(total)
    })
  })
}
```

#### 2. 찾은 전역변수를 지역변수로 바꾸기

```tsx
function calc_cart_total() {
  let total = 0 // 지역변수로..
  cost_ajax(cart, function (cost) {
    total += cost
    shipping_ajax(cart, function (shipping) {
      total += shipping
      update_total_dom(total)
    })
  })
}
```

### 전역변수를 인자로 바꾸기

> `cart`는 지역변수로 바꾸기 어려우니 암묵적 입력을 인자로 바꿔주자

#### 1. 암묵적 인자 확인하기

```tsx
function add_item_to_cart(name, price, quantity) {
  cart = add_item(cart, name, price, quantity)
  calc_cart_total()
}

function calc_cart_total() {
  let total = 0
  cost_ajax(cart, function (cost) {
    total += cost
    shipping_ajax(cart, function (shipping) {
      total += shipping
      update_total_dom(total)
    })
  })
}
```

#### 2. 암묵적 입력을 인자로 바꾸기

```tsx
function add_item_to_cart(name, price, quantity) {
  cart = add_item(cart, name, price, quantity)
  calc_cart_total(cart)
}

function calc_cart_total(cart) {
  // 명시적 인자로..
  let total = 0
  cost_ajax(cart, function (cost) {
    total += cost
    shipping_ajax(cart, function (shipping) {
      total += shipping
      update_total_dom(total)
    })
  })
}
```

## 재사용하기 더 좋은 코드로 만들기

> 비동기 함수는 값을 리턴할 수 없기 때문에 콜백 함수로 전달하자  
> → `함수 본문을 콜백으로 바꾸기` 리팩토링

#### callback으로 바꾸기

```tsx
function add_item_to_cart(name, price, quantity) {
  cart = add_item(cart, name, price, quantity)
  calc_cart_total(cart, update_total_dom) // 업데이트 함수를 콜백으로 전달
}

function calc_cart_total(cart, callback) {
  let total = 0
  cost_ajax(cart, function (cost) {
    total += cost
    shipping_ajax(cart, function (shipping) {
      total += shipping
      callback(total) // 콜백으로..
    })
  })
}
```

#### 비동기 호출에서 명시적인 출력을 위해 리턴값 대신 콜백을 사용할 수 있습니다

- 비동기 호출은 바로 리턴이 되지만 결과 값은 콜백이 호출되어야 얻을 수 있음
- 비동기 코드에서 계산된 값은 이벤트 루프에서 나중에 실행되기 때문에 즉시 리턴 값으로 받을 수 없음
- 그래서 콜백을 사용해서 비동기 호출의 결과를 받음 → 결과가 준비되었을 때 결과를 인자에 넣어 콜백 호출
- 함수형 프로그래밍에서는 콜백 함수로 비동기 함수에서 액션을 빼내는 데 사용

> continuation(후속문. 여기선 콜백)을 전달하는 형태의 프로그래밍 방식을 CPS(Continuation-Passing Style)라고 한다고 합니다..
>
> - [Continuation-passing style](https://en.wikipedia.org/wiki/Continuation-passing_style)
> - [callback, cps, call/cc 그리고 monad](https://velog.io/@jsonkim/callback-cps-callcc)
