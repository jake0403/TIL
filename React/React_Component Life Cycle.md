### Component Life Cycle

* component, react class component는 단순히 render 말고 더 많은걸 가지고 있다.
* life cycle method는 기본적으로 react가 component를 생성하는 그리고 없애는 방법이다.
* component가 생성 될 때, render 전에 호출되는 몇 가지 function이 있다.
* component가 render 된 후, 호출되는 다른 function들이 존재한다.
* mounting, updating, unmounting



#### mounting

- mounting 이란  태어나는 것
- constructor()   => javascript에서 class를 만들 때 호출되는 것이다.
  - component가 mount될 때, component가 screen에 표시 될 때, component가 Website에 갈 때, constructor를 호출한다.
- 그 후에 Render가 이루어짐.
- 그 뒤에는 componentDidMount()가 이루어진다.



#### updating

개발자로 인해 일어나는 변화를 update라고 한다. (setState를 할 때 마다)



axios

``` react
import React from "react";
import axios from "axios";

class App extends React.Component{
    state = {
        isLoading:true,
        movies: []
    };
    getMovies = async () => {
      const movies = await axios.get("json url : e.g. https://yts-proxy.now.sh/list_movie");
    componentDidMount(){
        this.getMovies();
      }
    render() {
        const{isLoading} = this.state;
        return<div>{isLoading? "Loading...." : "we are ready"}</div>
    }
  }

}
```

