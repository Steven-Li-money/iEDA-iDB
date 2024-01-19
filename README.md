# iEDA-iDB中数据解析器Parser原理
### 功能：
- “数据中心层”：使用结构化的文件流来存储数据，通过文件解析器Parsers读取数据，并且抽象成各个点工具所需要的数据结构，具体文件包含verilog、lib、sdf文件等
### tcl脚本执行流程
#### verilog文件解析流程 erilog_init -path ./result/verilog/gcd.v -top gcd
- verilog_init命令注册（与run_iPL脚本类似），在tcl解释器中注册tcl命令，实现与c++交互
···C++
registerTclCmd(CmdInitVerilog, "verilog_init");
```
- 

