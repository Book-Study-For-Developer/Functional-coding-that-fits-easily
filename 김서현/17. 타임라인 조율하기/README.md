# 17. 타임라인 조율하기

- [cut()으로 모든 병렬 콜백 기다리기](#cut으로-모든-병렬-콜백-기다리기)
- [코드에 Cut() 적용하기](#코드에-cut-적용하기)
- [딱 한 번만 호출하는 기본형](#딱-한-번만-호출하는-기본형)
- [암묵적 시간 모델 vs 명시적 시간 모델](#암묵적-시간-모델-vs-명시적-시간-모델)
- [타임라인 사용법](#타임라인-사용법)

---

### 좋은 타임라인의 원칙

1. 타임라인은 적을수록 이해하기 쉬움
2. 타임라인은 짧을수록 이해하기 쉬움
3. 공유하는 자원이 적을 수록 이해하기 쉬움
4. 자원을 공유한다면 서로 조율해야 함 - 올바른 결과를 주지 않는 실행 순서를 없앰
5. 시간을 일급으로 다룸
    - 17장에서는 시간을 다룰 수 있는 대상으로 생각
    - 액션의 순서와 타이밍을 맞추는 건 어려우나 타임라인을 관리하는 재사용 가능한 객체 만들면 타이밍 문제 쉽게 처리 가능
        - 호출 순서와 반복은 직접 다룰 수 있음

---

## cut()으로 모든 병렬 콜백 기다리기

1. 병렬로 진행하는 최적화 후 추가 버그 발생 
    
    ```tsx
    function add_item_to_cart(item) {
      cart = add_item(cart, item);
      update_total_queue(cart);
    }
    
    function calc_cart_total(cart, callback) {
      var total = 0;
      cost_ajax(cart, function (cost) {
        total += cost;
      // 최적화 작업(닫는 괄호 옮겨짐) 후 추가 버그 발생
      });
      shipping_ajax(cart, function (shipping) {
        total += shipping;
        callback(total);
      });
    }
    
    function calc_cart_worker(cart, done) {
      calc_cart_total(cart, function (total) {
        update_total_dom(total);
        done(total);
      });
    }
    
    function DroppingQueue(max, worker) { ... }
    
    var update_total_queue = DroppingQueue(1, calc_cart_worker);
    ```
    
2. 컷으로 타임라인을 앞부분과 뒷부분으로 나누어 실행 가능한 순서 줄임
    
    ```tsx
    // num: 기다릴 타임라인의 수
    // callback: 모든 것이 끝났을 때 실행할 콜백
    function Cut(num, callback) {
      // 카운터를 0으로 초기화
      var num_finished = 0;
      // 리턴되는 함수는 타임라인이 끝났을 때 호출
      return function () {
        // 함수를 호출할 때마다 카운터가 증가
        num_finished += 1;
        if (num_finished === num) {
          // 마지막 타임라인이 끝났을 때 콜백 호출
          callback();
        }
      };
    }
    ```
    

---

## 코드에 Cut() 적용하기

1. Cut()을 보관할 범위
    - 응답 콜백 끝에서 done() 불러야 하는 상황이라 두 응답 콜백 만드는 calc_cart_total() 함수 범위에 Cut()을 만드는 게 좋을 것
2. Cut()에 어떤 콜백을 넣을지
    
    ```tsx
    function calc_cart_total(cart, callback) {
      var total = 0;
      // Cut() 적용
      var done = Cut(2, function () {
        callback(total);
      });
      cost_ajax(cart, function (cost) {
        total += cost;
        // 추가
        done();
      });
      shipping_ajax(cart, function (shipping) {
        total += shipping;
        // callback(total);
        // 추가
        done();
      });
    }
    ```
    
3. 복잡성의 필요성
    - 우리가 다루는 복잡성
        1. 비동기 웹 요청
        2. 결과를 합쳐야 하는 두 개의 API 응답
        3. 예측 불가능한 사용자의 액션
    - 더 좋은 UX를 위해 보통 복잡성이 생김 - 그게 고려되려면 복잡성은 피할 수 없을 것

---

## 딱 한 번만 호출하는 기본형

```tsx
// 액션을 전달
function JustOnce(action) {
  // 함수가 실행됐는지 기억
  var alreadyCalled = false;
  return function (a, b, c) {
    // 실행한 적이 있다면 바로 종료
    if (alreadyCalled) return;
    // 함수 처음 실행 - 함수가 실행됐다고 생각하고 실행된 사실을 기록함
    alreadyCalled = true;
    // 인자와 함께 액션 호출
    return action(a, b, c);
  };
}
```

---

## 암묵적 시간 모델 vs 명시적 시간 모델

- 자바스크립트의 시간 모델
    1. 순차적 구문은 순서대로 실행됨
    2. 두 타임라인에 있는 단계는 왼쪽 먼저 실행되거나 오른쪽 먼저 실행될 수 있음
    3. 비동기 이벤트는 새로운 타임라인에서 실행됨
    4. 액션은 호출될 때마다 실행됨
- 간단한 프로그램에서 암묵적 시간 모델은 좋으나 실행 방식을 바꾸지 못함
- 함수형 개발자는 필요한 실행 방식에 가깝게 새로운 시간 모델 만듦
    - 비동기 콜백 사용할 때 새로운 타임라인 만들지 않도록 큐 만듦
    - 여러 번 호출해도 한 번만 실행되는 액션을 만들기 위해 JustOnce() 기본형 만듦

---

## 타임라인 사용법

1. 타임라인은 적을수록 이해하기 쉬움 → 타임라인 수를 줄임
    - 스레드, 비동기 호출, 서버 요청 줄여 적은 타임라인 만들면 시스템이 단순해짐
2. 타임라인은 짧을수록 이해하기 쉬움 → 타임라인 길이를 줄임
    - 타임라인에 있는 액션 줄이기
    - 액션을 계산으로 바꾸기
    - 암묵적인 입력과 출력 없애기
3. 공유하는 자원이 적을 수록 이해하기 쉬움 → 공유 자원을 없앰
    - 가능하면 단일 스레드에서 공유 자원에 접근
4. 자원을 공유한다면 서로 조율해야 함 → 동시성 기본형으로 자원을 공유함
    - 자원을 안전하지 않게 공유한다면 큐나 락을 이용해 안전하게 공유할 수 있는 방법으로 바꾸기
5. 시간을 일급으로 다룸 → 동시성 기본형으로 조율함
    - 타임라인을 조율하기 위해 Promise나 컷과 같은 것으로 액션에 순서나 반복을 제한하기
