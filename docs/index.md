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




## Utils
- window.XE.Utils
