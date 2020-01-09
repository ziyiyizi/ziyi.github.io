---
title: springboot项目启动后执行的类
author: ziyi
tags:
  - spring boot
index_img: /img/mybg.jpg
banner_img: /img/mybg.jpg
categories:
  - spring boot
comments: true
---

# springboot项目启动后执行的类

```
@Component
@Slf4j
@EnableBinding(PerformanceOutput.class)
/**
 * 据实践 使用stream发送第一条消息均会失败 因此在启动后发送一条无效消息 以确保之后的消息不会出现此问题
 */
public class AfterServiceStartRunner implements ApplicationRunner {

    @Autowired
    private PerformanceOutput performanceOutput;

    /**
     * 会在服务启动完成后立即执行
     */
    @Override
    public void run(ApplicationArguments args) throws Exception {
        performanceOutput.output().send(MessageBuilderUtil.buildMessage("performance已启动", "performance"));
    }
}
```