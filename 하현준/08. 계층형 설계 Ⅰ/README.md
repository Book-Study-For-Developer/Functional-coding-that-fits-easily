## 계층형 설계란 무엇인가요?

말 그대로 소프트웨어를 계층으로 구성하는 기술이다. 

### 계층형 설계 패턴

**패턴1. 직접 구현**

- 직접 구현된 함수를 읽을 때, 함수 시그니처가 나타내고 있는 문제를 함수 본문에서 적절한 구체화 수준에서 해결해야 한다.

**패턴2. 추상화 벽**

- 인터페이스를 사용하여 세부 구현을 감추어 제공한다.

**패턴3. 작은 인터페이스**

- 비지니스 개념을 나타내는 중요한 인터페이스는 작고 강력한 동작으로 구성하는 것이 좋다.

**패턴4. 편리한 계층**

- 소프트웨어를 더 빠르고 고품질로 제공하는 데 도움이 되는 계층에 투자해야 하지, 그냥 단순히 좋아서 계층을 추가하면 안된다.

### 직접 구현 패턴을 적용하기

```tsx
function freeTieClip(cart) {
  let hasTie = false
  let hasTieClip = false;
  
  for(var i = 0; i < cart.length; i++) {
    let item = cart[i];
    if(item.name === "tie") {
      hasTie = true;
    }
    if(item.name === "tie clip") {
      hasTieClip = true;
    }
  }
  
  if(hasTie && !hasTieClip) {
    let tieClip = make_item("tie clip", 0);
    return add_item(cart, tieClip);
  }
  return cart;
}

// 함수 추출하기
function freeTieClip(cart: Cart) {
  const hasTie = isInCart(cart, "tie");
  const hasTieClip = isInCart(cart, "tie clip");
  
  if(hasTie && !hasTieClip) {
    const tieClip = make_item("tie clip", 0);
    return add_item(cart, tieClip);
  }
  
  return cart;
}

function isInCart(cart: Cart, name: string) {
  return cart.some((cartItem) => {
	  return cartItem.name === name;
  }).length !== 0; 
}
```

저수준의 반복문을 isInCart함수로 추출해내어 비슷한 수준의 함수로 변경한다.

여기서 중요한 점은 `freeTieClip`이 사용하는 **모든 함수는 장바구니가 배열인지 몰라도 된다.** 
즉, **함수가 모두 비슷한 계층에 존재한다는 의미와 같다.**

## 3단계 줌 레벨

1. 전역 줌 레벨

전역 줌 레벨이 기본 줌 레벨이고, 계층 사이에 상호 관계를 포함해서 모든 영역을 살펴볼 수 있다.

2. 계층 줌 레벨

계층 줌 레벨은 한 계층과 연결된 바로 아래 계층을 볼 수 있는 줌 레벨이다.

3. 함수 줌 레벨

함수 하나와 바로 아래 연결된 함수를 볼 수 있다.

### 함수 줌 레벨을 사용하여 함수 하나가 가진 화살표를 비교해보자.

![image](https://github.com/user-attachments/assets/91acae5f-cdb6-4e64-ab2b-ad0877a5bab0)

두개의 긴 화살표를 줄여야 한다. 즉 중간 계층에 함수를 두어 화살표의 길이를 줄이는 것이다.

```tsx
function remove_item_by_name(cart: Cart, name: string) {
	let idx = null;
	cart.forEach((cart_item, i) => {
		if (cart_item.name === name) {
			idx = i
		}
	})
	if (idx !== null) {
		cart.splice(idx, 1);
	}
}

function remove_item_by_name(cart: Cart, name: string) {
	const idx = indexOfItem(cart, name);
	if (idx !== null) {
		cart.splice(idx, 1);
	}
}

function indexOfItem(cart: Cart, name: string) {
	const index = cart.findIndex((cartItem) => {
		return cartItem.name === name;
	});
	return index === -1 ? null : index;
}
```

이전의 함수보다 읽기 쉬워졌고 다이어그램도 더 읽기 쉬워졌다.

<img width="659" alt="image" src="https://github.com/user-attachments/assets/2d227702-125c-4af5-91ce-a77d805daeb4">


### 다른 함수로 연습해보기

```tsx
// 배열 인덱스를 직접 참조
function setPriceByName(cart: Cart, name: string, price: number) {
  const i = indexOfItem(cart, name);
  if(i !== null) {
    const item = cart[i];
    return arraySet(cart, i, setPrice(item, price));
  }
  return cart;
}

// 배열 인덱스를 참조하지 않고 함수를 통해 가져오기
function setPriceByName(cart, name, price) {
  const i = indexOfItem(cart, name);
  if(i !== null) {
    const item = arrayGet(cart, i);
    return arraySet(cart, i, setPrice(item, price));
  }
  return cart;
}

function indexOfItem(cart, name) {
  for(var i = 0; i < cart.length; i++) {
   // 책의 코드..
    if(arrayGet(cart, i).name === name)
      return i;
  }
  return null;
}

function arrayGet(array: Array, idx: number) {
  return array[idx];
}
```

**배열 인덱스를 직접 참조하면서 설계가 좋다고 느꼈을 때**

- 배열을 쓸 때 편리했다.

**배열 인덱스를 직접 참조하지 않으면서 설계가 좋다고 느꼈을 때**

- 분리된 계층이 필요할 때
- 일반화된 함수의 사용이 필요할 때
- 추상화된 계층을 하나 더 만들고 싶을 때

> 내가 느낀점은.. 
> 
> 과도한 일반화가 아닌가 싶었다. 
> 배열에서 인덱스를 직접 참조하든, 함수를 통해 참조하든 결국 js에서 제공하는 문법 중 하나인데  이렇게 되면 함수내에서 에러처리를 더 해줘야 하는게 아닌가 했다.
> 
> 아마도 여러 패턴 중 한개를 보여주는 것이니 하나만 사용하면 이러한 단점이 생기는게 아닌가 생각이 든다.
>
