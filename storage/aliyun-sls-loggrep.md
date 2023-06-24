- [LogGrep: Fast and Cheap Cloud Log Storage by Exploiting both Static and Runtime Patterns](https://web.cse.ohio-state.edu/~wang.7564/papers/eurosys23-final39.pdf)

- 日志服务几乎是现代所有中大型软件所必需的辅助运维工具, 阿里云 sls 则是国内日志服务的前沿。本篇 paper 描述了一种高效的日志压缩手段。

- sls 的日志分为三种类型:
  - 在线日志: 被频繁查询的日志，主要用于监控系统状态。
  - 近线日志: 只有出现问题时才会被查询，经过一段时间(一般是 6 - 12 个月才会被归档为离线日志)。
  - 离线日志: 几乎不可能被查询的日志。

- 分为三种类型的原因很简单, 不同的查询模式在 压缩率 压缩速度 查询延迟 都有不同的权衡。用户每天产生的日志高达 PB 级，其中大多都是无用的信息，离线日志需要更高的压缩比，来减少长时间存储的成本。在线日志被频繁查询，不需要过度压缩，反而需要更多的索引数据。近线日志则需要考虑取舍。

- sls 目前使用 gzip + grep. [CLP](https://www.usenix.org/system/files/osdi21-rodrigues.pdf) 是目前最先进的日志搜索工具.

- sls 目前将原始日志写入到 64MB 的 LogBlock 中. sls 会在后台压缩这些 LogBlock.

- 对于日志搜索，常见的方法是通过解压之后再使用 grep 等标准工具，然后这些方法会产生很长的查询延迟。

- 对于一个应用程序而言, 日志的格式几乎是不变的, 考虑:
  ```rust
  info!("send request error, request_id: {} err_msg: {} ", xxx, xxx);
  output:
  [2022.08.22 08:33 12122] [info] xxx.rs: send request error, request_id: 123i123i91209391dasd, err_msg: 1923iajkkad
  ```
  这里是说，例如上方的例子: **"send request error request_id: " + "{} + " {} + "error_msg: " + "{}"** 这个字符串会在该应用程序中大量出现，而且只有 request_id: {} 和 err_msg: {} 这两个字段的后两个变量是变化的. CLP 称这些静态字符串是令牌. CLP 可以确定这些令牌是来自模版的静态部分还是变量。CLP 使用标识符替换其中的静态部分，并存储其中的变量令牌。

  笔者这里不过多介绍 CLP, 感兴趣可以看看其他介绍.

  但由于这种方法大量编排了静态字符串，对于搜索实在是不太满意。

- 对于单条日志而言，一般来说，这条日志中的变量大多都是同一类型，相同类型的值往往具有相似特性，从而获得更好的压缩性价比。

- LogGrep 将同一个变量放到相同的分区中。而不是连续存储来自不同变量的值。

- 为了进一步将日志划分为更细粒度的分区，LogGrep 提出了利用静态和运行时模式来结构化和过滤日志。

### 一些疑问:
- sls 每条日志都带有时间戳吗? 虽然论文说 aliyun all logs have timestamps. 

- aliyun 在线日志使用 ElasticSearch, 感觉不可相信。

- LogGrep 地址 [log_grep_github_link](https://github.com/THUBear-wjy/LogGrep).


### 感想：
- 事实上，关于日志本地压缩就很早有人研究过，本文中所提出的这种日志压缩方式在一些日志库就有实现，即落盘只存令牌，由于我们认为其是常量字符串，需要查询再用专有的工具解压即可。请见 https://github.com/PlatformLab/NanoLog, 论文link https://www.usenix.org/system/files/conference/atc18/atc18-yang.pdf, 这个就说了将单机应用程序日志延迟格式化。不过似乎不应该在日志库中做吧？我还没见过什么场景单机日志落盘成为瓶颈。如果有请考虑使用高性能现代化的nvme硬盘或者考虑使用更高性能的日志库。请放弃 glog (强烈谴责 glog(C++ version))!!!

- 这论文作者本科计算机，硕士哲学专业。有被惊讶到。
