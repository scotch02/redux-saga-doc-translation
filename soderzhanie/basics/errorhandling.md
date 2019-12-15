# 2.4 Обработка ошибок

В этом разделен мы увидим как обрабатывать случай отказа из предывдущего примера. Давайте предположим что наша функция API `Api.fetch` возвращает Promise который прероеходит в состояние rejected когда удаленный fetch терпит неудачу по какой-то причине.

Мы хотим обработать эти ошибки внутри нашей Саги отправляя `PRODUCTS_REQUEST_FAILED` action в Store.

Мы можем ловить ошибки внутри саги, используя знакомый `try/catch` синтакс.

```javascript
import Api from './path/to/api'
import { call, put } from 'redux-saga/effects'

// ...

function* fetchProducts() {
  try {
    const products = yield call(Api.fetch, '/products')
    yield put({ type: 'PRODUCTS_RECEIVED', products })
  }
  catch(error) {
    yield put({ type: 'PRODUCTS_REQUEST_FAILED', error })
  }
}
```

Для того, чтобы проверить случай отказа, мы будем использовать метод `throw` Генератора

```javascript
import { call, put } from 'redux-saga/effects'
import Api from '...'

const iterator = fetchProducts()

// expects a call instruction
assert.deepEqual(
  iterator.next().value,
  call(Api.fetch, '/products'),
  "fetchProducts should yield an Effect call(Api.fetch, './products')"
)

// create a fake error
const error = {}

// expects a dispatch instruction
assert.deepEqual(
  iterator.throw(error).value,
  put({ type: 'PRODUCTS_REQUEST_FAILED', error }),
  "fetchProducts should yield an Effect put({ type: 'PRODUCTS_REQUEST_FAILED', error })"
)
```

В этом случае мы передаем методу `throw` ложную ошибку. Это приведет к тому что Генератор прервет текущий поток и запустит блок catch.

Конечно, вас не заставляют обрабатывать шибки вашего API внутри блоков `try`/`catch`. Вы можете также сделать ваш API сервис возвращающим нормальное значение с каким-то флагом ошибки внутри него. Например, вы можете ловить Promise rejections and отображать их на объект с полем ошибки.

```javascript
import Api from './path/to/api'
import { call, put } from 'redux-saga/effects'

function fetchProductsApi() {
  return Api.fetch('/products')
    .then(response => ({ response }))
    .catch(error => ({ error }))
}

function* fetchProducts() {
  const { response, error } = yield call(fetchProductsApi)
  if (response)
    yield put({ type: 'PRODUCTS_RECEIVED', products: response })
  else
    yield put({ type: 'PRODUCTS_REQUEST_FAILED', error })
}
```

