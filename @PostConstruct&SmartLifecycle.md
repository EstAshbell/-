记录开发遇到的一个问题

redis抢锁后通过手动启动的方式启动kafka监听发现@KafkaListerner并没有注入进去



项目执行逻辑：在redis抢到锁后执行方法将@KafkaListener启动起来
之前抢到锁的方法是使用@PostConstruct注解实现的，但是当@KafkaListener 的autoStartup改为false后，

```java
@KafkaListener(id = "listener1",
            topics = {"#{'${kafka.topic}'.split(',')}"},
            groupId = "group-dev" ,idIsGroup = false,autoStartup = "false"
    )
```

通过kafka统一管理来手动启动kafka监听时

```java
@Autowired
private KafkaListenerEndpointRegistry registry;

@PostConstruct
public void init(){
	//xxx抢到锁后执行回调方法，这里指的是通过
    MessageListenerContainer listenerContainer = registry.getListenerContainer("listener1");
    if (!listenerContainer.isRunning()){
            listenerContainer.start();
    }
}
```

会发现listenerContainer为null，没有找到该id的监听。





- @PostConstruct注解的方法将会在**该**bean依赖注入完成后被自动调用。
- 实现SmartLifecycle接口后是在Spring加载和初始化**所有**bean后被调用