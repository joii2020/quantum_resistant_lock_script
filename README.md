
# Quantum Resistant Lock Script

## 原始方案
### 生成密钥
生成256对长为256bits的随机数，作为私钥。一共 256*2*256 （128kbits）
对生成的512个数进行哈希，这些生成的哈希作为公钥。

### 签名
* 把之前生成的私钥分成两组：A和B
* 把需要签名的消息进行哈希，生成一个256bits的数据。
* 对于这256bits数据的每一位，根据该位的值从私钥AB中选择对应的数（如果为0，选择 A，如果为1则选择B）。最终会产生256*256bits的数据作为签名数据

### 验证
* 分别对签名数据的每一位进行哈希
* 根据message选择公钥中对应的数据，并对比两组数据是否相等
* 循环上面的步骤


### 优劣
* 会产生很大的数据：公钥签名均为16k
* 性能问题： 生成公钥需要 512次哈希，验证签名需要256次哈希
* 私钥不能重复使用，否则会有严重的安全问题


## 缩短签名方案
https://gist.github.com/karlgluck/8412807
### 生成密钥和签名
* 获取文档的哈希值 256bits 作为messge
* 将这256bits 切分为32个块，每块8bits的数据，分为AB两组。
* 对应每个块都生成一对256bits的随机数 一共64个
* 将上面生成的随机数进行258（2^bits + 2）次哈希，这样会得到一个链状结构，最后得到的哈希作为公钥
* 对其中一块进行签名：8bits转换为int为n（ 0 <= n <= 255 ），从对应的A链中取第n个和B链中的第256-n个作为签名数据

### 验证
* 对message的32块 8bits数据分别校验
* 将8bits数据转换为n
* 将签名中A组的数据进行 258-n 次哈希，B组中进行 n+2 次哈希
* 得到的结果与hash进行比较

### 优劣
* 该算法通过时间换取空间来缩短密钥长度
* 产生的公钥长度为2k，签名数据为2k
* 生成公钥需要 258 * 64 次哈希
* 校验需要最多 256 * 64 次哈希
* 哈希链会不会出现重复的值，是否需要进行数学证明

# END

代码测试结果：正常
|        | 生成   | 签名   | 验证 | 
| ------ | ---   | ----   | ---- |
| 正常    | 512   |  0     | 256 |
| 短密钥  | 16512 | 8256   | 8256 |

是不是可以在短密钥的基础上再做优化：
只生成一对私钥和公钥，每 258 次循环生成的公钥就是下一组的私钥？