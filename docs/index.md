# DataPilot

DataPilot 是一个面向金融等多个领域，进行数据分析和知识管理的智能代理系统，支持自动化数据表、工具、知识库的注册、管理和查询，集成了知识图谱、社区报告、脚本关系抽取等多种功能，适用于企业级数据治理和智能分析场景。

## 主要特性

- **知识库管理**：支持 Markdown 文档分块、嵌入、检索，自动构建知识图谱。
- **元数据管理**：自动注册数据表，抽取列描述、数据类型、可连接关系，支持 MySQL、本地 CSV。
- **工具管理**：自动注册工具，抽取工具与数据表/列/文件的输入输出关系。
- **可视化**：支持知识图谱和元数据关系的 HTML 可视化。

## 快速开始

1. **运行主流程**

    推荐使用如下命令启动（入口为 [`datapilot/workflow.py`](datapilot/workflow.py)）：

    ```bash
    # PROJECT_ROOT为项目所在路径
    cd $PROJCET_ROOT
    export $PATHPATH=$PROJECT_ROOT/datapilot
    python -m datapilot.workflow
    ```

2. **用户指令**

    在 `workflow.py` 文件，在 `__main__` 部分修改 `user_query` 变量进行交互

    ```python
    if __name__ == "__main__":
        user_query = "请输入您的分析任务"
        # 示例1：数据分析任务
        # user_query = "分析一下各个区域的消费水平，并生成一份报告。所有需要保存的文件都放在 /data1/agent/machengyuan/intermediate_res3 路径下"
        
        # 示例2：菜谱搜索任务
        # user_query = "帮我搜索一下番茄炒鸡蛋怎么做"
        asyncio.run(run_agent_workflow(user_query))
    ```


## 整体流程

以下节点通过**Router**进行如图所示的流转控制，并且支持多轮反思与反馈

1. **Coordinator**：接收用户需求，分析并分发任务。
2. **Perceptor**：感知可用数据和工具，补充上下文。
3. **Planner** ：根据感知模块制定具体任务计划。
4. **Invoker** 执行计划，调用工具，收集结果。

## 核心组件

### LangGraphCoordinator

- **职责**：实现**Coordinator**模块作为智能体的“大脑”，负责接收用户需求，分析当前工作流状态，并且可以进行反思（reflect）。
- **核心方法**：
    - `reflect(state: BaseState)`: 反思当前状态，分析需求与上下文。

### LangGraphPerceptor

- **职责**：负责“感知”当前可用的数据和工具，可从数据库、知识库、工具服务（如 MCP 服务、函数等）等内容中检索相关可用信息。
- **核心方法**：
    - `search_knowledge(query: str)`: 检索知识库，获取背景知识。
    - `search_database(query: str, knowledge: str)`: 检索数据库和工具，获取可用数据。
    - `get_mcp_tools()`:获取可用MCP服务
    - `get_available_tools()`: 获取当前可用工具列表。

### LangGraphPlanner

- **职责**：根据协调器和感知器提供的信息，制定具体的任务执行计划，并且并对计划进行可行性验证。
- **核心方法**：
    - `generate_plan(state: BaseState)`: 生成任务执行计划。
    - `validate_plan(plan: str, state: BaseState)`: 验证计划的合理性。

### LangGraphInvoker

- **职责**：负责进行工具调用并对结果进行收集。后续可扩展为支持反思和用户反馈。
- **核心方法**：
    - `generate_tool_calls(state: BaseState)`: 生成工具调用列表。
    - `execute_tool_calls(tool_calls: List[Dict])`: 执行工具调用，返回结果。



## LangGraphRouter

- **职责**：负责对整个工作节点的流转进行调度。

