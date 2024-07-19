## promise 学习
### 1、为什么需要promise?
````
例子1：通过ajax请求id，再根据id请求用户名，再根据用户名获取email

通过(例子1)我们使用ajax 回调，需要在回调函数中嵌套回调（这种回调被称为回调地狱），而promise 解决了这一问题
````

### 2、Promise 的基本使用
Promise 是一个构造函数，通过new关键字实例化对象
> 语法
````javascript {.line-numbers}
new Promise((resolve , reject)=>{})
````
- Promise 接收一个函数作为参数
- 在参数函数中接收两个参数
  - resolve:
  - reject:
 > promise 实例

promise 实例有两个属性
- state:状态
- result:结果
#### 1) promise的状态
第一种状态：pending(准备，待解决，进行中)
第二种状态：fulfilled(已完成，成功)
第三种状态：rejected(已拒绝，失败)
#### 2) promise状态的改变
同意调用resolve()和reject()改变当前promise对象的状态
>示例1
````javascript {.line-numbers}
const p= new Promise((resolve,reject)=>{
	resolve()
})
console.dir(p); //状态改成fulfilled
````

>示例2
````javascript {.line-numbers}
const p= new Promise((resolve,reject)=>{
	reject()
})
console.dir(p);//状态改成rejected
````

- resolve()：调用函数，使当前promise对象的状态改成fulfilled
- reject()：调用函数，使当前promise对象的状态改成rejected
  - promise 状态的修改是一次性的

#### 3) promise结果
````javascript {.line-numbers}
const p= new Promise((resolve,reject)=>{
                    //通过resolve ，传递参数，改变promise 对象的结果
                    //resolve('成功结果')
                    reject('失败结果')
                })
                console.dir(p);
````

### 3、promise方法
#### 4) then方法
```` javascript{.line-numbers}
const p= new Promise((resolve,reject)=>{
  //通过resolve ，传递参数，改变promise 对象的结果
  resolve('成功结果')
  // reject('失败结果')
  })
  console.dir(p);

  const t= p.then((value)=>{
  console.log("成功调用")  
  },(reason)=>{
  console.log("失败调用")
  })
  console.log("123",t);
  //当使用debug模式，打断点t 返回的promise 状态是pending，后面javascript 异步执行完 .then方法如果没有返回值 会默认返回null 并且状态改为 fulfilled
````

#### 5) catch方法
``` javascript {.line-numbers}
 const p=new Promise((resolve,reject)=>{

      //演示第二种错误情况
      console.log(a)

  })
  //catch中的参数函数什么时候会被执行
  //1.当promise的状态改为rejected时，被执行
  //2.当promise执行体中出现代码错误的时候，catch被执行，捕获错误（thorw new Error（'错误')这种情况catch也会被捕获)
  p.catch((reason)=>{
      console.log('失败',reason)
  })
```
### 4、解决之前说的ajax 回调地狱问题
> 使用promise 链式调用
``` javascript {.line-numbers}
      // 封装ajax请求
function getData(url, data = {}){
	return new Promise((resolve, reject) => {
  	$.ajax({
      // 发送请求类型
    	type: "GET",
      url: url,
      data: data,
      success: function (res) {
      	// 修改Promise状态为成功, 修改Promise的结果res
        resolve(res)
      },
      error:function (res) {
      	// 修改Promise的状态为失败,修改Promise的结果res
        reject(res)
      }
    })
  })
}

// 调用函数
getData("data1.json")
  .then((data) => {
  	// console.log(data)
    const { id } = data
    return getData("data2.json", {id})
  })
  .then((data) => {
  	// console.log(data)
    const { usename } = data
    return getData("data3.json", {usename})
  })
  .then((data) => {
  	console.log(data)
  })
  ```
  这样写的好处有：
  1）Promise 提供了 .catch() 方法，允许你在链的末端捕获并处理所有异步操作中发生的错误，而不需要在每个回调中单独处理错误。
  2) Promise 允许你以链式调用的方式处理异步操作的结果，而不是将回调函数嵌套在其他回调函数中。这使得代码更加清晰、易于理解和维护。
  3) Promise 允许你使用 .then() 和 .catch() 来定义异步操作的成功和失败路径，这使得控制流程更加直观。

  ### 5、使用 async/await 来简化 Promise 的使用
  ``` javascript {.line-numbers}
  // 封装ajax请求
      function getData(url, data = {}) {
        return new Promise((resolve, reject) => {
          $.ajax({
            // 发送请求类型
            type: "GET",
            url: url,
            data: data,
            success: function (res) {
              // 修改Promise状态为成功, 修改Promise的结果res
              resolve(res);
            },
            error: function (res) {
              // 修改Promise的状态为失败,修改Promise的结果res
              reject(res);
            },
          });
        });
      }

      // 使用 async/await 调用函数
      async function fetchDataSequentially() {
        try {
          const data1 = await getData("data1.json");
          const { id } = data1;
          const data2 = await getData("data2.json", { id });
          const { username } = data2; // 这里应该是 username 而不是 usename
          const data3 = await getData("data3.json", { username });
          console.log(data3);
        } catch (error) {
          console.error("An error occurred:", error);
        }
      }

      // 调用 async 函数
      fetchDataSequentially();
```
1. 定义了一个新的异步函数 fetchDataSequentially：使用 async 关键字定义，这样你就可以在函数内部使用 await。
2.  使用 await 等待每个 getData 调用：这会暂停 fetchDataSequentially 函数的执行，直到对应的 Promise 被解决。
3. 避免了嵌套回调和 Promise 链，使代码结构更清晰。