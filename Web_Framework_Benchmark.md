# Web Framework Benchmark

## 1. 介绍
网站 [https://www.techempower.com](https://www.techempower.com)目前已经对于常用的web框架提供了13轮的测试，框架包括了常见的各种语言, 前几名的主流语言分别为c/c++/scala/java/go。第13轮的blog：[framework-benchmarks-round-13](https://www.techempower.com/blog/2016/11/16/framework-benchmarks-round-13/)

该网站的测试结果对于我们选择语言和框架具备非常好的参考价值。 测试的源码Github地址：[FrameworkBenchmarks](https://github.com/TechEmpower/FrameworkBenchmarks)

测试的Framework具备生产环境运行，测试过程中禁止打印日志。 参见：[Source And Requirements](https://www.techempower.com/benchmarks/#section=code&hw=ph&test=fortune)

测试的种类包括：

1. JSON serialization
2. Single database query
3. Multiple database queries
4. Fortunes
5. Database updates
6. Plaintext


## 2. 测试环境

[environments](https://www.techempower.com/benchmarks/#section=environment)

1. **ServerCentral**

	Physical hardware environment for Rounds 13 and beyond. Provided by ServerCentral. Dell R910 (4x 10-Core E7-4850 CPUs) application server; Dell R710 (2x 4-Core E5520 CPUs) database server; switched 10-gigabit Ethernet

2. **Azure**

	Cloud environment for Rounds 13 and beyond. Microsoft Azure D3v2 instances; switched gigabit Ethernet.

3. **Peak**

	Physical hardware environment for Rounds 9 through 12. Dell R720xd dual Xeon E5-2660 v2 (40 HT cores) with 32 GB memory; database servers equipped with SSDs in RAID; switched 10-gigabit Ethernet.

4. **i7**
	
	Physical hardware environment for Rounds 1 through 8. Sandy Bridge Core i7-2600K workstations with 8 GB memory (early 2011 vintage); database server equipped with Samsung 840 Pro SSD; switched gigabit Ethernet

5. **AWS**
	
	Cloud environment for Rounds 1 through 12. Amazon EC2 c3.large instances (2 vCPU each); switched gigabit Ethernet (m1.large was used through Round 9).
	
## 3. 测试结果展现

![res](http://www.do1618.com/wp-content/uploads/2016/12/web_framework_benchmark.png)

## 3. 其他相关资源

jakubkulhan提供了Node.js, Spray, Erlang, Http-kit, Warp, Tornado,和 Puma的benchmark： [hit-server-bench](https://github.com/jakubkulhan/hit-server-bench)

### Requests per second

| Environment          | Req/s (c=1) | Req/s (c=10) | Req/s (c=50) | Req/s (c=100) |
|----------------------|------------:|-------------:|-------------:|--------------:|
| Node.js              |    10215.47 |     38447.10 |     51362.60 |      52722.19 |
| Spray                |    10681.03 |     41761.57 |     50912.04 |      52746.62 |
| Erlang               |       24.70 |       246.98 |      1240.38 |       2474.18 |
| Http-kit             |     8789.63 |     61355.26 |     73457.67 |      73475.39 |
| Warp                 |    12467.62 |     41547.37 |     53475.75 |      53864.91 |
| Tornado              |     2356.82 |      4590.33 |      4422.08 |       4155.61 |
| Puma                 |    10048.54 |     19289.60 |     16280.00 |      16698.86 |
| Netty                |    28409.44 |    116605.01 |    186109.06 |     180001.91 |
| Spark (Jetty)        |    10982.23 |     44533.38 |     31624.97 |      57425.96 |
| Spring Boot (Tomcat) |     2652.85 |     13450.45 |     26336.17 |      31288.19 |
| Reactor              |    12899.17 |     50113.60 |     66310.16 |      65718.33 |
| PHP-FPM              |     2907.34 |      9968.78 |      9004.46 |       9425.08 |
| HHVM                 |     1323.99 |      6434.97 |      9192.72 |       9743.09 |
| ReactPHP (PHP)       |     1636.49 |      9454.17 |     13676.61 |      12061.91 |
| ReactPHP (HHVM)      |     2087.04 |     12185.55 |     16151.42 |      12036.11 |


### Average latency

| Environment          | Latency (c=1) | Latency (c=10) | Latency (c=50) | Latency (c=100) |
|----------------------|--------------:|---------------:|---------------:|----------------:|
| Node.js              |       83.59us |       253.57us |         1.03ms |          2.19ms |
| Spray                |      686.19us |         0.92ms |         2.71ms |          2.59ms |
| Erlang               |       40.60ms |        40.58ms |        40.35ms |         40.43ms |
| Http-kit             |       98.01us |       770.13us |       683.12us |          1.36ms |
| Warp                 |      116.25us |       333.56us |         1.30ms |          2.18ms |
| Tornado              |      414.73us |         2.28ms |        12.21ms |         24.95ms |
| Puma                 |       96.37us |       510.70us |         2.83ms |          5.88ms |
| Netty                |       53.17us |       113.52us |       635.84us |          1.18ms |
| Spark (Jetty)        |        2.04ms |       617.83us |         6.23ms |          7.94ms |
| Spring Boot (Tomcat) |        7.09ms |       815.26us |         2.25ms |          3.81ms |
| Reactor              |        2.19ms |         3.93ms |         6.72ms |          8.49ms |
| PHP-FPM              |      355.04us |         1.26ms |         6.37ms |         11.60ms |
| HHVM                 |      824.73us |         1.53ms |         5.62ms |         10.47ms |
| ReactPHP (PHP)       |      626.27us |         1.05ms |         3.81ms |         10.83ms |
| ReactPHP (HHVM)      |      481.31us |       811.58us |         3.21ms |         15.36ms |


