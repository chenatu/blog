title: spring-boot单元测试
date: 2016/10/28
categories:
- coding
tags:
- java
- spring
---
本文仅适用spring-boot 1.4版本以后的写法，至于1.4以前的版本，还是建议升级到1.4:)

如果仅仅是直接调用接口函数进行测试的话，非常简单增加。在测试类增加一些注解就好了。利用`@Autowired`也能将接口给注解进来。

````
@RunWith(SpringRunner.class)
@SpringBootTest
public class ApiTest {
    @Autowired
    MessageApi messageApi;
    ...
````

同时spring-boot-test也直接支持mockmvc。如果测试controller的话会用得到，像下面这样就就好了

````
@RunWith(SpringRunner.class)
@SpringBootTest
@AutoConfigureMockMvc
public class ControllerTest {
    @Autowired
    private MockMvc mockMvc;

    @Test
    public void testControllerMethods() {
    	MvcResult result = mockMvc.perform(get("/get-receive-message-abstracts").param("siteId", "webtrn").param("uid", "lucy")
                    .param("limit", "100")).andExpect(status().isOk())
                    .andExpect(jsonPath("$", hasSize(10))).andExpect(jsonPath("$[9].title", is("hello0"))).andReturn();
    }
````

其中MockMvc可以模拟http对于controller的请求主要用到的函数在我的测试用例里面都列出来了。大家开发的时候直接看javadoc就好了。