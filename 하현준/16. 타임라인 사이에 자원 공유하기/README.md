## 장바구니에 아직 버그가 있다.

DOM 자원을 공유하기 때문에 실행 순서에 있어서 버그가 존재한다. 두 타임라인이 DOM을 공유하고 있기 때문에 잠재적으로 문제가 있다.

### DOM이 업데이트되는 순서를 보장해야 한다.

클릭한 순서대로 DOM이 업데이트돼야 하는데 해당 업데이트는 통제가 불가능하다.

이를 통제하기 위해서는 큐를 사용하는 것이 좋다. 공유 자원이지만 안전하게 공유되기 때문에 순서대로 작업을 꺼내 쓸 수 있기 때문이다.

### 자바스크립트에는 큐 자료 구조가 없기 때문에 만들어야 한다.

큐는 자료 구조이지만 타임라인 조율에 사용된다면 `동시성 기본형(concurrency primitive)`이라고 부른다.

- 동시성 기본형이란 자원을 안전하게 공유할 수 있는 재사용 가능한 코드를 말한다.

## 큐를 만들어서 해결하기

### 큐에서 처리할 작업을 큐에 넣기

큐에서 처리할 작업을 하나의 액션으로 바꿔 타임라인을 이동시켜야 한다.

```tsx
// 큐에 넣는 액션 추가하기
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

let queue_items = [];

function update_total_queue(cart) {
  queue_items.push(cart);
}
```

### 큐에 있는 첫 번째 항목을 실행한다.

큐에 넣었던 항목중에 가장 첫 번째 항목을 꺼내어 작업을 실행시켜야 한다.

```tsx

function add_item_to_cart(item) {
  cart = add_item(cart, item);
  update_total_queue(cart);
}

function calc_cart_total(cart, callback) {
  var total = 0;
  cost_ajax(cart, function(cost) {
    total += cost;
    shipping_ajax(cart, function(shipping) {
      total += shipping;
      callback(total);
    });
  });
}

let queue_items = [];

function runNext() {
  let cart = queue_items.shift();
  calc_cart_total(cart, update_total_dom);
}

function update_total_queue(cart) {
  queue_items.push(cart);
  setTimeout(runNext, 0);
}
```

그러나 여기서 동시에 두 항목이 처리된다면 막을 방법이 없다. 이를 해결하기 위해서는 working 변수를 통해 관리해야 한다. ([react 에서도 비슷한 패턴을 사용한 예시](https://react.dev/reference/react/useEffect#fetching-data-with-effects))

### 두 번째 타임라인이 첫 번째 타임라인과 동시에 실행되는 것을 막기

```tsx
let queue_items = [];
let working = false;

function runNext() {
	if (working) {
		return;
	}
	working = true;
  let cart = queue_items.shift();
  calc_cart_total(cart, update_total_dom);
}

function update_total_queue(cart) {
  queue_items.push(cart);
  setTimeout(runNext, 0);
}
```

### 다음 작업을 시작할 수 있도록 calc_cart_total() 콜백 함수를 고치기

working이 한번 true가 된 이후 다시 false로 처리하는 부분이 없다. 그렇기에 이를 다시 원상태로 돌려놓고 실행시키는 코드가 필요하다.

```tsx
function runNext() {
	if (working) {
		return;
	}
  working = true;
  let cart = queue_items.shift();
  calc_cart_total(cart, function(total) {
    update_total_dom(total);
    working = false; // 처리 후에 원상태로 돌려 놓기
    runNext();
  });
}
```

배열이 비어있을 때 예외처리를 해줘야 한다. 비어있더라도 계속해서 실행되기 때문에..

### 항목이 없을 때 멈추게 하기

```tsx
function runNext() {
	if (working) {
		return;
	}
	if (queue_items.length === 0) {
		return;
	}
  working = true;
  let cart = queue_items.shift();
  calc_cart_total(cart, function(total) {
    update_total_dom(total);
    working = false; // 처리 후에 원상태로 돌려 놓기
    runNext();
  });
}
```

### 변수와 함수를 함수 범위로 넣기

마지막을 전역 변수를 참조하고 있는 부분을 제거해보자

```tsx

function Queue() {
  let queue_items = [];
  let working = false;

  function runNext() {
    if (working) {
      return;
    }
    if (queue_items.length === 0) {
      return;
    }
    working = true;
    let cart = queue_items.shift();
    calc_cart_total(cart, function(total) {
      update_total_dom(total);
      working = false;
      runNext();
    });
  }

  return function(cart) {
    queue_items.push(cart);
    setTimeout(runNext, 0);
  };
}

// 큐 만들기!
const update_total_queue = Queue();
```

나는 함수도 괜찮은데 여기서부터 class 문법으로 instance 만들어서 해도 괜찮아 보인다.
책과 다르게 class를 사용해서 해보려고 한다.

```tsx
type QueueWorker<DataType> = (data: DataType, done: (val: any) => void) => void;

interface QueueItem<DataType> {
  data: DataType;
  callback: (val: any) => void;
}

class Queue {
  private queue_items: Array<QueueItem<DataType>> = [];
  private working: boolean = false;
	
	enqueue() {
		this.queue_items.push(cart);
		setTimeout(this.runNext, 0);
	}
	
	private runNext() {
		if (this.working || this.queue_items.length === 0) {
      return;
    }
  
    this.working = true;
    
    // 일반적인 형태로 변경해야 할 듯합니다. 일단 여긴 문법에 눈감아주세요.. ㅎㅎ
    let cart = this.queue_items.shift();
   
    calc_cart_total(cart, function(total) {
      update_total_dom(total);
      this.working = false;
      this.runNext();
    });
	}
}
```

### 작업이 끝났을 때 실행하는 콜백을 받기

작업이 끝났을 때 콜백을 실행시켜주는 함수로 바꿔보자

```tsx
type QueueWorker<DataType> = (data: DataType, done: (val: ResultType) => void) => void;

interface QueueItem<DataType, ResultType> {
  data: DataType;
  callback: (val: ResultType) => void;
}

class Queue<DataType, ResultType> {
  private queue_items: Array<QueueItem<DataType>> = [];
  private working: boolean = false;
  private worker: QueueWorker<DataType>;
	
	constructor(worker: QueueWorker<DataType>) {
		this.worker = worker;
	}
	
  enqueue(data: DataType, callback: (val: ResultType) => void = () => {}) {
		this.queue_items.push({
			 data,
			 callback: callback || () => {}
		});
		setTimeout(this.runNext, 0);
	}
	
	private runNext() {
		if (this.working || this.queue_items.length === 0) {
      return;
    }
    
    this.working = true;
    const item = this.queue_items.shift() as QueueItem<DataType>;
    
    // 화살표 함수를 써야 this 문제가 없음
	  this.worker(item.data, () => {
		  this.working = false;
		  this.runNext();
	  }
	}
}

const update_total_queue = new Queue<Cart, Total>(calc_cart_worker);

function calc_cart_worker(cart: Cart, done: (total: number) => void) {
	calc_cart_total(cart, function(total: number) {
		update_total_dom(total);
		done(total);
	})
}
```

### 작업이 완료되었을 때 콜백 부르기

```tsx
class Queue<DataType, ResultType> {
  private queue_items: Array<QueueItem<DataType>> = [];
  private working: boolean = false;
  private worker: QueueWorker<DataType>;
	
	constructor(worker: QueueWorker<DataType>) {
		this.worker = worker;
	}
	
  enqueue(data: DataType, callback: (val: ResultType) => void = () => {}) {
		this.queue_items.push({
			 data,
			 callback: callback || (() => {})
		});
		setTimeout(this.runNext, 0);
	}
	
	private runNext() {
		if (this.working || this.queue_items.length === 0) {
      return;
    }
    
    this.working = true;
    const item = this.queue_items.shift() as QueueItem<DataType>;
    
	  this.worker(item.data, (val: ResultType) => {
		  this.working = false;
		  // 콜백 호출하기
      setTimeout(() => item.callback(val), 0);
		  this.runNext();
	  }
	}
}

// 사용 예시
type Cart = { };
type Total = number;

const update_total_queue = new Queue<Cart, Total>(calc_cart_worker);

function calc_cart_worker(cart: Cart, done: (total: Total) => void) {
	calc_cart_total(cart, function(total) {
		update_total_dom(total);
		done(total);
	})
}
```

최종적으로 완성된 Queue 코드이다.

큐를 사용해 DOM 업데이트를 같은 타임라인에서 하도록 만들었기 때문에 순서 문제가 생기기 않는다.

책에서는 드로핑 큐를 통해 새로운 작업이 들어오면 건너뛸 수 있도록 만든 개선된 큐를 제안한다.
이는 사용자가 빠르게 항목을 눌러 추가하더라도 클릭 한 횟수만큼 기다리는 것이 아닌 **최대 개수를 정해서 큐를 태우는** 것이다.

**드로핑 큐**

```tsx
class DroppingQueue<DataType, ResultType> {
  private queue_items: Array<QueueItem<DataType>> = [];
  private working: boolean = false;
  private worker: QueueWorker<DataType>;
  private max: number;
	
	constructor(max: number, worker: QueueWorker<DataType>) {
		this.worker = worker;
		this.max = max;
	}
	
  enqueue(data: DataType, callback: (val: ResultType) => void = () => {}) {
		this.queue_items.push({
			 data,
			 callback: callback || (() => {})
		});
		while(this.queue_items.length > max) {
			this.queue.items.shift();
		}
		setTimeout(this.runNext, 0);
	}
	
	private runNext() {
		if (this.working || this.queue_items.length === 0) {
      return;
    }
    
    this.working = true;
    const item = this.queue_items.shift() as QueueItem<DataType>;
    
	  this.worker(item.data, (val: ResultType) => {
		  this.working = false;
		  // 콜백 호출하기
      setTimeout(() => item.callback(val), 0);
		  this.runNext();
	  }
	}
}

// 사용 예시
const update_total_queue = new DroppingQueue<Cart, Total>(1, calc_cart_worker);
```

큐 항목이 한 개 이상 늘어나지 않기 때문에 서버의 응답을 최대 두 번만 기다리면 된다.

💡책에서는 함수내부에 return을 했는데, class로 변경하면서 return 대신 메서드를 직접 사용해야 한다.

```tsx
update_total_queue.enqueue(); 
```