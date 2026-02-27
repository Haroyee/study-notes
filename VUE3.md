# VUE3

一、创建

在要创建的位置打开cmd命令窗口

1、创建项目

```powershell
npm init vue@latest
```

2、下载模块

```powershell
cnpm install
```

3、启动项目

```powershell
npm dev
```

二、Axios(中文文档：https://www.axios-http.cn/docs/req_config)

1、下载

```powershell
npm install axios
```

2、导入

```javascript
import axios from 'axios'
```

3、查(常用get)

```javascript
const get = ()=>{
axios.get(url:'请求的后台接口地址').then((res)=>{data.list = res.data});
}
```

4、增（常用post）

```javascript
const post = ()=>{
axios.post(url:'请求的后台接口地址',需要传递的数据).then((res)=>{console.log(res.data)});
}
```

5、删（delete）

```javascript
const delete = ()=>{
axios.delete(url:'请求的后台接口地址',需要传递的数据).then((res)=>{console.log(res.data)});
}
```

6、更新(常用put)

```javascript
const put = ()=>{
axios.put(url:'请求的后台接口地址',需要传递的数据).then((res)=>{console.log(res.data)});
}
```

7、异步（以查为例）

```javascript
async const get =()=>{
    let res = await axios.get(url:'请求的后台接口地址');
    data.list = res.data;
}
```

8、request请求

```javascript
async const request =()=>{
    let data = {};
    let res = await axios.request({url:'请求的后台接口地址',
                                  method:'GET',
                                  data:data,
                                  headers:{
                                       isToken: false,
      							     repeatSubmit: false
                                  }
                                  timeout:2000});
    data.list = res.data;
}
```

9、拦截器

```
import axios from 'axios'
const service = axios.create({
  // axios中请求配置有baseURL选项，表示请求URL公共部分
  baseURL: '后端',
  // 超时
  timeout: 10000
})
```

