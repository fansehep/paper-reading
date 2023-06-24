- [CLP: efficient and scalable search on compressed text Logs](https://www.usenix.org/system/files/osdi21-rodrigues.pdf)

- CLP 其实和 sls loggrep 那篇的思想有点类似. 都是存储静态字符串.

- CLP 将单个日志分为三个block
  - LogType 日志类型, 通常是高度重复的静态文本
  - Variable 变量值
  - TimeStamp 时间戳

- CLP 对于变量值, 将其分为两类, 一类是大量的重复的, 例如 标识符 和 用户名, 另一类是不重复的。CLP 将前者称为字典变量, 因为 CLP 将他们提取并且存储在一个 dict 字典中.

![](/png/clp-princle.jpg)

![](/png/clp-compression-speed.jpg)

![](/png/clp-search-speed.jpg)

看上去这压缩比令人震惊🤯.
搜索速度也快.
有空看看这个 CLP 是咋实现的.
