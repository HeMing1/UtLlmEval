# 单元测试生成大模型测评

> 本测评模型仅基于JAVA代码单元测试用例生成

### 测评点
1. 测试方法命名是否规范
   1. 风格是否能与整体代码仓库保持一致
   2. 测试方法名的元素是否符合规范：参考BDD的范式
2. 能否支持不同测试框架的选择（Junit4、Junit5、Mockito等）
3. 测试用例的三个要素是否完整：Given/When/Then
4. 能否识别所有分支逻辑
   1. IF-ELSE类型分支逻辑
   2. SWITCH类型分支逻辑
   3. TRY-CATCH类型分支逻辑
5. 依赖类的处理方式是否合理
   1. 能否选择合适的方式模拟依赖类（使用Dummy\Mock\Stub\Fake\Spy等方式）
   2. 能否不进行任何模拟，使用真实依赖类
   3. 能否正确使用Mockito等Mock框架
6. 能否正确使用测试框架
7. 正确性
   1. 能成功编译
   2. 能成功执行
8. 符合单元测试的FIRST原则 
   1. Fast 快速 
   2. Isolated 隔离 
   3. Repeatable 可重复 
   4. Self-verifying 自我验证 
   5. Timely 及时

