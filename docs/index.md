# XpressEngine3 Front-End

XpressEngine 3(이하 XE)의 Front-End의 기능 및 동작은  `XE` 변수에 담겨 있으며, App 및 Event, AOP를 이용할 수 있습니다.

## window.XE
`window.XE`를 노출하고 있으며, XE에서 제공하는 기능이 담겨있습니다.

- Event를 구독, 트리거 할 수 있도록 EventEmitter를 확장하고 있습니다
- Request, Router, Lang 등 내장된 App을 사용할 수 있는 상태로 포함하고 있습니다.
  - 각 App은 EventEmitter를 확장하고 있으며, 개별적으로 이벤트를 관리합니다.
- `XE.app()` 메소드로 Request 등 등록된 App의 instance를 전달하는 [Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/promise)를 반환합니다.

## EventEmitter
EventEmitter는 이벤트에 대한 listener를 관리하고 트리거하는데 사용됩니다.

- $$emit, $$on, $$once, $$off, $$offAll
- `$$on`, `$$once`
    - callback 첫번째 argument는 발생된 이벤트의 이름
- before 옵션으로 순서를 조정할 수 있습니다.
    - 이름을 가진 listener만 대상으로 할 수 있습니다.

```js
XE.$$on('eventName', /*callback*/(eventName, arg1/*, arg2, ...*/) => {
    console.debug('emitted', eventName, arg1, arg2)
}, options)

XE.$$on('setup', (eventName, arg) => {
    console.debug('emitted', eventName, arg)
}, { name: 'low'})

XE.$$on('setup', (eventName, arg) => {
    console.debug('emitted', eventName, arg)
}, { name: 'common', before: 'low' } )
```

## XE.app()
- XE Object에 포함되는 모듈은 App을 확장하였으며, Singleton임
- EventEmitter를 확장하고 있음
- XE Core에서 다루는 것만을 고려했으므로 외부에서 이를 직접 확장하여 사용하기는 어려울 수 있음
- `boot` method를 포함하며, XE Object와 back-end로부터 전달된 주요 설정을 전달 받아 처리할 수 있음

```js
// XE.app()은 Promise를 반환 함

// boot된 instance를 반환
XE.app('Request').then((appRequest) => {
    appRequest.get()
})

// instance를 반환 (아직 boot 되지 않았을 수 있음)
XE.app('Request', (appRequest) => {
    appRequest.get()
})
```

---

## Apps

### Lang
#### XE.Lang.trans( id [, parameter] )
back-end로부터 전달된 다국어 목록에서 지정된 ID의 다국어를 반환합니다.

- id (string)
    - 다국어가 정의된 key값을 지정합니다. ( 필수 요소 )
- parameter (object)
    - object타입으로 2번째 인자에 사용하면 해당 번역에 지정된 문자열을 치환합니다.

back-end에서 다국어를 front-end에 전달하기 위해 아래와 같이 두 가지 방법을 사용할 수 있습니다. `expose_trans` PHP 함수 또는 blade 지시자로 사용할 수 있습니다.
```php
// PHP helper 함수 (core/src/Xpressengine/Support/helpers.php)
expose_trans('xe::name'); // '이름'

// blade directive
@expose_trans('xe::msgHello') // '안녕하세요. :msg'
```

front-end에 전달되지 않은 다국어는 namespace(구분자 `::` 앞의 문자열)를 제거한 key를 반환합니다('xe::msgHello'은 'msgHello').


```js
//javascript
var name = XE.Lang.trans('xe::name');
var msg = XE.Lang.trans('xe::msgHello', {msg: '반갑습니다.'});
console.log(name) // 이름 or Name
console.log(msg) // '안녕하세요. 반갑습니다.'
```

---

### Request
XE에 비동기 요청을 요청하고 처리하기 위해 사용합니다. 요청에 대한 결과는 Promise를 반환합니다.

공통으로 다음과 같은 arguments를 지정할 수 있습니다.
- url (string. required)
  - 요청보낼 URL 값
- options (object. optional)
  - 서버로 전송할 data 및 parameter, header

지원하는 Method 목록은 아래와 같습니다. HEAD를 제외하고 `XE.get()`과 같이 `XE` 객체에서 요청할 수도 있습니다.

- GET: `XE.Request.get()`
- POST: `XE.Request.post()`
- DELETE: `XE.Request.delete()`
- PUT: `XE.Request.put()`
- HEAD: `XE.Request.head()`

#### GET 요청 예시
```js
XE.get('/path/to/target', { data: { id: 123 } }).then(function (response) {
  console.log(response);
  // => response = { data, status, statusText }
});
```

---

### Validator
front-end로 Validation Rule를 전달하기위해 back-end에서 다음과 같이 사용할 수 있습니다.

```php
XeFrontend::rule('myRuleName', [
    'email' => 'required|email',
    'password' => 'required'
]);
```


#### XE.formValidate( $form )
폼 요소에 있는 입력값의 유효성을 검사합니다. Form element의 `data-valid` attribute로 지정된 rule로 입력 밧을 검사합니다. 유효성 검사를 하고자 하는 내용을 '|'구분하여 지정하면 여러개의 유효성을 체크하게되고 유효성이 통과하지 못할 경우 메시지를 노출합니다.

#### Attribute
##### data-rule-alert-type (form)
유효성 체크를 통과하지 못할 경우 보여질 메시지형태를 정의합니다. 지원하는 메시지 출력 형태는 아래와 같습니다.

- toast
  - XE.toast 팝업의 형태로 메시지가 노출됩니다.
- form
  - 해당 필드 요소하단에 메시지가 노출됩니다.
  
```html
<form data-rule-alert-type="toast">
...
</form>
```

##### data-valid (input element)
입력 값을 검사할 rule을 지정합니다.

```html
<input type="text" name="email" data-valid="required|email" />
```

지원하는 룰은 아래 목록에서 확인할 수 있습니다.

- required
  - 필수 요소일 경우 사용합니다.
- checked:min-max
  - checkbox, raido 타입에 사용됩니다.
  - **첫번째 엘리먼트**에만 해당 요소가 정의되어야 합니다.
  - min, max는 number 타입으로 지정합니다. radio button에서는 필수 체크시 **checked:1**로 표기 합니다.
- alpha
  - 알파벳으로만 사용되는 필드 값을 체크합니다.
- alphanum 또는 alpha_num
  - 알파벳 또는 숫자 값을 체크합니다.
- min
  - 최소 입력 글자를 체크합니다
- max
  - 최대 입력 글자를 체크합니다
- email
  - 이메일 형식인지 체크합니다.
- url
  - url형식으로 입력되었는지 체크합니다.
- numeric
 - 숫자값만 입력되었는지 체크합니다.
- between:min,max
 - 필드 값이 최소, 최대에 만족하는지 체크합니다.
- accepted
 - 필드의 값이 yes, on, 1, 또는 _true_이어야 합니다.
- alpha_dash
 - 필드의 값이 (숫자나 기호가 아닌) 알파벳[자음과 모음] 문자 및 숫자와 dash(-), underscore(_)로 이루어져야 합니다.
- array
 - 필드의 값이 배열형태이어야 합니다.
- boolean
 - 필드의 값이 반드시 true, false, 1, 0, "1", "0" 이어야 합니다.
- date
 - 필드의 값이 strtotime PHP 함수에서 인식할 수 있는 올바른 날짜여야 합니다.
- date_format:format
 - 필드의 값이 반드시 주어진 format과 일지해야 합니다.
- digits:value
 - 필드의 값이 반드시 숫자여야 하고, 길이가 value이어야 합니다
- digits_between:min,max
 - 필드의 값이 주어진 min과 max사이의 길이를 가져야 합니다.
- filled
 - 필드가 존재하는 경우 값이 비어있으면 안됩니다.
- integer
 - 필드의 값이 정수여야 합니다.
- ip
 - 필드의 값이 IP 주소여야 합니다.
- mimes:foo,bar...
 - 파일의 MIME 타입이 주어진 확장자 리스트 중에 하나와 일치해야 합니다.
- regex:pattern
 - 필드의 값이 주어진 정규식 표현과 일치해야 합니다.
- json
 - 필드의 값이 유효한 JSON 문자열이어야 합니다.
- string
 - 필드의 값이 반드시 문자열이어야 합니다.

##### data-valid-name (input element)
validation 실패 시 'resources/lang/ko|en/validation.php' 파일에 정의된 다국어를 사용하여 `:attribute` 기본값으로 엘리먼트의 name속성을 사용하고 있습니다. validation 실패 메시지에서 치환되는 `:attribute`를 input element의 name 대신 사용자 정의할 수 있습니다.

```html
<!-- XE form sample -->
<form id='form' action="/users" method="POST" data-rule-alert-type="toast">
  <input type="text" name="name" data-valid="required" data-valid-name="이름(비실명)" />
  <input type="password" name="current_password" data-valid="required" data-valid-name="현재 비밀번호" />
  <input type="password" name="password" data-valid="required" data-valid-name="새 비밀번호" />
  <input type="password" name="password_confirmation" data-valid="required" data-valid-name="새 비밀번호(확인)" />
</form>
```

```javascript
XE.formValidate($('#form'));
// => 'password_confirmation은(는) 필수입력 항목입니다.'
// 대신 '새 비밀번호(확인)은(는) 필수입력 항목입니다.' 메시지 출력
```

##### data-valid-message (input element)
'새 비밀번호(확인)은(는) 필수입력 항목입니다.'와 같은 정돈하지 않은 듯한 메시지를 사용자 정의하여 변경할 수 있습니다.

```html
<!-- XE form sample -->
<form>
  <input type="password" name="password_confirmation" data-valid="required" data-valid-message="새 비밀번호는 동일하게 입력해주세요" />
</form>
// => '새 비밀번호는 동일하게 입력해주세요'
```


### DynamicLoadManager



## Utils
- window.XE.Utils
