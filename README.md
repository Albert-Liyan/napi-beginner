
##### 代码工程
`node-gyp` 编译addons的工具链
`node-addon-api` addons开发的helper project
`windows-build-tools` windows环境需要安装的编译工具，否则npm run执行报错

node-addon-api@1.7.2  
node-gyp@9.3.1 

#### Demo1：最简单native cpp
napi demo工程
`binding.gyp` file contains all the files that need to be compiled and all the include files / libraries that the project will be using

执行`npm run build`
```
在此解决方案中一次生成一个项目。若要启用并行生成，请添加“-m”开关。
  nothing.vcxproj -> D:\Code\node_js\napi_project\test-addon\build\Release\\nothing.lib
  main.cpp
  win_delay_load_hook.cc
  正在生成代码
  Previous IPDB not found, fall back to full compilation.
  All 14 functions were compiled because no usable IPDB/IOBJ from previous compilation was found.
  已完成代码的生成
  testaddon.vcxproj -> D:\Code\node_js\napi_project\test-addon\build\Release\\testaddon.node
gyp info ok
```
执行`node index.js`
```
addon {}
```

#### Demo2:Exporting Hello World C++ function
`HelloWrapped` 是NAPI封装函数，所有的export C++ function流程都是创建NAPI封装函数、添加`exports`对象。
所有的封装函数都需要设置Napi namespace的input params、returnValue，这里`HelloWrapped` 中`returnValue`是output，没有input参数。
所有的封装函数都必须传入`CallbackInfo`参数，包含上下文信息、input params。
```
std::string functionexample::hello(){
  return "Hello World";
}
Napi::String functionexample::HelloWrapped(const Napi::CallbackInfo& info) 
{
  Napi::Env env = info.Env();
  Napi::String returnValue = Napi::String::New(env, functionexample::hello());
  
  return returnValue;
}
Napi::Object functionexample::Init(Napi::Env env, Napi::Object exports) 
{
  exports.Set(
"hello", Napi::Function::New(env, functionexample::HelloWrapped)
  );
 
  return exports;
}
```
#### Demo3:Exporting function with input params
```
// C++ function
int add(int a, int b){
  return a + b;
}

// NAPI封装函数
Napi::Number functionexample::AddWrapped(const Napi::CallbackInfo& info) {
    Napi::Env env = info.Env();
    if (info.Length() < 2 || !info[0].IsNumber() || !info[1].IsNumber()) {
        Napi::TypeError::New(env, "Number expected").ThrowAsJavaScriptException();
    } 

    Napi::Number first = info[0].As<Napi::Number>();
    Napi::Number second = info[1].As<Napi::Number>();

    int returnValue = functionexample::add(first.Int32Value(), second.Int32Value());
    
    return Napi::Number::New(env, returnValue);
}

// export C++ function添加exports对象
exports.Set("add", Napi::Function::New(env, functionexample::AddWrapped));

// test
console.log('hello ', testAddon.add(5, 10));
```

#### Demo4:Exporting a Hello World C++ Class

`func`定义export出去给JS的类，`func`赋值给constructor（C++静态函数引用`static Napi::FunctionReference constructor;`）；

```
Napi::Function func = DefineClass(env, "ClassExample", 
{
  InstanceMethod("add", &ClassExample::Add),
  InstanceMethod("getValue", &ClassExample::GetValue),
});

constructor = Napi::Persistent(func);
constructor.SuppressDestruct(); 
```
自测test
```
const classInstance = new testAddon.ClassExample(4.3);
console.log('Testing class initial value : ',classInstance.getValue());
console.log('After adding 3.3 : ',classInstance.add(3.3));

执行结果：
Testing class initial value :  4.3
index.js:8
After adding 3.3 :  7.6
```