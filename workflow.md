# 工作流引擎开发

## 技术选择

* 协议 BPMN2.0 https://www.omg.org/spec/BPMN/2.0/
* 引擎 Flowable https://www.flowable.org
* 设计器 AntV G6 https://www.yuque.com/antv/g6

## BPMN2.0

### 什么是BPMN2.0

全称BUSINESS PROCESS MODEL AND NOTATION。

Business Process Model and Notation has become the de-facto standard for business processes diagrams. It is intended to be used directly by the stakeholders who design, manage and realize business processes, but at the same time be precise enough to allow BPMN diagrams to be translated into software process components. BPMN has an easy-to-use flowchart-like notation that is independent of any particular implementation environment.

[机翻]业务流程模型和符号已经成为业务流程图的事实标准。它旨在由设计、管理和实现业务流程的涉众直接使用，但同时要足够精确，以允许将BPMN图转换为软件流程组件。BPMN有一个易于使用的流程图式符号，它独立于任何特定的实现环境。

参考文档 http://www.mossle.com/docs/jbpm4devguide/html/bpmn2.html#completeExample
示例文档 https://www.omg.org/cgi-bin/doc?dtc/10-06-02.pdf


### collaboration 参与方
定义流程中的各方，例如本系统、用户、第三方系统等。
```
<collaboration id="onestore">
    <participant name="采购系统" processRef="Demo001-p1" id="Demo001-c1"/>
</collaboration>
```

### process 流程
定义一个可执行的流程
```<process isExecutable="true" id="Demo001-p1" name="Demo001-p1">``` 

### lane & pool 泳道和池
泳道和池将一个process划分为多个区域和阶段方便展示和理解。引擎中的流程可以省略。
```
<laneSet id="ls_1-1">
  <lane name="1st level support" partitionElementRef="tns:FirstLevelSupportResource" id="_1-9"> 
    <flowNodeRef>_1-13</flowNodeRef> 
    <flowNodeRef>_1-26</flowNodeRef> 
    <flowNodeRef>_1-77</flowNodeRef> 
    <flowNodeRef>_1-128</flowNodeRef> 
    <flowNodeRef>_1-150</flowNodeRef> 
    <flowNodeRef>_1-201</flowNodeRef> 
    <flowNodeRef>_1-376</flowNodeRef>
  </lane>
  <lane name="2nd level support" partitionElementRef="tns:SecondLevelSupportResource" id="_1-11"> 
    <flowNodeRef>_1-252</flowNodeRef> 
    <flowNodeRef>_1-303</flowNodeRef> 
    <flowNodeRef>_1-325</flowNodeRef>
  </lane>
</laneSet>
```

### event 事件

#### startEvent 开始

简单的开始节点，流程从这个节点开始向后运行，并且是手动执行
```
<startEvent name="开始节点" id="start1559811801412"/>
```

循环开始执行，`startEvent`内有一个`timerEventDefinition`定时器的定义, 每天的17:37:09开始执行流程
```
<startEvent name="定时器" id="timeB1563010389061" isInterrupting="false">
  <timerEventDefinition>
    <timeCycle flowable:endDate="">9 37 17 * * ?</timeCycle>
  </timerEventDefinition>
</startEvent>
```

#### endEvent 结束

简单的结束节点，如果一个流程有多个分支，可能会有多个结束节点
```
<endEvent name="结束节点" id="stop1563010286102"/>
```

### task 任务

#### serviceTask 服务任务

#### scriptTask 脚本任务

#### sendTask 发送任务

#### 

### gateway 网关

### sequenceFlow 流向


## Flowable


## AntV G6
