# iEDA-iDB中数据解析器Parser原理
### 功能：
- “数据中心层”：使用结构化的文件流来存储数据，通过文件解析器Parsers读取数据，并且抽象成各个点工具所需要的数据结构，具体文件包含verilog、lib、sdf文件等
### tcl脚本执行流程
#### verilog文件解析流程 erilog_init -path ./result/verilog/gcd.v -top gcd
- verilog_init命令注册（与run_iPL脚本类似），在tcl解释器中注册tcl命令，实现与c++交互
```C++
registerTclCmd(CmdInitVerilog, "verilog_init");
```
- 实现CmdInitVerilog命令，在iEDA/src/interface/tcl/tcl_idb/tcl_db_file.h以及tcl_db_file.cpp中
```C++
class CmdInitVerilog : public TclCmd
{
 public:
  explicit CmdInitVerilog(const char* cmd_name);
  ~CmdInitVerilog() override = default;

  unsigned check() override;
  unsigned exec() override;

 private:
  // private function
  // private data
};
```
- 接着实现readVerilog(path_string, top_module);函数
- readVerilog命令在iEDA/src/platform/data_manager/idm.cpp中实现
```C++
bool DataManager::readVerilog(string path, string top_module)
{
  if (_idb_builder == nullptr || _idb_lef_service == nullptr || _layout == nullptr) {
    return false;
  }

  if (!initVerilog(path, top_module)) {
    return false;
  }

  return true;
}
```
- - 这个方法的作用是读取并初始化Verilog文件,步骤包括：
  - 检查必要组件：确保_idb_builder、_idb_lef_service和_layout等必要组件已初始化
  - 调用initVerilog：使用给定的路径和顶层模块名初始化Verilog文件
  - 返回结果：如果初始化成功则返回true，否则返回false
- 然后在iEDA/src/platform/data_manager/idm_init.cpp是实现这个initVerilog函数
```C++
bool DataManager::initVerilog(string verilog_path, string top_module)
{
  _idb_def_service = _idb_builder->buildVerilog(verilog_path, top_module);
  _design = get_idb_design();

  return _idb_def_service == nullptr ? false : true;
}
```
- - 这个方法用于具体实现Verilog文件的初始化：
  - 构建Verilog服务：通过_idb_builder->buildVerilog创建一个处理Verilog文件的服务
  - 获取并设置设计信息：设置设计相关的数据结构
  - 判断服务是否成功创建：如果服务创建失败（即返回nullptr），则返回false；否则返回true
- 然后实现buildVerilog中的（iEDA/src/database/manager/builder/builder.cpp）VerilogRead类
```C++
IdbDefService* IdbBuilder::buildVerilog(string file, std::string top_module_name)
{
  if (_def_service != nullptr) {
    delete _def_service;
    _def_service = nullptr;
  }

  IdbLayout* layout = _lef_service->get_layout();
  _def_service = new IdbDefService(layout);

  if (IdbDefServiceResult::kServiceFailed == _def_service->VerilogFileInit(file.c_str())) {
    std::cout << "Read Verilog file failed..." << endl;
    return nullptr;
  } else {
    std::cout << "Read Verilog file success : " << file << endl;
  }

  std::shared_ptr<VerilogRead> verilog_read = std::make_shared<VerilogRead>(_def_service);
  verilog_read->createDb(file.c_str(), top_module_name);

  return _def_service;
}
```
- - 这是一个关键的方法，它在IdbBuilder类中创建和初始化Verilog处理服务：
  - 重置服务：如果已存在_def_service，则删除并重置
  - 初始化布局：通过_lef_service获取布局信息
  - 创建新的IdbDefService实例：使用布局信息来创建服务
  - 处理Verilog文件：尝试初始化Verilog文件。如果失败，输出错误信息并返回nullptr
  - 创建和管理Verilog读取实例：使用VerilogRead类来进一步处理Verilog文件
- VerilogRead的实现是基于iEDA/src/database/manager/parser/verilog/VerilogReader.hh文件的，使用parser解析
#### def文件解析流程 def_init -path ./result/iTO_fix_fanout_result.def
- def_init命令注册（与run_iPL脚本类似），在tcl解释器中注册tcl命令，实现与c++交互
```C++
registerTclCmd(CmdInitDef, "def_init");
```
- 实现CmdInitDef命令，在iEDA/src/interface/tcl/tcl_idb/tcl_db_file.h以及tcl_db_file.cpp中
```C++
class CmdInitDef : public TclCmd
{
 public:
  explicit CmdInitDef(const char* cmd_name);
  ~CmdInitDef() override = default;

  unsigned check() override;
  unsigned exec() override;

 private:
  // private function
  // private data
};
```
- 接着实现readDef(def_path);函数
- 下面与verilog类似，就不解释了，也许不是用的parser

