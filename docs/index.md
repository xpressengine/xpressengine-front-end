# FE

## window.XE
`window` object에는 `window.XE`를 노출하고 있으며, 코어에서 지원하는 대부분의 기능이 여기에 담김

- EventEmitter가 확장되어 있음
- Request, Router, Lang 등의 App을 사용할 수 있는 상태로 포함하고 있음
- `XE.app()` 메소드로 Request 등 등록된 App의 instance를 전달하는 [Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/promise)를 반환함

```js
// XE.app()은 Promise를 반환 함

// boot된 instance를 반환
XE.app('Request').then((requestInstance) => {
    requestInstance.get()
})

// instance를 반환 (아직 boot 되지 않았을 수 있음)
XE.app('Request', (requestInstance) => {
    requestInstance.get()
})
```

## EventEmitter
EventEmitter는 이벤트에 대한 listener를 관리하기 위해 제작됨

- $$emit, $$on, $$once, $$off, $$offAll
- `$$on`, `$$once`
    - callback 첫번째 argument는 발생된 이벤트의 이름임
        - (미구현) 다중 이벤트를 구독할 수 있도록 개선 예정이어서 이러한 규칙을 가짐
- before 옵션으로 순서를 조정할 수 있음
    - 이름을 가진 listener만 대상으로 할 수 있음

```js
XE.$$on('eventName', /*callback*/(eventName, arg1, arg2/*, arg3, ...*/) => {
    console.debug('emitted', eventName, arg1, arg2)
}, options)

XE.$$on('setup', (eventName, arg) => {
    console.debug('emitted', eventName, arg)
}, { name: 'low'})

XE.$$on('setup', (eventName, arg) => {
    console.debug('emitted', eventName, arg)
}, { name: 'common', before: 'low' } )
```

## App
- XE Object에 포함되는 모듈은 App을 확장하였으며, Singleton임
- EventEmitter를 확장하고 있음
- XE Core에서 다루는 것만을 고려했으므로 외부에서 이를 직접 확장하여 사용하기는 어려울 수 있음
- `boot` method를 포함하며, XE Object와 back-end로부터 전달된 주요 설정을 전달 받아 처리할 수 있음

## Utils
- window.XE.Utils

## Router

## Request

## Lang

## DynamicLoadManager

## Validator
