# 2.1. Использование хелперов Saga

`redux-saga` предоставляет вспомогательные эффекты, которые обарачивают внутренние функции для запуска задач, когда определенные action-ы отправляются в Store.

Вспомогательные функции построены поверх API нижнего уровня. В расширенном/продвинутом разделе мы увидим, как эти функции могут быть реализованы. 

Первая функция `takeEvery` является наиболее знакомой и обеспечивает поведение похожее на `redux-thunk`.

Давайте рассмотрим пример с AJAX. При каждом нажатии на кнопку Fetch, мы диспатчим action `FETCH_REQUESTED` и хотим обработать его запуском таска, который будет извлекать данные с сервера.

Сначала мы создаем задачу, которая будет выполнять асинхронное действие: 

```javascript
import { call, put } from 'redux-saga/effects'

export function* fetchData(action) {
   try {
      const data = yield call(Api.fetchUser, action.payload.url)
      yield put({type: "FETCH_SUCCEEDED", data})
   } catch (error) {
      yield put({type: "FETCH_FAILED", error})
   }
}
```

Чтобы таск запускался для каждого action `FETCH_REQUESTED`:

```javascript
import { takeEvery } from 'redux-saga/effects'

function* watchFetchData() {
  yield takeEvery('FETCH_REQUESTED', fetchData)
}
```

В приведенном выше примере, `takeEvery` позволяет одновременно запускать несколько экземпляров `fetchData` одновременно. В данный момент мы можем запустить новый таск `fetchData` пока есть все ещё запущеные таски `fetchData`  которые еще не завершены.

Если мы хотим получить ответ только от последнего запроса \(например, чтобы всегда показывать последнюю версию данных\) мы можем использовать `takeLatest` helper:

```javascript
import { takeLatest } from 'redux-saga/effects'

function* watchFetchData() {
  yield takeLatest('FETCH_REQUESTED', fetchData)
}
```

В отличие от `takeEvery`, `takeLatest` позволяет запустить только один таск `fetchData` в любой момент. И это будет последний запущенный таск. Если предыдущая задача все еще запущена когда другой таск `fetchData` запущен, предыдущая задача будет автоматически отменена.

Если у вас есть несколько Sagas которые наблюдают за различными action, вы можете создать несколько наблюдателей с помощью этих встроенных хелперов, которые будут вести себя так, будто их запустили через `fork` \(мы поговорим о `fork` позже. На данный момент будем просто счиать что этот Effect позволяет нам запускать несколько саг в фоновом режиме\).

Пример:

```javascript
import { takeEvery } from 'redux-saga/effects'

// FETCH_USERS
function* fetchUsers(action) { ... }

// CREATE_USER
function* createUser(action) { ... }

// use them in parallel
export default function* rootSaga() {
  yield takeEvery('FETCH_USERS', fetchUsers)
  yield takeEvery('CREATE_USER', createUser)
}
```

