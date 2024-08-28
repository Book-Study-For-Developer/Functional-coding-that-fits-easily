# 03. 액션과 계산, 데이터의 차이를 알기

- [액션, 계산, 데이터](#액션-계산-데이터)
- [어디에나 적용할 수 있는 액션, 계산, 데이터](#어디에나-적용할-수-있는-액션-계산-데이터)
- [새로 만드는 코드에 함수형 사고 적용하기](#새로-만드는-코드에-함수형-사고-적용하기)
- [이미 있는 코드에 함수형 사고 적용하기](#이미-있는-코드에-함수형-사고-적용하기)

## 액션, 계산, 데이터


| 액션 | 계산 | 데이터 |
| --- | --- | --- |
| 실행 시점과 횟수에 의존<br/>외부 세계에 영향을 주거나 받는 것<br/>언제 실행 (순서), 얼마나 실행 (반복) | 입력으로 출력을 계산<br/>외부 세계에 영향 주거나 받지 않음<br/>실행 시점이나 횟수에 의존하지 않음 | 이벤트에 대한 사실<br/>일어난 일에 대한 결과<br/>의미 있게 처리한 입력 장치로 얻은 정보 |
| 부수 효과, 순수하지 않은 함수 | 순수 함수, 수학 함수 |  |
| 이메일 보내기, DB 읽기 | 최댓값 찾기, 이메일 주소가 올바른지 확인 | 사용자가 입력한(이벤트) 이메일 주소(사실)<br/>은행 API로 읽은(이벤트) 달러 수량 (사실) |

### 🍭 액션

- 사용하기 어려우나 소프트웨어를 실행하려는 가장 중요한 이유임
- 액션을 잘 사용하기 위한 방법
  - 가능한 한 적게 사용. 가급적 대신 계산 사용
  - 가능한 한 작게 만듦. 액션과 관련 없는 코드(결정, 계획과 관련된 부분)는 모두 제거
  - 외부 세계와 상호작용하는 것 제한. 내부에 계산과 데이터만 있고 가장 바깥에 액션이 있는 게 이상적. 18장 어니언 아키텍처 참고
  - 액션이 호출 시점에 의존하는 것을 제한

### 🍭 계산

- 참조 투명(referentially transparant)함: 계산을 호출하는 코드를 계산 결과로 바꿀 수 있음
- 장점 (액션과 비교했을 때)
  - 테스트 용이 - 외부에 영향 주지 않음 → 가급적 액션보다 계산 사용하는 게 좋음
  - 기계적 분석 쉬움 - 정벅 분석에서 자동화된 분석은 중요
  - 조합하기 좋음 - 더 큰 계산 만들 수 있음. high order 계산 사용.
  - 계산 쓰면 걱정하지 않아도 됨
    1. 동시에 실행되는 것
    2. 과거에 실행됐던 것이나 미래에 실행할 것
    3. 실행 횟수
- 단점
  - 실행 전에 어떤 일이 일어날지 알 수 없음. 블랙박스

### 🍭 데이터

- 불변성
  - 카피 온 라이트 copy on write - 변경할 때 복사본 만듦
  - 방어적 복사 defensive copy - 보관하려는 데이터의 복사본 만듦
- 장점: 데이터 자체로 할 수 있는 일이 없음 → 데이터 그대로 이해 가능
  - 직렬화: 직렬화된 데이터는 전송하거나 데이터에 저장했다가 읽기 쉬움
  - 동일성 비교 용이
  - 자유로운 해석
- 단점
  - 해석이 반드시 필요

<br />

## 어디에나 적용할 수 있는 액션, 계산, 데이터

1. 액션, 계산, 데이터는 어디에나 적용 가능
2. 액션 안에는 계산, 데이터, 또 다른 액션이 숨어있을 수도 있음 - 액션을 더 작은 것으로 나누고 나누는 것을 언제 멈춰야 할지 아는 게 중요
3. 계산은 더 작은 계산과 데이터로 나누고 연결 가능
4. 데이터는 데이터만 조합 가능 - 데이터는 다른 영향 주지 않음 → 데이터 찾는 일을 먼저 해야
5. 계산은 때로 ‘머릿속에서’ 일어남 - 사고 과정에 녹아있는 계산 존재 (장을 보는데 무엇을 사야 할지) → 결정, 계획은 계산일 가능성이 높음

<br />


## 새로 만드는 코드에 함수형 사고 적용하기

- 명세
  - 친구 10명 이상 추천하면 더 좋은 쿠폰 보내줌
  - 친구 10명을 추천하면 best 쿠폰, 모든 사용자는 good 쿠폰, 사용자에게 전달하지 않는 bad 쿠폰 존재
  - 이메일 DB - 이메일, 추천 수
  - 쿠폰 DB - 쿠폰, 등급
- 쿠폰 보내는 과정

  1. 데이터베이스에서 구독자 가져오기
  2. 데이터베이스에서 쿠폰 목록 가져오기
  3. 보내야 할 이메일 목록 만들기
  4. 이메일 전송하기

  ```tsx
  // 1. 데이터베이스에서 구독자 가져오기 - 액션
  // 데이터베이스에서 가져온 구독자 - 데이터
  var subscribe = {
    email: "sam@pmail.com",
    rec_count: 16,
  };

  // 쿠폰 등급은 문자열 - 데이터베이스 테이블의 rank 열 값과 동일
  var rank1 = "best";
  var rank2 = "good";

  // 구독자가 받을 쿠폰 등급을 결정하는 함수 - 계산
  function subCouponRank(subscriber) {
    if (subscriber.rec_count >= 10) {
      return "best";
    } else {
      return "good";
    }
  }

  // 2. 데이터베이스에서 쿠폰 목록 가져오기 - 액션
  // 데이터베이스에서 가져온 쿠폰 - 데이터
  var coupon = {
    code: "10PERCENT",
    rank: "bad",
  };

  // 특정 등급의 쿠폰 목록을 선택하는 함수 - 계산
  /**
   * @param {coupons} Object 전체 쿠폰 목록
   * @param {rank} string 선택할 등급
   */
  function selectCouponsByRank(coupons, rank) {
    var ret = [];
    for (var c = 0; c < coupons.length; c++) {
      var coupon = coupons[c];
      // 쿠폰이 주어진 등급에 맞으면 쿠폰 코드를 배열에 넣음
      if (coupon.rank === rank) {
        ret.push(coupon.code);
      }
      return ret;
    }
  }

  // 3. 보내야 할 이메일 목록 만들기
  // 이메일 - 데이터
  var message = {
    from: "newsletter@coupondog.co",
    to: "sam@pmail.com",
    subject: "Your weekly coupons inside",
    body: "Here are your coupons...",
  };

  // 구독자가 받을 이메일 목록 계획 - 계산
  /**
   * @param {subscriber} Object 구독자
   * @param {goods} Object good 쿠폰 목록
   * @param {bests} Object bad 쿠폰 목록
   */
  function emailForSubscriber(subscriber, goods, bests) {
    var rank = subCouponRank(subscriber);
    if (rank === "best") {
      // 등급 결정
      return {
        // 이메일 만들어 리턴
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

  // 보낸 이메일 목록 준비 - 계산
  function emailsForSubscribers(subscribers, goods, bests) {
    var emails = [];
    for (var s = 0; s < subscribers.length; s++) {
      var subscriber = subscribers[s];
      var email = emailForSubscriber(subscriber, goods, bests);
      emails.push(email);
    }
    return emails;
  }

  // 4. 이메일 전송하기
  // 이메일 보내기 - 액션: 앞에서 이미 계획한 목록 순회하며 보내기
  function sendIssue() {
    var coupons = fetchCouponsFromDB();
    var goodCoupons = selectCouponsByRank(coupons, "good");
    var bestCoupons = selectCouponsByRank(coupons, "best");
    // 최적화하려면 구독자 목록을 페이지네이션으로 잘라서 가져오면 됨
    var subscribers = fetchSubscribersFromDB();
    var emails = emailsForSubscribers(subscribers, goodCoupons, bestCoupons);
    for (var e = 0; e < emails.length; e++) {
      var email = emails[e];
      emailSystem.send(email);
    }
  }
  ```

<br />


## 이미 있는 코드에 함수형 사고 적용하기

자회사에 수수료 보내기

```tsx
// 2. 액션을 호출함 -> 함수 전체가 액션
function figurePayout(affiliate) {
  var owed = affiliate.sales * affiliate.commission;
  if (owed > 100) {
    // 100달러 이하면 송금 안 함
    sendPayout(affiliate.bank_code, owed); // 1. 액션
  }
}

// 4. 액션을 호출함 -> 함수 전체가 액션
function affiliatePayout(affiliates) {
  for (var a = 0; a < affiliates.length; a++) {
    figurePayout(affiliates[a]); // 3. 액션 함수를 호출 -> 액션
  }
}

// 6. 액션을 호출함 -> 함수 전체가 액션
function main(affiliates) {
  affiliatePayout(affiliates); // 5. 액션 함수를 호출 -> 액션
}
```

⇒ 액션을 호출하는 함수 또한 액션이 되기에 액션 조심해서 사용해야
