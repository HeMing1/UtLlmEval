## 交互层单元测试

在Web应用中用户交互层的主要职责是接收外部各种协议的请求，并调度系统作出相应的操作和返回数据。根据不同项目的情况可以将代码进行合适的分层，在本示例代码中，将交互层代码大致分为两层，Controller层和Validator
层（随项目变化不断迭代），根据单一职责原则，需保障每一层的实现代码职责单一，同时测试代码也保持职责单一。

### 交互层职责
| **#** | **职责**       | **描述**                                                     |
| ----- | -------------- | ------------------------------------------------------------ |
| 1.    | 监听 HTTP 请求 | Controller应该响应某些 URL、HTTP 方法和内容类型。                |
| 2.    | 反序列化输入   | Controller应该解析传入的 HTTP 请求并根据 URL、HTTP 请求参数和请求正文中的变量创建 Java 对象，以便我们可以在代码中使用它们。 |
| 3.    | 验证输入       | Controller是防止错误输入的第一道防线，因此它是我们可以验证输入的地方。 |
| 4.    | 调用业务逻辑   | 解析输入后，Controller必须将输入转换为业务逻辑期望的模型并将其传递给业务逻辑。 |
| 5.    | 序列化输出     | Controller获取业务逻辑的输出并将其序列化为 HTTP 响应。           |
| 6.    | 转换异常       | 如果在某个地方发生异常，Controller应将其转换为对用户有意义的错误消息和 HTTP 状态。 |


### 测试关注点
#### Controller
Controller层的代码主要用来处理用户请求和返回处理结果，测试过程中需要考虑
* Http请求的URL、Header、Content等
* 外部Input的校验逻辑
* 内部Service、Validator、Adapter等核心逻辑的调度
* 请求和返回的格式是否正确，如Rest请求的path、requestBody、response等
> 必要的时候可以使用MockMvc、RestAssured等框架进行端到端的单元测试

#### Request
Request作为接收用户请求模型，核心关注点主要包括：
* 反序列化的正确性，如Rest请求中从json到对象的转化
* 使用SpringValidator做校验时，对校验规则的测试
* Request到系统其他类型模型的转换（也可抽象到Factory、Convert等模式）

#### Validator
对用户请求对象的校验
* 校验过程中产生的异常
* 校验可能会有很多的分支逻辑，根据业务性质选择测试用例粒度

### 使用的工具
* SpringBootTest 集成了大部分单测框架如JUnit，hamcrest，assertj，mockito

### 参考文档
* https://spring.io/guides/gs/testing-web/

### 以下为使用springMvc实现交互层代码示例

``` java
package com.example.demo.interactive.controller;

import javax.validation.Valid;
import javax.validation.constraints.NotNull;

import com.example.demo.domain.entity.Order;
import com.example.demo.interactive.request.OrderCreateRequest;
import com.example.demo.interactive.response.ResponseWrapper;
import com.example.demo.interactive.validator.OrderRequestValidator;
import com.example.demo.service.OrderApplicationService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class OrderController {

    private final OrderApplicationService orderApplicationService;
    private final OrderRequestValidator orderRequestValidator;

    @Autowired
    public OrderController(OrderApplicationService orderApplicationService, OrderRequestValidator orderRequestValidator) {

        this.orderApplicationService = orderApplicationService;
        this.orderRequestValidator = orderRequestValidator;
    }

    @PostMapping("/orders")
    public ResponseWrapper create(@RequestBody @Valid @NotNull OrderCreateRequest orderCreateRequest) {
        orderRequestValidator.validateCreate(orderCreateRequest);
        Order order = orderCreateRequest.convert();
        orderApplicationService.create(order);
        return ResponseWrapper.success();
    }
}

```

### 使用通义灵码生成的单元测试

![img.png](img.png)

```java
package com.example.demo.interactive.controller;


import com.example.demo.domain.entity.Order;
import com.example.demo.interactive.request.OrderCreateRequest;
import com.example.demo.interactive.response.ResponseWrapper;
import com.example.demo.interactive.validator.OrderRequestValidator;
import com.example.demo.service.OrderApplicationService;
import org.junit.Before;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.mockito.InjectMocks;
import org.mockito.Mock;
import org.mockito.MockitoAnnotations;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTest;
import org.springframework.boot.test.mock.mockito.MockBean;
import org.springframework.http.MediaType;
import org.springframework.test.context.junit4.SpringRunner;
import org.springframework.test.web.servlet.MockMvc;
import org.springframework.test.web.servlet.setup.MockMvcBuilders;

import static org.mockito.Mockito.doNothing;
import static org.mockito.Mockito.doThrow;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.post;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.jsonPath;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;

@RunWith(SpringRunner.class)
@WebMvcTest(OrderController.class)
public class OrderControllerApiTest {

    @Autowired
    private MockMvc mockMvc;

    @MockBean
    private OrderApplicationService orderApplicationService;

    @MockBean
    private OrderRequestValidator orderRequestValidator;

    @InjectMocks
    private OrderController orderController;

    @Before
    public void setup() {
        MockitoAnnotations.initMocks(this);
        mockMvc = MockMvcBuilders.standaloneSetup(orderController).build();
    }

    @Test
    public void create_ValidRequest_ReturnsSuccess() throws Exception {
        OrderCreateRequest request = new OrderCreateRequest();
        request.setAddress("address");
        request.setMemberId(1L);
        request.setOrderAmountTotal(new BigDecimal("100"));
        request.setOrderNo("orderNo");
        request.setProductAmountTotal(new BigDecimal("200"));
        request.setProductCount(100L);
        request.setProductId(1L);
        request.setSupplierId(1L);

        doNothing().when(orderRequestValidator).validateCreate(request);
        doNothing().when(orderApplicationService).create(request.convert());

        mockMvc.perform(post("/orders")
                .contentType(MediaType.APPLICATION_JSON)
                .content("{\n"
                        + "    \"orderNo\":\"orderNo\",\n"
                        + "    \"supplierId\":1,\n"
                        + "    \"address\":\"address\",\n"
                        + "    \"productId\":1,\n"
                        + "    \"orderAmountTotal\":100,\n"
                        + "    \"productCount\":100,\n"
                        + "    \"productAmountTotal\":200,\n"
                        + "    \"memberId\":1\n"
                        + "}"))
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.success").value(true))
                .andExpect(jsonPath("$.message").value("OK"));
    }

    @Test
    public void create_InvalidRequest_ThrowsConstraintViolationException() throws Exception {
        OrderCreateRequest request = new OrderCreateRequest();
        request.setAddress(""); // 无效的地址

        doThrow(new ConstraintViolationException("Validation failed", null))
                .when(orderRequestValidator).validateCreate(request);

        mockMvc.perform(post("/orders")
                .contentType(MediaType.APPLICATION_JSON)
                .content("{\n"
                        + "    \"orderNo\":\"orderNo\",\n"
                        + "    \"supplierId\":1,\n"
                        + "    \"address\":\"\",\n"
                        + "    \"productId\":1,\n"
                        + "    \"orderAmountTotal\":100,\n"
                        + "    \"productCount\":100,\n"
                        + "    \"productAmountTotal\":200\n"
                        + "}"))
                .andExpect(status().isBadRequest());
    }
}
```

### 单元测试生成结果分析
1. 正确性：执行失败，对测试框架的使用有一定偏差，可能是测试框架版本和训练所用代码版本不一致导致，微调后可正确执行
2. 有效性：生成的单元测试代码已经正确实现了对OrderController类的create方法的测试，并使用了MockMvc进行对HTTP请求的模拟测试
3. 丰富性：测试用例的丰富度足够，对正确输入、异常输入、边界输入都做了覆盖
4. 测试设计：对于SpringMvc框架的代码直接选择了MockMvc进行测试，在简单的工程中是不错的选择
5. 通过对话引导，不使用MockMvc和Mockito进行测试，测试结果如下，整体符合预期

```java
package com.example.demo.interactive.controller;

import com.example.demo.domain.entity.Order;
import com.example.demo.interactive.request.OrderCreateRequest;
import com.example.demo.interactive.response.ResponseWrapper;
import com.example.demo.interactive.validator.OrderRequestValidator;
import com.example.demo.service.OrderApplicationService;
import org.junit.Before;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.mockito.InjectMocks;
import org.mockito.Mock;
import org.mockito.MockitoAnnotations;
import org.mockito.junit.MockitoJUnitRunner;

import javax.validation.ConstraintViolation;
import javax.validation.Validation;
import javax.validation.Validator;
import javax.validation.ValidatorFactory;
import java.math.BigDecimal;
import java.util.Set;

import static org.junit.Assert.assertEquals;
import static org.junit.Assert.assertTrue;
import static org.mockito.Mockito.doNothing;
import static org.mockito.Mockito.doThrow;

@RunWith(MockitoJUnitRunner.class)
public class OrderControllerApiTest {

    private OrderController orderController;

    @Mock
    private OrderApplicationService orderApplicationService;

    @Mock
    private OrderRequestValidator orderRequestValidator;

    private Validator validator;

    @Before
    public void setup() {
        MockitoAnnotations.initMocks(this);
        ValidatorFactory factory = Validation.buildDefaultValidatorFactory();
        validator = factory.getValidator();
        orderController = new OrderController(orderApplicationService, orderRequestValidator);
    }

    @Test
    public void should_return_success_result_when_create_order_given_order_service_create_success() {
        OrderCreateRequest request = new OrderCreateRequest();
        request.setAddress("address");
        request.setMemberId(1L);
        request.setOrderAmountTotal(new BigDecimal("100"));
        request.setOrderNo("orderNo");
        request.setProductAmountTotal(new BigDecimal("200"));
        request.setProductCount(100L);
        request.setProductId(1L);
        request.setSupplierId(1L);

        doNothing().when(orderRequestValidator).validateCreate(request);
        doNothing().when(orderApplicationService).create(request.convert());

        ResponseWrapper response = orderController.create(request);

        assertEquals(true, response.isSuccess());
        assertEquals("OK", response.getMessage());
    }

    @Test(expected = ConstraintViolationException.class)
    public void should_throw_constraint_violation_exception_when_create_order_given_invalid_request() {
        OrderCreateRequest request = new OrderCreateRequest();
        request.setAddress(""); // 无效的地址
        request.setMemberId(1L);
        request.setOrderAmountTotal(new BigDecimal("100"));
        request.setOrderNo("orderNo");
        request.setProductAmountTotal(new BigDecimal("200"));
        request.setProductCount(100L);
        request.setProductId(1L);
        request.setSupplierId(1L);

        Set<ConstraintViolation<OrderCreateRequest>> violations = validator.validate(request);
        assertTrue(violations.size() > 0);

        orderController.create(request);
    }
}

```
