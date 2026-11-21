> ## Documentation Index
> Fetch the complete documentation index at: https://docs.camel-ai.org/llms.txt
> Use this file to discover all available pages before exploring further.

# Memory

> [card] What is Memory?
>
> The CAMEL Memory module gives your AI agents a flexible, persistent way to **store, retrieve, and manage information**, across any conversation or task.
>
> With memory, agents can maintain context, recall key details from previous chats, and deliver much more coherent, context-aware responses.
>
> Memory is what transforms a “chatbot” into a smart, adaptable assistant.

> [card] Basic Usage: LongtermAgentMemory
>
> This is the fastest way to enable true memory for your agents: store, retrieve, and leverage context across interactions.
>
> > [tab] Python Example
> >
> > ```python theme={"system"}
> > from camel.memories import (
> >     ChatHistoryBlock,
> >     LongtermAgentMemory,
> >     MemoryRecord,
> >     ScoreBasedContextCreator,
> >     VectorDBBlock,
> > )
> > from camel.messages import BaseMessage
> > from camel.types import ModelType, OpenAIBackendRole
> > from camel.utils import OpenAITokenCounter
> >
> > # Initialize the memory
> > memory = LongtermAgentMemory(
> >     context_creator=ScoreBasedContextCreator(
> >         token_counter=OpenAITokenCounter(ModelType.GPT_4O_MINI),
> >         token_limit=1024,
> >     ),
> >     chat_history_block=ChatHistoryBlock(),
> >     vector_db_block=VectorDBBlock(),
> > )
> >
> > # Create and write new records
> > records = [
> >     MemoryRecord(
> >         message=BaseMessage.make_user_message(
> >             role_name="User",
> >             meta_dict=None,
> >             content="What is CAMEL AI?",
> >         ),
> >         role_at_backend=OpenAIBackendRole.USER,
> >     ),
> >     MemoryRecord(
> >         message=BaseMessage.make_assistant_message(
> >             role_name="Agent",
> >             meta_dict=None,
> >             content="CAMEL-AI.org is the 1st LLM multi-agent framework and "
> >                     "an open-source community dedicated to finding the scaling law "
> >                     "of agents.",
> >         ),
> >         role_at_backend=OpenAIBackendRole.ASSISTANT,
> >     ),
> > ]
> > memory.write_records(records)
> >
> > # Get context for the agent
> > context, token_count = memory.get_context()
> >
> > print(context)
> > print(f"Retrieved context (token count: {token_count}):")
> > for message in context:
> >     print(f"{message}")
> > ```
>
> > [tab] Output
> >
> > ```markdown theme={"system"}
> > >>> Retrieved context (token count: 49):
> > {'role': 'user', 'content': 'What is AI?'}
> > {'role': 'assistant', 'content': 'AI refers to systems that mimic human intelligence.'}
> > ```

> [card] Integrate Memory into a ChatAgent
>
> Assign memory to any agent and watch your AI recall and reason like a pro.
>
> > [tab] Python Example
> >
> > ```python theme={"system"}
> > from camel.agents import ChatAgent
> >
> > # Define system message for the agent
> > sys_msg = BaseMessage.make_assistant_message(
> >     role_name='Agent',
> >     content='You are a curious agent wondering about the universe.',
> > )
> >
> > # Initialize agent
> > agent = ChatAgent(system_message=sys_msg)
> >
> > # Set memory to the agent
> > agent.memory = memory
> >
> > # Define a user message
> > usr_msg = BaseMessage.make_user_message(
> >     role_name='User',
> >     content="Tell me which is the 1st LLM multi-agent framework based on what we have discussed",
> > )
> >
> > # Sending the message to the agent
> > response = agent.step(usr_msg)
> >
> > # Check the response (just for illustrative purpose)
> > print(response.msgs[0].content)
> > ```
>
> > [tab] Output
> >
> > ```markdown theme={"system"}
> > >>> CAMEL AI is recognized as the first LLM (Large Language Model) multi-agent framework. It is an open-source community initiative focused on exploring the scaling laws of agents, enabling the development and interaction of multiple AI agents in a collaborative environment. This framework allows researchers and developers to experiment with various configurations and interactions among agents, facilitating advancements in AI capabilities and understanding.
> > ```

> [card] Core Components of CAMEL Memory
>
> > [accordion] MemoryRecord
> >
> > **What it is:**
> > The basic data unit in CAMEL’s memory system—everything stored/retrieved flows through this structure.
> >
> > **Attributes:**
> >
> > * **message**: The content, as a `BaseMessage`
> > * **role\_at\_backend**: Backend role (`OpenAIBackendRole`)
> > * **uuid**: Unique identifier for the record
> > * **extra\_info**: Optional metadata (key-value pairs)
> >
> > **Key methods:**
> >
> > * `from_dict()`: Build from a Python dict
> > * `to_dict()`: Convert to dict for saving/serialization
> > * `to_openai_message()`: Transform into an OpenAI message object
>
> > [accordion] ContextRecord
> >
> > **What it is:**
> > Result of memory retrieval from `AgentMemory`, scored for context relevance.
> >
> > **Attributes:**
> >
> > * **memory\_record**: The original `MemoryRecord`
> > * **score**: How important/relevant this record is (float)
>
> > [accordion] MemoryBlock (Abstract Base Class)
> >
> > **What it is:**
> > The core “building block” for agent memory, following the Composite design pattern (supports tree structures).
> >
> > **Key methods:**
> >
> > * `write_records()`: Store multiple records
> > * `write_record()`: Store a single record
> > * `clear()`: Remove all stored records
>
> > [accordion] BaseContextCreator (Abstract Base Class)
> >
> > **What it is:**
> > Defines strategies for generating agent context when data exceeds model limits.
> >
> > **Key methods/properties:**
> >
> > * `token_counter`: Counts message tokens
> > * `token_limit`: Max allowed tokens in the context
> > * `create_context()`: Algorithm for building context from chat history
>
> > [accordion] AgentMemory (Abstract Base Class)
> >
> > **What it is:**
> > Specialized `MemoryBlock` for direct agent use.
> >
> > **Key methods:**
> >
> > * `retrieve()`: Get `ContextRecord` list
> > * `get_context_creator()`: Return the associated context creator
> > * `get_context()`: Return properly sized chat context

> [card] Memory Block Implementations
>
> > [accordion] ChatHistoryBlock
> >
> > **What it does:**
> > Stores and retrieves recent chat history (like a conversation timeline).
> >
> > **Initialization:**
> >
> > * `storage`: Optional (default `InMemoryKeyValueStorage`)
> > * `keep_rate`: Historical message score weighting (default `0.9`)
> >
> > **Methods:**
> >
> > * `retrieve()`: Get recent chats (windowed)
> > * `write_records()`: Add new records
> > * `clear()`: Remove all chat history
> >
> > **Use Case:**
> > Best for maintaining the most recent conversation flow/context.
>
> > [accordion] VectorDBBlock
> >
> > **What it does:**
> > Uses vector embeddings for storing and retrieving information based on semantic similarity.
> >
> > **Initialization:**
> >
> > * `storage`: Optional vector DB (`QdrantStorage` by default)
> > * `embedding`: Embedding model (default: `OpenAIEmbedding`)
> >
> > **Methods:**
> >
> > * `retrieve()`: Get similar records based on query/keyword
> > * `write_records()`: Add new records (converted to vectors)
> > * `clear()`: Remove all vector records
> >
> > **Use Case:**
> > Ideal for large histories or when semantic search is needed.
>
> **Difference** `string`
>
> **Key Differences:**
>
> * **Storage:** ChatHistoryBlock uses key-value storage. VectorDBBlock uses vector DBs.
> * **Retrieval:** ChatHistoryBlock retrieves by recency. VectorDBBlock retrieves by similarity.
> * **Data:** ChatHistoryBlock stores raw messages. VectorDBBlock stores embeddings.

> [card] Agent Memory Implementations & Advanced Usage
>
> > [card] ChatHistoryMemory
> >
> > **What is it?**
> > An **AgentMemory** implementation that wraps `ChatHistoryBlock`.
> > **Best for:** Sequential, recent chat context (simple conversation memory).
> >
> > **Initialization:**
> >
> > * `context_creator`: `BaseContextCreator`
> > * `storage`: Optional `BaseKeyValueStorage`
> > * `window_size`: Optional `int` (retrieval window)
> >
> > **Methods:**
> >
> > * `retrieve()`: Get recent chat messages
> > * `write_records()`: Write new records to chat history
> > * `get_context_creator()`: Get the context creator
> > * `clear()`: Remove all chat messages
>
> > [card] VectorDBMemory
> >
> > **What is it?**
> > An **AgentMemory** implementation that wraps `VectorDBBlock`.
> > **Best for:** Semantic search—find relevant messages by meaning, not just recency.
> >
> > **Initialization:**
> >
> > * `context_creator`: `BaseContextCreator`
> > * `storage`: Optional `BaseVectorStorage`
> > * `retrieve_limit`: `int` (default `3`)
> >
> > **Methods:**
> >
> > * `retrieve()`: Get relevant messages from the vector DB
> > * `write_records()`: Write new records and update topic
> > * `get_context_creator()`: Get the context creator
>
> > [card] LongtermAgentMemory
> >
> > **What is it?**
> > Combines **ChatHistoryMemory** and **VectorDBMemory** for hybrid memory.
> > **Best for:** Production bots that need both recency & semantic search.
> >
> > **Initialization:**
> >
> > * `context_creator`: `BaseContextCreator`
> > * `chat_history_block`: Optional `ChatHistoryBlock`
> > * `vector_db_block`: Optional `VectorDBBlock`
> > * `retrieve_limit`: `int` (default `3`)
> >
> > **Methods:**
> >
> > * `retrieve()`: Get context from both history & vector DB
> > * `write_records()`: Write to both chat history & vector DB
> > * `get_context_creator()`: Get the context creator
> > * `clear()`: Remove all records from both memory blocks
>
> > [card] Mem0Storage Integration
> >
> > Add [Mem0](https://mem0.ai/) for cloud-based memory with automatic sync.
> >
> > **Initialization Params:**
> >
> > * `api_key`: (optional) Mem0 API authentication
> >
> > * `agent_id`: (optional) Agent association
> >
> > * `user_id`: (optional) User association
> >
> > * `metadata`: (optional) Dict of metadata for all memories
> >
> > > [tab] Python Example
> > >
> > > ```python theme={"system"}
> > > from camel.memories import ChatHistoryMemory, ScoreBasedContextCreator
> > > from camel.storages import Mem0Storage
> > > from camel.types import ModelType
> > > from camel.utils import OpenAITokenCounter
> > >
> > > memory = ChatHistoryMemory(
> > >     context_creator=ScoreBasedContextCreator(
> > >         token_counter=OpenAITokenCounter(ModelType.GPT_4O_MINI),
> > >         token_limit=1024,
> > >     ),
> > >     storage=Mem0Storage(
> > >         api_key="your_mem0_api_key",  # Or set MEM0_API_KEY env var
> > >         agent_id="agent123"
> > >     ),
> > >     agent_id="agent123"
> > > )
> > > # ...write and retrieve as usual...
> > > ```
> >
> > > [tab] Output
> > >
> > > ```markdown theme={"system"}
> > > >>> Retrieved context (token count: 49):
> > > {'role': 'user', 'content': 'What is CAMEL AI?'}
> > > {'role': 'assistant', 'content': 'CAMEL-AI.org is the 1st LLM multi-agent framework and an open-source community dedicated to finding the scaling law of agents.'}
> > > ```
> >
> > **Why use this?**
> >
> > * Cloud persistence of chat history
> > * Simple setup and config
> > * Sequential retrieval—conversation order preserved
> > * Syncs across sessions automatically
> >
> > **Use when:** you need reliable, persistent chat history in the cloud (not advanced semantic search).

> [card] Advanced Topics
>
> > [card] Customizing Context Creator
> >
> > You can subclass `BaseContextCreator` for advanced control.
> >
> > > [tab] Python Example
> > >
> > > ```python theme={"system"}
> > > from camel.memories import BaseContextCreator
> > >
> > > class MyCustomContextCreator(BaseContextCreator):
> > >     @property
> > >     def token_counter(self):
> > >         # Implement your token counting logic
> > >         return
> > >
> > >     @property
> > >     def token_limit(self):
> > >         return 1000
> > >
> > >     def create_context(self, records):
> > >         # Implement your context creation logic
> > >         pass
> > > ```
>
> > [card] Customizing VectorDBBlock
> >
> > You can use custom embeddings or vector DBs.
> >
> > > [tab] Python Example
> > >
> > > ```python theme={"system"}
> > > from camel.embeddings import OpenAIEmbedding
> > > from camel.memories import VectorDBBlock
> > > from camel.storages import QdrantStorage
> > >
> > > vector_db = VectorDBBlock(
> > >     embedding=OpenAIEmbedding(),
> > >     storage=QdrantStorage(vector_dim=OpenAIEmbedding().get_output_dim()),
> > > )
> > > ```
>
> > [card] Performance Considerations
> >
> > * For production, use persistent storage (not just in-memory).
> > * Optimize your context creator for both relevance and token count.


This documentation is built and hosted on [Mintlify](https://mintlify.com), a developer documentation platform.