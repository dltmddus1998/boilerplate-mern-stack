### 1. MongoDB 연결하기

1. 서버와 클라이언트에서 Dependencies 다운받기 

```jsx
$ npm install
```

1. `dev.js` 생성

```jsx
// local에서 작업할 때
module.exports = {
    mongoURI:'mongodb+srv://dltmddus1998:MSeungGuri21!@reactredux.4bos1.mongodb.net/myFirstDatabase?retryWrites=true&w=majority' 
}
```

1. `cluster` 생성 및 연결 
    1. 클러스터 생성 후 데이터베이스 사용자명과 비밀번호를 설정하여 `dev.js` 에 다음과 같이 복붙하면 된다.

```
$ brew install mongosh

$ mongosh "mongodb+srv://reactredux.4bos1.mongodb.net/myFirstDatabase" --username dltmddus1998x
```

### 2. 상품 업로드 페이지 만들기

**Step1. 비어 있는 업로드 페이지 생성**

---

```jsx
import React from 'react'

function UploadProductPage() {
    return (
        <div>
            UploadProductPage
        </div>
    )
}

export default UploadProductPage
```

**Step2. 업로드 페이지 Route 만들기**

---

```jsx
import React, { Suspense } from 'react';
import { Route, Switch } from "react-router-dom";
import Auth from "../hoc/auth";
// pages for this product
import LandingPage from "./views/LandingPage/LandingPage.js";
import LoginPage from "./views/LoginPage/LoginPage.js";
import RegisterPage from "./views/RegisterPage/RegisterPage.js";
import NavBar from "./views/NavBar/NavBar";
import Footer from "./views/Footer/Footer";
import UploadProductPage from './views/UploadProductPage/UploadProductPage.js';

//null   Anyone Can go inside
//true   only logged in user can go inside
//false  logged in user can't go inside

function App() {
  return (
    <Suspense fallback={(<div>Loading...</div>)}>
      <NavBar />
      <div style={{ paddingTop: '69px', minHeight: 'calc(100vh - 80px)' }}>
        <Switch>
          <Route exact path="/" component={Auth(LandingPage, null)} />
          <Route exact path="/login" component={Auth(LoginPage, false)} />
          <Route exact path="/register" component={Auth(RegisterPage, false)} />
          <Route exact path="/product/upload" component={Auth(UploadProductPage, true)} /> 
          {/* 업로드페이지는 로그인한 사람만 들어갈 수 있으므로 true 
							path --> /product/upload */}
        </Switch>
      </div>
      <Footer />
    </Suspense>
  );
}

export default App;
```

**Step3. 업로드 페이지 탭 만들기**

---

```jsx
/* eslint-disable jsx-a11y/anchor-is-valid */
import React from 'react';
import { Menu } from 'antd';
import axios from 'axios';
import { USER_SERVER } from '../../../Config';
import { withRouter } from 'react-router-dom';
import { useSelector } from "react-redux";

function RightMenu(props) {
  const user = useSelector(state => state.user)

  const logoutHandler = () => {
    axios.get(`${USER_SERVER}/logout`).then(response => {
      if (response.status === 200) {
        props.history.push("/login");
      } else {
        alert('Log Out Failed')
      }
    });
  };

  // 로그 아웃되어 있는 상태
  if (user.userData && !user.userData.isAuth) {
    return (
      <Menu mode={props.mode}>
        <Menu.Item key="mail">
          <a href="/login">Signin</a>
        </Menu.Item>
        <Menu.Item key="app">
          <a href="/register">Signup</a>
        </Menu.Item>
      </Menu>
    )
    // 로그인 되어있는 상태
  } else {
    return (
      <Menu mode={props.mode}>
        {/* upload */}
        <Menu.Item key="upload">
          <a href="/product/upload">Upload</a>
        </Menu.Item>
        <Menu.Item key="logout">
          <a onClick={logoutHandler}>Logout</a>
        </Menu.Item>
      </Menu>
    )
  }
}

export default withRouter(RightMenu);
```

**Step4. Drop Zone을 제외한 Form을 만들기**

---

`antd` 를 통해 css form을 가져와준다. (좀 더 보기 좋게 하기 위해서)

```jsx
import React from 'react'
import { Typography, Button, Form, Input } from 'antd';

const { Title } = Typography;
const { TextArea } = Input;

function UploadProductPage() {
    return (
        <div style={{ maxWidth: '700px', margin: '2rem auto' }}>
            <div style={{ textAlign: 'center', marginBottom: '2rem' }}>
                <Title level={2}> 여행 상품 업로드 </Title>
            </div>

            <Form>
                {/* DropZone */}
                <br />
                <br />
                <label>이름</label>
                <Input value="안녕" />
                <br />
                <br />
                <label>설명</label>
                <TextArea />
                <br />
                <br />
                <label>가격</label>
                <Input />
                <br />
                <br />
                <select>
                    <option></option>
                </select>
                <br />
                <br />
                <Button>
                    확인
                </Button>
            </Form>
        </div>
    )
}

export default UploadProductPage
```

**Step5. 모든 Input을 위한 onChange Function 만들기**

---

onChange라는 이벤트를 통해서 input의 value 값이 변하도록 해준다. 

```jsx
import React, { useState } from 'react'
import { Typography, Button, Form, Input } from 'antd';

const { TextArea } = Input;

// select --> options
const Continents = [
    { key: 1, value: 'Africa' },
    { key: 2, value: 'Europe' },
    { key: 3, value: 'Asia' },
    { key: 4, value: 'North America' },
    { key: 5, value: 'South America' },
    { key: 6, value: 'Australia' },
    { key: 7, value: 'Antarctica' }
]

function UploadProductPage() {

    // state 설정
    const [Title, setTitle] = useState("");
    const [Description, setDescription] = useState("");
    const [Price, setPrice] = useState(0);
    const [Continent, setContinent] = useState(1);
    const [Images, setImages] = useState([]);

    // eventhandler
    const titleChangeHandler = (event) => {
        // 타이핑을 할때마다 value를 바꿔준다
        setTitle(event.currentTarget.value);
    }

    const descriptionChangeHandler = (event) => {
        setDescription(event.currentTarget.value);
    }

    const priceChangeHandler = (event) => {
        setPrice(event.currentTarget.value);
    }

    const continentChangeHandler = (event) => {
        setContinent(event.currentTarget.value);
    }

    return (
        <div style={{ maxWidth: '700px', margin: '2rem auto' }}>
            <div style={{ textAlign: 'center', marginBottom: '2rem' }}>
                <h2 level={2}> 여행 상품 업로드 </h2>
            </div>

            <Form>
                {/* DropZone */}
                <br />
                <br />
                <label>이름</label>
                <Input onChange={titleChangeHandler} value={Title} />
                <br />
                <br />
                <label>설명</label>
                <TextArea onChange={descriptionChangeHandler} value={Description} />
                <br />
                <br />
                <label>가격($)</label>
                <Input type="number" onChange={priceChangeHandler} value={Price} />
                <br />
                <br />
                <select onChange={continentChangeHandler} value={Continent}>
                    {/* map을 이용하여 option 표현 */}
                    {Continents.map(item => (
                        <option key={item.key} value={item.key}>
                            {item.value}
                        </option>
                    ))}
                </select>
                <br />
                <br />
                <Button>
                    확인
                </Button>
            </Form>
        </div>
    )
}

export default UploadProductPage
```

### 3. 파일 업로드 컴포넌트 만들기
