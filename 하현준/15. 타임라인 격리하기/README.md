## 두 번 빠르게 클릭하니 문제가?!

구매 버튼을 빠르게 두 번 클릭했을 때 문제가 발생(버그가 클릭 속도와 관련이 있음)

```tsx
function add_item_to_cart(name: string, price: number, quantity: number) {
	cart = add_item(cart, name, price, quantity);
	calc_cart_total();
}

function calc_cart_total() {
	total = 0;
	cost_ajax(cart, function(cost) {
		total += cost;
		shipping_ajax(cart, function(shipping) {
			total += shipping;
			update_total_dom(total);
		})
	})
}
```

타임라인 다이어그램을 보면 시간에 따라 어떤 일이 일어나는지 볼 수 있다.

즉, 두 번 빠르게 클릭했을 때 **total을 읽고 쓰는 부분**에서 문제가 발생한다.

### 두 가지 타임라인 다이어그램 기본 규칙

1. 두 액션이 순서대로 나타나면 같은 타임라인에 넣는다.

```tsx
sendEmail1();
sendEmail2();
```

타임라인에는 **계산이 아닌 액션**만 그린다.

1. 두 액션이 동시에 실행되거나 순서를 예상할 수 없다면 분리된 타임라인에 넣는다.

```tsx
setTimeout(sendEmail1, Math.random() * 10000);
setTimeout(sendEmail2, Math.random() * 10000);
```

두 액션의 실행 시점이 무작위이기 때문에 어떤 액션이 먼저 실행될지 알 수 없다.

### 액션 순서에 관한 두 가지 사실

1. ++ 와 += 는 사실 세단계이다.

자바스크립트에서 연산자를 짧게 쓸 수 있는 문법이 있다.

```tsx
total += 1;

// 풀어쓰면..
let temp = total; // 읽기 (액션)
temp = temp + 1; // 더하기
total = temp; // 쓰기 (액션)
```

즉 타임라인에는 두개의 액션으로 다이어그램에 표기해야한다.

1. 인자는 함수를 부르기 전에 실행한다.

```tsx
console.log(total);

// 풀어쓰면..
let temp = total; // 읽기  (액션)
console.log(temp);
```

즉 전역변수를 읽는 것이므로 이또한 다이어그램에 표기해야 한다.

### add-to-cart 타임라인 그리기

1. 액션을 확인한다.
2. 순서대로 실행되거나 동시에 실행되는 액션을 그린다.
3. 플랫폼에 특화된 지식을 사용해 다이어그램을 단순하게 만든다.

```tsx
function add_item_to_cart(name: string, price: number, quantity: number) {
  // 1. cart 읽기
  // 2. cart 쓰기
	cart = add_item(cart, name, price, quantity); 
	calc_cart_total();
}

function calc_cart_total() {
  // 3. total 쓰기
	total = 0;
	// 4. cart 읽기
	// 5. cost_ajax() 부르기
	cost_ajax(cart, function(cost) {
	  // 6. total 읽기
	  // 7. total 쓰기
		total += cost;
		// 8. cart 읽기
		// 9. shipping_ajax() 부르기
		shipping_ajax(cart, function(shipping) {
			// 10. total 읽기
	    // 11. total 쓰기
			total += shipping;
			// 12. total 읽기
			// 13. update_total_dom() 부르기
			update_total_dom(total);
		})
	})
}
```

위의 코드는 13단계의 액션으로 나눠져있다.

![image](https://github.com/user-attachments/assets/1e84316b-c797-4e5f-867c-c85504bdb01a)

### 타임라인 다이어그램으로 순서대로 실행되는 코드에도 두 가지 종류가 있다.

- 순서가 섞일 수 있는 코드
    - 두 액션 사이에 시간은 얼마나 걸릴지 알 수 가 없다. 액션 1과 액션 2 사이에 시간이 얼마나 걸릴지 모르니까 다른 타임라인의 액션의 순서가 섞일 수 있다.
- 순서가 섞이지 않는 코드
    - 두 액션이 차례로 실행되는데 그 사이에 다른 작업이 끼어들지 못한다.

즉, 타임라인이 짧을수록 관리가 쉽고, 박스가 적은 것이 더 좋다.

### 타임라인 다이어그램으로 동시에 실행되는 코드는 순서를 예측할 수 없다는 것을 알 수 있다.

동시에 실행되는 코드는 타임라인 다이어그램에 나란히 표현한다. 나란히 표현된다고 해서 액션 1과 액션2가 항상 정확히 동시에 실행된다는 의미는 아니다.

## 좋은 타임라인의 원칙

1. 타임라인은 적을수록 이해하기 쉽다.
    1. 타임라인이 하나라면 모든 액션은 앞의 액션다음에 바로 실행된다.
2. 타임라인은 짧을수록 이해하기 쉽다.
    1. 단계를 줄인다면 **실행 가능한 순서의 수**도 많이 줄일 수 있다.
3. 공유하는 자원이 적을수록 이해하기 쉽다.
    1. 신경 써야할 실행 가능한 순서를 줄일 수 있다.
4. 자원을 공유한다면 서로 조율해야 한다.
    1. 안전하게 공유(올바른 순서대로 자원을 쓰고 돌려준다)할 수 있어야 한다.
5. 시간을 일급으로 다룬다.
    1. 재사용 가능한 객체를 만들어 타이밍 문제를 해결하자

### 타임라인 단순화하기

타임라인 다이어그램을 그렸다면 이제는 타임라인을 단순화할 차례

1. 하나의 타임라인에 있는 모든 액션을 하나로 통합한다
    
    ![image](https://github.com/user-attachments/assets/bc19817a-661c-478d-8a6f-e130b0d1c9e5)
    
2. 타임라인이 끝나는 곳에서 새로운 타임라인이 하나 생긴다면 통합한다.
    
    ![image](https://github.com/user-attachments/assets/43311194-2f08-4e97-b9db-6a1a4802c9b4)
    

### 완성된 타임라인 읽기

타임라인 다이어그램이 완성되었다면 액션의 실행 순서를 알 수 있다. 실행 순서를 알게되면 버그를 쉽게 찾을 수 있다.

실행 순서를 통해 코드가 잘 동작하는지 확인하여 버그를 미리 예방할 수 있다.

## 버그 해결하기

**공유하는 자원을 없애 문제를 해결하기**

- total 전역변수를 제거하고 지역변수로 변경하기
- cart 암묵적 입력을 인자로 변경하기

```tsx
function add_item_to_cart(name: string, price: number, quantity: number) {
	cart = add_item(cart, name, price, quantity);
	calc_cart_total(cart);
}

function calc_cart_total(cart) {
	let total = 0;
	cost_ajax(cart, function(cost) {
		total += cost;
		shipping_ajax(cart, function(shipping) {
			total += shipping;
			update_total_dom(total);
		})
	})
}
```

### 재사용하기 더 좋은 코드 만들기

재사용하기 쉽게 만들려면 비동기 호출 내부에서 total을 계산한 뒤 사용하는 함수를 콜백으로 넘겨주면 된다.

```tsx
function add_item_to_cart(name: string, price: number, quantity: number) {
	cart = add_item(cart, name, price, quantity);
	calc_cart_total(cart, update_total_dom);
}

function calc_cart_total(cart, callback) {
	let total = 0;
	cost_ajax(cart, function(cost) {
		total += cost;
		shipping_ajax(cart, function(shipping) {
			total += shipping;
			// 콜백함수로 변경하기
			callback(total);
		})
	})
}
```

이렇게 되면 total의 값을 비동기 함수내부에서도 리턴받을 수 있게 되었다.

여기까지 하면서 생각이 든 건 타임라인에 대한 설명을 엄청 해주는데, 타임라인을 통해 해결한 느낌이 별로 없었다. 전역변수를 없애고, 암묵적 입력을 없애니 그냥 해결..? 오잉? 예시를 타임라인에 맞추서 좀더 해줬으면 어떤가 했다. 

책의 예시와는 별개로 실무에서 이런 비슷한 타이밍 이슈를 겪은적이 있는데 이때 해결한 방식은 업데이트를 한번에 처리하는 것으로 해결하였다. 

코드로 예시를 들자면, 각각의 컴포넌트에서 상태에 따라 특정 id값으로 변경해야 했는데 이를 각각 업데이트 하니 stepId가 바뀌는 타이밍과 imageId가 바뀌는 타이밍이 별개로 돌다보니 이미지가 간헐적으로 아주 짧게 이전 단계의 image로 보이는 이슈가 있었다.

```tsx
// 기존
// component A
useEffect(() => {
	if (status === 'start') {
		setStateA(imageId);
	}
}, [status, id])

// component B
useEffect(() => {
	if (status === 'start') {
		setStateB(stepId);
	}
}, [status, id])

// 수정
// parent (A, B 상위에 있는 컴포넌트)
useEffect(() => {
	if (status === 'start') {
		setState((prev) => ({ ...prev, stepId, imageId }));
	}
})

```

즉, 상위에서 status에 따라 id를 한번에 처리하도록 로직을 변경했다.
