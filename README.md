# 리덕스를 사용해 리액트 애플리케이션 상태 관리

## UI 준비하기

### 프레젠테이셔널 컴포넌트와 컨테이너 컴포넌트 분리 패턴

- 리액트 프로젝트에서 리덕스 사용 시 가장 많이 사용하는 패턴
- 프레젠테이셔널 컴포넌트 : 상태 관리 이뤄지지 않고 props 받아와 화면에 UI 보여주기만 하는 컴포넌트, src/components 경로에 저장
- 컨테이너 컴포넌트 : 리덕스와 연동되어 있는 컴포넌트로, 리덕스로부터 상태 받아오기도 하고 리덕스 스토어에 액션 디스패피 하기도 함, src/containers 컴포넌트에 작성
- 해당 패턴 사용시 코드의 재사용성 높아지고 관심사의 분리가 이뤄져 UI 작성할 때 집중가능

## 리덕스 관련 코드 작성하기

- 액션 타입, 액션 생성 함수, 리듀서 코드 작성 -> 각각 다른 파일에 작성 or 기능별로 묶어 파일 하나에 작성
  - actions, contants, reducers 세 디렉터리 만들고 기능별로 파일 하나씩 : 리덕스 공식 문서에 사용되는 기본적 방식이지만 새로운 액션 만들 때마다 세 종류의 파일 모두 수정해 불편할 수 있음
  - 기능별로 파일 하나에 몰아서 작성 : Ducks 패턴이라고 부름

### 모듈 작성

모듈 : Ducks 패턴 사용해 액션 타입, 액션 생성 함수, 리듀서 작성한 코드

1. 액션 타입 정의
   - 액션 타입은 대문자로 정의, 문자열 내용은 '모듈이름/액션이름' 형태로 작성(액션 이름 충돌되지 않도록)
2. 액션 생성 함수 만들기
   - 앞부분에 export 키워드 들어감 : 추후 해당 함수를 다른 파일에서 불러와 사용 가능
3. 초기 상태 및 리듀서 함수 만들기
   - 리듀서는 export default로 내보내줌
     ; export는 여러개 내보낼 수 있지만 export default는 단 한개만 가능, 불러오는 방식도 다름(export는 {} 안, export default는 그냥 적어줌

### 루트 리듀서 만들기

- createStore 함수 사용해 스토어 만들때는 리듀서 하나만 사용해야 하기때문에 기존에 만든 리듀서를 하나로 합쳐줘야 함
- 리덕스에서 제공하는 combineReducers 유틸 함수 사용

## 리액트 애플리케이션에 리덕스 적용

스토어 만들고 리액트 애플리케이션에 리덕스 적용하는 작업은 src 디렉터리의 index.js에서 이뤄짐

### 스토어 만들기

import { createStore } from 'redux'; 에 취소선 그어짐

- 기능 문제 없이 작동
- 취소선 없애기 위해
  - redux 공식문서에서 권장하는 RTK(Redux Toolkit) 사용
  - 이름만 바꿔서 사용 : import { legacy_createStore as createStore } from 'redux';

### Provider 컴포넌트 사용해 프로젝트에 리덕스 적용

리액트 컴포넌트에서 스토어 사용할 수 있도록 App 컴포넌트를 react-redux에서 제공하는 Provider 컴포넌트로 감싸기

- store를 props로 전달해줘야 함

### Redux DevTools 설치 및 적용

Redux DevTools는 리덕스 개발자 도구로, 크롬 확장 프로그램으로 설치해 사용

- 패키지도 설치해 적용하면 코드 깔끔 ; redux-devtools-extension

## 컨테이너 컴포넌트 만들기

- 컨테이너 컴포넌트 : 리덕스 스토어와 연동된 컴포넌트
- 컴포넌트를 리덕스와 연동하려면 react-redux에서 제공하는 connect 함수 사용해야 함
  : connect(mapStateToProps, mapDispatchToProps)(연동할 컴포넌트)

  - mapStateToProps : 리덕스 스토어 안의 상태를 컴포넌트의 props로 넘겨주기 위해 설정하는 함수
  - mapDispatchToProps : 액션 생성 함수를 컴포넌트의 props로 넘겨주기 위해 사용하는 함수
  - mapStateToProps, mapDispatchToProps 미리 선언 후 사용하는 대신 connect 함수 내부에 익명함수 형태로 선언해도 됨

  ```
  export default connect(
  state => ({
    number: state.counter.number,
  }),
  dispatch => ({
    increase: () => dispatch(increase()),  // {return dispatch(increase())}로 적어도 됨(디스패치 코드 한줄이라 생략 가능)
    decrease: () => dispatch(decrease()),
  }),
  )(CounterContainer);
  ```

  - 액션 생성 함수 많아질 경우 각 함수 호출하고 dispatch로 감싸기 번거로워짐

        1. 리덕스에서 제공하는 bindActionCreators 유틸 함수 사용

        ```
        import { bindActionCreators } from 'redux';
        (...)
        export default connect(
        state => ({
            number: state.counter.number,
        }),
        dispatch =>
            bindActionCreators(
            {
                increase,
                decrease,
            },
            dispatch,
            ),
        )(CounterContainer);
        ```

        2. mapDispatchToProps에 해당하는 파라미터를 함수 형태가 아닌 액션 생성 함수로 이뤄진 객체 형태로 넣어주기 : connect 함수가 내부적으로 bindActionCreators 작업 대신 해줌

        ```
        export default connect(
            state => ({
                number: state.counter.number,
            }),
            {
                increase,
                decrease,
            },
        )(CounterContainer);
        ```

- connect 함수 호출 후 또 다른 함수 반환, 반환된 함수에 컴포넌트를 파라미터로 넣어주면 리덕스와 연동된 컴포넌트 만들어짐 :

  ```
  const makeContainer = connect(mapStateToProps,mapDispatchToProps)
  makeContainer(타깃 컴포넌트)
  ```

## 리덕스 더 편리하게 사용

액션 생성 함수, 리듀서 작성 시 redux-actions 라이브러리와 immer 라이브러리 활용하면 훨씬 편하게 사용 가능

1. redux-actions
   - 리듀서 작성시 switch/case 문이 아닌 handleActions 함수 사용해 각 액션마다 업데이트 함수 설정하는 형식으로 작성가능
     - handleActions 함수 첫번째 파라미터에는 각 액션에 대한 업데이트 함수 넣어주고, 두번째 파라미터에는 초기 상태 넣어줌
   - 액션 생성 함수에서 파라미터 필요로 할 때, createAction으로 액션 만들면 액션에 필요한 추가 데이터는 payload 이름 사용
     - 모든 추가 데이터값을 action.payload로 사용하기 때문에 나중에 리듀서 코드 다시 볼때 헷갈릴 수 있음 => 객체 비구조화 할당 문법으로 action 값의 payload 이름 새로 설정
2. immer
   - 객체 구조 복잡해지거나 객체로 이뤄진 배열 다룰 경우 immer 사용하면 휠씬 편리하게 상태 관리 가능
   - 간단한 리듀서에 immer 사용하면 오히려 코드 더 길어짐

## Hooks 사용해 컨테이너 컴포넌트 만들기

리덕스 스토어와 연동된 컨테이너 컴포넌트 만들 때 connect 함수 사용하는 대신 react-redux에서 제공하는 Hooks 사용할 수도 있음

1. useSelector로 리덕스 상태 조회
   - 사용법 : const 결과 = useSelector(상태 선택 함수)
   - 상태 선택 함수는 mapStateToProps와 형태 똑같음
2. useDispatch 사용해 액션 디스패치

   - 컴포넌트 내부에서 스토어의 내장 함수 dispatch 사용할 수 있게 해줌
   - 사용법 :

   ```
   const dispatch = useDispatch();
   dispatch({type: 'SAMPLE_ACTION' });
   ```

3. useStore 사용해 리덕스 스토어 사용

   - useStore는 컴포넌트에서 스토어에 직접 접근해야 하는 상황에만 사용해야 함
   - 사용법 :

   ```
   const store = useStore();
   store.dispatch({type: 'SAMPLE_ACTION' });
   store.getState();
   ```

4. useActions 유틸 Hook 만들어 사용
   - react-redux에 내장된 상태로 릴리즈 될 계획이었으나 제외된 Hook으로, 그 대신 공식 문서에서 그대로 복사해 사용할 수 있도록 제공
   - 참고링크 : <https://react-redux.js.org/next/api/hooks#recipe-useactions>
   - useActions는 두가지 파라미터 필요 : 첫번째 파라미터는 액션 생성 함수로 이뤄진 배열, 두번째 파라미터는 deps 배열로, 이 배열 안에 들어있는 원소가 바뀌면 액션을 디스패치하는 함수를 새로 만들게 됨

### connect 함수와의 주요 차이점

- connect 함수 사용해 컨테이너 컴포넌트 만들었을 경우, 해당 컨테이너 컴포넌트의 부모 컴포넌트가 리렌더링 될 때 해당 컴포넌트의 props가 바뀌지 않았다면 리렌더링이 자동으로 방지되어 성능 최적화 됨
- useSelector 사용해 리덕스 상태 조회했을 때는 최적화 작업이 자동으로 이뤄지지 않으므로, 성능 최적화 위해 React.memo를 컨테이너 컴포넌트에 사용해줘야 함 ex) export default React.memo(컴포넌트이름)
