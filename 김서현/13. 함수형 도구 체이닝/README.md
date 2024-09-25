# 13. 함수형 도구 체이닝

- [체이닝 적용](#체이닝-적용)
- [체이닝 팁](#체이닝-팁)
- [체이닝 디버깅 팁](#체이닝-디버깅-팁)
- [다양한 함수형 도구](#다양한-함수형-도구)
- [값을 만들기 위한 reduce()](#값을-만들기-위한-reduce)

## 체이닝 적용

- 우수 고객의 가장 비싼 구매 구하기
    1. 우수 고객(3개 이상 구매)를 거름 (filter)
    2. 우수 고객을 가장 비싼 구매로 바꿈 (map)
    
    ```tsx
    function biggestPurchasesBestCustomers(customers) {
      // 1. 우수 고객 거르기
      var bestCustomers = filter(customers, function (customer) {
        return customer.purchases.length >= 3;
      });
      // 2. 고객의 가장 비싼 구매 가져와 체인에 담음
      var biggestPurchases = map(bestCustomers, function (customer) {
        // 가장 비싼 구매
        return maxKey(customer.purchases, { total: 0 }, function (purchase) {
          return purchase.total;
        });
      });
      return biggestPurchases;
    }
    ```
    
- 체인 명확하게 만들기
    1. 방법 1: 단계에 이름 붙이기
        
        ```tsx
        function biggestPurchasesBestCustomers(customers) {
          var bestCustomers = selectBestCustomers(customers); // 1단계
          var biggestPurchases = getBiggestPurchases(bestCustomers); // 2단계
          return biggestPurchases;
        }
        
        // 1
        function selectBestCustomers(customers) {
          return filter(customers, function (customer) {
            return customer.purchases.length >= 3;
          });
        }
        
        // 2
        function getBiggestPurchases(customers) {
          return map(customers, getBiggestPurchase);
        }
        
        // 고차함수를 함수로 빼냄
        function getBiggestPurchase(customer) {
          return maxKey(customer.purchases, { total: 0 }, function (purchase) {
            return purchase.total;
          });
        }
        ```
        
    2. 방법 2: 콜백에 이름 붙이기 - 더 명확, 재사용하기도 좋음
        
        ```tsx
        function biggestPurchasesBestCustomers(customers) {
          var bestCustomers = filter(customers, isGoodCustomer); // 1단계
          var biggestPurchases = map(bestCustomers, getBiggestPurchase); // 2단계
          return biggestPurchases;
        }
        
        function isGoodCustomer(customer) {
          return customer.purchases.length >= 3;
        }
        
        function getBiggestPurchase(customer) {
          return maxKey(customer.purchases, { total: 0 }, getPurchaseTotal);
        }
        
        function getPurchaseTotal(purchase) {
          return purchase.total;
        }
        ```
        
    - filter(), map() 모두 새로운 배열을 만들지만 만들어진 배열이 필요 없을 때 가비지 컬렉터가 빠르게 처리해 문제되지 않음.
    - **스트림 결합**: 체인 최적화
        - 두 번 map() 부르는 동작 한 단계로 합칠 수 있음
            
            ```tsx
            var names = map(customers, getFullName);
            var nameLengths = map(names, stringLength);
            ```
            
            ```tsx
            var nameLengths = map(customers, function (customer) {
              return stringLength(getFullName(customer));
            });
            ```
            
        - filter()
            
            ```tsx
            var goodCustomers = filter(customers, isGoodCustomer);
            var withAddresses = filter(goodCustomers, hasAddress);
            ```
            
            ```tsx
            var withAddresses = filter(customers, function (customer) {
              return isGoodCustomer(customer) && hasAddress(customer);
            });
            ```
            
        - reduce()
            
            ```tsx
            var purchaseTotals = map(purchases, getPurchaseTotal);
            var purchaseSum = reduce(purchaseTotals, 0, plus);
            ```
            
            ```tsx
            var purchaseSum = reduce(purchases, 0, function (total, purchase) {
              return total + getPurchaseTotal(purchase);
            });
            ```
            

## 체이닝 팁

- 데이터 만들기 - 함수형 도구는 배열 전체를 다룰 때 잘 동작함. 배열 일부에 대해 동작하는 반복문이 있다면 배열 일부를 새로운 배열로 나눌 수 있음.
- 배열 전체 다루기 - 전체 배열 한번에 처리.
- 작은 단계로 나누기
- 조건문을 filter()로 바꾸기 - 반복문 안에 조건문 대신 앞 단계에서 filter()로 거르기
- 유용한 함수로 출력하기
- 개선을 위해 실험하기
- 예제
    - 기존 코드
        
        ```tsx
        var answer = [];
        var window = 5;
        
        // 배열 개수만큼 계산
        for (var i = 0; i < array.length; i++) {
          var sum = 0;
          var count = 0;
          // 작은 구간 반복
          for (var w = 0; w < window; w++) {
            // 새 인덱스 계산
            var idx = i + w;
            if (idx < array.length) {
              // 값 누적
              sum += array[idx];
              count += 1;
            }
          }
          answer.push(sum / count);
        }
        ```
        
    1. 데이터 만들기
        
        ```tsx
        var answer = [];
        var window = 5;
        
        for (var i = 0; i < array.length; i++) {
          var sum = 0;
          var count = 0;
          // 하위 배열로 만듦
          var subarray = array.slice(i, i + window);
          // 반복문으로 배열 반복
          for (var w = 0; w < subarray.length; w++) {
        	  // 하위 배열의 합과 개수 구함
            sum += subarray[w];
            count += 1;
          }
          // 평균 구하기 위해 나눔
          answer.push(sum / count);
        }
        ```
        
    2. 한 번에 전체 배열 조작하기
        
        ```tsx
        var answer = [];
        var window = 5;
        
        for (var i = 0; i < array.length; i++) {
          var subarray = array.slice(i, i + window);
          answer.push(average(subarray));
        }
        ```
        
    3. 작은 단계로 나누기
        
        ```tsx
        var indices = [];
        // 인덱스가 들어잇는 배열 만듦
        for (var i = 0; i < array.length; i++) indices.push(i);
        
        var window = 5;
        // 인덱스 배열에 map() 사용해 각 항목마다 인덱스 가지고 콜백 부름
        var answer = map(indices, function (i) {
          var subarray = array.slice(i, i + window); // 하위 배열 만들기
          return average(subarray); // 평균 계산하기
        });
        ```
        
    4. 코드 단계 나누고 인덱스 배열 만드는 코드 빼내기
        
        ```tsx
        var window = 5;
        
        // 단계 1: 인덱스 배열 생성
        var indices = range(0, array.length);
        
        // 단계 2: 하위 배열 만들기
        var windows = map(indices, function (i) {
          return array.slice(i, i + window);
        });
        
        // 단계 3: 평균 계산하기
        var answer = map(windows, average);
        ```
        

## 체이닝 디버깅 팁

- 구체적인 것을 유지하기 - 각 단계에서 어떤 것을 하고 있는지 알기 쉽게 이름 지어야
- 출력해보기 - 각 단계 사이에 로그 찍어보기
- 타입을 따라가 보기

## 다양한 함수형 도구

- `pluck()`: map()으로 특정 필드값 가져오기 위해 콜백 작성하는 일 안 해도 됨
    - 함수
        
        ```tsx
        function pluck(array, field) {
          return map(array, function (object) {
            return object[field];
          });
        }
        ```
        
    - 사용법
        
        ```tsx
        var prices = pluck(products, "price");
        ```
        
    - 비슷한 도구
        
        ```tsx
        function invokeMap(array, method) {
          return map(array, function (object) {
            return object[method]();
          });
        }
        ```
        
- `concat()`: 중첩된 배열을 한 단계의 배열로 만듦
    - 함수
        
        ```tsx
        function concat(arrays) {
          var ret = [];
          forEach(arrays, function (array) {
            forEach(array, function (element) {
              ret.push(element);
            });
          });
          return ret;
        }
        ```
        
    - 사용법
        
        ```tsx
        var purchaseArrays = pluck(customers, "purchases");
        var allPurchases = concat(purchaseArrays);
        ```
        
    - 비슷한 도구
        
        ```tsx
        function concatMap(array, f) {
          return concat(map(array, f));
        }
        ```
        
- `frequenciesBy()`, `groupBy()`: 개수 세거나 그룹화. 객체 또는 맵 리턴
    - 함수
        
        ```tsx
        function frequenciesBy(array, f) {
          var ret = {};
          forEach(array, function (element) {
            var key = f(element);
            if (ret[key]) ret[key] += 1;
            else ret[key] = 1;
          });
          return ret;
        }
        
        function groupBy(array, f) {
          var ret = {};
          forEach(array, function (element) {
            var key = f(element);
            if (ret[key]) ret[key].push(element);
            else ret[key] = [element];
          });
          return ret;
        }
        ```
        
    - 사용법
        
        ```tsx
        var howMany = frequenciesBy(products, function (p) {
          return p.type;
        });
        ```
        
    - 비슷한 도구
        
        ```tsx
        var groups = groupBy(range(0, 10), isEven);
        ```
        

## 값을 만들기 위한 reduce()

- reduce()로 값 만들기
    
    ```tsx
    var itemsAdded = ["shirt", "shoes", "shirt", "socks", "hat", "...."];
    
    var shoppingCart = reduce(itemsAdded, {}, function (cart, item) {
      // 추가하려는 제품이 장바구니에 없는 경우
      if (!cart[item])
        return add_item(cart, {
          name: item,
          quantity: 1,
          price: priceLookup(item),
        });
      // 이미 장바구니에 있는 경우
      else {
        var quantity = cart[item].quantity;
        return setFieldByName(cart, item, "quantity", quantity + 1);
      }
    });
    ```
    
- 함수 분리한 버전
    
    ```tsx
    var shoppingCart = reduce(itemsAdded, {}, addOne);
    
    function addOne(cart, item) {
      if (!cart[item])
        return add_item(cart, {
          name: item,
          quantity: 1,
          price: priceLookup(item),
        });
      else {
        var quantity = cart[item].quantity;
        return setFieldByName(cart, item, "quantity", quantity + 1);
      }
    }
    ```
    
- 삭제 추가
    
    ```tsx
    var itemOps = [
      ["add", "shirt"],
      ["add", "shoes"],
      ["remove", "shirt"],
      ["add", "socks"],
      ["remove", "hat"],
    ];
    
    var shoppingCart = reduce(itemOps, {}, function (cart, itemOp) {
      var op = itemOp[0];
      var item = itemOp[1];
      // 동작에 따라 알맞은 함수 실행
      if (op === "add") return addOne(cart, item);
      if (op === "remove") return removeOne(cart, item);
    });
    
    function removeOne(cart, item) {
      // 장바구니에 제품 없다면 아무것도 하지 않음
      if (!cart[item]) return cart;
      else {
        var quantity = cart[item].quantity;
        // 수량이 하나일 땐 제품 삭제
        if (quantity === 1) return remove_item_by_name(cart, item);
        // 아니면 수량 줄임
        else return setFieldByName(cart, item, "quantity", quantity - 1);
      }
    }
    ```
