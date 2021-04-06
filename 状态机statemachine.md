# 1 简单的示例

pom依赖：

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.4.4</version>
    <relativePath/>
</parent>


<dependencies>
    <dependency>
        <groupId>org.springframework.statemachine</groupId>
        <artifactId>spring-statemachine-starter</artifactId>
    </dependency>

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>

    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
    </dependency>
</dependencies>

<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.statemachine</groupId>
            <artifactId>spring-statemachine-bom</artifactId>
            <version>3.0.0</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>

<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
    </plugins>
</build>
```



启动类：

```java
@SpringBootApplication
public class Application{
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```



状态和事件枚举

```java
public enum States {
    SI,
    S1,
    S2
}
```

```java
public enum Events {
    E1, E2
}
```



状态机配置

```java
@Configuration
@EnableStateMachine
@Slf4j
public class StateMachineConfig
        extends EnumStateMachineConfigurerAdapter<States, Events> {

    /**
     * @Author lucky
     * @Description StateMachine 配置
     * @Date 14:56 2021/4/6
     * @Param StateMachineConfigurationConfigurer<States, Events> config
     * @return void
     **/
    @Override
    public void configure(StateMachineConfigurationConfigurer<States, Events> config)
            throws Exception {
        config
            .withConfiguration()
                .autoStartup(true)
                .listener(listener());
    }

    /**
     * @Author lucky
     * @Description 状态机状态的初始化配置
     * @Date 14:57 2021/4/6
     **/
    @Override
    public void configure(StateMachineStateConfigurer<States, Events> states)
            throws Exception {
        states
            .withStates()
                .initial(States.SI)
                    .states(EnumSet.allOf(States.class));
    }

    /**
     * @Author lucky
     * @Description 状态机 流转配置
     * @Date 14:57 2021/4/6
     **/
    @Override
    public void configure(StateMachineTransitionConfigurer<States, Events> transitions)
            throws Exception {
        transitions
            .withExternal()
            	// 当发生了Events.E1 事件，状态机从States.SI 流转到 States.S1
                .source(States.SI).target(States.S1)
                .event(Events.E1)
                .and()
            .withExternal()
                .source(States.S1).target(States.S2)
                .event(Events.E2);
    }

    /**
     * @Author lucky
     * @Description 状态机监听器bean
     * @Date 14:57 2021/4/6
     **/
    @Bean
    public StateMachineListener<States, Events> listener() {
        return new StateMachineListenerAdapter<States, Events>() {
            @Override
            public void stateChanged(State<States, Events> from, State<States, Events> to) {
                // 状态机第一次初始化的时候 from 是 null 值
                log.info("状态机状态流转到：{}状态",to.getId());
            }
        };
    }
}
```

测试：

```java
private final StateMachine<States, Events> stateMachine;

@Test
void test_01() {
    stateMachine.sendEvent(Events.E1);
    stateMachine.sendEvent(Events.E2);
}
```



输出：

状态机状态流转到：SI状态

状态机状态流转到：S1状态

状态机状态流转到：S2状态