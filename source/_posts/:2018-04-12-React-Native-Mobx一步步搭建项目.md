---
title: React Native + Mobx一步步搭建项目
date: 2018-04-12 10:06:40
tags: "React Native"
categories: "React Native"
---

![banner](https://raw.githubusercontent.com/Wuguanghua/blogImg/master/0528-02.png)

   本文将简单介绍React Native 结合 Mobx 构建项目的过程，我也是初探Mobx，文中如有错误还请评论指出，内容很简单，大概10分钟左右看完。
 至于为什么选择Mobx而不用Redux？其实Redux之前也在项目中用过，用了一段时间感觉虽然他在大型项目中做了很多工作，但使用起来代码量很多，
 尤其是redux-saga中间件的使用步骤确实很繁琐，所以想尝试一种新的数据管理方式，当我碰到Mobx时，我觉得很有意思，便开始了小的尝试，做了一个小demo，于是就有了这篇文章。
 关于Redux和Mobx的对比，参考这篇文章：[我为什么从Redux迁移到了Mobx][2]
 
  <!-- more -->
 开始搭建：
 
 前提是你已经搭好环境，并且能成功跑起来;
 否则的话先进行：[Mac环境搭建,目标：ios][1] 


 本项目使用的依赖：
``` javascript
  "dependencies": {
    "lodash": "^4.17.5",
    "mobx": "^4.1.0",
    "mobx-react": "^5.0.0",
    "native-base": "^2.3.5",
    "react": "^16.3.0-alpha.1",
    "react-native": "0.54.4",
    "react-navigation": "^1.5.8"
  },
```
 
### 准备工作 ###

#### 1. 安装相应依赖：mobx 和 mobx-react;

``` bash
npm i mobx mobx-react --save
```

#### 2. 安装一些 babel 插件，以支持 ES7 的 decorator 特性:
``` bash
npm i babel-plugin-transform-decorators-legacy babel-preset-react-native-stage-0 --save-dev

```
#### 3. 打开 .babelrc 没有就创建一个，配置 babel 插件:
``` bash
touch .babelrc
```
写入：
``` javascript
{
 'presets': ['react-native'],
 'plugins': ['transform-decorators-legacy']
}
```
** 补充：ES7装饰器语法在编译器可能会报错；我这里用的是vscode，分享解决办法：**
 1. 找到 首选项 > 设置 > 工作区设置;
 2. 加入以下代码：
``` javascript
    "javascript.implicitProjectConfig.experimentalDecorators": true
```


至此准备工作已经差不多了，接下来写代码

----

### 加入Mobx ###
#### 1. 根目录下新建 js文件夹； js下新建`store`文件夹；
#### 2. `store`下新建`index.js` 作用是合并每个`store`到一个`store`中去，最终通过 `<Provider {...store}>`方式注入`<App/>`;

#### 3.分别在`store`下新建几个js，示例：
``` javascript
// home
import { observable, action } from "mobx";

class HomeStore {
  @observable text; // 注册变量，使其成为可检测的
  @observable num;

  constructor() {
    this.num = 0; // 初始化变量，可以定义默认值
    this.text = "Hello, this is homePage!!!";
  }

  @action  // 方法推荐用箭头函数的形式
  plus = () => {
    this.num += 1;
  };

  @action
  minus = () => {
    this.num -= 1;
  };
}

const homeStore = new HomeStore(); 

export { homeStore };
```

``` javascript
// user
import { observable, action } from "mobx";

class UserStore {
  @observable userInfo;
  @observable text;

  constructor() {
    this.userInfo = "123";
    this.text = "Hello, this is UserPage!!!";
  }

  @action
  getListData = () => {
    fetch(`http://yapi.demo.qunar.com/mock/5228/record/list`)
      .then(
        action("fetchRes", res => {
          return res.json();
        })
      )
      .then(
        action("fetchSuccess", data => {
          return (this.userInfo = data);
        })
      )
      .catch(
        action("fetchError", e => {
          console.log(e.message);
        })
      );
  };
}

const userStore = new UserStore();

export { userStore };

```
#### 4. 通过index文件合并：
``` javascript
import { homeStore } from "./home";
import { saleStore } from "./sale";
import { userStore } from "./user";

const store = { homeStore, saleStore, userStore };

export default store;
```
#### 5. 在组件中使用Mobx：

js目录下新建tabs文件夹，新建HomeScreen.js
``` javascript
import React, { Component } from "react";
import { View, Text, StyleSheet, TouchableOpacity } from "react-native";
import { connect } from "mobx-react";
import { observer, inject } from "mobx-react";
import { Button, Container } from "native-base";
import Headers from "../common/components/header";

@inject(["homeStore"]) // 注入对应的store
@observer // 监听当前组件
class HomeScreen extends Component {
  constructor(props) {
    super(props);
    this.store = this.props.homeStore; //通过props来导入访问已注入的store
    this.state = {
    };
  }

  render() {
    const { text, num } = this.store;
    return (
      <Container>
        <Headers 
        title="首页" 
        type='index' 
        navigation={this.props.navigation}
        />
        <Text>{text}</Text>
        <Button primary onPress={() => this.store.plus()}>
          <Text>add</Text>
        </Button>
        <Text>{num}</Text>
        <Button primary onPress={() => this.store.minus()}>
          <Text>Minu</Text>
        </Button>
      </Container>
    );
  }
}

export default HomeScreen;

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: "center",
    alignItems: "center",
    backgroundColor: "#F5FCFF"
  }
});

```
#### 6. 目前组件中还拿不到当前组件的`store`，接下来注入`store`；
在初始化的项目结构中，项目运行的入口文件是`index.js`,注册了同级目录下的`App.js`;
``` javascript
// index.js

import { AppRegistry } from 'react-native';
import App from './App';

AppRegistry.registerComponent('demo', () => App);
```
由于需要在`App.js`上套入`store` 所以改写结构，
##### 1. js文件夹下新建`App.js`

`touch App.js`

可以把根目录下`App.js`的内容copy过来，然后删掉根目录下`App.js`文件;
##### 2. js目录下新建 `setup.js` 文件； 
``` javascript
import React from "react";
import { Provider } from "mobx-react";
import App from "./App";
import store from "./store";

export default function setup() {
  class Root extends React.Component {
    render() {
      return (
        <Provider {...store}>
          <App />
        </Provider>
      );
    }
  }
  return Root;
}
```
##### 3. 修改index启动页代码：
``` javascript
import { AppRegistry } from 'react-native';
import setup from "./js/setup";

AppRegistry.registerComponent('******', () => setup());
```
现在`store`已经注入，在每个组件中也可以拿到当前`store`的数据了；可以进行代码开发了；


  [1]: https://segmentfault.com/a/1190000013947391
  [2]: https://tech.youzan.com/mobx_vs_redux/