# Fullstack_7h_cloneStudy_react_django

- 7개의 강의로 이루어진 fullstack 클론 코딩 입니다.
- 주소 : https://www.youtube.com/playlist?list=PLillGF-RfqbbRA-CIUxlxkUpbq0IFkX60

- Django and React together

- Option 1. React in its own "frontend" Django app: load a single HTML template and let React manage the frontend (difficulty: medium)

- Option 2. Django REST as a standalone API + React as a standalone SPA (difficulty: hard, it involves some form of token-based authentication)

- 이 강좌는 option 2로 진행

- https://www.django-rest-framework.org/api-guide/serializers/ API Guide 보기
```
pip install django djangorestframework django-rest-knox
```
- python manage.py startapp leads
- installed_apps, rest_framework 앱추가
- 모델 생성
```python
class Lead(models.Model):
    name = models.CharField(max_length=100)
    email = models.EmailField(max_length=100, unique=True)
    message = models.CharField(max_length=500, blank=True)
    created_at = models.DateTimeField(auto_now_add=True)
```

- 마이그레이션생성하고 실행시키기. 
- 데이터베이스에 테이블과 컬럼을 넣기위해

- python manage.py makemigrations 앱이름
- migrations 폴더에서 모델생성 확인하기
- python manage.py migrate
- admin, auth 등 마이그레이션들이 적용됨
- 시리얼라이저 만들기
- DRF doucument에서의 serializers 정의 
- "Serializers allow complex data such as querysets and model instances to be converted to native Python datatypes that can then be easily rendered into JSON, XML or other content types."
- 앱 폴더에 serializers.py 만들고 class 작성
- 앱 폴더에 api.py 만들고 작성
- 뷰셋이란 기본적으로 full crud api 를 만들게 해주는 것이라고 설명가능( create, read, update and delete)


```
class LeadViewSet(viewsets.ModelViewSet):
    queryset = Lead.objects.all()
    # 일단은 전체 허용으로 
	permission_classes = [
        permissions.AllowAny
    ]
	serializer_class = LeadSerializer
```

- 프로젝트 urls 수정
```
urlpatterns = [
    path('', include('leads.url')),
]
```
- 앱에 urls.py 추가 
```python
from rest_framework import routers
from .api import LeadViewSet

router = routers.DefaultRouter()
router.register('api/leads', LeadViewSet, 'leads')

urlpatterns = router.urls
```
---
## Postman 이용하여 테스트하기 
- post, header는 Content-Type application/json 설정, body는 raw로
```
{
    "name":"choi",
    "email":"abc@naver.com",
    "message":"test"
}
```
- status 201 Created, created_at 필드는 자동으로 시간정보가 들어가있음

- 만들고나서 GET으로 http://localhost:8000/api/leads/1 테스트
- DELETE도 http://localhost:8000/api/leads/1    뒤에 슬래쉬 안하면 안없어짐
- http://localhost:8000/api/leads/1/


---
# Part 2 Implementing React
- 앱추가 
- python manage.py startapp frontend 
- mkdir -p ./frontend/src/components
- static, templates 폴더 생성 
- npm init -y
- npm i -D webpack webpack-cli
- npm i -D @babel/core babel-loader @babel/preset-env @babel/preset-react babel-plugin-transform-class-properties
- npm i react react-dom prop-types

## package.json 확인하기

- .barbelrc 생성
```
{
  "presets": ["@babel/preset-env", "@babel/preset-react"],
  "plugins": ["transform-class-properties"]
}
```
- webpack.config.js 생성
```
module.exports = {
  module: {
    rules: [
      {
        test: /\.js$/,
        exclude: /node_modules/,
        use: {
          loader: "babel-loader"
        }
      }
    ]
  }
}
```
- package.json 수정 
```
"scripts": {
-    "test": "echo \"Error: no test specified\" && exit 1"
+ 	
	    "dev": "webpack --mode development --watch ./lead_manager_react_django/frontend/src/index.js --output ./lead_manager_react_django/frontend/static/frontend/main.js",
    "build": "webpack --mode production ./lead_manager_react_django/frontend/src/index.js --output ./lead_manager_react_django/frontend/static/frontend/main.js"
  },
```

- src 폴더에 index.js 만들고 해당 코드 작성
```
import App from './components/App';
```
- components 폴더에 App.js 생성 
```
import React, {Component} from "react";
import ReactDOM from 'react-dom';

class App extends Component{
    render() {
        return <h1>React App</h1>
    }
}

ReactDOM.render(<App />, document.getElementById('app'));
```

- template/frontend 에 index.html 작성 
```
	... 생략
	<body>
	 <div id="app"></div>
    {% load static %}
    <script src="{% static "frontend/main.js" %}"></script>
	</body>
```

- frontend 앱추가한것 setteing.py 에 추가하기 
- frontend/views.py 코드 추가 
```
def index(request):
    return render(request, 'frontend/index.html')
```

- frontend에 urls.py 추가 
```
from django.urls import path
from . import views
urlpatterns = [
    path('', views.index)
]
```
- 프로젝트 urls.py 에 frontend.urls 추가

## npm run dev 
- --output 대신에 --output-path 로 설정
## output 확인하기 frontend/static/frontend에 main.js 생성확인 

## python manage.py runserver  404 에러 나서 package.json 다시 수정함
- 확인해보니 main.js 폴더와 main.js 파일이 생성되는걸 확인 
```
    "dev": "webpack --mode development --watch ./frontend/src/index.js --output-path ./frontend/static/frontend/",
    "build": "webpack --mode production ./frontend/src/index.js --output-path ./frontend/static/frontend/"
```
---
## component 추가 
- layout 폴더 생성 Header.js 생성 class-based component
- es7 react 설치하면 컴포넌트 쉽게 생성할수있음 RCE tap(파이참이라서 안됨 ㅠ)
- navbar를 만들거라함 
- bootstrap docu에 component/Navbar 
- 코드 복사후 class 부분을 컨트롤 D 로 className으로 바꿔줌 
---
- App.js 에 import Header from './layout/Header';
- package.json watch 옵션 넣으면 새로고침으로 바로 확인가능 
---
## leads 폴더 생성 
- Form.js, Dashboard.js, Leads.js class-based component생성
- dashboard.js 는 function으로 생성, Fragment 태그로 되있고 안에 Form, Leads 컴포넌트를 가지고있음
```
import React, {Fragment} from "react";
import Form from './Form';
import Leads from './Leads';

export  default function Dashboard(){
    return(
        <Fragment>
            <Form />
            <Leads />

        </Fragment>
    )
}
```

## 갑자기 Fragment 태그?
- 컴포넌트가 여러 엘리먼트를 return 할 때 fragment를 사용하면 여러 자식을 그룹화 할 수 있다. 
- 이외에도 short syntax가 있다. 

- App.js에 Dashboard컴포넌트 추가하기 
```App.js
return (
            <Fragment>
                <Header />
                <div className="container">
                    <Dashboard />
                </div>
            </Fragment>
            );
```

## 새로 고침 해도 안될 때 캐시 비우기 하면된다.
- 도구 더보기 \> 인터넷 기록 삭제 

--- 
# 3강 Redux & HTTP 
- chrome web store 에서 Redux DevTools 다운
- npm i redux react-redux redux-thunk redux-devtools-extension

## redux ?
- state container 라고함, state를 관리하기위해 필요함
- Store 라는애를 통해 컴포넌트들에게 state를 저장,유지,전달 한다고함
- 스토어 : 무엇이 일어날지를 나타내는 action, 그리고 action에 따라 상태를 수정하는 reducer를 저장하는 객체
- 리듀서 : action을 통해 어떤 행동을 정의하면 어플리케이션 상태가 어떻게 바뀌는지 특정하게 되는 함수 action 타입에 따라 state를 반환
- 리덕스 쉽게설명 : https://medium.com/@ca3rot/%EC%95%84%EB%A7%88-%EC%9D%B4%EA%B2%8C-%EC%A0%9C%EC%9D%BC-%EC%9D%B4%ED%95%B4%ED%95%98%EA%B8%B0-%EC%89%AC%EC%9A%B8%EA%B1%B8%EC%9A%94-react-redux-%ED%94%8C%EB%A1%9C%EC%9A%B0%EC%9D%98-%EC%9D%B4%ED%95%B4-1585e911a0a6
- 리덕스는 언론에 비유할 수 있다. 모든 정보는 리덕스가 가지고있다.
- 어떤 컴포넌트가 구성원들에게 전달하고 싶은 정보가 있으면 소문대신에 리덕스라는 언론사에 제보한다.
- 리덕스는 전체 컴포넌트에 방송을 한다. 
- 전체 컴포넌트에게 하고 싶은 말이 있으면 리덕스를 통해 스토에 제보한다. 
- 방송은 필요하지 않은 사람에게도 소식이 전달되서 리액트와 리덕스를 연결해주는 react-redux를 사용하여 필요한 컴포넌트에만 소식을 전한다.

## store.js 생성
```store.js
import { createStore, applyMiddleware } from "redux";
import { composeWithDevTools } from 'redux-devtools-extension';
import thunk from "redux-thunk";
import rootReducer from './reducers';

const initialState = {};

const middleware = [thunk];

const store = createStore(
    rootReducer,
    initialState,
    composeWithDevTools(applyMiddleware(...middleware))
);

export default store;
```
- reducers 폴더 생성, index.js(combindreducer), leads.js(action 타입에 따라서 state리턴해줌) 추가
- actinos 폴더생성 leads.js(axios사용하여 비동기 http Request), types.js(const string 정의) 

- 리덕스와함께 어떤 컴포넌트와 일할때는 connect가 필요, react redex 패키지에 있음
- mapStateToProps
- leads 컴포넌트 수정 
```
import {connect} from "react-redux";
import PropTypes from 'prop-types';
import {getLeads} from "../../actions/leads";
export class Leads extends Component {
    static propTypes ={
        leads: PropTypes.array.isRequired
    }
    componentDidMount() {
        this.props.getLeads();
    }

    render() {
        return (
            <div>
                <h1>Leads List</h1>
            </div>
        );
    }
}

const mapStateToProps = state => ({
    leads: state.leads.leads
});
export default connect(mapStateToProps)(Leads);
```

---
## reducer 에서 State 확인 장고에서 만든 leads 확인 
## request의해 action이 만들어지고 django가 leads를 받고 leads가 state로 들어온것
