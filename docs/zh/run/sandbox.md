# 沙箱

在我们所设想的使用情况下，SubQuery节点通常是由一个受信任的主机来管理的。 而用户向节点所提交的 SubQuery 项目代码并不完全可信。

Some malicious code is likely to attack the host or even compromise it, and cause damage to the data of other projects in the same host. Therefore, we use the [VM2](https://www.npmjs.com/package/vm2) sandbox secured mechanism to reduce risks. This: 因此，我们使用 [VM2](https://www.npmjs.com/package/vm2) 沙盒安全机制来降低风险。 如下：

- 在受隔离的系统中运行不受信任的代码，这些代码不会访问主机的网络和文件系统，除非通过我们注入沙箱的公共接口。

- 使用安全的调用方法，并在沙箱之间交换和接受数据。

- 对许多已知的攻击方法免疫


## 限制

- 为了限制访问某些内置模块，只有 `conflict`, `buffer` `crypto`,`util` 和 `path` ，上述代码是白名单。

- 我们支持使用**CommonJS**和**CommonJS**库编写的
第三方模块，如`@polkadt/*`，它们默认使用ESM。</p></li> 
  
  - 禁止使用`HTTP`和`WebSocket`的任何模块。</ul>
