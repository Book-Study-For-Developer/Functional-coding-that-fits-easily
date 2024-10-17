# 16. 타임라인 사이에 자원 공유하기

- [큐를 사용해 DOM이 업데이트되는 순서 보장하기](#큐를-사용해-dom이-업데이트되는-순서-보장하기)
- [큐를 재사용할 수 있도록 만들기](#큐를-재사용할-수-있도록-만들기)
- [큐를 건너뛰도록 만들기](#큐를-건너뛰도록-만들기)

---

### 좋은 타임라인의 원칙

1. 타임라인은 적을수록 이해하기 쉬움
2. 타임라인은 짧을수록 이해하기 쉬움
3. 공유하는 자원이 적을 수록 이해하기 쉬움
4. 자원을 공유한다면 서로 조율해야 함 - 올바른 결과를 주지 않는 실행 순서를 없앰
   - 16장에서는 타임라인끼리 공유하는 자원 조율 위해 재사용 가능한 방법 만들 것
5. 시간을 일급으로 다룸

---

## 큐를 사용해 DOM이 업데이트되는 순서 보장하기

1. 장바구니에 존재하는 버그 - DOM 자원을 공유해서 두 타임라인 사이에 실행 순서가 중요
   - 첫 번째 클릭, 두 번째 클릭 타임라인 - 왼쪽 다음 오른쪽이 기대한 결과이지만 오른쪽 다음 왼쪽 케이스도 생길 수 있음.
2. DOM이 업데이트되는 순서를 보장해야
   - 오른쪽 먼저 실행되는 것 막아야 하나 DOM 업데이트는 통제 불가능함
   - → 큐 사용
3. 자바스크립트에서 큐 만들기
   - 큐는 자료구조이나, 타임라인 조율에 사용한다면 `동시성 기본형`(자원을 안전하게 공유할 수 있는 재사용 가능한 코드)이라 부름.
   - JS에는 큐 자료구조가 없어서 만들어야 함
4. 큐에서 처리할 작업을 분리 - 동시에 실행되는 것 막기

   ```tsx
   function add_item_to_cart(item) {
     cart = add_item(cart, item);
     // calc_cart_total(cart, update_total_dom);

     // 1. 큐에서 처리할 작업을 큐에 넣기
     // 클릭 핸들러에서 큐에 항목 추가
     update_total_queue(cart);
   }

   // 기존 코드
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

   // 6. 변수와 함수를 함수 범위로 넣기
   // 작업한 모든 코드를 Queue() 함수 내부에 넣어서 전역변수를 Queue()의 지역변수로 만듦
   function Queue() {
     var queue_items = [];
     // 3. 두 번째 타임라인이 첫 번째 타임라인과 동시에 실행되는 것 막기
     // 현재 동작하고 있는 다른 작업이 있는지 확인
     var working = false;

     function runNext() {
       // 3. 두 번째 타임라인이 첫 번째 타임라인과 동시에 실행되는 것 막기
       // 동시에 타임라인 두 개가 동작하는 것을 막음
       if (working) return;
       // 5. 항목이 없을 때 멈추게 하기
       // 큐가 비어있으면 멈춤
       if (queue_items.length === 0) return;
       working = true;

       // 2. 큐에 있는 첫 번째 항목 실행
       // 배열의 첫 번째 항목을 꺼내 카트에 넣음
       var cart = queue_items.shift();
       calc_cart_total(cart, function (total) {
         // 4. 다음 작업을 시작할 수 있도록 콜백 함수 고치기
         // 작업 완료 표시하고 다음 작업 시작
         update_total_dom(total);
         working = false;
         runNext();
       });
     }

     // 6. 변수와 함수를 함수 범위로 넣기
     // update_total_queue로 쓸 함수 리턴
     return function (cart) {
       // 1. 큐에서 처리할 작업을 큐에 넣기
       // 배열 끝에 항목 추가
       queue_items.push(cart);

       // 2. 큐에 있는 첫 번째 항목 실행
       // 큐에 항목 추가하고 워커 시작
       setTimeout(runNext, 0);
     };
   }

   var update_total_queue = Queue();
   ```

---

## 큐를 재사용할 수 있도록 만들기

```tsx
// 2. 워커 행동을 바꿀 수 있도록 밖으로 뺌 - 인자 추가
// function Queue() {
function Queue(worker) {
  var queue_items = [];
  var working = false;

  function runNext() {
    if (working) return;
    if (queue_items.length === 0) return;
    working = true;

    // var cart = queue_items.shift();
    var item = queue_items.shift();

    // 1. done() 함수 빼내기
    // 2. 워커 행동을 바꿀 수 있도록 밖으로 뺌
    // calc_cart_total(cart, function (total) {
    //   update_total_dom(total);
    //   working = false;
    //   runNext();
    // });

    // 3. 작업이 끝났을 때 실행하는 콜백을 받기
    // worker(cart, function () {
    worker(item.data, function (val) {
      working = false;
      // 4. 작업이 완료됐을 때 콜백 부르기
      // item.callback을 비동기로 부름
      setTimeout(item.callback, 0, val);
      runNext();
    });
  }

  // 3. 작업이 끝났을 때 실행하는 콜백을 받기
  // return function(cart) {
  //   queue_items.push(cart);
  return function (data, callback) {
    queue_items.push({
      // 배열에 데이터와 콜백 모두 넣음
      data: data,
      callback: callback || function () {},
    });
    setTimeout(runNext, 0);
  };
}

// 1. done() 함수 빼내기
// calc_cart_worker()의 콜백함수를 done()이라 이름짓고 빼냄
// 큐에서 하는 일(calc_cart_total() 부르기)과
// 큐를 반복해 처리하는 코드(runNext()를 부르는 코드) 분리
// 2. 워커 행동을 바꿀 수 있도록 밖으로 뺌
// 일반적인 큐 만들기 위해 특정 도메인에 한정적인 기능 분리
// Queue 함수의 인자로 넘겨 사용
// 특정 도메인에 한정됨 - 구체적 인자명 사용
function calc_cart_worker(cart, done) {
  calc_cart_total(cart, function (total) {
    update_total_dom(total);
    done(total);
  });
}

var update_total_queue = Queue(calc_cart_worker);
```

- Queue()
  - 액션에 새로운 능력을 줄 수 있는 고차함수
  - 액션에 순서를 보장해줌
  - 동시성 기본형(자원을 안전하게 공유할 수 있는 재사용 가능한 코드)

---

## 큐를 건너뛰도록 만들기

- 기존 큐: 작업이 끝나야 다음 작업 시작됨 → 연속 요청 들어올 경우 느림
- 드로핑 큐: 새로운 작업이 들어오면 건너뛸 수 있음

```tsx
// function Queue(worker) {
// max: 보관할 수 있는 최대 큐 크기 넘김
function DroppingQueue(max, worker) {
  var queue_items = [];
  var working = false;

  function runNext() {
    if (working) return;
    if (queue_items.length === 0) return;
    working = true;
    var item = queue_items.shift();
    worker(item.data, function (val) {
      working = false;
      setTimeout(item.callback, 0, val);
      runNext();
    });
  }

  return function (data, callback) {
    queue_items.push({
      data: data,
      callback: callback || function () {},
    });
    // 큐에 추가한 후에 항목이 max 넘으면 모두 버림
    while (queue_items.length > max) {
      queue_items.shift();
    }
    setTimeout(runNext, 0);
  };
}

function calc_cart_worker(cart, done) {
  calc_cart_total(cart, function (total) {
    update_total_dom(total);
    done(total);
  });
}

// var update_total_queue = Queue(calc_cart_worker);
// 한 개 이상은 모두 버림
var update_total_queue = DroppingQueue(1, calc_cart_worker);
```
