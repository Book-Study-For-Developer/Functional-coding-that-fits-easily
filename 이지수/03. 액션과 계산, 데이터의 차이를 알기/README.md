## 액션과 계산, 데이터

### 액션

- 실행 시점과 횟수에 의존
- 부수효과, 부수효과가 있는 함수, 순수하지 않은 함수
- ex) 이메일 보내기, 데이터베이스 읽기

### 계산

- 입력으로 출력을 계산
- 순수함수, 수학함수
- ex) 최대값 찾기, 이메일 주소가 올바른지 확인하기

### 데이터

- 이벤트에 대한 사실
- ex) 사용자가 입력한 이메일 주소, 은행 API로 읽은 달러 수량
- 문제에 대해 생각할 때(코딩 전), 코딩할 때, 코드를 읽을 때 액션, 계산, 데이터를 구분하는 기술을 적용해 볼 수 있음

## 액션과 계산, 데이터는 어디에나 적용할 수 있습니다

```jsx
A -> 액션
C -> 계산
D -> 데이터
```

장보기 과정을 보면

```jsx
냉장고 확인하기(A) → 운전해서 상점으로 가기(A) → 필요한 것 구입하기(A) → 운전해서 집으로 오기(A)
```

단순히 모든 것이 액션으로 분류하지말고 액션에 숨어 있는 다른 액션이나 계산 또는 데이터를 찾아 나누는 것이 좋다

ex) 냉장고 확인하기

```jsx
현재 재고(D)
```

ex) 필요한 것 구입하기

```jsx
현재 재고(D) → 필요한 재고(D) → 재고 빼기(C) → 장보기 목록(D) → 목록에 있는 것 구입하기(A)
```

필요한 재고 - 현재 재고 = 장보기 목록

## 장보기 과정에서 배운 것

1. 액션과 계산, 데이터는 어디에나 적용 가능
2. 액션 안에는 계산과 데이터, 또 다른 액션이 숨어있을지도 모름
3. 계산은 더 작은 계산과 데이터로 나누고 연결 가능
   - 계산을 더 작은 계산으로 나눌 수 있는 상황도 있음
   - 첫번째 계산의 결과 데이터가 두번째 계산의 입력이 됨
4. 데이터는 데이터만 조합 가능
5. 계산은 때로 ‘우리 머리속에서’ 일어남
   - 장보다가 갑자기 장보기 목록을 작성하지 않는 것처럼 머리속에서 나도 모르게 일어남
   - 어떤 단계에서 무엇인가 결정해야 할 것이 있는지, 또는 무엇인가 계획해서 방법을 찾아야 할 것이 있는지를 체크
   - 결정과 계획이 계산이 될 가능성이 높음

> 💡 **데이터 자세히 알아보기**
>
> 이벤트에 대한 사실, 일어난 일의 결과
>
> 1. **카피-온-라이트 (Copy-on-Write)**: 변경할 때 복사본 만들기
> 2. **방어적 복사 (Defensive Copy)**: 보관하려고 하는 데이터의 복사본을 만들기
>
> ### 데이터의 장점
>
> - **직렬화**: 직렬화된 데이터는 전송하거나 저장했다가 읽기가 쉬움
> - **동일성 비교**: 계산이나 액션은 서로 비교하기가 어려움
> - **자유로운 해석**: 여러 가지 방법으로 해석 가능
>
> ### 데이터의 단점
>
> - **해석 필요**: 해석하지 않은 데이터는 쓸모없는 바이트일 뿐
>
> 데이터는 언제나 쉽게 해석할 수 있도록 표현하는 것이 함수형 프로그래밍에서 중요한 기술!

## 새로 만드는 코드에 함수형 사고 적용하기

구독자들에게 이메일로 쿠폰을 매주 보내주는 서비스

친구를 10명 추천하면 더 좋은 쿠폰을 보내주려고 함

쿠폰은 ‘bad’, ‘good’, ‘best’등급이 있음

- bad: 사용 x
- good: 모든 사용자들에게 전달
- best: 추천을 많이 한 사용자를 위한 쿠폰

코드 작성에 있어서 필요한 단계를 적어보기

- 이메일 보내기
- 데이터베이스에서 구독자 가져오기
- 쿠폰에 등급 매기기
- 추천한 친구 수 가져오기
- 추천한 친구수에 따라 어떤 쿠폰 받을 지 결정하기

```jsx
- 이메일 보내기 A
- 데이터베이스에서 구독자 가져오기 A
- 쿠폰에 등급 매기기 D
- 데이터베이스에서 쿠폰 읽기 A
- 이메일 제목 D
- 이메일 주소 D
- 추천 수 D
- 어떤 이메일이 쿠폰을 받을지 결정하기 C
- 구독자 DB 레코드 D
- 쿠폰 DB 레코드 D
- 구독자 목록 DB 레코드 D
- 이메일 본문 D
```

### 쿠폰 보내는 과정 그려보기

1. 데이터베이스에서 구독자 가져오기
2. 데이터베이스에서 쿠폰 목록 가져오기
3. 보내야할 이메일 목록 만들기
4. 이메일 전송하기

### 쿠폰 보내는 과정 구현하기

```jsx
// 데이터베이스에서 가져온 구독자 데이터
var subscribe = {
  email: "sam@pmail.com",
  rec_count: 16,
};

// 쿠폰 등급은 문자열
var rank1 = "best";
var rank2 = "good";

// 쿠폰 등급을 결정하는 함수
function subCouponRank(subscriber) {
  if (subscriber.rec_count >= 10) {
    return "best";
  } else {
    return "good";
  }
}

// 데이터베이스에서 가져온 쿠폰 데이터
var coupon = {
  code: "10PERCENT",
  rank: "bad",
};

// 특정 등급의 쿠폰 목록을 선택하는 계산
function selectCouponsByRank(coupons, rank) {
  var ret = [];
  for (var c = 0; c < coupons.length; c++) {
    var coupon = coupons[c];
    if (coupon.rank === rank) {
      ret.push(coupon.code);
    }
    return ret;
  }
}

// 이메일 데이터
var message = {
  from: "newsletter@coupondog.co",
  to: "sam@pmail.com",
  subject: "Your weekly coupons inside",
  body: "Here are your coupons...",
};

// 구독자가 받을 이메일을 계획하는 계산
function emailForSubscriber(subscriber, goods, bests) {
  var rank = subCouponRank(subscriber);
  if (rank === "best") {
    return {
      from: "newsletter@coupondog.co",
      to: "sam@pmail.com",
      subject: "Your best weekly coupons inside",
      body: "Here are your the best coupons: " + bests.join(", "),
    };
  } else {
    return {
      from: "newsletter@coupondog.co",
      to: "sam@pmail.com",
      subject: "Your good weekly coupons inside",
      body: "Here are your the good coupons: " + goods.join(", "),
    };
  }
}

// 보낸 이메일 목록 준비하기
function emailsForSubscribers(subscribers, goods, bests) {
  var emails = [];
  for (var s = 0; s < subscribers.length; s++) {
    var subscriber = subscribers[s];
    var email = emailForSubscriber(subscriber, goods, bests);
    emails.push(email);
  }
  return emails;
}

// 이메일 보내기 - 액션으로 모든 기능을 하나로 묶음
function sendIssue() {
  var coupons = fetchCouponsFromDB();
  var goodCoupons = selectCouponsByRank(coupons, "good");
  var bestCoupons = selectCouponsByRank(coupons, "best");
  var subscribers = fetchSubscribersFromDB();
  var emails = emailsForSubscribers(subscribers, goodCoupons, bestCoupons);
  for (var e = 0; e < emails.length; e++) {
    var email = emails[e];
    emailSystem.send(email);
  }
}
```

데이터를 먼저 구현하고 계산을 구현한 다음에 마지막으로 액션을 구현하는 것이 함수형 프로그래밍의 일반적인 순서!

> 💡 **계산 자세히 알아보기**
>
> 입력값으로 출력값을 만드는 것
>
> 실행 시점과 횟수에 관계없이 항상 같은 입력값에 대해 같은 출력값을 줌
>
> 순수함수, 수학함수
>
> ### 액션보다 계산이 좋은 점
>
> - 테스트하기 쉬움
> - 기계적인 분석이 쉬움
> - 조합하기 좋음
>
> 동시에 실행되는 것, 과거에 실행되었던 것이나 미래에 실행할 것, 실행 횟수를 신경쓰지 않아도 됨
>
> ### 계산의 단점
>
> - 실행하기 전에 어떤 일이 발생할지 알 수 없음

## 이미 있는 코드에 함수형 사고 적용하기

```jsx
function figurePayout(affiliate) {
  var owed = affiliate.sales * affiliate.commission;
  if (owed > 100) {
    sendPayout(affiliate.bank_cod, owed);
  }
}

function affiliatePayout() {
  for (var a = 0; a < affiliates.length; a++) {
    figurePayout(afiliates[a]);
  }
}

function main(affiliates) {
  affiliatePayout(affiliates);
}
```

위 코드에 액션이 1개라고 생각할 수 있지만 모든 함수가 액션이 된다

왜냐하면 `sendPayout` 함수가 액션이고 이걸 호출하는 함수 `figurePayout`도 액션이 되고 `figurePayout`를 호출하는 `main`함수도 액션이 됨

액션을 부르는 함수가 액션이 되고 그 함수를 부르는 함수도 액션이 되어 액션 한개가 코드 전체로 퍼져 나감

⇒ 코드가 호출 시점이나 횟수에 의존하는지를 체크해보기

> 💡 **액션 자세히 알아보기**
>
> 외부 세계에 영향을 주거나 받는 것
>
> 실행 시점과 횟수에 의존
>
> - **언제 실행되는지** - 순서
> - **얼마나 실행되는지** - 반복
>
> 순수하지 않은 함수, 부수 효과 함수
>
> ### 액션 잘 사용하기
>
> - 가능한 액션을 적게 사용
> - 액션은 가능한 작게
> - 액션이 외부 세계와 상호작용하는 것을 제한
> - 액션이 호출 시점에 의존하는 것을 제한

## 요점 정리

- 액션과 계산, 데이터를 구분하기
- 액션은 실행 시점이나 횟수에 의존
- 계산은 입력값으로 출력값을 만드는 것, 실행 시점이나 횟수에 의존 X
- 데이터는 이벤트에 대한 사실
- 액션 < 계산 < 데이터
