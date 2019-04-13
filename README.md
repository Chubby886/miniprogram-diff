# miniprogram-diff

小程序setData diff算法

## 目的
为了解决小程序内因setData执行频繁且数据传输量大而引发的页面渲染阻塞问题。

## 使用
可以对setData做一个新的封装，Promise化使用
```javascript
import diff from './src/diff';

var isEmpty = function isEmpty(params) {
    //console.log(typeof params, params)
    try {
        if (params == null || params == undefined) { return true; }
        //判断数字是否是NaN
        if (typeof params === "number") {
            if (isNaN(params)) { return true; }
            else { return false; }
        }
        //判断参数是否是布尔、函数、日期、正则，是则返回false
        if (typeof params === "boolean" || typeof params === "function" || params instanceof Date || params instanceof RegExp) { return false; }
        //判断参数是否是字符串，去空，如果长度为0则返回true
        if (typeof params === "string") {
            if (params.trim().length == 0) { return true; }
            else { return false; }
        }
        if (typeof params === 'object') {
            //判断参数是否是数组，数组为空则返回true
            if (params instanceof Array) {
                if (params.length == 0) { return true; }
                else { return false; }
            }
            //判断参数是否是对象，判断是否是空对象，是则返回true
            if (params instanceof Object) {
                //判断对象属性个数
                if (ownPropertyNames(params).length == 0) { return true; }
                else { return false; }
            }
        }
    } catch (e) { console.log(e); return false; }
}

// this指向的是Page对象
this.update = (data) => {
    return new Promise((resolve, reject) => {
        if (Object.prototype.toString.call(data) !== OBJECTTYPE) {
            reject('Error data type');
            return;
        }
        const result = diff(data, this.data);
        if ( isEmpty(result) ) {
            if ( isEmpty(data)){ reject(null); return; } 
            this.setData(data, () => { resolve(data); }); return;
        }
        
        this.setData(result, () => {
            resolve(result);
        });
    });
}
```
封装后使用
```javascript
this.update({
    a: 1,
    b: {
        c: 2
    },
    d: [1, 2, 3]
}).then((res) => {
    // 渲染成功回调
    // do something
});
```
