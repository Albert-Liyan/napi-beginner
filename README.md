
#### npm install

node-addon-api@1.7.2
node-gyp@9.3.1

#### Demo1

`npm run build`
```
npm run build
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
`node index.js`
```
addon {}
```