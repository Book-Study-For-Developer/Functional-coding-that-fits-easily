## 코드에 대한 최적화를 진행하니 버그가 발생

```tsx
function add_item_to_cart(item) {
  cart = add_item(cart, item);
  update_total_queue(cart);
}

function calc_cart_total(cart, callback) {
  let total = 0;
  cost_ajax(cart, function(cost) {
    total += cost;
    shipping_ajax(cart, function(shipping) {
      total += shipping;
      callback(total);
    });
  });
}

function calc_cart_worker(cart, done) {
  calc_cart_total(cart, function(total) {
    update_total_dom(total);
    done(total);
  });
}

const update_total_queue = DroppingQueue(1, calc_cart_worker);

// 최적화를 진행한 코드
function add_item_to_cart(item) {
  cart = add_item(cart, item);
  update_total_queue(cart);
}

function calc_cart_total(cart, callback) {
  let total = 0;
  cost_ajax(cart, function(cost) {
    total += cost;
  }); // 괄호의 위치가 바뀜
  
  shipping_ajax(cart, function(shipping) {
    total += shipping;
    callback(total);
  });
}

function calc_cart_worker(cart, done) {
  calc_cart_total(cart, function(total) {
    update_total_dom(total);
    done(total);
  });
}
```

병렬 처리로 순서를 바꾸어서 실행 속도는 빨라졌지만 버그가 생김

책에서는 타임라인을 그리면서 버그를 분석하는데, 여기선 생략하겠습니다.. ㅎㅎ
(함수형 및 타임라인을 그리지 않고 봐도 저렇게 바꿔버리면 처리 순서가 바뀌기 때문에 당연히 버그가 생길 느낌)

최적화(?) 를 통해 빨라진 이유는 타임라인을 그려보았을 때 동시에 실행시키기 때문에 응답을 기다리지 않아도 되어 더 빨리 끝나게 된다.

그렇다면 버그를 없애면서 빠르게 실행하는 방법은 무엇일까? ⇒ 그것은 바로 동시성 기본형을 만들어 해결하는 것이다. (책의 초반에 나왔던 3개의 로봇 피자에서 커팅을 쓰는게 여기서 나온다.)

## 모든 병렬 콜백 기다리기

![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/253d1ac1-0c8d-4179-8d90-21ade38e0aea/55d411ac-dfd4-43a2-a64b-58d5e3c19e4a/image.png)

두 응답이 모두 처리되고 나서 마지막으로 DOM 업데이트를 하도록 한다.

이 점선을 바로 컷이라고 하고 순서를 보장해주는 역할을 한다.
컷을 만들면 **타임라인을 앞부분과 뒷부분으로 나눠주는 장점**이 있다.

### Cut 함수 만들기

```tsx

/**
 * @param {number} num - 기다릴 타임라인 수
 * @param {VoidFunction} callback 모든 것이 끝났을 때 실행할 콜백
 */
function Cut(num, callback) {
	// 기다릴 타임라인
	let num_finished = 0;
	return function() {
		// 함수가 호출할 때마다 카운터가 증가.
		num_finished += 1;
		if (num_finished === num) {
			callback();
		}
	}
}

// 사용 예시
function calc_cart_total(cart, callback) {
	let total = 0;
	const done = Cut(2, function() {
		callback(total);
	});
	
	cost_ajax(cart, function(cost) {
		total += cost;
		done();
	})
	
	shipping_ajax(cart, function(shipping) {
		total += shipping;
		done();
	})
}
```

done이 두번 호출이 되어야만 다음 callback이 실행될 수 있는 것이다.

> `Promise.all`이랑 뭐가 다르냐?

책에서 저자도 당연히 구현된 동시성 기본형을 사용하라고 한다.
그러나 책에서는 javascript에서만 한정하는 것이 아니기에 이러한 함수도 소개한 것이다.
> 

### 복잡한 구현이 필요한 이유

1. 비동기 웹 요청
    1. ajax요청을 사용하지 않으면 없앨 수 있다.(사용자가 인터랙션을 하면 새로고침..) 이것도 현실적으로 불가능
2. 결과를 합쳐야 하는 두 개의 API 응답
    1. 하나의 API로 처리할 수 있으면 되는데, 백엔드의 입장에서 올바르지 않을 수 있다.
3. 예측 불가능한 사용자의 액션
    1. 사용자 인터랙션을 적게 만들면 없앨 수 있다. 그러나  이렇게 만드는 것은 현실적으로 불가능..

## 딱 한 번만 호출하는 기본형

사이트에 처음 방문한 사용자에게만 호출하는 함수를 만들고 싶다.

```tsx

/**
 * @param {VoidFunction} action 실행할 액션
 */
function JustOnce(action) {
	let alreadyCalled = false;
	return function(a, b, c) {
		if (alreadyCalled) return;
		alreadyCalled = true;
		return action(a, b, c);
	}
}

// 사용 예시
const sendAddToCartTextOnce = JustOnce(sendAddToCartText);
```

sendAddToCartTextOnce 함수는 이제 여러번 호출하더라도 한번만 호출하는 함수가 되었다.

### 암묵적 시간 모델 vs 명시적 시간 모델

자바스크립트의 시간 모델 

1. 순차적 구문은 순서대로 실행
2. 두 타임라인에 있는 단계는 왼쪽 먼저 실행되거나, 오른쪽 먼저 실행될 수 있다.
3. 비동기 이벤트는 새로운 타임라인에서 실행된다.
4. 액션은 호출할 때마다 실행된다. (반복)

암묵적 시간모델을 사용하는 것은 간단한 프로그램에서는 좋지만 실행 방식을 바꾸기는 어렵다.

그럴때 앞에서 만든 동시성 기본형을 통해 시간 모델을 명시적으로 쓸 수 있다.