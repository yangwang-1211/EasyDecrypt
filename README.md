# EasyDecrypt

    在现代应用中（App、小程序、Web），大量接口具备以下特征：

      ● 请求包/响应包数据加密（AES/SM4/RSA等）

      ● 服务端校验签名、随机数、时间戳（Sign/Nonce/Timestamp等）

    导致安全测试人员在使用 Burp/Yakit 等抓包工具时面临几个常见问题：

      ● Burp/Yakit 只能看到加密后的密文

      ● 修改参数后签名校验失败

      ● 重放数据包每次都需要修改报文的 Nonce 等防重放字段 

### ✨ 工具介绍

---

1. 该工具基于 Python + Mitmproxy 构建一个上下游代理架构，主要用于解决安全测试中流量加密不可见、验签无法绕过的难题。工具亮点：

    ● 无需编写代码来处理加解密/签名，工具内置“加解密/签名/字符串处理/编码转化”等规则

    ● 配置好规则即可对请求/响应报文按规则处理，对用户完全透明

2. 上游代理：

    ● 负责拦截来自浏览器/小程序的请求报文，对业务层加密报文进行自动解密和结构还原，使 Burp 等抓包工具能够直接查看和修改明文数据
   
    ● 接收 Burp 修改后的明文响应，自动完成重新加密与签名生成，确保数据能够以符合业务协议的形式正确发往浏览器/小程序

3. 下游代理：

    ● 负责接收 Burp 修改后的明文请求，自动完成重新加密与签名生成，确保数据能够以符合业务协议的形式正确发送到目标服务器
   
    ● 介绍来自服务器的响应报文，对加密的响应报文进行自动解密和结构还原，使 Burp 等抓包工具能够直接查看和修改明文数据

     <img src="https://github.com/yangwang-1211/EasyDecrypt/blob/78921ebf49ce02128496fed1cc59c85c6550f9b4/img/1.png" width="600">

### 🚀 界面展示

---

1. 启动上/下游代理规则界面主要用于管理整个代理链路的规则加载与监听端口设置，核心由三部分组成：

    ● 上/下游代理规则：用于展示当前可用的规则集，并支持按需选择和加载不同的规则

    ● 监听端口：用于设置代理链路的关键通信端口，其中包括

      - 上游监听端口（用于接收浏览器/小程序流量，默认为 8081）
  
      - Burp监听端口（作为中间抓包与调试节点，默认为 8080）
  
      - 下游监听端口（用于转发至目标服务器的出口端口，默认为 8082）
  
    ● 日志窗体：记录执行规则过程中报错接口、报错信息等内容。方便排查问题

     <img src="https://github.com/yangwang-1211/EasyDecrypt/blob/78921ebf49ce02128496fed1cc59c85c6550f9b4/img/2.png" width="600">

2. 上/下游代理规则配置页面， 其核心目标是将请求/响应的加密、解密、签名逻辑抽象为可配置的规则集合，并在代理链路中动态生效。该界面具体组件如下

    ● 规则基础内容：

      - 规则名称：唯一主键，保存后无法修改
  
      - 匹配域名：用于匹配请求/响应包的域名
  
      - 匹配路径： 支持多路径精确匹配（暂不支持正则匹配）
  
      - 请求/响应：分为request和response两种类型

     <img src="https://github.com/yangwang-1211/EasyDecrypt/blob/78921ebf49ce02128496fed1cc59c85c6550f9b4/img/3.png" width="600" />

    ● 规则集合管理：记录上/下游保存过的所有规则（上/下游规则通过菜单栏的 Upstream 和 Downstream 区分）

      - 🟠图标：点击后展示指定规则的“基础内容”和对请求报文的“操作函数集合”
  
      - 🟢图标：点击后展示指定规则的“基础内容”和对响应报文的“操作函数集合”

     <img src="https://github.com/yangwang-1211/EasyDecrypt/blob/78921ebf49ce02128496fed1cc59c85c6550f9b4/img/4.png" width="600" />

    ● 操作函数集合：用于定义和展示某条规则内部的处理逻辑链路

     <img src="https://github.com/yangwang-1211/EasyDecrypt/blob/78921ebf49ce02128496fed1cc59c85c6550f9b4/img/5.png" width="600" />

    ● 操作函数配置窗体：点击新增按钮弹出操作函数配置窗口，通过“函数选择 + 输入输出变量选择”的方式定义请求/响应报文的处理逻辑

      - 函数类别/Type1： 用于定义操作函数的一级分类，例如编码/解码、加密/解密、哈希算法、时间戳生成等，用于确定操作函数的功能域
  
      - 具体函数实现/Type2：在 Type1 分类基础上进一步细化具体算法或实现方式，具体可用函数如下图6

     <img src="https://github.com/yangwang-1211/EasyDecrypt/blob/73b1f2e62cd0b109148c85958561b4db2a5bb40c/img/6.png" width="600" />

      - 输出变量定义/Output： 用于定义当前操作函数的结果变量名，执行结果将绑定至该变量，可在后续函数节点中作为输入被继续调用
  
      - 输入参数绑定/Input： 根据 Type1 与 Type2 的选择动态生成对应的输入参数结构， 用于从请求/响应报文中提取变量或引用上游节点的输出变量，实现跨节点数据流传递

     <img src="https://github.com/yangwang-1211/EasyDecrypt/blob/78921ebf49ce02128496fed1cc59c85c6550f9b4/img/7.png" width="600" />

    ● 规则执行与验证：

      - 验证规则：点击下方验证按钮。自上而下执行操作函数，并将执行结果输出到执行日志窗口，同时会将当前配置的“规则基础内容 + 操作函数集合 + 请求/响应报文”保存
  
      - 左边文本框中保存为待处理的请求/响应报文，可通过在 Burp 等抓包工具中直接复制
  
      - 右边文本框中保存为执行规则后的请求/响应报文

     <img src="https://github.com/yangwang-1211/EasyDecrypt/blob/78921ebf49ce02128496fed1cc59c85c6550f9b4/img/8.png" width="600" />

### 📖 工具使用

---

1. Upstream/Downstream 窗体
   
    ● 输入规则名称、匹配域名、匹配路径、请求/响应
  
    ● 根据请求/响应的类型，在左边文本框中复制 Burp 的请求/响应报文
  
    ● 添加操作函数，对请求/响应报文进行逻辑处理（建议每添加一个操作函数做一次验证，这样既能实时保存进度，也能查看当前步骤是否存在问题）
  
    ● 验证规则的可用性（至此规则保存后可重复使用，无需再次配置）
  
2. Start 窗体

    ● 选择保存的上/下游规则
  
    ● 输入上游、下游、Burp监听端口，启动上/下游代理
  
3. 本机代理配置

    ● 配置浏览器/小程序的下一跳代理到“上游监听端口”

    ● 配置 Burp 的下一跳代理到“下游监听端口”

     <img src="https://github.com/yangwang-1211/EasyDecrypt/blob/73b1f2e62cd0b109148c85958561b4db2a5bb40c/img/9.png" width="600" />

     <img src="https://github.com/yangwang-1211/EasyDecrypt/blob/73b1f2e62cd0b109148c85958561b4db2a5bb40c/img/10.png" width="600" />

4. Mitmproxy 证书安装（火狐浏览器需要在浏览器中安装证书，具体安装方式可以自行百度）

    ● windows 主机选择 Cert 文件夹下的 .cer 证书

    ● 双击证书-安装证书-受信任的根证书颁发机构

     <img src="https://github.com/yangwang-1211/EasyDecrypt/blob/73b1f2e62cd0b109148c85958561b4db2a5bb40c/img/11.png" width="600" />

     <img src="https://github.com/yangwang-1211/EasyDecrypt/blob/73b1f2e62cd0b109148c85958561b4db2a5bb40c/img/12.png" width="600" />

5. 正常访问指定规则的网站，在 Burp 中查看请求/响应报文是否按照规则修改

### 🛠️ 注意

---

1. 项目保存位置禁止存在中文,否则会启动失败
2. 可能会在 Header 头添加 X-Mitm 字段，不要随意更改。最终该字段会被删除
3. 运行工具可能报毒（KDF.cpython-38.pyc），具体原因未知，可将该工具文件夹放到杀毒软件白名单（项目完全开源未打包为exe，可自行进行病毒查杀和代码分析，保证绿色无毒）
4. 借助 <a href="https://github.com/SwagXz/encrypt-labs">Encrypt-Labs</a> 靶场进行工具实战

    ● <a href="https://github.com/yangwang-1211/EasyDecrypt/wiki/Repeate">防重放</a>
    
    ● <a href="https://github.com/yangwang-1211/EasyDecrypt/wiki/AES">AES加解密</a>

### ⭐ 结语

---

    若工具使用存在任何问题可提Issue,也可添加下方QQ私信(备注Github即可)
    
    制作不易，项目完全开源，若有其他需求后续也会持续更新。方便的话给个星⭐

<img src="https://github.com/yangwang-1211/EasyDecrypt/blob/cadd3d00d4361659868fbea511f9eaab9b8ed0a5/img/QQ.png" width="200" />
