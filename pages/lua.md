- cjson 转换过程中 空数组 变为 空对象的问题
- https://github.com/openresty/lua-cjson#decode_array_with_array_mt
- ```lua
  -- init cjson lib config
  require("cjson").decode_array_with_array_mt(true)
  require("cjson.safe").decode_array_with_array_mt(true)
  ```
-
- ![Beginning Lua Programming ( PDFDrive ).pdf](../assets/Beginning_Lua_Programming_(_PDFDrive_)_1650425888119_0.pdf) #book
- ![700021 自己动手实现Lua：虚拟机、编译器和标准库.pdf](../assets/700021_自己动手实现Lua：虚拟机、编译器和标准库_1650730285040_0.pdf) #book