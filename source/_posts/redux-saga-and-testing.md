---
title: Redux-Saga - redux-saga-test-plan 測試範例
tags:
  - Redux
  - Redux-Saga
  - redux-saga-test-plan
  - unit-test
  - integration-test
date: 2022-10-01 17:03:00
---


最近在專案中第一次使用 redux-saga ，對一個 saga 寫測試的時候，看到既有的一些 saga 測試使用 [redux-saga-test-plan](https://github.com/jfairbank/redux-saga-test-plan)。在閱讀文件的範例過程中，把 `testSaga` 和 `expectSaga` 兩個測試用法搞糊塗，於是仔細地讀了裡面的說明。兩者的差別如下：

1. `testSaga`: 在乎 saga 的順序、每一個被 yield 的 effect 樣貌對不對。
2. `expectSaga`: 在乎 saga 的結果對不對。


## 利用範例說明

下面的範例是在描述用戶在登入成功時，要接著去把用戶資訊拉回來，更新到 redux store 的 state 裡面。

`sagas/auth/selectors.js`
```javascript
export const getToken = state => state.auth.token;
```

`sagas/auth/actions.js`
``` javascript
export const LOGIN_SUCCESSFULLY = 'LOGIN_SUCCESSFULLY';
```

`sagas/user/actions.js`
``` javascript
export const RESET_USER_INFO = 'RESET_USER_INFO';
export const SET_USER_INFO = 'SET_USER_INFO';
export const LOAD_USER_INFO_ERROR = 'LOAD_USER_INFO_ERROR''
```

`sagas/user/reducers.js`
``` javascript
import { RESET_USER_INFO, SET_USER_INFO, LOAD_USER_INFO_ERROR } from './actions';

const initialState = {
	info: null,
	error: null,
};

export default function (state = initialState, action) {
	switch(action.type) {
		case RESET_USER_INFO:
			return Object.assign({}, state, {
				info: null,
				error: null
			});
		case SET_USER_INFO:
			return Object.assign({}, state, {
				info: action.payload
			});
		case LOAD_USER_INFO_ERROR:
			return Object.assign({}, state, {
				error: action.payload
			});
		default:
			return state;
	}
}
```

`sagas/user/index.js`
``` javascript
import { takeLatest, select } from 'redux-saga/effects';
import * as actions from './actions';
import * as authActions from '../auth/actions';
import { getToken } from '../auth/selectors';

export const fetchUserInfoError = new Error('token is needed');

export const fetchUserInfo = async (token) => {
	if (token == null) throw fetchUserInfoError;

	const result = fetch('https://api-url.com/user', {
		headers: {
			'Authorization': token,
			'Accept': 'application/json'
		}
	}).then(response => response.json());

	return result;
};

export function* loadUserInfo() {
	const token = yield select(state => state.auth.token);

	try {
		yield put({ type: actions.RESET_USER_INFO });
		const userInfo = yield call(fetchUserInfo, token);
		yield put({ type: actions.SET_USER_INFO, payload: userInfo });
	} catch(e) {
		yield put({ type: actions.LOAD_USER_INFO_ERROR, payload: e });
	}
}

export default function* saga() {
	yield takeLatest(authActions.LOGIN_SUCCESSFULLY, loadUserInfo);
}
```

## testSaga 用法
因為需要確保 saga 的執行順序，以及每一個被 yield 的 effect 如同預期，所以用法有幾個原則：
1. 先執行 `testSaga(saga, ...args)`
2. 再按照順序執行 effects ，每一個 effect 執行之前都要先做 `.next()` 到達當前的 effect
3. 最後要執行 `.isDone()` ，看看這個 saga 是否已經執行完畢

下面的程式碼分別針對 `saga()` 和 `loadUserInfo` 兩個 saga 做測試，很單純地只是在測試運作的順序有沒有按照預期那樣而已。如果把 `loadUserInfo` 的測試範圍涵蓋到 error 的情況，就必須要開不同的 `testSaga` 去驗證，可參考官方文件的[範例](https://redux-saga-test-plan.jeremyfairbank.com/unit-testing/)。

`sagas/user.spec.js`

``` javascript
import { testSaga } from 'redux-saga-test-plan';
import mainSaga, { fetchUserInfoError, fetchUserInfo, loadUserInfo } from './user';
import * as actions from './user/actions';
import { getToken } from './auth/selectors';
import * as authActions from './auth/actions';

it('test mainSaga', () => {
	testSaga(mainSaga)
		.next()
		
		.takeLatest(authActions.LOGIN_SUCCESSFULLY, loadUserInfo)

		.next()
		
		.isDone();
});

it('test loadUserInfo', () => {
	const fakeToken = undefined;
	const fakeUserInfo = undefined;

	// 測試成功的步驟
	testSaga(loadUserInfo)
		// 開始 saga 的執行步驟
		.next()

		// 驗證 saga yield 的 effect 是否和 `select` 這個傳入 selector 的 effect 相同
		.select(getToken)

		// 直接走到下一個 yield 的 effect
		.next()

		// 驗證 saga yield 的 effect 是否和 `put` 這個傳入 action 的 effect 相同
		.put({ type: actions.RESET_USER_INFO })

		.next()

		// 驗證 saga yield 的 effect 是否和 `call` 這個傳入 function 和傳入參數的 effect 相同
		.call(fetchUserInfo, fakeToken)

		.next()

		// 驗證 saga yield 的 effect 是否和 `put` 這個傳入 action 的 effect 相同
		.put({ type: actions.SET_USER_INFO, payload: fakeUserInfo })

		// 驗證 saga 是否已經完成
		.isDone();

	// 測試發生錯誤的步驟
	testSaga(loadUserInfo)
		.next()
		.select(getToken)
		.next()
		.put({ type: actions.RESET_USER_INFO })
		.next()
		.call(fetchUserInfo, fakeToken)
		.next()

		// 這邊開始不一樣
		.throw(fetchUserInfoError)
		.put({ type: LOAD_USER_INFO_ERROR, payload: fetchUserInfoError })
		.next()
		.isDone();
});
```

## expectSaga 用法

在使用 `expectSaga` 的時候，因為目的是要確保模擬的輸入參數、設定的情境，產出的結果能夠符合預期。因此，用法上有幾個原則：

1. 會影響結果的 effect 都要測試到
2. 只需要 mock saga 在執行每一個 effect 的結果
3. 用 `.provide()` 可以 mock [固定的 effect 結果](https://redux-saga-test-plan.jeremyfairbank.com/integration-testing/mocking/static-providers.html)，也可以 mock [動態的結果](https://redux-saga-test-plan.jeremyfairbank.com/integration-testing/mocking/dynamic-providers.html)
4. 最後再執行 `.run()` 

下面是官方範例：

``` javascript
import { call, put, take } from 'redux-saga/effects';
import { expectSaga } from 'redux-saga-test-plan';
import * as matchers from 'redux-saga-test-plan/matchers';
import { throwError } from 'redux-saga-test-plan/providers';
import api from 'my-api';

// 要測試的 saga
function* userSaga(api) {
  try {
    const action = yield take('REQUEST_USER');
    const user = yield call(api.fetchUser, action.payload);
    const pet = yield call(api.fetchPet, user.petId);

    yield put({
      type: 'RECEIVE_USER',
      payload: { user, pet },
    });
  } catch (e) {
    yield put({ type: 'FAIL_USER', error: e });
  }
}

it('fetches the user', () => {
	// 假資料
  const fakeUser = { name: 'Jeremy', petId: 20 };
  const fakeDog = { name: 'Tucker' };

  return expectSaga(userSaga, api)
	  // mocked api calls with result
    .provide([
      [call(api.fetchUser, 42), fakeUser],
      [matchers.call.fn(api.fetchPet), fakeDog],
    ])
		// mocked side effect
    .put({ 
      type: 'RECEIVE_USER',
      payload: { user: fakeUser, pet: fakeDog },
    })
		// 觸發 saga 
    .dispatch({ type: 'REQUEST_USER', payload: 42 })
    .run();
});

it('handles errors', () => {
	// 假資料
  const error = new Error('error');

  return expectSaga(userSaga, api)
		// mocked api call with result
    .provide([
      [matchers.call.fn(api.fetchUser), throwError(error)]
    ])
		// mocked side effect
    .put({ type: 'FAIL_USER', error })
		// 觸發 saga 
    .dispatch({ type: 'REQUEST_USER', payload: 42 })
    .run();
});
```

## 結論
1. `testSaga` 目的在於 branching 是否如預期
2. `expectSaga` 目的在於結果是否如預期
