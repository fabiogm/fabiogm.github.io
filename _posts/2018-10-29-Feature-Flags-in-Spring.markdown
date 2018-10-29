---
layout: post
title:  "Feature Flags"
date:   2017-04-29 20:20:15 -0300
categories: programming, java, spring
---

Feature flags (also feature toggles) are useful when you want the ability to turn some application behavior on or off. Let's say you are rolling out
a new functionality. It might be the case that you don't want to make it available from the get go, maybe part of the functionality is not complete. It might
also be the case that you only want to show this functionality to a subset of users. That old-fashioned way of doing this is using conditional statements

```java
if (bazFunctionalityEnabled) {
    showBaz();
} else {
    showFoo();
}
``` 

There are several libraries for aiding in the process of creating feature toggles. Also Spring ships with an annotation called `@ConditionalOnProperty` that
is also helpful on those scenarios. This annotation tells Spring whether or not to create a `Bean` based on a property. Let's say you have a `Bean` for listening
to a queue

```java
@Component
public class FooSubscriber {
    private final MessageConverter<Event> messageConverter;
    private FooService fooService;

    @Autowired
    public FooSubscriber(FooService fooService) {
        this.messageConverter = new MessageConverter<>(Event.class);
        this.fooService = fooService;
    }

    @RabbitListener(queues = {"#{@fooQueue}"})
    public void receiveMessage(byte[] message) {
        Event event = messageConverter.convert(message);

        fooService.save(e.getId());
    }
}
``` 

You might want to disable processing of messages. Maybe the server is too busy processing messages. Just add conditional annotation

```java
@Component
@ConditionalOnProperty(name = "feature.enable.foo.processing", matchIfMissing = true)
public class FooSubscriber {
    (...)
}
```

This way, if `feature.enable.foo.processing` is equal to `true` in `application.properties` the bean will be created and consequently messages will be
processed. Otherwise, Spring won't create the bean. Notice the argument `matchIfMissing` with a value of `true`. It tells Spring to consider the property
as being true if it is not defined. This way you can define a default behavior in case that property is not defined.

This is even more convenient if you use distributed configuration tools like Spring Cloud Config. If you want to turn a certain feature off just change the
configuration in Cloud Config repository, push those changes, restart the app and you will have the desired result.

In order to check more than one property, just supply an array of property names in the `name` parameter

```java
@ConditionalOnProperty(name = {"feature.enable.message.processing", "feature.enable.foo.processing"}, matchIfMissing = true)
```

See more in the [official docs](http://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/autoconfigure/condition/ConditionalOnProperty.html).

## Advanced Checks with @ConditionalOnExpression and Spring Expression Language

If you need more elaborated checks you can use Spring Expression Language (SpEL) expressions inside `@ConditionalOnExpression` annotations. Example:

```java
@Component
@ConditionalOnExpression("${messaging.enable} && ${messaging.throttle} < 10")
public class FooSubscriber {
    (...)
}
```

### Bean Creation on Class Properties

You can also conditionally define multiple beans inside a class and use the presence or absence of this class as flag for another class. It is necessary to
extend `AllNestedConditions` abstract class from Spring

```java
public class FooMessagingFlag extends AllNestedConditions {
    @Bean
    @ConditionalOnProperty("feature.enable.messaging")
    publiMessagingBean 
}


@Conditional({FooMessagingFlag.class})
@Configuration
```


