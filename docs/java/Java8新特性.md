

## Java8新特性

### 测试代码

```java
public static void main(string[] args){
	
}

```

> [!NOTE]
> An alert of type 'note' using global style 'callout'.

> [!TIP]
> An alert of type 'tip' using global style 'callout'.


> [!WARNING]
> An alert of type 'warning' using global style 'callout'.


> [!DANGER]
> An alert of type 'danger' using global style 'callout'.


> [!NOTE|style:flat]
> An alert of type 'note' using alert specific style 'flat' which overrides global style 'callout'.


> [!COMMENT]
> An alert of type 'comment' using style 'callout' with default settings.

### Section X

```plantuml
@startuml
Alice -> Bob: Authentication Request
Bob --> Alice: Authentication Response
Alice -> Bob: Another authentication Request
Alice <-- Bob: another authentication Response
@enduml
```

<!-- panels:start -->
<!-- div:title-panel -->

  (...) - Awesome title

<!-- div:left-panel -->

  (...) - Awesome explanation

<!-- div:right-panel -->


  (...) - Awesome example

<!-- panels:end -->


## 时序图

```plantuml
@startuml
skinparam ParticipantBackgroundColor #DeepSkyBlue


participant "odps" as odps
participant "用户洞察中心" as uic
participant "insadvisor" as insadvisor
participant "insservicesale" as insservicesale
participant "alg" as alg
participant "nlu意图识别模型" as nlu
participant "lindorm" as lindorm
database "obkv" as obkv

odps->obkv: 0.同步数据
group#DeepSkyBlue 定时任务
insservicesale->odps: 1.捞取出所有的被点击过的主动服务透出文案，sug推荐文案
insservicesale->insadvisor:2.调用insadvisor获取文案意图
insservicesale->insservicesale: 3.落库透出文案与意图对应关系日志
insservicesale->obkv:4.更新文案意图关系表的意图字段


end group


@enduml
```