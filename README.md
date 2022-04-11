# unit-test-project

单元测试的示例，

1. spock及mockito
    1. 常见测试、简单注入
        ```java
        @InjectMocks
        private SkuService skuService = new SkuServiceImpl();

       @Mock
       private SkuDao skuDao;
        ```

       ```groovy
       /*
       @see https://github.com/marcingrzejszczak/spock-subjects-collaborators-extension
       */
        @Subject // @InjectMocks
        InjectServiceImpl injectService

        @Collaborator // 类似于Mockito的@Mock
        private OtherService otherService = Mock();

       ```
    2. for循环调用
        1. void方法
        2. 有返回值，多次调用不同
       1. 根据入参响应不同值
    3. 多线程模拟
       ```java
         Mockito.doAnswer((invocation) ->{
            ((Runnable)invocation.getArguments()[0]).run();
            return null;
        }).when(asyncExecutor).execute(any(Runnable.class));
       ```
       ```groovy
        asyncExecutor.execute(_) >> { Runnable it -> it.run() }
       ```
    4. 复杂对象的比较
        1. junit
       ```java
        /** 入参*/
       doReturn(skuList).when(otherService).querySkuByCity(ArgumentMatchers.argThat((it) ->
                it.getId() == city.getId() && it.getName().equals(city.getName())
        ));
       verify(otherService, times(1)).
                doSomethingNoReturn(ArgumentMatchers.argThat((it) -> {
                    return it.getId() == city.getId() && it.getName().equals(city.getName());
                }));
            
       /**响应*/
        MatcherAssert.assertThat(BigDecimal.valueOf(10.0), Matchers.comparesEqualTo(BigDecimal.TEN));
       
        Assertions.assertIterableEquals(skuList, rList);
       //比较属性
        Assertions.assertAll(() ->{
            Assertions.assertEquals(city.getId(),r.getId());
            Assertions.assertEquals(city.getName(),r.getName());
        });
       
       
       // 直接比较toString，前提是重写toString()
        Assertions.assertEquals(city.toString(), r.toString());

       ```
        2. spock
        ```groovy
        /** 入参 闭包校验**/
        otherService.querySkuByCity({ City it ->
        it.getId() == city.getId() && it.getName().equals(city.getName())
        }) >> skuList 
       
        1 * otherService.doSomethingNoReturn({City it ->
            it.getId() == city.getId()&&it.getName()==it.getName()
        })
       
       /**响应**/
       //junit 中响应校验在在spock都可以使用
        // 一般使用 == 就可以验证，groovy末主动调用equals方法，包括BigDecimal
       //复杂对象使用闭包
       
        
        ```
    5. 断言
        1. spock 是隐性断言，直接在expect或when 语句块中 比较对象即可
        2. mockito 需要调用Assert 相关的方法
    6. mock、spy、stub
        1. mock 模拟对象响应、记录模拟对象的调用情况、验证模拟对象
        2. stub 主要功能之一是模拟对象响应
        3. spy 在调用时默认调用真实方法
    7. 参数化测试
        1. spock 灵活，在测试方法体内，where 打码块就可以指定参数测试，更加符合思维逻辑
       ```groovy
        def querySkuByCityId() {
           given:
           cityService.getCity(cityId) >> city
           and:
           otherService.querySkuByCity({ City it ->
           it.getId() == city.getId() && it.getName().equals(city.getName())
           }) >> skuList
    
           when:
           List<Sku> rList = injectService.querySkuByCityId(cityId);
           then:
           Assertions.assertIterableEquals(skuList, rList);
    
           where:
           cityId || city               | skuList
           1L     || new City(1L, "北京") | [new Sku(1L, "1"), new Sku(2L, "2")]
           2L     || new City(2L, "上海") | [new Sku(1L, "1"), new Sku(2L, "2")]
        }
        ```
        2. junit5 需要使用注解 @ParameterizedTest 表明是参数化测试，再使用其他注解来提供不同的参数，eg:@MethodSource、@ValueSource
       ```java
       @DisplayName("根据城市id获取SKU列表")
       @ParameterizedTest  //与注解Test互斥
       @MethodSource(value = "argProvider")
       void querySkuByCityId(List<Sku> skuList, long cityId, City city) {
            doReturn(city).when(cityService).getCity(cityId);
            doReturn(skuList).when(otherService).querySkuByCity(ArgumentMatchers.argThat((it) ->
                    it.getId() == city.getId() && it.getName().equals(city.getName())
            ));
            List<Sku> rList = injectService.querySkuByCityId(cityId);
            Assertions.assertIterableEquals(skuList, rList);
       }
    
       static Stream<Arguments> argProvider() {
           return Stream.of(
           Arguments.of(Lists.newArrayList(new Sku(1L, "1"), new Sku(2L, "2")), 1L, new City(1L, "北京")),
           Arguments.of(Lists.newArrayList(new Sku(1L, "1"), new Sku(2L, "2")), 2L, new City(2L, "上海"))
           );
       }

       ```
    8. 异常情况
       
    9. 对象初始化
        1. java必须严格按照语法进行new，如果没有专门的初始化方法，需要调用大量的get、set 方法，进行强制语法检查，好处是在运行前就可以发现问题，ide会提示
        2. groovy 语法比较灵活，在new的时候可以直接指定字段值、也可以使用跟java相同的方法，坏处是在可能字段可能拼错，不能及时检查出来，只能在运行时才可以发现
    10. 测试结构
        1. spock 在测试用例中按照BDD的模式规定了用例结构，风格统一，在进行review时容易发现问题，有迹可循，上手比较快
        2. junit/mockito 没有指定任何结构，支持BDD开发，但是需要开发同学自己在写的时候注意，可能每位同学的编写风格都不一致
2. 在Spring进行测试
3. 测试mybatis数据库操作，使用h2
4. 测试覆盖率、断言覆盖率


