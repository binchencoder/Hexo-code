---
title: Arthas
date: 2019-11-29 20:53:08
tags:
	- Java
	- 后端
	- 线上诊断
categories:
	- 工具
---

# Arthas

**Arthas 是Alibaba开源的Java诊断工具，深受开发者喜爱。**
1. 当你遇到以下类似问题而束手无策时，Arthas可以帮助你解决：
2. 这个类从哪个 jar 包加载的？为什么会报各种类相关的 Exception？
3. 我改的代码为什么没有执行到？难道是我没 commit？分支搞错了？
4. 遇到问题无法在线上 debug，难道只能通过加日志再重新发布吗？
5. 线上遇到某个用户的数据处理有问题，但线上同样无法 debug，线下无法重现！
6. 是否有一个全局视角来查看系统的运行状况？
7. 有什么办法可以监控到JVM的实时运行状态？
8. 怎么快速定位应用的热点，生成火焰图？

## 下载

下载arthas-boot.jar，然后用java -jar的方式启动：

```sh
curl -O https://alibaba.github.io/arthas/arthas-boot.jar

java -jar arthas-boot.jar
```

> 如果从github下载有问题，可以使用gitee镜像
>
> curl -O https://arthas.gitee.io/arthas-boot.jar

## watch

**方法执行数据观测**

让你能方便的观察到指定方法的调用情况。能观察到的范围为：返回值、抛出异常、入参，通过编写OGNL表达式进行对应变量的查看。

watch 的参数比较多，主要是因为它能在 4 个不同的场景观察对象


参数名称 | 参数说明
---|---
class-pattern |	类名表达式匹配
method-pattern |	方法名表达式匹配
express	| 观察表达式
condition-express |	条件表达式
[b] | 在方法调用之前观察
[e] | 在方法异常之后观察
[s] | 在方法返回之后观察
[f] | 在方法结束之后(正常返回和异常返回)观察
[E] | 开启正则表达式匹配，默认为通配符匹配
[x:] | 指定输出结果的属性遍历深度，默认为 1

### Samples

> **watch**
```java
watch com.xxx.uac.service.OrgService getCompanyStopedUserDeptNode "{params, returnObj}" params[0]==10 -b -s -x 5

watch com.xxx.search.searchcenter.spi.sp.Searcher execute "{params, returnObj}" "params[0].{? #this.routings[0]==2}.size()>0" -x 5
```

## References

- https://alibaba.github.io/arthas/
- https://github.com/alibaba/arthas