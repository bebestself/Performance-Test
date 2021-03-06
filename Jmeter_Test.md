# Jmeter性能测试脚本开发
## 步骤：
1. 测试计划中新建线程组并对名称进行定义（测试场景）
2. 通过Fiddler抓包获取场景下依次发送的请求，添加相应的取样器，并对名称进行定义（请求的作用）
3. 添加一个配置元件-HTTP请求默认值，放置线程组上面，对服务器的ip和端口进行配置
4. 依次获取抓包得到的请求的请求方式，请求路径，请求参数进行设置
- **注意：请求参数不能写死，而是通过之前预埋的数据进行读取和设置。首先，密码为固定的123456。其次用户名在数据预埋时按照相应的格式进行的设计：以python开头，随后以自然数增长，所以在登录请求下可以添加一个配置元件-计数器，方便对用户名进行组装。计数器设置有：开始值，递增值，（最大值，数字格式）*可选*，以及引用名称进行定义，如user_no**
- **对参数进行合理的填写、引用和组装，实现使用不同的用户名去执行登录请求，动态的请求。其中用户名按照格式为：python${user_no}，_通过${}去引用变量是Jmeter的语法_**
5. **获取登录信息请求设置成功后，可以得到一个响应，需要对该请求获取响应断言，在该请求下添加断言，根据其返回值格式设置响应断言，如json断言，需要对验证点进行提取以及预期返回值是否匹配，分别为Assert Json Path exists字段（$.键名），Expected Value字段进行设置**
6. 在测试计划下添加监听器-查看结果树，验证场景是否正确并进行调试操作。
- **注意凡是和登录相关的操作，首先需要在测试计划下添加配置元件-HTTP Cookie管理器**
7. 进行信息查询场景的抓包，并添加相应的取样器，定义名称，设置参数。
- **如果请求的参数所代表含义不明确，去问开发。该请求中参数为时间戳（精确到毫秒），需要通过Jmeter生成一个时间戳去引用到参数中。方法为：Tools-函数助手。选择需要的函数，比如时间戳为__time，点击生成，拷贝函数字符串到需要设置的参数位置即可。**
- **获取顾客信息的请求参数顾客编号为模拟真实使用场景，也不能写死，根据预埋的顾客编号格式，此处为p（1-15000），所以请求中我们去随机取一个数，用到函数助手，选择__Random，生成指定范围内的随机数，设置范围内的最小值和最大值，生成并拷贝函数字符串，然后通过添加配置元件-用户定义的变量来保存每一次所取到的随机数方便后面验证**
8. 接口请求有返回值，需要根据返回值的验证点进行断言，json断言在断言路径字段设置$.cno，断言值字段设置为之前保存的用户定义的变量
9. 在测试计划中添加监听器-**聚合报告**（性能测试必须的），以记录请求的相关信息。聚合报告相关字段含义：
- label：每个jmeter元件的名称。请求名字
- samples：发出请求数量，比如模拟10个用户，每个用户循环10次，这里显示100
- Average：平均响应时间（单位：毫秒）。默认是单个Request的平均响应时间，当使用了Transaction Controller（事务控制器）时，也可以以Transaction（事务）为单位显示平均响应时间
- Median：中位数，50%的用户响应时间均小于这个数，单位毫秒
- 90%Line：90%的请求响应时间均小于这个时间，单位毫秒
- Min：最小响应时间
- Max：最大响应时间
- Error%：本次测试中出现错误的请求的数量/请求的总数
- Throughput：吞吐量。默认情况下表示每秒完成的请求数
10. 通常我们以事务的角度查看而不是单个请求的方式，在线程组中添加逻辑控制器-事务控制器并进行定义。（登录事务，查询客户信息事务，查询经办人信息事务）
11. 将事务相关的请求多选拉入事务控制器中，并勾选Generate parent sample选项（勾选后在聚合报告中仅以事务为单位进行显示）
12. 为模拟真实用户场景，在线程组下添加逻辑控制器-仅一次控制器，并将登录事务拉入仅一次控制器下。这样设置后若修改线程组的循环次数，仅一次控制器循环次数为1次
13. 为模拟真实用户场景，在需要输入参数的请求下添加定时器-高斯随机定时器，高斯随机定时器设置主要为固定延迟和偏差
14. 在线程组下，添加逻辑控制器-吞吐量控制器，吞吐量控制器设置有两种选项，固定吞吐量和按比例划分，此处为按比例划分
15. 将需要设置吞吐量控制的事务拉入吞吐量控制器下，
