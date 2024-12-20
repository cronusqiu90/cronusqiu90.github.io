# FRIDA Hook Java 层代码


### 设备
- PC 
    - 推荐使用 `Linux` 系统, `Windows` 系统也可以
    - `Android Studio` &amp; `Android SDK`
    - `Android Soursce Code` 
    - 运维工具 `htop`, `jnettop` (Linux) 
- 手机
    - 解 `Bootloader` 锁
    - `TWRP` 刷入 Recovery 分区
    - 安装 `Magisk`, 修改并刷入 `boot.img`
    - (Optional) 刷 `kali-nethunter` 系统



### 环境配置
1. 使用 `Visual Studio Code` 和对应的插件

2. 安装 adb 工具

3. 安装 python 和 nodejs

4. frida 模块

客户端（工具）: `frida-tools` 和 `objection`
```shell
pip install frida-tools
pip install objection
```


服务端: `frida-server`
下载地址: `https://github.com/frida/frida/releases`
需要注意的是：
1. frida-server的版本要与 `frida` 版本一致；
2. frida-server的架构需要和测试机的系统以及架构一致；



推送手机:
```shell
[user@localhost ~]# adb push frida-server /data/local/tmp
[user@localhost ~]# adb shell
[shell@xm8 ~]# su
[root@xm8 ~]# cd /data/local/tmp
[root@xm8 ~]# chmod 777 frida-server
# 启动 frida-server, 使用默认端口
[root@xm8 ~]# ./frida-server 

```

检查是否成功:
```shell
[user@localhost ~]# adb devices
List of devices attached
0123456789ABCDEF        device

[user@localhost ~]# frida-ps -Uai
 PID  Name
-----  -----------
 1234 frida-server
 ...  
```

### Frida 基础
`frida` 有两种操作模式:
1. `CLI` 命令行模式：通过 shell 命令直接将 js 脚本注入进程
2. `RPC` 使用 Python 脚本控制将 js 脚本注入进程

首先编写一个简单的 apk 应用（默认具备Android应用的开发基础）
java 代码:
```java
package org.example.demo01;

import androidx.appcompat.app.AppCompatActivity;

import android.os.Bundle;
import android.view.View;
import android.widget.Button;
import android.widget.TextView;

public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        Button btn = findViewById(R.id.button);
        btn.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                TextView tv = findViewById(R.id.tv);
                tv.setText(Talk(1, 2));
            }
        });
    }

    String Talk(int n1, int n2){
        return &#34;Talk:&#34; &#43; String.valueOf(n1 &#43; n2);
    }
   
}
```

#### 调用成员函数
编写一个 js 脚本 hook 按钮点击事件，并打印出参数和返回值:
```javascript

function main() {
    console.log(&#34;start hooking &#34;);
    Java.perform(function() {
        console.log(&#34;Inside java perform function&#34;);
        //定位类
        var MainActivity = Java.use(&#34;org.example.demo01.MainActivity&#34;);
        console.log(&#34;Java.Use.Successfully!&#34;); //定位类成功！

        // 使用 `implementation` 关键词实现实例化对应
        MainActivity.Talk.implementation = function (x, y) {
            console.log(&#34;MainActivity.Talk:&#34;, x, y);            
            // 调用原有的方法
            // return this.Talk(x, y);            
            // 替换参数
            return this.Talk(10, 20);

        }
    });
}

setImmediate(main);

```


对 `Talk` 方法增加一个 `overload` 方法:
```java
// ...

    String Talk(int n1, int n2){
        return &#34;Talk:&#34; &#43; String.valueOf(n1 &#43; n2);
    }

    String Talk(String str){
        return &#34;Talk:&#34; &#43; str.toUpperCase();
    }

// ...

```

`JS` 脚本:
```javascript

function main() {
    console.log(&#34;start hooking &#34;);
    Java.perform(function() {
        console.log(&#34;Inside java perform function&#34;);
        //定位类
        var MainActivity = Java.use(&#34;org.example.demo01.MainActivity&#34;);
        console.log(&#34;Java.Use.Successfully!&#34;); //定位类成功！

        // 使用 `overload` 关键词来指定函数入参
        // 使用 `implementation` 关键词实现实例化对应
        MainActivity.Talk.overload(&#39;int&#39;, &#39;int&#39;).implementation = function (x, y) {
            console.log(&#34;MainActivity.Talk:&#34;, x, y);            
            return this.Talk(10, 20);
        }

        // MainActivity.Talk.overload(&#39;java.lang.String&#39;).implementation = function (str) {
        //     console.log(&#34;MainActivity.Talk:&#34;, str);            
        //     return this.Talk(str);
        // }
    });
}

setImmediate(main);

```


#### 调用静态函数
`MainActivity` 中新增方法:
```java
// ...

    void hello() {
        Log.d(&#34;com.example.demo01&#34;, &#34;hello invoked&#34;);
    }

    public static String staticHello(){
        Log.d(&#34;com.example.demo01&#34;, &#34;staticHello invoked&#34;);
    }

// ...

```

`JS` 脚本:
```javascript

function main() {
    console.log(&#34;start hooking &#34;);
    Java.perform(function() {
        console.log(&#34;Inside java perform function&#34;);
        //定位类
        var MainActivity = Java.use(&#34;org.example.demo01.MainActivity&#34;);
        console.log(&#34;Java.Use.Successfully!&#34;); //定位类成功！

        // 静态函数调用
        MainActivity.staticHello()

        // 实例函数调用
        // `Java.choose` 是在内存中查找对应的类，前提是要保证类已经加载到内存中；
        // 运行脚本前，需要先运行应用；
        Java.choose(&#34;org.example.demo01.MainActivity&#34;, {
            onMatch: function(instance) {
                console.log(&#34;instance:&#34;, instance);
                instance.hello();
            },
            onComplete: function() {
                console.log(&#34;onComplete&#34;);
            }
        });
    }
}

setImmediate(main);

```

### RPC及其自动化
在 `CLI` 模式下完成了对应用的分析，通过编写 `Python` 脚本实现自动化运行注入。

#### 简单调用
```javascript
// hook.js
function CallHello() {
    Java.perform(function() {
        Java.choose(&#34;org.example.demo01.MainActivity&#34;, {
            onMatch: function(instance) {
                instance.hello();
            },
            onComplete: function() {
            }
        });
    }
}

rpc.exports = {
    // 导出名（key）不可以有大写字母或者下划线!!!
    callhello: CallHello,
}

```

```python
# loader.py
import frida
import sys
import time

def on_message(message, data):
    if message[&#39;type&#39;] == &#39;send&#39;:
        print(&#34;[*] {0}&#34;.format(message[&#39;payload&#39;]))
    else:
        print(message)


device = frida.get_usb_device()
process = device.attach(&#34;com.example.demo01&#34;)

with open(&#34;hook.js&#34;) as f:
    jsCode = f.read()
script = process.create_script(jsCode)

script.on(&#39;message&#39;, on_message)
script.load()

while True:
    script.exports.callhello()
    time.sleep(1)
```

####  互联互通
```javascript
//hook.js
Java.perform(function () {
    console.log(&#34;Inside java perform function&#34;);
    // 通过 hook TextView 控件
    Java.use(&#34;android.widget.TextView&#34;).setText.overload(&#39;java.lang.CharSequence&#39;).implementation = function (x) {
        // 将参数发送给 Python 脚本
        var strSend = x.toString();
        send(strSend);

        // 接收 Python 脚本返回的结果
        var strRecv;
        recv(function (received_json_objection) {
            strRecv = received_json_objection.mData;
            console.log(&#34;strRecv &#34;, strRecv)
        }).wait()

        // 更新UI
        var javaStr = Java.use(&#39;java.lang.String&#39;).$new(strRecv);
        var result = this.setText(javaStr)
        console.log(&#34;javaStr result&#34;, javaStr, result)

        return result
    }
})
```

```python
# loader.py
import frida
import sys

def on_message(message, payload):
    print(&#34;message:&#34;, message)
    if message[&#39;type&#39;] == &#39;send&#39;:
        script.post({&#34;mData&#34;: &#34;hello, from python script!&#34;})



device = frida.get_usb_device()
process = device.attach(&#34;com.example.demo01&#34;)
with open(&#34;hook.js&#34;) as f:
    jsCode = f.read()
script = process.create_script(jsCode)
script.on(&#39;message&#39;, on_message)
script.load()

input()
```

### 参数类型、转换



---

> 作者: 迷途小狼崽  
> URL: https://cronusqiu90.github.io/posts/reverse/android/frida/hook_java/  

