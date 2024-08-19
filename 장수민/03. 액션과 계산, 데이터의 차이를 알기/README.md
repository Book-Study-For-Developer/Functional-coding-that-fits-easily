> **Note**
> 3장의 목표

- 액션과 계산, 데이터가 어떻게 다른지 알아보기
- 문제에 대해 생각하거나 코드를 작성할 때, 읽을 때 액션과 계산, 데이터를 구분해서 적용해보기
- 액션이 코드 전체로 퍼질 수 있다는 것을 이해하기
- 이미 있는 코드에서 어떤 부분이 액션인지 찾아보기

## 액션과 계산, 데이터는 어디에나 적용할 수 있습니다

(책이랑 조금 다르게 헬스장 가는 과정을 예로 들었습니다.)

```
1. 헬스장 열었는지 확인하기
2. 걸어서 헬스장 가기
3. 운동하기
4. 걸어서 다시 집으로 오기
```

위 타임라인은 모두 액션이다.

1. _헬스장이 닫는 날은 불규칙합니다._
2. _걷는 행위를 연속으로 두 번하면 그만큼 힘들어서 운동을 제대로 하지 못하게 됩니다._
3. _컨디션에 따라 운동하려는 부위가 달라질 수도 있고 내 의지와는 달리 기구에 사람이 차 있어서 다른 운동을 하게 될 수도 있어서 시점이 중요합니다._
4. 2번처럼 동일하게 액션입니다.

<br>

이제 함수형 프로그래밍을 적용해서 액션과 계산, 데이터로 다시 나눠보자.

![헬스장 가기 타임라인](./헬스장%20가기%20타임라인.png)

> **Note**
> 액션과 계산, 데이터로 나눌 때 참고하면 좋을 점

- 걸어서 헬스장 가기 액션에는 헬스장 위치나 걸어가는 경로가 데이터로 포함되어 있긴 하지만, 관심사가 아니므로 분리하지 않아도 된다.
- 액션 안에는 계산과 데이터, 또 다른 액션이 숨어 있을 수 있다. 처음에는 '운동하기'였던 액션이 '컨디션 확인하기', '기구 사용 현황 알기', '운동하기'로 다시 세분화됐다.
- 계산은 더 작은 계산과 데이터로 나누고 연결할 수 있다.
- 데이터는 데이터만 조합할 수 있으므로, 가장 먼저 데이터를 찾는 일이 중요하다.

## 데이터에 대해 자세히 알아보기

**일어난 일의 결과를 기록한 것이자, 이벤트에 대한 사실**

- 데이터 구조로 의미를 담을 수 있다.
- 장점으로는,
  - 직렬화된 데이터는 저장했다가 읽기 쉽다.
  - 동일한지 비교하기 쉽다.
  - 데이터는 여러 가지 방법으로 해석하기 좋다.
- 단점으로는,
  - 해석이 반드시 필요하다. 만약 없다면, 공간만 차지하는 바이트일 뿐이다.

## 구독자들에게 쿠폰 보내기 서비스에 함수형 사고 적용하기

액션(A)과 계산(C), 데이터(D)로 순서 상관 없이 분류해보겠습니다.

- 이메일 보내기(A)
- 데이터베이스에서 구독자 가져오기(A)
- 쿠폰 등급(D)
- 데이터베이스에서 쿠폰 정보 가져오기(A)
- 이메일 제목(D)
- 이메일 주소(D)
- 추천 수(D)
- 어떤 이메일이 쿠폰 받을지 결정하기(C)
- 구독자 DB 레코드(D)
- 쿠폰 DB 레코드(D)
- 쿠폰 목록 DB 레코드(D)
- 구독자 목록 DB 레코드(D)
- 이메일 본문(D)

## 쿠폰 보내는 과정 구현하기

**데이터를 파악하는 것으로 시작해서 계산과 추가 데이터를 도출한 뒤에, 마지막에 액션으로 모두 묶기**

```js
// 데이터베이스에서 가져온 구독자 데이터
const subscriber = {
  email: "sam@pmail.com",
  rec_count: 16,
};

// 쿠폰 등급 매기기(계산) & 반환되는 쿠폰 등급 데이터
function subCouponRank(subscriber) {
  if (subscriber.rec_count >= 10) return "best";
  else return "good";
}

// 데이터베이스에서 가져온 쿠폰 데이터
const coupon = {
  code: "10PERCENT",
  rank: "bad",
};

// 특정 등급의 쿠폰 목록 선택하기(계산)
function selectCouponsByRank(coupons, rank) {
  return coupons.filter((coupon) => coupon.rank === rank);
}

// 이메일 데이터
const message = {
  from: "newsletter@coupondog.co",
  to: "sam@pmail.com",
  subject: "Your weekly coupons inside",
  body: "Here are your coupons ...",
};

// 구독자가 받을 이메일을 계획하는 계산
function emailForSubscriber(subscriber, goods, bests) {
  // 계산 하위에 다른 계산
  const rank = subCouponRank(subscriber);
  if (rank === "best") {
    return {
      from: "newsletter@coupondog.co",
      to: subscriber.email,
      subject: "Your weekly coupons inside",
      body: `Here are your coupons: ${bests.join(", ")}`,
    };
  } else {
    return {
      from: "newsletter@coupondog.co",
      to: subscriber.email,
      subject: "Your weekly coupons inside",
      body: `Here are your coupons: ${goods.join(", ")}`,
    };
  }
}

// 이메일 목록 준비하기(계산)
function emailsForSubscribers(subscribers, goods, bests) {
  return subscribers.map((subscriber) =>
    emailForSubscriber(subscriber, goods, bests)
  );
}

// 이메일 보내기(액션)
function sendIssue() {
  const coupons = fetchCouponsFromDB(); // 액션
  const goodCoupons = selectCouponsByRank(coupons, "good"); // 계산
  const bestCoupons = selectCouponsByRank(coupons, "best"); // 계산
  const subscribers = fetchSubscribersFromDB(); // 액션
  const emails = emailsForSubscribers(subscribers, goodCoupons, bestCoupons); // 계산

  emails.forEach((email) => {
    emailSystem.send(email); // 액션
  });
}
```

## 계산에 대해 자세히 알아보기

**입력값으로 출력값을 만드는 것으로, 실행 시점과 횟수에 관계없이 항상 같은 입력에 대해서는 출력을 반환하는 순수 함수**

- 계산에는 연산을 담을 수 있다. 알고리즘 문제를 해결하기 위한 함수를 작성하고 연산을 하도록 시키는 것도 계산이다.
- 액션보다 계산이 좋은 이유는,
  - 테스트하기 쉽다.
  - 기계적인 분석이 쉽다.
  - 계산은 조합하기 쉽다.
- 계산을 쓰면서 걱정하지 않아도 될 것들은,
  - 동시에 실행되는 것
  - 과거에 실행되었거나 미래에 실행할 것
  - 실행 횟수
- 계산의 단점으로는,
  - 결국 입력값으로 실행을 거쳐야 결과를 알 수 있는 블랙박스 구조

## 이미 있는 코드에 함수형 사고 적용하기

```js
function figurePayout(affiliate) {
  const owed = affiliate.sales * affiliate.commission;
  if (owed > 100) sendPayout(affiliate.bank_code, owed); // 액션
}

function affiliatePayout(affiliates) {
  affiliates.forEach((affiliate) => {
    figurePayout(affiliate); // 액션
  });
}

function main(affiliates) {
  affiliatePayout(affiliates); // 액션
}
```

> **Warn**
> 위 코드는 액션이 프로그램 전체로 퍼진 모습이다.

- 함수 내부에서 액션을 부르고 있다면, 그 함수도 액션이 된다.

### 액션은 다양한 형태로 나타납니다

자바스크립트에서 자주 발생할 수 있는 액션의 예시로는,

- `window.alert`: 팝업 창 띄우기
- `console.log`: 콘솔 출력
- `new Date()`: 부르는 시점에 따라 다른 출력값
- 변수 참조, 속성 참조, 배열 참조: 변수나 객체, 배열이 공유되어 있으면서 변경 가능한 객체라면 참조하는 시점에 따라 값이 다를 수 있다.
- 값 할당: 공유하기 위해 값을 할당했고 변경 가능하다면 다른 코드에 영향을 주기 때문에 액션이다.
- 속성 삭제: 다른 코드에 영향을 주기 때문에 액션이다.

### 액션에 대해 자세히 알아보기

**외부 세계(전역)에 영향을 주거나 받는 것으로, 실행 시점과 횟수에 의존한다.**

- 액션으로 외부 세상에 영향을 줄 수 있으므로, 어떤 일을 하려는지 아는 것이 중요하다.
- 일반적으로 순수하지 않은 함수, 혹은 부수 효과 함수라고 부른다.
- 액션은 사용하기 어렵지만, 필요할 때는 사용해야 한다.
- 그 때 잘 사용하기 위해서는,
  - 가능한 액션을 적게 사용해야 한다.
  - 액션은 가능한 작게 만든다. 결정이나 계획과 관련된 계산은 빼낸다.
  - 내부에 계산과 데이터만 있고 가장 바깥쪽에 액션이 있는 구조가 이상적이다.
  - 액션이 호출 시점에 의존하는 것을 제한한다.

<hr>

### 느낀 점

- 책에서 데이터와 계산을 사용할 때 어떤 장점이 있는지 적은 반면에, 액션은 그런 내용이 없는 것을 보아하니 확실히 액션을 신중히 다뤄야 한다는 게 느껴진다.

### 의문이 드는 점

```js
function calculateOwed(affiliate) {
  // 계산
  return affiliate.sales * affiliate.commission;
}

function main(affiliates) {
  // 액션
  affiliates.forEach((affiliate) => {
    const owed = calculateOwed(affiliate);
    if (owed > 100) sendPayment(affiliate.bank_code, owed);
  });
}
```

- 이렇게 계산을 빼내고, 액션이 최대한 중첩되지 않게 코드를 짠다면 괜찮을까요?
