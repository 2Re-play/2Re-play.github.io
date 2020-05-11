---
title: "[React, Redux, Redux-saga] 흔하디 흔한 투두리스트 아니고 로그인 - 1"
comments: true
author: jaehyun
date: 2020-05-11
tags:
  - react
  - redux-saga
  - redux

---

### 개요 
react를 처음으로 입문한다면 흔하디 흔한 투두리스트, 카운터 예제를 가장 많이 보게된다.  
필자도 react를 투두리스트, 카운터와 같은 예제로 실습을 진행했다.  
하지만 실제 프로젝트에 들어가면 서버와 비동기 통신을 해야하고, 상태 관리 등등 많은 개념이 필요해진다.  
좀 더 편한 상태 관리를 위해 redux를 도입하고 비동기 통신을 위해 redux-saga를 이용하게 됐다.  
하지만 모든 서비스에 필요한 흔하디 흔한 로그인, 회원가입 예제가 없어 처음에 감을 잡기 힘들어 조금이나마 도움이 되기 위해 포스팅하게 됐다.  

간단한 설명만 진행하기 위해 프로젝트 구조화, 예제 코드는 간단하게만 작성한다. 

---

### redux action 만들기
최근에는 redux의 action, reducer등을 하나의 파일에서 작성하는 DUCKS 패턴을 많이 사용하지만  
필자는 모두 분리 시켰다.  
redux를 사용한다면 어떠한 변화를 일으키기 위해서는 action이라는 것을 발생시켜줘야한다.  
액션은 하나의 객체로 표현되면 type이외의 값은 개발자 마음대로 설정할 수 있다.
필자는 로그인을 위해 필요한 액션을 SIGNIN_REQUEST, SIGNIN_SUCCESS, SIGNIN_FAIL 3가지로 만들었다.

src/redux/actions 디렉토리에 auth.js을 생성한다.  
액션 생성 함수를 직접 객체를 리턴하는 함수로 작성해도 되지만  
redux-actions 모듈의 createAction 함수를 사용하면 좀 더 간결하고 쉽게 액션을 생성 할 수 있다.  
creatAction으로 액션을 생성할 때 첫번째 인자는 액션 타입, 두번째 인자는 이외의 데이터를 자유롭게 객체로 넣을 수 있다.  

```jsx harmony
import { createAction } from 'redux-actions';

// actions
export const SIGNIN_REQUEST = 'auth/SIGNIN_REQUEST';
export const SIGNIN_SUCCESS = 'auth/SIGNIN_SUCCESS';
export const SIGNIN_FAIL = 'auth/SIGNIN_FAIL';

// create actions
export const signInRequestAction = createAction(SIGNIN_REQUEST, user => (user));
export const signInSuccessAction = createAction(SIGNIN_SUCCESS, token => (token));
export const signInFailAction = createAction(SIGNIN_FAIL);
```

### redux reducer 만들기 
redux action을 dispatch시켜 상태에 어떠한 변화를 주고 싶다면  
redux reducer에서 실질적으로 state값을 변경시키는 작업을 진행한다.
리덕스를 사용할 때 여러개의 리듀서를 만들고 이를 하나의 루트 리듀서(Root Reducer)로 만들 수 있다.

src/redux/reducer/auth.js를 생성한다.    
위에서 3가지로 구분한 액션별로 변경될 state값을 action객체에서 받아 불변성을 유지하면 넣어주면 된다.  
reducer를 구현할 때 switch문으로 많이들 작성하지만 redux-actions 모듈의 handleActions를 사용하게 되면 좀 더 간결한 코드를 작성할 수 있다.  

```jsx harmony
import handleActions from "redux-actions/es/handleActions"
import {
    SIGNIN_FAIL,
    SIGNIN_SUCCESS,
    SIGNIN_REQUEST
} from "../action/auth"

// state initialize
const initialState = {
    loading : false,
    payload : {
        token : ''
    },
}

// reducer
export default handleActions({
    [SIGNIN_REQUEST]: (state, action) => ({
        ...state,
        loading: true
    }),
    [SIGNIN_SUCCESS]: (state, action) => ({
        ...state,
        loading: false,
        payload: {
            token: action.payload.token
        }
    }),
    [SIGNIN_FAIL]: (state, action) => ({
        ...state
    })
}, initialState);
```

### root reducer로 여러개의 reducer 합치기
src/redux/reducer/index.js를 생성한다.  
해당 파일에서는 여러개의 reducer를 root reducer로 합치는 작업을 한다.  

```jsx harmony
import { combineReducers } from "redux"
import auth from "./auth"

export default combineReducers({
    auth
});
```

### saga 구현하기  
redux-saga는 제너레이터 함수를 구현하여 사용한다.  
src/redux/saga/authSaga.js 파일을 생성한다.  

해당 예제 saga 제너레이터 함수에서는 api 호출 함수가 구현되어있다고 가정한다.  
redux-saga모듈의 call 함수를 통해 특정 함수를 비동기로 호출할 수 있다.  
put 함수로 액션을 디스패치할 수 있다.  

```jsx harmony
import { call, put, takeEvery } from "redux-saga/effects"
import { requestSignIn } from "../apis/auth"
import { SIGNIN_REQUEST } from "../redux/action/auth"
import { signInSuccessAction, signInFailAction } from "../redux/action/auth"

function* postSignInSaga(action) {
    try {
        const email = action.payload.id;
        const password = action.payload.pw;
        const result = yield call(requestSignIn, email, password); // call 을 사용하면 특정 함수를 호출하고, 결과물이 반환 될 때까지 기다려줄 수 있다.
        if(result.message === "SUCCESS") {
            yield put(signInSuccessAction({token : result.data.token})); // 성공 액션 디스패치
        } else {
            yield put(signInFailAction({error: "로그인 실패"})); // 실패 액션 디스패치
        }
    } catch (e) {
        yield put(signInFailAction({error: e.message})); // 실패 액션 디스패치
    }
}

// 사가들을 합치기
export function* postsSaga() {
    yield takeEvery(SIGNIN_REQUEST, postSignInSaga);
}
```

### root saga로 여러개의 saga 합치기
src/redux/saga/index.js 파일을 생성하여 여러개의 saga를 root saga로 합쳐준다.  

```jsx harmony
import { all } from "../../node_modules/redux-saga/effects"
import { postsSaga } from "./authSaga"

export default function* rootSaga() {
    yield all([
      postsSaga(),
    ]);
}
```

### 요약 
- redux action을 생성하는 코드  
- 변화된 action을 감지하여 state를 변경해주는 redux reducer 코드
- 서버로부터 로그인 통신을 하기위한 redux-saga 미들웨어 코드
를 작성했다.  
아직까지는 이 모든게 하나로 연결되어 있지 않다!  
왜냐하면 아직 이 모든 로직이 돌 수 있게 해주는 action을 컴포넌트에 붙이지 않았기 때문이다.  
다음 포스팅에서 SIGNIN_REQUEST 액션을 디스패치하는 코드와 간단한 로그인 컴포넌트를 만들 예정이다!  
설명이 부족한 부분이 많아 앞으로 계속해서 수정할 예정이다!  

