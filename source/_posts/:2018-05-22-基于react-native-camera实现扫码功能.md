---
title: 基于react-native-camera实现扫码功能
date: 2018-03-22 17:59:02
categories: "React Native"
tags: "react-native"
---

![Banner](https://user-gold-cdn.xitu.io/2018/3/6/161fbdd2fb0b3826?imageView2/1/w/1304/h/734/q/85/format/webp/interlace/1)

> 组件库文档：https://github.com/react-native-community/react-native-camera

### 安装
1. `npm install react-native-camera --save`
2. `react-native link react-native-camera`

> 使用最新的稳定版：
> 将你的package.json这样配置：
> `"react-native-camera": "git+https://git@github.com/react-native-community/react-native-camera"`；

<!-- more -->

### 配置（IOS）
如果上一步的link显示成功，则xcode里的配置基本完成，如果失败，可以手动配置；
#### 手动配置
1.你已经完成 `npm install react-native-camera --save`；
2. 在XCode中，在项目导航器中右键单击`Libraries➜Add Files to [your project's name]`;
3. 转到`node_modules➜ react-native-camera并添加RNCamera.xcodeproj`;
4. 展开`RNCamera.xcodeproj➜ Products`文件夹;
5. 在XCode中，在项目导航器中选择您的项目。添加`libRNCamera.a到您的项目Build Phases➜Link Binary With Libraries`;


6. 点击`RNCamera.xcodeproj`项目导航器并转到`Build Settings`选项卡。确保“All”开启（而不是'Basic'）。在该`Search Paths`部分中，查找`Header Search Paths`并确保它包含两者`$(SRCROOT)/../../react-native/React`并将它们`$(SRCROOT)/../../../React`标记为`recursive`。




### 使用

只需`import { RNCamera } from react-native-camera 模块中 取出<RNCamera/>`标签。

引用标签：

``` javaScript
<RNCamera
         ref={ref => {
             this.camera = ref;
         }}
         style={styles.preview}
         type={RNCamera.Constants.Type.back}
         flashMode={RNCamera.Constants.FlashMode.on}
         onBarCodeRead={this.onBarCodeRead}
         >  
</RNCamera>
```
### 属性
#### autoFocus 

值：（RNCamera.Constants.AutoFocus.on默认）或RNCamera.Constants.AutoFocus.off

使用该autoFocus属性指定相机的自动对焦设置。
`RNCamera.Constants.AutoFocus.on将其打开，RNCamera.Constants.AutoFocus.off将其关闭。`

#### flashMode
指定相机的闪光模式
值：`（RNCamera.Constants.FlashMode.off默认）RNCamera.Constants.FlashMode.on`；
#### onBarCodeRead
检测到条形码时，将调用指定的方法；
事件包含data（条形码中的数据）和type（检测到的条形码类型）



### 绘制扫码界面
 代码：
 
 ```javaScript
 render() {
        return (
            <View style={styles.container}>
                <RNCamera
                    ref={ref => {
                        this.camera = ref;
                    }}
                    style={styles.preview}
                    type={RNCamera.Constants.Type.back}
                    flashMode={RNCamera.Constants.FlashMode.on}
                    onBarCodeRead={this.onBarCodeRead}
                >
                    <View style={styles.rectangleContainer}>
                        <View style={styles.rectangle}/>
                        <Animated.View style={[
                            styles.border,
                            {transform: [{translateY: this.state.moveAnim}]}]}/>
                        <Text style={styles.rectangleText}>将二维码放入框内，即可自动扫描</Text>
                    </View>
                    </RNCamera>
            </View>
        );
    }
 ```

在 Camera 组件中绘制一个绿色的正方形 View，并在下方加上文案提示。随后就是绘制绿色方框中滚动的线。使用 RN 中的 Animated 组件来实现动画效果。 首先在 componentDidMount 函数中初始化动画函数。

```javaScript
componentDidMount() {
    this.startAnimation();
}

startAnimation = () => {
    this.state.moveAnim.setValue(0);
    Animated.timing(
        this.state.moveAnim,//初始值
        {
            toValue: -200,
            duration: 1500,
            easing: Easing.linear
        }//结束值
    ).start(() => this.startAnimation());//开始
};
```
并且记得在构造函数中初始化 state：

```javaScript
constructor(props) {
    super(props);
    this.state = {
        moveAnim: new Animated.Value(0)
    };
}
```
通过 startAnimation 函数的递归使得这个动画呈现一个循环的效果。更加详细的动画效果可以参考文档中的说明。 接下来就是当 Camera 扫描出结果后的处理了，这里通过 onBarCodeRead 函数来处理扫描结果：

```javaScript
//  识别二维码
    onBarCodeRead = (result) => {
        const { navigate } = this.props.navigation;
               const {data} = result;
               //路由跳转到webView页面；
            navigate('Sale', {
                url: data
            })
    };
```

完整版示例：

```javaScript
class ScanScreen extends Component {
    constructor(props) {
        super(props);
        this.state = {
            moveAnim: new Animated.Value(0)
        };
        this.title = '扫描二维码';
    }

    componentDidMount() {
        this.startAnimation();
    }

    startAnimation = () => {
        this.state.moveAnim.setValue(0);
        Animated.timing(
            this.state.moveAnim,//初始值
            {
                toValue: -200,
                duration: 1500,
                easing: Easing.linear
            }//结束值
        ).start(() => this.startAnimation());//开始
    };
    //  识别二维码
    onBarCodeRead = (result) => {
        const { navigate } = this.props.navigation;
               const {data} = result;
            navigate('Sale', {
                url: data
            })
    };

    render() {
        return (
            <View style={styles.container}>
                <RNCamera
                    ref={ref => {
                        this.camera = ref;
                    }}
                    style={styles.preview}
                    type={RNCamera.Constants.Type.back}
                    flashMode={RNCamera.Constants.FlashMode.on}
                    onBarCodeRead={this.onBarCodeRead}
                >
                    <View style={styles.rectangleContainer}>
                        <View style={styles.rectangle}/>
                        <Animated.View style={[
                            styles.border,
                            {transform: [{translateY: this.state.moveAnim}]}]}/>
                        <Text style={styles.rectangleText}>将二维码放入框内，即可自动扫描</Text>
                    </View>
                    </RNCamera>
            </View>
        );
    }
}

export default ScanScreen;

const styles = StyleSheet.create({
    container: {
        flex: 1,
        flexDirection: 'row'
    },
    preview: {
        flex: 1,
        justifyContent: 'flex-end',
        alignItems: 'center'
    },
    rectangleContainer: {
        flex: 1,
        alignItems: 'center',
        justifyContent: 'center',
        backgroundColor: 'transparent'
    },
    rectangle: {
        height: 200,
        width: 200,
        borderWidth: 1,
        borderColor: '#00FF00',
        backgroundColor: 'transparent'
    },
    rectangleText: {
        flex: 0,
        color: '#fff',
        marginTop: 10
    },
    border: {
        flex: 0,
        width: 200,
        height: 2,
        backgroundColor: '#00FF00',
    }
});
```






