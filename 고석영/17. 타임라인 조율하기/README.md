### 버그 코드

- `shipping_ajax()`를 `cost_ajax()` 콜백 안에서 호출하는 대신 `cost_ajax()` 다음 바로 실행하게 변경하면서 버그 발생
- 두 개의 비동기 호출 순서 문제

```tsx
function add_item_to_cart(item) {
  cart = add_item(cart, item)
  update_total_queue(cart)
}

function calc_cart_total(cart, callback) {
  let total = 0
  cost_ajax(cart, function (cost) {
    total += cost
  })

  shipping_ajax(cart, function (shipping) {
    total += shipping
    callback(total)
  })
}

function calc_cart_worker(cart, done) {
  calc_cart_total(cart, function (total) {
    update_total_dom(total)
    done(total)
  })
}

const update_total_queue = DroppingQueue(1, calc_cart_worker)
```

### 단계 1: 액션 확인하기

![image.png](https://drek4537l1klr.cloudfront.net/normand/Figures/figure_17-1.png)

> `total`은 지역변수이지만 액션에 포함

### 단계 2: 모든 액션 그리기

![image.png](https://drek4537l1klr.cloudfront.net/normand/Figures/f0480-01.jpg)

### 단계 3: 다이어그램 단순화하기

- 자바스크립트 스레드를 적용
- 타임라인이 끝나는 곳에서 생기는 타임라인 하나로 합치기

![image.png](https://drek4537l1klr.cloudfront.net/normand/Figures/f0482-02.jpg)

### 실행 가능한 순서 분석하기

![image.png](https://drek4537l1klr.cloudfront.net/normand/Figures/f0484-02.jpg)

- 기존 타임라인 두 응답을 **순서대로** 대기
- 최적화 타임라인: 두 응답을 **병렬로** 대기

## 동시성 기본형

### 모든 병렬 콜백 기다리기

#### 원하는 결과

> 점선(컷)으로 두 콜백이 모두 끝날 때까지 기다린다는 것을 나타내보자.

![image.png](https://drek4537l1klr.cloudfront.net/normand/Figures/f0486-01.jpg)

#### 컷의 장점

- 타임라인을 앞부분과 뒷부분으로 나눠 따로 분석 가능
- 실행 가능한 순서를 줄여 애플리케이션의 복잡성 감소

### 타임라인을 나누기 위한 동시성 기본형

- 서로 기다릴 수 있는 간단하고 재사용 가능한 기본형 필요

> 단일 스레드인 자바스크립트의 장점을 활용하여 가능한 동기적으로 접근하는 간단한 변수로 동시성 기본형

### `Cut()` 동시성 기본형

```tsx
function Cut(num, callback) {
  let num_finished = 0
  return function () {
    num_finished += 1
    if (num_finished === num) {
      callback()
    }
  }
}

const done = Cut(3, function () {
  console.log('3 timelines are finished')
})
```

### 코드에 `Cut()` 적용하기

#### 고려해야 할 사항

- `Cut()`을 보관할 범위 → 두 응답 콜백을 받는 `calc_cart_total()`에 적용
- `Cut()`에 어떤 콜백을 넣을지 → `Cut()`에서 `calc_cart_total()`에서 받은 콜백을 적용

```tsx
function calc_cart_total(cart, callback) {
  let total = 0
  const done = Cut(2, function () {
    callback(total)
  })

  cost_ajax(cart, function (cost) {
    total += cost
    done()
  })
  shipping_ajax(cart, function (cost) {
    total += shipping
    done()
  })
}
```

> - 불확실한 순서 → 병렬 처리하는 액션이 모두 완료될 때까지 기다렸다 순서대로 실행하므로 ✅

- 빠르게 동작하는지 → 병렬 실행으로 ✅
- 여러 번 클릭했을 때→ `Cut()`으로 하나로 합쳐지므로 ✅
  >

#### `Cut()`의 장점

> 복잡성은 바꾸지 않으려고 하는 선택들로부터 생깁니다.

- 비동기 웹 요청
- 결과를 합쳐야 하는 두 개의 API 응답
- 예측 불가능한 사용자의 액션

### 여러 번 실행해도 한 번만 실행하는 `JustOnce()`

```tsx
function sendAddToCartText(number) {
  sendTextAjax(number, `Thanks for adding something to your cart.`)
}

function JustOnce(action) {
  let alreadyCalled = false
  return function (a, b, c) {
    if (alreadyCalled) return
    alreadyCalled = true
    return action(a, b, c)
  }
}

const sendAddToCartTextOnce = JustOnce(sendAddToCartText)
```

### 암묵적 시간 모델을 명시적 시간 모델로

#### 순서

- 순차적 구문은 순서대로 실행
- 두 타임라인에 있는 단계는 왼쪽 먼저 실행되거나, 오른쪽 먼저 실행될 수 있음
- 비동기 이벤트는 새로운 타임라인에서 실행

#### 반복

- 액션은 호출할 때마다 실행

> 암묵적 시간 모델의 실행 방식이 애플리케이션에서 필요한 실행 방식과 딱 맞을 일이 거의 없기 때문에 함수형 사고로 필요한 실행 방식에 가깝게 새로운 시간 모델을 만들어서 사용하자.
