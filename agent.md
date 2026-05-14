from dotenv import load_dotenv

from langchain.agents import create_agent

from langchain.chat_models import init_chat_model

from langchain_core.messages import HumanMessage

from langchain_core.prompts import ChatPromptTemplate, MessagesPlaceholder

from langgraph.checkpoint.memory import MemorySaver

from langchain.agents.middleware import AgentMiddleware

from langchain.agents.middleware.types import ModelRequest, ModelResponse

from custom_tools import query_order, create_ticket, search_knowledge_base

import time

import os

  
  

load_dotenv()

DEEPSEEK_API_KEY = os.getenv("DEEPSEEK_API_KEY")

llm = init_chat_model(

    model="deepseek-chat",

    api_key=DEEPSEEK_API_KEY,

    base_url="https://api.deepseek.com"

)

  

tools = [query_order, create_ticket, search_knowledge_base]

  

system_instruction = """你是「智购商城」的 AI 客服助理。

  

## 行为规则

1. 始终保持礼貌和专业

2. 先尝试用知识库回答；知识库没有再查订单；都解决不了才建工单

3. 永远不要编造信息—不确定就说"我帮你转接人工客服"

4. 回答简洁，不超过 3 句话

5. 涉及退款/投诉时，优先级设为 high

  

## 可用工具

- search_knowledge_base：搜索产品知识库

- query_order：查询订单状态

- create_ticket：创建人工客服工单"""

  

class LoggingMiddleware(AgentMiddleware):

    def wrap_model_call(self, request: ModelRequest, handler) -> ModelResponse:

        start = time.time()

        response = handler(request)  # 调用下一个中间件或模型

        print(f"[LOG] LLM 调用耗时 {time.time() - start:.2f}s")

        return response

  

checkpointer = MemorySaver()

  

agent = create_agent(

    model=llm,                       # ① 直接传入 ChatOpenAI 实例

    tools=[query_order, create_ticket, search_knowledge_base], # ② 工具列表

    system_prompt=system_instruction,            # ③ 参数名是 system_prompt

    middleware=[LoggingMiddleware()],# ④ 注意实例化，加括号

    checkpointer=checkpointer,       # ⑤ 检查点保存器

    # max_iterations=5               # ⚠️ 1.0 版本已移除该参数，如需控制循环次数，请使用中间件

)

  

config  = {"configurable":{"thread_id":"customer-session-123"}}

  

result1 = agent.invoke(

    {"messages": [HumanMessage("你们的退款政策是什么")]},

    config=config

)

print(result1.get("messages")[-1].content)