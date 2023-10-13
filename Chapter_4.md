<aside>
✔️ 살펴볼 내용

- 테스트하기 쉽고 재 사용성이 좋은 코드를 위한 함수형 기술에 대해 알아보기
- 액션에서 계산을 빼내는 방법
</aside>

### 기본 기능

- 쇼핑 중에 장바구니에 담겨있는 금액 합계를 볼 수 있는 코드입니다.

```jsx
// 장바구니 제품과 금액 합계를 담고 있는 전역변수
var shopping_cart = [];
var shopping_cart_total = 0;

function add_item_to_cart(name, price) {
  // 장바구니에 제품 담기
  shopping_cart.push({
    name: name,
    price: price,
  });
  // 장바구니 금액 합계 업데이트
  calc_cart_total();
}

function calc_cart_total() {
  shopping_cart_total = 0;
  for (var i = 0; i < shopping_cart.length; i++) {
    // 모든 제품값 더하기
    shopping_cart_total += item.price;
  }
  // 금액 합계를 반영하기 위해 DOM 업데이트
  set_cart_total_dom();
}
```

### 무료 배송비 계산하기 추가하기

- 위의 코드에서 구매 합계가 20달러 이상이면 무료 배송을 새로 제공하여 20달러가 넘는 제품에 한하여 무료 배송 아이콘을 표시해 주려고 합니다.
- 해당 코드를 절차적인 방법으로 먼저 작성합니다.

```jsx
function update_shipping_icons() {
  // 페이지의 모든 구매버튼을 가져와서 적용합니다.
  var buy_buttons = get_buy_buttons_dom();
  for (var i = 0; i < buy_buttons.length; i++) {
    var button = buy_bottons[i];
    var item = button.item;
    // 무료배송이 가능한지 확인하여 아이콘을 보여주거나 보여주지 않습니다.
    if (item.price + shopping_cart_total >= 20)
      button.show_free_shipping_icon();
    else button.hide_free_shipping_icon();
  }
}
```

- 합계 금액이 바뀔 때마다 아이콘을 업데이트 하기 위해 `calc_cart_total()` 함수 안에 `update_shipping_icons()` 함수를 불러옵니다.

```jsx
function calc_cart_total() {
  shopping_cart_total = 0;
  for (var i = 0; i < shopping_cart.length; i++) {
    shopping_cart_total += item.price;
  }
  set_cart_total_dom();
  // 아이콘을 업데이트하는 코드 추가합니다.
  update_shipping_icons();
}
```

### 테스트를 쉽게 할 수 있는 방법

- 현재 코드는 테스트 하기 어렵게 작동합니다.
- 현재 테스트를 진행하기 위해서는 위와 같은 테스트를 만들어야 합니다.

  1. 브라우저 설정
  2. 페이지 로드
  3. 장바구니에 제품 담기 버튼 클릭
  4. DOM이 업데이트될 때까지 기다리기
  5. DOM에서 값 가져오기
  6. 가져온 문자열 값을 숫자로 바꾸기
  7. 예상하는 값과 비교하기

- **테스트를 더 쉽게 하려면 다음과 같은 조건**이 필요합니다.
  1. DOM 업데이트와 비즈니스 규칙은 분리되어야 한다.
  2. 전역변수가 없어야 한다.

### **재 사용하기 쉽게 만들기**

- 결제팀과 배송팀에서도 위에 작성한 코드를 사용하고 싶었지만 다음과 같은 이유로 사용할 수 없었습니다.
  1. 장바구니 정보를 전역변수에서 읽어오고 있지만, 결제팀과 배송팀은 데이터베이스에서 장바구니 정보를 읽어옵니다.
  2. 결과를 보여주기 위해 DOM을 직접 바꾸고 있지만, 결제팀은 영수증을, 배송팀을 운송장을 출력해야 합니다.

```jsx
function update_shipping_icons() {
  var buy_buttons = get_buy_buttons_dom();
  for (var i = 0; i < buy_buttons.length; i++) {
    var button = buy_bottons[i];
    var item = button.item;
    // 결제팀과 배송팀에서 이 미즈니스 규칙을 사용하려고 합니다.
    // 전역 변수인 shopping_cart_total가 있어야 사용 가능 합니다.
    if (item.price + shopping_cart_total >= 20)
      // DOM이 있어야 실행 가능 합니다.
      button.show_free_shipping_icon();
    else button.hide_free_shipping_icon();
  }
  // return 값이 없어 결과를 받을 수 없습니다.
}
```

- **재사용성**을 좋게 하려면 아래와 같은 조건이 필요합니다.
  1. 전역변수에 의존하지 않아야 합니다.
  2. DOM을 사용할 수 있는 곳에서 실행된다고 가정하면 안 됩니다.
  3. 함수가 결괏값을 리턴해야 합니다.

### **액션과 계산, 데이터를 구분하기**

- 모든 전역변수, DOM에서 엘리먼트 변경를 사용하는 코드들은 액션으로 구분됩니다.
- 따라서 위의 코드들은 **모두 액션**입니다.

### **함수에는 입력과 출력이 있다**

- 모든 함수에는 입력과 출력이 있으며 **입력**은 함수가 계산을 하기 위한 외부 정보이고, **출력**은 함수 밖으로 나오는 정보나 어떤 동작입니다.
- 모든 입력과 출력은 암묵적일 수 있으며 **암묵적 입력**과 **출력**이 있으면 **액션**이 됩니다.
  - 인자 : 명시적
  - 전역변수, 콘솔 : 암묵적
- 암묵적 입력과 출력을 **부수 효과**라고 부릅니다.

### 테스트와 재 사용성은 입출력과 관련 있습니다.

- **테스트를 더 쉽게 하기 위한 조건**

  1. DOM 업데이트와 비즈니스 규칙은 분리되어야 합니다.
     - DOM 업데이트는 함수에서 나오기 때문에 출력. 하지만 리턴값이 없어서 암묵적 출력이 됩니다.
  2. 전역변수가 없어야 합니다.

- **재 사용성을 좋게 하기 위한 조건**
  1. 전역변수에 의존하지 않아야 합니다.
  2. DOM을 사용할 수 있는 곳에서 실행된다고 가정하면 안 됩니다.
  3. 함수가 결괏값을 리턴해야 합니다.
- 테스트와 재사용성을 좋게 하기 위해서는 **암묵적 입력**과 **출력**을 **제거**해야 합니다.

### 액션에서 계산 빼내기

```jsx
function calc_cart_total() {
  shopping_cart_total = 0;
  // 계산에 해당되는 코드 시작
  for (var i = 0; i < shopping_cart.length; i++) {
    var item = shopping_cart[i];
    shopping_cart_total += item.price;
  }
  // 계산에 해당되는 코드 끝
  set_cart_total_dom();
  update_shipping_icons();
  update_tax_dom();
}

function calc_total() {
  shopping_cart_total = 0; // 전역 변수
  // shopping_cart 전역 변수
  for (var i = 0; i < shopping_cart.length; i++) {
    var item = shopping_cart[i];
    shopping_cart_total += item.price;
    // 전역변수 변경
  }
}
```

- 이 함수는 명시적인 입력과 명시적인 출력이 없기 때문에 **아직도 액션 함수**입니다.

```jsx
function calc_cart_total() {
  shopping_cart_total = calc_total(**shopping_cart**); // shopping_cart를 인자로 넣은 함수의 계산 결과를 전역변수에 할당

  set_cart_total_dom();
  update_shipping_icons();
  update_tax_dom();
}

function calc_total(**cart**) { // 전역변수 대신 인자를 만들어 사용
  **var total = 0;  // 지역변수로 변경**
  for(var i = 0; i < shopping_cart.length; i++) {
    var item = shopping_cart[i];
    total += item.price;
  }
  **retrun total;**
}
```

- 최종적으로 암묵적 출력과 암묵적 입력 없앤 코드
- 이러한 리팩터링은 **서브루틴 추출하기**라고도 합니다.
  - 동작을 유지하며 코드를 바꾸는 것 (지역변수를 입력으로 받고 지역변수를 리턴한다)

### 단계별 계산 추출

1. 계산 코드를 찾아 뺴냅니다.
2. 새 함수에 암묵적 입력과 출력을 찾습니다.
3. 암묵적 입력은 인자로, 압묵적 출력은 리턴값으로 바꿉니다.

---

## 요점 정리

- **액션**은 **암묵적 입력**과 **출력**을 가지고 있습니다.
- **계산**은 **암묵적 입력**이나 **출력**이 없어야 합니다.
- **암묵적 입력**은 **인자**, **암묵적 출력**은 **리턴값**으로 바꿀 수 있습니다.
- **테스트**와 **재 사용성**을 높이기 위한 **암묵적 입출력을 없애는 작업**은 함수형 프로그래밍 내용과 동일합니다.
- **함수형 원칙**을 사용하면 액션이 줄어들고 계산이 늘어나게 됩니다.
- **액션을 계산으로 바꾸는 과정**을 ‘서브루틴 추출하기’ 라고 합니다.
