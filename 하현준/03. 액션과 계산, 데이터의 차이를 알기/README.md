## 액션과 계산, 데이터는 어디에나 적용할 수 있다.

장보기 과정을 예를 든다면, 각 단계는 모두 액션에 해당한다.

```jsx
냉장고 확인하기 -> 운전해서 상점으로 가기 -> 필요한 것 구입하기 -> 운전해서 집으로 오기
```

그러나 액션보다는 계산과 데이터를 선호하기 때문에 우리는 액션에서 숨어있는 계산이나 데이터를 찾아 분리해야 한다.

ex) `필요한 것을 구입하기`에서 데이터와 계산을 분리해내기

- 데이터: `현재 재고`, `필요한 재고`
- 계산: `재고 ‘빼기’`
- 데이터: `장보기 목록`
- 액션: `목록에 있는 것 구입하기`

(장보기 목록 = 필요한 재고 - 현재 재고)

즉, 필요한 것을 구입하는 하나의 액션에서 여러개의 데이터와 계산을 만들어서 간단한 액션을 만들게 된 것이다.

**계속 나누다 보면 점점 더 복잡해진다고 생각할 수 있지만, 액션에 숨어 있는 다른 액션이나 계산 또는 데이터를 발견하기 위해 나눌 수 있는 만큼 나누는 것이 좋습니다.**

**💡여기서 배운 것**

1. 액션과 계산, 데이터는 어디에나 적용할 수 있다.⭐️
2. 액션 안에는 계산과 데이터, 또 다른 액션이 숨어 있을지도 모른다.⭐️⭐️
3. 계산은 더 작은 계산과 데이터로 나누고 연결할 수 있다.⭐️
4. 데이터는 데이터만 조합할 수 있다.⭐️
5. 계산은 대로 ‘우리 머리속에서’ 일어난다. ⭐️⭐️⭐️
    1. 현실에서 사람의 머릿속에서는 쉽게 생각할 수 있는 것들이 있어 계산이 잘 보이지 않는다.
    (ex) 장보기 목록도 재고를 나누어서 직접 계산해서 구매하지 않는다.

> **데이터에 대해서**

데이터는 **이벤트에 대한 사실**이다. 
함수형 프로그래머는 데이터 구조를 만들기 위해 두 가지 원칙을 사용

1. copy-on-write: 변경할 때 복사본을 만듦
2. defensive copy: 보관하려는 데이터의 복사본을 만듦

장점
- 직렬화: 직렬화된 데이터는 읽고 쓰기가 쉽다.
- 동일성 비교: 데이터는 비교하기가 쉽다.
- 자유로운 해석

단점
- 해석이 항상 필요하며, 해석하지 않은 데이터는 쓸모없는 바이트이다.
> 

## 쿠폰 서비스 코드에 함수형 사고 적용하기

이메일로 쿠폰을 구독받을 수 있는 서비스이다.

요구 사항

- 쿠폰 등급은 ‘bad’, ‘good’, best’ 등급을 가지고 있다.
    - bad: 사용되지 않는 쿠폰
    - good: 모든 사용자들에게 전달되는 쿠폰
    - best: 추천을 많이 한 사용자를 위한 쿠폰
- 10명 이상 추천한 사용자는 더 좋은 쿠폰을 받을 수 있다.

이제 이러한 서비스를 구축하는데 있어서 필요한 액션(A)과 계산(C), 데이터(D)를 분류해보자.

- 이메일 보내기 **`A`**
- 데이터베이스에서 구독자 가져오기 **`A`**
- 쿠폰에 등급 매기기 **`D`**
- 데이터베이스에서 쿠폰 읽기 **`A`**
- 이메일 제목 **`D`**
- 이메일 주소 **`D`**
- 추천 수 **`D`**
- 어떤 이메일이 쿠폰을 받을지 결정하기 **`C`**
- 구독자 DB 레코드 **`D`**
- 쿠폰 DB 레코드 **`D`**
- 쿠폰 목록 DB 레코드 **`D`**
- 구독자 목록 DB 레코드 **`D`**
- 이메일 본문 **`D`**

### 쿠폰 보내는 과정 구현하기

```tsx
// 데이터베이스에서 가져온 구독자 데이터
type Subscriber = {
	email: string,
	rec_count: number
}
const subscriber: Subscriber = {
	email: 'sam@pmail.com',
	rec_count: 16
}

// 쿠폰 등급을 결정하는 함수 
function subCouponRank(subscriber: Subscriber) {
	if(subscriber.rec_counter >= 10) {
		return 'best';
	} else {
		return 'good';
	}
}

// 데이터베이스에서 가져온 쿠폰 데이터
type Rank = 'bad' | 'good' | 'best';
type Coupon = {
	code: string,
	rank: Rank,
}
cosnt coupon = {
	code: '10PERCENT',
	rank: 'bad',
}

// 특정 등급의 쿠폰 목록을 선택하는 계산
function selectCouponByRank(coupons: Array<Coupon>, rank: Rank) {
	// 책의 코드와 다르게 변형했습니다.
	const ret = coupons.filter((coupon) => {
		return coupon.rank === rank;
	})
	return ret;
}

// 이메일 데이터
type Message = {
	from: string,
	to: string,
	subject: string,
	body: string,
}
const message = { 
	from: 'newsletter@coupondog.co',
	to: 'sam@pmail.com',
	subject: 'Your weekly coupons inside',
	body: 'Here are your coupons ...',
}

// 구독자가 받을 이메일을 계획하는 계산
function emailForSubscriber(subscriber: Subscriber, goods: Array<Coupon>, bests: Array<Coupon>) {
	// 쿠폰의 등급
	const rank = subCouponRank(subscriber);
	// 책의 코드와 다르게 변형했습니다.
	return { 
		from: 'newsletter@coupondog.co',
		to: 'sam@pmail.com',
		subject: 'Your weekly coupons inside',
		body: `Here are the ${rank} coupons: ${(rank === 'best' ? bests : goods).join(", ")}`,
	}
}

// 보낸 이메일 목록을 준비하기
function emailsForSubscribers(subscribers: Array<Subscriber>, goods: Array<Coupon>, bests: Array<Coupon>) {
	// 책의 코드와 다르게 변형했습니다.
	const emails = subscribers.map((subscribe) => {
		return emailForSubscriber(subscribe, goods, bests);
	})
	return emails;
} 

// 이메일 보내기를 위한 액션
function sendIssue() {
	const coupons = fetchCouponsFromDB();
	const goodCoupons = selectCouponsByRank(coupons, 'good');
	const bestCoupons = selectCouponsByRank(coupons, 'best');
	const emails = emailsForSubscribers(subscribers, goodCoupons, bestCoupons);
	
	// 책의 코드와 다르게 변형했습니다.
	emails.forEach((email) => {
		emailSystem.send(email);	
	})
}
```

> **계산에 대해서**

계산은 **입력값으로 출력값**을 만드는 것이다. 
보통 **`순수 함수`**라고 부르는 것 중에 하나이다.

액션 보다 계산이 좋은 점
- 테스트하기 쉽다.
- 기계적인(정적인) 분석은 쉽다
- 계산은 분석이 쉽다.

단점
- 실행하기 전에 어떤 일이 발생할 지 알 수 없다.
> 

## 이미 있는 코드에 함수형 사고 적용하기

```tsx
type Affiliate = {
	sales: number;
	commission: number;
	bank_code: string;
}
function figurePayout(affiliate: Affiliate) {
	const owed = affiliate.sales * affiliate.commission;
	if (owed > 100) {
	// 하나의 액션이 있다고 생각할 수 있다.
	 sendPayout(affiliate.bank_code, owed);
	}
}
 
function affiliatePayout(affiliates: Array<Affiliate>) {
	affiliates.forEach((affiliate) => {
		figurePayout(affiliate);
	})
 }
 
function main(affiliates: Array<Affiliate>) {
	affiliatePayout(affiliates);
}
```

그러나 위의 코드는 하나의 액션이 아니다. 

함수 안에서 액션을 호출하고 있기 때문에 함수 전체가 액션이고, 이를 타고 타고 올라가면 **결국 모든 함수가 액션**이 돼버린다.

> **액션에 대해서**

액션은 외부 세계에 영향을 주거나 받는 것을 말한다. 
- **순서**: 언제 실행되는지
- **반복**: 얼마나 실행되는지
액션은 일반적으로 **순수하지 않은 함수, 부수 효과 함수**라고 부른다.

액션은 다루기가 힘들지만 꼭 써야 한다.
- 가능한 액션을 적게 사용한다.
- 액션은 가능한 작게 만든다.
- 액션이 외부 세계와 상호작용하는 것을 제한해라.
- 액션이 호출 시점에 의존하는 것을 제한한다.
>