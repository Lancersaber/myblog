---
title: Activiti API
time: 2019-11-02
categories: Activiti
tags: Activiti
---

# Activiti API
---
ProcessEngine API是与Activiti交互的最常见方式。中心起点是ProcessEngine，可以按照配置部分中所述的多种方式创建 。从ProcessEngine，您可以获取包含工作流/ BPM方法的各种服务。ProcessEngine和服务对象是线程安全的。因此，您可以为整个服务器保留对其中之一的引用。
![api.services.png](https://i.loli.net/2019/11/02/XmiH2ZJc5v3a8Sh.png)

```
ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();

RuntimeService runtimeService = processEngine.getRuntimeService();
RepositoryService repositoryService = processEngine.getRepositoryService();
TaskService taskService = processEngine.getTaskService();
ManagementService managementService = processEngine.getManagementService();
IdentityService identityService = processEngine.getIdentityService();
HistoryService historyService = processEngine.getHistoryService();
FormService formService = processEngine.getFormService();
DynamicBpmnService dynamicBpmnService = processEngine.getDynamicBpmnService();
```

ProcessEngines.getDefaultProcessEngine()会在首次调用时初始化并构建一个流程引擎，此后始终返回相同的流程引擎。可以使用ProcessEngines.init() 和正确创建和关闭所有流程引擎ProcessEngines.destroy()。

ProcessEngines类将扫描所有activiti.cfg.xml和activiti-context.xml文件。对于所有activiti.cfg.xml文件，将以典型的Activiti方式构建流程引擎：ProcessEngineConfiguration.createProcessEngineConfigurationFromInputStream(inputStream).buildProcessEngine()。对于所有activiti-context.xml文件，将以Spring方式构建流程引擎：首先创建Spring应用程序上下文，然后从该应用程序上下文中获取流程引擎。

所有服务都是无状态的。这意味着您可以轻松地在群集中的多个节点上运行Activiti，每个节点都访问同一数据库，而不必担心哪台计算机实际执行了先前的调用。无论在何处执行，对任何服务的任何调用都是幂等的。

## 各个节点的Service

### RepositoryService
当使用Activiti引擎时，RepositoryService可能是第一个需要的服务。这项服务提供运营管理和操纵deployments和process definitions。此处无需赘述，流程定义是BPMN 2.0流程的Java对应项。它代表了流程中每个步骤的结构和行为。A deployment是Activiti引擎中包装的单位。一个部署可以包含多个BPMN 2.0 xml文件和任何其他资源。一个部署中包含的内容的选择取决于开发人员。它的范围从单个流程BPMN 2.0 xml文件到整个流程包和相关资源（例如，部署hr-processes）可能包含与HR流程的一切）。在RepositoryService允许deploy这样的包。部署部署意味着将其上载到引擎，在这里将检查并解析所有进程，然后再将它们存储在数据库中。从那时起，系统便知道该部署，并且现在可以启动该部署中包括的任何过程。

此外，这项服务还可以
* 查询引擎已知的部署和流程定义。

* 挂起并激活整个部署或特定过程定义。挂起表示无法对其进行进一步的操作，而激活是相反的操作。

* 检索各种资源，例如部署中包含的文件或引擎自动生成的流程图。

* 检索流程定义的POJO版本，该版本可用于使用Java而不是xml自省流程。

### RuntimeService
它处理启动流程定义的新流程实例。如上所述，a process definition定义了过程中不同步骤的结构和行为。流程实例是这种流程定义的一种执行。对于每个流程定义，通常会同时运行许多实例。这RuntimeService也是用于检索和存储的服务process variables。这是特定于给定流程实例的数据，并且可以由流程中的各种构造使用（例如，专用网关通常使用流程变量来确定选择哪个路径来继续该流程）。

## 使用Activiti服务
如上所述，与Activiti引擎进行交互的方式是通过org.activiti.engine.ProcessEngine类实例公开的服务。以下代码段假定您具有有效的Activiti环境，即您可以访问有效的org.activiti.engine.ProcessEngine。如果您只想尝试下面的代码，则可以下载或克隆Activiti单元测试模板，将其导入IDE中，然后将testUserguideCode()方法添加到org.activiti.MyUnitTest单元测试中。

这个小教程的最终目标是要有一个能正常工作的业务流程，该流程模仿公司中一个简单的假期请求流程：
![api.vacationRequest.png](https://i.loli.net/2019/11/02/zswSxg3GMAHL5Iy.png)

### 部署流程
与静态数据相关的所有内容（例如流程定义）都可以通过RepositoryService访问。从概念上讲，每个此类静态数据都是Activiti引擎存储库的内容。

为了使Activiti引擎知道此过程，我们必须首先对其进行部署。部署意味着引擎将BPMN 2.0 xml解析为可执行文件，并为部署中包括的每个流程定义添加新的数据库记录。这样，当引擎重新启动时，它仍然会知道所有已部署的进程：
```
ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
RepositoryService repositoryService = processEngine.getRepositoryService();
repositoryService.createDeployment()
  .addClasspathResource("org/activiti/test/VacationRequest.bpmn20.xml")
  .deploy();

Log.info("Number of process definitions: " + repositoryService.createProcessDefinitionQuery().count());
```

### 启动流程实例
将流程定义部署到Activiti引擎后，我们可以从中启动新流程实例。对于每个流程定义，通常都有许多流程实例。流程定义是蓝图，而流程实例是其运行时执行。
与进程的运行时状态相关的所有内容都可以在RuntimeService中找到。有多种启动新流程实例的方法。在以下代码段中，我们使用在流程定义xml中定义的键来启动流程实例。我们还在流程实例启动时提供了一些流程变量，因为第一个用户任务的描述将在其表达式中使用它们。通常使用过程变量，因为它们为特定过程定义的过程实例赋予了含义。通常，流程变量使流程实例彼此不同。

```
//定义流程变量
Map<String, Object> variables = new HashMap<String, Object>();
variables.put("employeeName", "Kermit");
variables.put("numberOfDays", new Integer(4));
variables.put("vacationMotivation", "I'm really tired!");

RuntimeService runtimeService = processEngine.getRuntimeService();
ProcessInstance processInstance = runtimeService.startProcessInstanceByKey("vacationRequest", variables);

// Verify that we started a new process instance
Log.info("Number of process instances: " + runtimeService.createProcessInstanceQuery().count());
```

### 完成任务
该过程开始时，第一步将是用户任务。这是系统用户必须执行的步骤。通常，此类用户将拥有一个任务收件箱，其中列出了该用户需要完成的所有任务。以下代码段显示了如何执行这种查询：
```
// Fetch all tasks for the management group
TaskService taskService = processEngine.getTaskService();
List<Task> tasks = taskService.createTaskQuery().taskCandidateGroup("management").list();
for (Task task : tasks) {
  Log.info("Task available: " + task.getName());
}
```

要继续该流程实例，我们需要完成此任务。对于Activiti引擎，这意味着您需要complete执行任务。以下片段显示了如何完成此操作：
```
Task task = tasks.get(0);

Map<String, Object> taskVariables = new HashMap<String, Object>();
taskVariables.put("vacationApproved", "false");
taskVariables.put("managerMotivation", "We have a tight deadline!");
taskService.complete(task.getId(), taskVariables);
```

现在，流程实例将继续进行下一步。在此示例中，下一步允许员工填写可调整其原始休假请求的表格。员工可以重新提交休假请求，这将导致流程循环回到启动任务。

### 暂停或激活流程
可以暂停流程定义。流程定义被挂起时，无法创建新的流程实例（将引发异常）。可以通过以下方式暂停流程定义RepositoryService：
```
repositoryService.suspendProcessDefinitionByKey("vacationRequest");
try {
  runtimeService.startProcessInstanceByKey("vacationRequest");
} catch (ActivitiException e) {
  e.printStackTrace();
}
```
要重新激活流程定义，只需调用其中一种repositoryService.activateProcessDefinitionXXX方法。
也可以挂起流程实例。暂停后，该过程将无法继续执行（例如，完成任务会引发异常），并且不会执行任何作业（例如计时器）。可以通过调用runtimeService.suspendProcessInstance方法来挂起流程实例。通过调用runtimeService.activateProcessInstanceXXX方法来再次激活流程实例。

## 查询API
...待写

## 流程变量
=====待写

## 瞬态变量

