# android源码编译步骤

1.导入环境 source build/envsetup.sh
2.lunch 选择版本
3.make 编译

| 编译指令               | 解释                   |
| ------------------ | -------------------- |
| m                  | 在源码树的根目录执行编译         |
| mm                 | 编译当前路径下所有模块，但不包含依赖   |
| mmm [module_path]  | 编译指定路径下所有模块，但不包含依赖   |
| mma                | 编译当前路径下所有模块，且包含依赖    |
| mmma [module_path] | 编译指定路径下所有模块，且包含依赖    |
| make [module_name] | 无参数，则表示编译整个Android代码 |
