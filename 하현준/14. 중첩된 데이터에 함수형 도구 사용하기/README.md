## 필드명을 명시적으로 만들기

```tsx
function incrementQuantity(item) {
	const quantity = item['quantity'];
	const newQuantity = quantity + 1;
	const newItem = objectSet(item, 'quantity', newQuantity);
	
	return newItem;
}

function incrementSize(item) {
	const size = item['size'];
	const newSize = size + 1;
	const newItem = objectSet(item, 'size', newSize);
	
	return newItem;
}
```

위의 두 코드는 **암묵적 인자를 드러내기**를 사용하여 리팩토링을 진행한 코드이다.

그러나 위 두 코드를 보면 또 다시 중복이 시작되고 있다. 각각의 함수들의 목적은 다르지만 비슷한 코드로 작성되고 있다.

### update() 도출하기

**암묵적 인자를 드러내기** + **함수 본문을 콜백으로 바꾸기** 두가지 기법을 사용해서 리팩토링을 진행해야 한다.

- 암묵적 인자를 드러내기 ⇒ ‘size’, ‘quantity’ 를 없애기
- 함수 본문을 콜백을 바꾸기 ⇒ 중복되는 앞부분과 뒷부분을 뽑아내어 함수로 만들기

```tsx
function update<T extends object, K extends keyof T>(object: T, key: K, modify: (value: T[K]) => T[K]) {
	const value = object[key];
	const newValue = modify(value);
	const newObject = objectSet(object, key, newValue);
	
	return newItem;
}

// 사용 예제
const employee = {
	name: 'kim',
	salary: 120000
}

// 내 연봉도 10퍼 올려줘 🚀
function raise10Percent(salary: number) {
	return salary * 1.1;
}

update(employee, 'salary', raise10Percent);
```

### 기존에 사용되던 함수들(조회, 변경, 설정)을 update함수로 교체하기

```tsx
function incrementField(item, field) {
	return update(item, field, function(value) {
		return value + 1;
	})
}

function halveField(item, field) {
	return update(item, field, function(value) {
		return value / 2;
	})
}
```

객체를 update에 전달하고 바꿀 값의 필드를 어떻게 변경할지 같이 전달하여 변경된 값을 리턴해주는 함수로 바꾸었다.

## 중첩된 데이터에 update 사용하기

API에는 중첩된 데이터가 많다. 단순히 1 뎁스의 객체가 아닐때가 많기 때문에 이를 `update`함수를 사용해서 처리해보자

```tsx
function incrementSize(item) {
	const options = item.options;
	const size = options.size;
	const newSize = size + 1;
	const newOptions = objectSet(options, 'size', newSize);
	const newItem = objectSet(item, 'options', newOptions);
	
	return newItem;
}

// update를 사용한 함수
function incrementSize(item) {
	const options = item.options;
	const newOptions = update(options, 'size', increment);
	const newItem = objectSet(item, 'options', newOptions);
	
	return newItem;
}

// 한번더 update를 사용해 리팩토링이 가능해보인다.
function incrementSize(item) {
	return update(item, 'options', function(options) {
		return update(options, 'size', increment);
	})
}
```

즉, update함수를 중첩해서 부르면 더 **깊은 단계로 중첩된 객체에도 사용이 가능**해진다.

함수 이름에 사용되는 암묵적 인자를 두 번(increment, ‘size’)이나 사용하고 있기 때문에 리팩토링이 필요하다.

```tsx
function updateOption(item, option, modify) {
	return update(item, 'options', function(options) {
		return update(options, option, modify);
	})
}
```

명시적으로 바꾸었지만 아직 `‘options’`가 해결되지 않았다. 

이것을 해결하기 위해서는 `update`에 필요한 키를 두개 받는 `update2`함수를 만들면 해결이 된다.

```tsx
function update2(object, key1, key2, modify) {
	return update(object, key1, function(options) {
		return update(options, key2, modify);
	})
}

// update2 함수를 사용해서 incrementSize코드를 다시 수정해보자.
function incrementSize(item) {
	return update2(item, 'options', 'size', increment)
}
```

이때 변경하고 싶은 키(중첩된 객체)를 찾아 들어가기 때문에 경로(path)라고 부를 수 있다.

(만약 중첩이 두개가 아니라 N개가 되면 그럴때마다 만들것인가? 라는 생각이 들었는데 책의 뒷부분에 해결하는 방식이 나오더라 ㅋㅋㅋ)

### update3 함수를 만드는 4가지 방법

```tsx
const cart = {
	shirt: {
		name: 'shirt',
		price: 13,
		options: {
			color: 'blue',
			size: 3
		}
	}
}

// 방법 1.
function incrementSizeByName(cart, name) {
	return update(cart, name, incrementSize);
}

// 방법 2.
function incrementSizeByName(cart, name) {
	return update(cart, name, function(item) {
		return update2(item, 'options', 'size', function(size) {
			return size + 1;
		})
	})
}

// 방법 3.
// 이건 진짜 아닌듯
function incrementSizeByName(cart, name) {
	return update(cart, name, function(item) {
		return update(item, 'options', function(options) {
			return update(options, 'size', function(size) {
				return size + 1;
			});
		})
	})
}

// 방법 4.
function incrementSizeByName(cart, name) {
	const item = cart[name];
	const options = item.options;
	const size = options.size;
	const newSize = size + 1;
	const newOptions = objectSet(options, 'size', newSize);
	const newItem = objectSet(item, 'options', newOptions);
	const newCart = objectSet(cart, name, newItem);

	return newCart;
}
```

4가지 방법 모두 좋은 방법처럼 보이진 않는다. `update3`함수를 도출해보자

```tsx
function update3(object, key1, key2, key3, modify) {
	return update(object, key1, function(object2) {
		return update(objec2, key2, key3, modify);
	})
}
```

update3 까지 만들었지만 언제까지나 만들 수는 없는 노릇, 더 깊은 중첨에도 대응하기 위한 코드를 만드려고 보니 **어느 정도 패턴**이 보인다.

**만약 updateX()를 만든다고 하면 update() 안에 updateX-1()를 불러주면 해결이 된다.
마지막으로 0번째 까지 오게 되면 중첩되지 않은 뎁스까지 온 것이기 때문에 변경만 해주면 된다.**

### nestedUpdate 함수 도출하기

```tsx
function updateX(object, depth, key1, key2, key3, modify) {
	return update(object, key1, function(value1) {
		return updateX(value, depth - 1, key2, key3, modify);
	})
}
```

여기서 문제가 보일 것이다. X라는 미지의 뎁스로 만들고 있지만 key3까지 보내고 있다.
그렇기 때문에 우리는 **키의 개수와 순서**를 지켜서 사용해야 한다.

```tsx
function updateX(object, keys, modify) {
	const key1 = keys[0];
	const restOfKeys = drop_first(keys);
	
	return update(object, key1, function(value1) {
		// 재귀에 처음 키는 빼고 다시 호출
		return updateX(value, restOfKeys, modify);
	})
}
```

재귀를 멈추기 위해서는 종료조건이 있어야 한다. 우리가 아는 것은 중첩된 객체의 키를 모두 돌았으면 멈춰야 한다. 이를 코드로 구현하자

```tsx
function updateX(object, keys, modify) {
	if (keys.length === 0) {
		return modify(object);
	}
	const key1 = keys[0];
	const restOfKeys = drop_first(keys);
	
	return update(object, key1, function(value1) {
		// 재귀에 처음 키는 빼고 다시 호출
		return updateX(value, restOfKeys, modify);
	})
}
```

이렇게 최종적으로 어떠한 중첩된 객체여도 변경할 키만 안다면 바꿀 수 있는 함수가 만들어 졌다. 마지막으로 함수명을 보편적인 것으로 바꾸면 끝이다.

```tsx
function nestedUpdate(object, keys, modify) {
	if (keys.length === 0) {
		return modify(object);
	}
	const key1 = keys[0];
	const restOfKeys = drop_first(keys);
	
	return update(object, key1, function(value1) {
		// 재귀에 처음 키는 빼고 다시 호출
		return nestedUpdate(value, restOfKeys, modify);
	})
}
```

### 시각화해서 보기

![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/253d1ac1-0c8d-4179-8d90-21ade38e0aea/69d93115-cc44-4428-b33d-6dac2ef4d9d7/image.png)

### 깊이 중첩된 구조를 설계할 때 생각할 점

nestedUpdate()를 쓰려면 긴 키의 경로를 알아야 한다. 키를 기억하는 것은 어렵다. 

api 사용해서 내려주는 응답값의 필드를 확인해서 키의 경로대로 입력을 해야 오류가 나지 않는다.

이것을 해결하게 해주는 방법이 바로 **추상화 벽**이다. 추상화 벽을 사용하여 사용할때 구체적인 것을 함수에 맡기고 사용하는 쪽에서는 함수의 이름만으로도 충분히 사용할 수 있도록 만들어 주면 되는 것이다.

```tsx
// 기존 코드
httpGet("http://my-blog.com/api/category/blog", function(blogCategory) {
    renderCategory(nestedUpdate(blogCategory, ['posts', '12', 'author', 'name'], capitalize));
});

// 리팩토링 된 코드
function updatePostById(category, id, modifyPost) {
	return nestedUpdate(category, ['posts', id], modifyPost);
}

function updateAuthor(post, modifyUser) {
	return update(post, 'author', modifyUser);
}

function capitalizeUserName(user) {
	return update(user, 'name', capitalize);
}

// 함수를 한번에 사용하기
httpGet("http://my-blog.com/api/category/blog", updatePostById(blogCategory, '12', function(post) {
	return updateAUthor(post, capitalizeUserName);
}));
```

좋아진 두 가지 이유

- 기억해야할 것이 네가지에서 세가지로 줄어듦
    - ‘post’, ‘12’, ‘author’, ‘name’ ⇒ name을 기억하지 않아도 됌
- 동작의 이름이 있으므로 각각의 동작을 기억하는 것이 쉬워짐
    - 어떤 키가 들어있는지 기억하지 않아도 된다. ⇒ 어떤 키가 들어있는지 몰라도 되지만 함수의 사용법을 알아야 하는 것으로 바뀐 것 같음

```tsx
// 위의 코드로 보면 이와 같은 데이터 형태로 추론이 가능하다.
const blogCategory = {
	posts: {
	  // ...
		12: {
		  // ...
			author: {
				name: 'kim'
				// ...
			}
		}
	}
}

type BlogCategory = {
	posts: Array<Post>
}

type Post = Record<number, Author>;
type Author = {
	name: string;
}
```

**순서를 기억**해야한다는 리스크가 너무 큰 것 같음, 순서를 보장해주지 않으면 동작하지 않을 수 있다는 리스크가 코드의 안정성을 떨어트리는게 아닌가 하는 생각이 들었음

그래서 3개 이상의 중첩을 사용해야 한다면 함수를 쪼개는게 맞는 것 같다. 그런데 3개 이상이 중첩이 아니면 스프레드 문법이 더 가독성이 높아보인다.. 흠..ㅋㅋㅋ

여기까지 읽고 든 나의 생각  

1.  함수형 프로그래밍을 도입하려면 팀전체가 함께 움직이는 것은 필연적일 듯 한 것 같다.
    1. 일부에만 적용하거나 일부 서비스에만 도입은 거의 불가능해 보임
2. 일반적으로 사용하던 코딩 방식이 아니여서 적용하는데 까지 시간이 걸릴 것 같다.
    1. 위의 함수만 예를 들어도 `nestedUpdate` 함수 사용법을 아는 건 둘째치고 만드는 것부터가 난이도가 매우 어려워보인다.