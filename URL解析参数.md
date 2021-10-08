## 描述

获取 url 中的参数  
1. 指定参数名称，返回该参数的值 或者 空字符串  
2. 不指定参数名称，返回全部的参数对象 或者 {}  
3. 如果存在多个同名参数，则返回数组
4. 不支持URLSearchParams方法

### 方法一
```js
/*
 * 方法一: 使用字符串凭借匹配字符 
 * 获取URl中的参数
 * 
 * @para url 
 * @para key 参数名
 * */
function getUrlParam(sUrl, sKey) {
  var left = sUrl.indexOf("?") + 1
  var right = sUrl.lastIndexOf("#")
  var parasString = sUrl.slice(left, right)
  var paras = parasString.split('&');
  var parasjson = {}
  paras.forEach(function (value, index, arr) {
    var a = value.split('=');
    parasjson[a[0]] !== undefined ? parasjson[a[0]] = [].concat(parasjson[a[0]], a[1]) : parasjson[a[0]] = a[1];
  });

  let result = arguments[1] !== void 0 ? (parasjson[arguments[1]] || '') : parasjson;
  return result
}
```

### 方法二
```js
/**
 * 方法二: 使用正则表达式匹配字符, 并使用正则replace方法替换
 * /[\?&]?(\w+)=(\w+)/g 核心正则: 匹配?和&号和后续的=号参数值
 */
function getUrlParam2(sUrl, sKey) {
  const reg = /[\?&]?(\w+)=(\w+)/g // /[?&]([^#&?]+)=([^#&\?=]+)/g;
  let result = {};
  let obj = {};
  sUrl.replace(reg, (group, catch1, catch2, index, str) => {
    // obj[catch1] = catch2;
    obj[catch1] === void 0 ? obj[catch1] = catch2 : obj[catch1] = [].concat(obj[catch1], catch2);
  });
  sKey === void 0 || sKey === '' ? result = obj : result = obj[sKey] || '';
  return result;
}
```

### 方法三
```js
/**
 * 方法三: 使用正则表达式匹配字符, 并使用正则Exec方法进行组装
 * /(\w+)=(\w+)/g 核心正则: 匹配=号两边的字符
 */
function getUrlParam3(sUrl, sKey) {
  var resObj = {};
  var reg = /(\w+)=(\w+)/g;
  while (reg.exec(sUrl)) {
    console.log(RegExp.$1, RegExp.$2)
    resObj[RegExp.$1] ? resObj[RegExp.$1] = [].concat(resObj[RegExp.$1], RegExp.$2) : resObj[RegExp.$1] = RegExp.$2;
  }
  if (sKey) {
    return (resObj[sKey] ? resObj[sKey] : '');
  }
  return resObj;
}
```

[关于replace()方法的回调函数参数](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String/replace#%E6%8C%87%E5%AE%9A%E4%B8%80%E4%B8%AA%E5%87%BD%E6%95%B0%E4%BD%9C%E4%B8%BA%E5%8F%82%E6%95%B0)