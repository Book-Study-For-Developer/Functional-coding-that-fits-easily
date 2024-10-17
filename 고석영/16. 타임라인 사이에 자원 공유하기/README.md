> #### 좋은 타임라인의 원칙

- 타임라인이 적을수록 이해하기 쉽습니다.
- 타임라인은 짧을수록 이해하기 쉽습니다.
- 공유하는 자원이 적을수록 이해하기 쉽습니다.
- 자원을 공유한다면 서로 조율해야 합니다.
- 시간을 일급으로 다룹니다.
  >

## DOM 자원을 공유하며 생기는 버그

```tsx
function add_item_to_cart(item) {
  cart = add_item(cart, item)
  calc_cart_total(cart, update_total_dom)
}

function calc_cart_total(cart, callback) {
  let total = 0
  cost_ajax(cart, function (cost) {
    total += cost
    shipping_ajax(cart, function (shipping) {
      total += shipping
      callbakc(total)
    })
  })
}
```

#### 타임라인 다이어그램

![image.png](https://drek4537l1klr.cloudfront.net/normand/Figures/f0443-01.jpg)

- 발생 원인: DOM 자원을 공유하면서 실행 순서에 문제가 생겨 발생
- 발생 내용: 첫 번째 DOM 업데이트가 나중에 실행되면 올바른 결과인 두 번째 결과를 덮어쓰게 됨
- 해결 방법: 업데이트 순서를 제한해 해결 → **큐!**

> [!NOTE]
>
> #### 큐(queue)
>
> - 넣은 순서대로 항목을 꺼낼 수 있는 데이터 구조
> - 주로 여러 타임라인에 있는 액션 순서를 조율하기 위해 사용
> - 큐를 통해 공유 자원을 안전하게 공유 가능

## 자바스크립트에서 큐 만들기

> 자바스크립트에는 큐 자료 구조가 없기 때문에 만들어야 합니다.

- 큐를 타임라인 조율에 사용하면 **동시성 기본형**이라 함
- 자바스크립트는 동시성 기본형으로 기본적으로 제공하지 않기 때문에 직접 만들어야 함

> [!NOTE]
>
> #### 동시성 기본형(concurrency primitive)
>
> - 자원을 안전하게 공유할 수 있는 재사용 가능한 코드

### 큐에서 처리할 작업을 큐에 넣기

> 큐에서 처리할 작업을 `큐에 추가`라는 하나의 액션으로 바꿔주자.

#### 타임라인 다이어그램

![image.png](https://drek4537l1klr.cloudfront.net/normand/Figures/f0449-01.jpg)

#### 주요 내용

- `queue_items`: 배열 큐 만들기
- `update_total_queue()`: 큐에 항목 추가

```tsx
function add_item_to_cart(item) {
  cart = add_item(cart, item)
  update_total_queue(cart) // 클릭 핸들러에서 큐에 항목 추가하기
}

function calc_cart_total(cart, callback) {
  let total = 0
  cost_ajax(cart, function (cost) {
    total += cost
    shipping_ajax(cart, function (shipping) {
      total += shipping
      callbakc(total)
    })
  })
}

// 큐
let queue_items = []

// 큐에 추가하고...
function update_total_queue(cart) {
  queue_item.push(cart)
}
```

### 큐에 있는 첫 번째 항목 실행하기

> 큐에 가장 앞에 있는 항목을 꺼내 작업을 시작하자.

#### 주요 내용

- `runNext()`: 배열에 첫 번재 항목을 꺼내 cart에 넣기
- `setTimeOut()`: 이벤트 루프에 작업을 추가 및 워커 실행

```tsx
function add_item_to_cart(item) {
  cart = add_item(cart, item)
  update_total_queue(cart)
}

function calc_cart_total(cart, callback) {
  let total = 0
  cost_ajax(cart, function (cost) {
    total += cost
    shipping_ajax(cart, function (shipping) {
      total += shipping
      callbakc(total)
    })
  })
}

let queue_items = []

function runNext() {
  const cart = queue_items.shift()
  calc_cart_total(cart, update_total_dom)
}

function update_total_queue(cart) {
  queue_item.push(cart)
  setTimeout(runNext, 0)
}
```

### 두 타임라인이 동시에 실행되는 것을 막기

> 이미 실행되는 작업이 있는지 확인해서 두 타임라인이 섞이지 않도록 만들어 보자.

#### 타임라인 다이어그램

![image.png](https://drek4537l1klr.cloudfront.net/normand/Figures/f0450-02.jpg)

#### 주요 내용

- `working`: 실행중인 작업이 있는지 확인하는 플래그
- 플래그를 통해 실행중인 작업을 확인하고 동시에 두 개가 동작하는 것을 방지

```tsx
function add_item_to_cart(item) {
  cart = add_item(cart, item)
  update_total_queue(cart)
}

function calc_cart_total(cart, callback) {
  let total = 0
  cost_ajax(cart, function (cost) {
    total += cost
    shipping_ajax(cart, function (shipping) {
      total += shipping
      callbakc(total)
    })
  })
}

let queue_items = []
let working = false

function runNext() {
  if (working) return
  working = true
  const cart = queue_items.shift()
  calc_cart_total(cart, update_total_dom)
}

function update_total_queue(cart) {
  queue_item.push(cart)
  setTimeout(runNext, 0)
}
```

### 현재 작업이 끝났을 때 다음 작업을 실행하기

> `calc_cart_totla()`에 새로운 콜백을 전달해 다음 작업을 실행해보자.

#### 타임라인 다이어그램

![image.png](https://drek4537l1klr.cloudfront.net/normand/Figures/f0451-02.jpg)

#### 주요 내용

- 비동기로 작업을 이어서 할 수 있는 반복 구조로 만들기 → **재귀**
- 작업 완료를 표시하고 다음 작업을 시작 → 반복..

```tsx
function add_item_to_cart(item) {
  cart = add_item(cart, item)
  update_total_queue(cart)
}

function calc_cart_total(cart, callback) {
  let total = 0
  cost_ajax(cart, function (cost) {
    total += cost
    shipping_ajax(cart, function (shipping) {
      total += shipping
      callbakc(total)
    })
  })
}

let queue_items = []
let working = false

function runNext() {
  if (working) return

  working = true
  const cart = queue_items.shift()
  calc_cart_total(cart, function (total) {
    update_total_dom(total)
    working = false
    runNext()
  })
}

function update_total_queue(cart) {
  queue_item.push(cart)
  setTimeout(runNext, 0)
}
```

### 항목이 없을 때 멈추게 하기

> 배열이 바닥이 났다면 실행을 멈추고 종료하게 만들어 보자.

#### 타임라인 다이어그램

![image.png](https://drek4537l1klr.cloudfront.net/normand/Figures/f0452-02.jpg)

#### 주요 내용

- `queue_items.length === 0`로 큐가 비었을 때 종료

```tsx
function add_item_to_cart(item) {
  cart = add_item(cart, item)
  update_total_queue(cart)
}

function calc_cart_total(cart, callback) {
  let total = 0
  cost_ajax(cart, function (cost) {
    total += cost
    shipping_ajax(cart, function (shipping) {
      total += shipping
      callbakc(total)
    })
  })
}

let queue_items = []
let working = false

function runNext() {
  if (working) return
  if (queue_items.length === 0) return

  working = true
  const cart = queue_items.shift()
  calc_cart_total(cart, function (total) {
    update_total_dom(total)
    working = false
    runNext()
  })
}

function update_total_queue(cart) {
  queue_item.push(cart)
  setTimeout(runNext, 0)
}
```

### 변수와 함수를 함수 범위로 넣기

> 큐에서 전역변수를 사용하고 있다. 함수로 감싸서 전역변수 사요을 지역변수로 바꿔보자.

#### 주요 내용

- `Queue()`: 전역변수와 사용하는 함수를 넣어 다른 곳에서 접근할 수 없게 감싸기 → 클로저 🤩
- 모든 큐 워커 코드를 `Queue()`에 집어넣고 함수를 리턴

```tsx
function Queue() {
  let queue_items = []
  let working = false

  function runNext() {
    if (working) return
    if (queue_items.length === 0) return

    working = true
    const cart = queue_items.shift()
    calc_cart_total(cart, function (total) {
      update_total_dom(total)
      working = false
      runNext()
    })
  }

  return function (cart) {
    queue_item.push(cart)
    setTimeout(runNext, 0)
  }
}

const update_total_queue = Queue()
```

> [!NOTE]
>
> #### 왜 콜백 안에서 `runNext()`를 호출해야 하나요? `calc_cart_total()` 다음에 `runNext()`를 호출하면 안 되나요?
>
> - `calc_cart_total()`이 비동기 호출이기 때문 → 미래 어떤 시점에 호출되는 작업을 포함
> - `runNext()`가 `calc_cart_total()` 다음에 있다면 ajax 요청이 진행되는 동안 다음 항목이 시작되어 버린다 😱

## 큐를 재사용할 수 있도록 만들기

### done() 함수 빼내기

> `함수 본문을 콜백으로 바꾸기` 리팩터링으로 큐를 반복해서 처리하는 코드와 큐에서 하는일 분리해 보자.

#### 주요 내용

- `worker()`: `total`을 업데이트하고 콜백함수 `done()`을 호출해 다음 작업을 진행
- `done()`: `worker()`에 콜백 함수 → 여기서는 진행중 작업을 확인하고 다음 작업을 하는 함수를 전달

> ❓ 질문...여기서 왜 total을 받을까요...😭

```tsx
function Queue() {
  let queue_items = []
  let working = false

  function runNext() {
    if (working) return
    if (queue_items.length === 0) return

    working = true
    const cart = queue_items.shift()
    function worker(cart, done) {
      calc_cart_total(cart, function (total) {
        update_total_dom(total)
        done(total)
      })
    }

    worker(cart, function () {
      working = false
      runNext()
    })
  }

  return function (cart) {
    queue_item.push(cart)
    setTimeout(runNext, 0)
  }
}

const update_total_queue = Queue()
```

### 워커 행동을 바꿀 수 있도록 밖으로 빼내기

> 장바구니에 추가하는 일만 하던 큐를 일반적인 큐로 바꾸기

#### 주요 내용

- `calc_cart_worker()`: 함수 안에 있던 work()를 밖으로 빼낸 함수 → 구체적인 함수명 사용
- `Queue(worker)`: worker를 인자로 → 일반적인 인자명 사용

```tsx
function Queue(worker) {
  let queue_items = []
  let working = false

  function runNext() {
    if (working) return
    if (queue_items.length === 0) return

    working = true
    const cart = queue_items.shift()

    worker(cart, function () {
      working = false
      runNext()
    })
  }

  return function (cart) {
    queue_item.push(cart)
    setTimeout(runNext, 0)
  }
}

function calc_cart_worker(cart, done) {
  calc_cart_total(cart, function (total) {
    update_total_dom(total)
    done(total)
  })
}

const update_total_queue = Queue(calc_cart_worker)
```

### 작업이 끝났을 때 실행하는 콜백 받기

> 작업 데이터와 콜백을 작은 객체로 만들어 큐에 넣기

#### 주요 내용

- `data`: 큐에 추가할 실제 데이터
- `callback`
  - 작업 완료 후 호출될 함수로 undefined일 경우 빈 함수 `() => {}`사용되도록 설정 → `callback`이 없는 경우에도 코드가 오류 없이 실행될 수 있도록 보장

```tsx
function Queue(worker) {
  let queue_items = []
  let working = false

  function runNext() {
    if (working) return
    if (queue_items.length === 0) return

    working = true
    const item = queue_items.shift()

    worker(item.data, function () {
      working = false
      runNext()
    })
  }

  return function (data, callback) {
    queue_items.push({
      data,
      callback: callback ?? () => {}
    })
    setTimeout(runNext, 0)
  }
}

function calc_cart_worker(cart, done) {
  calc_cart_total(cart, function (total) {
    update_total_dom(total)
    done(total)
  })
}

const update_total_queue = Queue(calc_cart_worker)
```

### 작업이 완료되었을 때 콜백 부르기

> 이제 작업 데이터와 함께 저장한 콜백을 불러보자.

```tsx
function Queue(worker) {
  let queue_items = []
  let working = false

  function runNext() {
    if (working) return
    if (queue_items.length === 0) return

    working = true
    const item = queue_items.shift()

    worker(item.data, function (val) {
      working = false
      setTimeout(item.callback, 0, val)
      runNext()
    })
  }

  return function (data, callback) {
    queue_items.push({
      data,
      callback: callback ?? () => {}
    })
    setTimeout(runNext, 0)
  }
}

function calc_cart_worker(cart, done) {
  calc_cart_total(cart, function (total) {
    update_total_dom(total)
    done(total)
  })
}

const update_total_queue = Queue(calc_cart_worker)
```

> [!NOTE]
>
> #### `Queue()`
>
> **`Queue()`는 어떤 함수를 새로운 타임라인에서 실행하고 한 번에 한 타임라인만 실행할 수 있도록 만들어주는 고차 함수**
>
> - 액션에 순서를 보장
> - 동시성 기본형
> - 모든 실행 가능한 순서를 제한하면서 동작
> - 기대하지 않는 실행 순서를 없애면 코드가 기대한 순서로 동작한다는 것을 보장 가능

### 공유하는 자원

- 장바구니 전역변수 → 모두 같은 박스에서 동기적으로 실행 ✅
- DOM → 큐를 사용해 DOM 업데이트를 같은 타임라인에서 하도록 변경 ✅
- 큐 → 모든 타임라인에서 서로 다른 네 단계에서 사용중.. → ✅ … 하지만 너무 느려요!

## 큐를 건너뛰도록 만들기

> 빠르게 4번 클릭한다면?…
>
> 결과는 마지막 합계만 DOM에 표시되면 되니까
> 드로핑 큐로 큐에 있는 마지막 업데이트만 처리할 수 있도록 하자.

### 드로핑 큐

#### 주요 내용

- `max`: 보관할 수 있는 최대 큐 크기
- `queue_items.length ≥ max`: max를 넘기는 항목들 버리기
- 아무리 빨리 항목을 추가해도 큐 항목이 한 개 이상 늘어나지 않음!

```tsx
function DroppingQueue(max, worker) {
  let queue_items = []
  let working = false

  function runNext() {
    if (working) return
    if (queue_items.length === 0) return

    working = true
    const item = queue_items.shift()

    worker(item.data, function () {
      working = false
      runNext()
    })
  }

  return function (data, callback) {
    queue_items.push({
      data,
      callback: callback ?? () => {}
    })
    while(queue_items.length >= 0)
      queue_items.shift()
    setTimeout(runNext, 0)
  }
}

function calc_cart_worker(cart, done) {
  calc_cart_total(cart, function (total) {
    update_total_dom(total)
    done(total)
  })
}

const update_total_queue = DroppingQueue(1, calc_cart_worker)function DroppingQueue(max, worker) {
  let queue_items = []
  let working = false

  function runNext() {
    if (working) return
    if (queue_items.length === 0) return

    working = true
    const item = queue_items.shift()

    worker(item.data, function () {
      working = false
      runNext()
    })
  }

  return function (data, callback) {
    queue_items.push({
      data,
      callback: callback ?? () => {}
    })
    while(queue_items.length >= 0)
      queue_items.shift()
    setTimeout(runNext, 0)
  }
}

function calc_cart_worker(cart, done) {
  calc_cart_total(cart, function (total) {
    update_total_dom(total)
    done(total)
  })
}

const update_total_queue = DroppingQueue(1, calc_cart_worker)
```
