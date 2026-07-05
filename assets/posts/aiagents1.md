# How AI Agents Actually Work. Models, Tools, Memory and Everything in Between.

Typically, it all begins with a simple demonstration. At least that's how it started with me. You request a model to generate some code, summarize a paper, or you simply ask a question, and it replies almost instantly with something that looks surprisingly meaningful and accurate. Then, a second demo comes along where the "same" system opens files, runs terminal commands, searches through documentation, fixes bugs in multiple files, and continues for several minutes without losing track. This is the moment where the question changes from "What the model can do?" to "What changed between these two?".

The truth is that the model hasn’t changed anything.

Much of the confusion around AI agents comes from the way people discuss them as if they’re just one single thing... but they are not.
In practice, there’s no single *agent component*. There’s a model at the center, and it’s supported by a system that provides structure, sets constraints, offers tools, keeps memory, and allows for actions over time. If you study real implementations, you’ll find that an agent is more like a runtime environment built around a language model than a brain.

To really understand why this matters, it’s usfeul to begin with the model itself and be clear about what it’s actually doing.


## A Model is not trying to be an Agent

...at least for now.

At its core, a language model, no matter how advanced, is essentially a function that predicts the next word based on a sequence of words, or more sophisticated, a function that predicts the next token in a sequence. The *intelligence* that people notice, comes from factors like its size, the training data, and its design, but the basic mechanism stays the same. It doesn’t have any goals, it doesn’t remember the past interactions, and it doesn’t decide to keep working on a task after it gives an answer. It simply generates the most probable continuation of text based on the context it receives.
This is where we encounter a subtle but crucial limitation. When you ask a model a question, it provides an answer. If you request it to keep reasoning, it will. And if you change your mind during the chat, it can adapt. But all of this reactive behavior is based on the current ***context window***.

As per the [ibm article](https://www.ibm.com/think/topics/context-window) :

> The context window (or context length) of a large language model (LLM) is the amount of text, in tokens, that the model can consider or “remember” at any one time. A larger context window enables an AI model to process longer inputs and incorporate a greater amount of information into each output.

The context window can be thought of as the equivalent of its working memory. It's the models entire world. Everything it knows about the task, the user, previous steps, constraints, and even its own past outputs must fit within that box. Once the box is full, older information is either discarded or summarized and there is no persistent memory beyond what you explicitly bring back into the conversation. And this is an important limitation that answers the question some people may ask "Why can't the model just do that ***x***?" or maybe "Why it does not follow the previous instructions that I have given?"... and the honest truth is that ***x*** often needs persistence, tool usage, or stateful control flow. And none of that is available with just a basic model call.

This is why a model can seem strong but also insufficient. It can reason beautifully in one shot, but it struggles with multi-step tasks that rely on external systems or long-running processes. That’s where agents come into play.


## Agents - The moment a model becomes something more

An agent is what you get when you stop treating the model as an endpoint and start treating it as a decision-making step inside a loop.

So instead of 

![LLM architecture](assets/images/llm_arch.png)

we have something closer to

![AI agent architecture](assets/images/llm_arch2.png)

This loop marks a real transformation. The model is now making ***decisions*** about ***actions*** within a system that can track the outcomes and use that ***feedback*** for the next steps. These actions could be anything from calling a search API, executing code, reading a file, writing to a database, or querying a [vector store](https://www.ibm.com/docs/en/watsonx/saas?topic=autoai-choosing-vector-store-rag-experiment). The main point here is that the model isn't the one doing the actual work but instead is responsible for deciding what comes next. Once you see it this way, the agent concept becomes much less enigmatic. It evolves into a well-organized system where the model is just one piece of a control loop that includes tools, memory, and execution logic.

The simplest way to see the difference is to compare a chat model with a coding agent. A chat model can explain how to fix a bug. A coding agent can open the repository, locate the relevant file, inspect logs, run tests, apply a fix, and iterate until the tests pass. The difference is not the reasoning ability. The difference is access to tools and a loop (***itteration***) that allows repeated interaction with the environment. That loop is where everything interesting happens.


## Tools - What an Agent *can* do

Tools are the bridge between the mode's internal reasoning and external systems. Without tools, a model is trapped inside text. With tools, it can exit the matrix and start affecting the real world. A tool can be anything that takes structured input and returns output. A search engine query, a database call, a function that runs Python code, a file system operation, an API request to a cloud service, or even another agent! The key point is that the tool has a well-defined interface. The model doesn't need to know how to the tool functions internally. It only needs to know how to make a request. In practice, a tool works as a dialogue between the model and system around it. The model outputs a structured instruction like "call this function with these inputs" and then an external ***orchestration*** layer (or simply "***the orchestrator***") executes the call and returns the result to the model as updated ***context***. 
This creates an iterative loop: The model requests an action, the system executes it, and the result becomes new context. Over time, this allows the system to do work that cannot be completed in a single forward pass.

The key design decision here is separation of concerns. The model is not responsible for reliability, retries, error handling, or integration logic. That complexity lives in the orchestration layer. The model only decides intent. This separation matters because using a ***tool*** may introduce failure and the llm's are not designed to deal with it. API calls can fail, files may not exist, code can crash, network calls may time out... If the model were responsible for all of that logic, it would quickly become unstable. Instead, the system handles those failures and feeds them back to the model in a controlled way.

Once ***tools*** are in play, the agent starts to look more like a program rather than just a prompt.


## Skills - What an Agent *should* do, when and how

As systems evolve, devs  quickly realize that simply calling tools isn't enough. While models can use these tools, they often do so inconsistently without proper guidance. This is where the concept of ***skills*** comes into play, even though different frameworks have their own names for them (capabilities, functions, roles etc).

A skill is not magic. It’s more like a reusable pattern that brings together tool usage, instructions, and sometimes a bit of intermediate logic to create a more advanced capability. For example, a "debugging skill" might involve reading error logs, identify stack trace location, search documentation, inspect relevant code, and execute test commands in a specific order. A "code generation skill" typically contains understanding the requirement, searching existing patterns in repo, generate initial code, generate tests, run tests, fix errors, optimize or refactor, validate output ... etc. The important thing is that skills are not new model abilities. They are orchestration patterns that constrain how the model uses tools and how intermediate results are interpreted. I think of it like giving the model a structured workflow rather than a blank canvas. Instead of hoping it figures out the right sequence of steps, you shape the space of possible actions. This is also where agents start to feel more reliable. Without structure, a model might call the wrong tool, jump to conclusions too early or be unpredictable. With skills, the system guides it toward consistent behavior.

But skills are not enough if the system cannot remember what is already done. This where memory fits.


## Memory - What an Agent *keeps*, across time

Memory is one of the most misunderstood parts of agentic systems. People often assume memory is just a database attached to a model, but in practice it is more layered than that.
There are two types of memory, the ***short-term memory*** and the ***long-term memory***.

The short-term memory, is basically the context window. This holds the immediate conversation, recent tool outputs, and any intermediate reasoning. It is transient and limited.
The long-term memory, is typically implemented through retrieval systems. This might include vector databases, structured logs, or even simple key value stores. The agent writes information to memory explicitly, and later retrieves it when needed based on similarity or structured queries. The important distinction is that memory is not passive. The model does not "remember" things in the human sense. It retrieves them when the ***orchestrator*** decides that retrieval is useful, when a query triggers it or even when the user requests it in a prompt.

This is where many systems start to get really intriguing. Instead of sticking to a fixed prompt that aims to cover all bases, agents continuously write their state out and pull it back in as needed. For example, an agent dealing with a large codebase might keep track of summaries of the modules it has explored, the decisions it has made, and the errors it has encountered. Later, when it goes back to a file, it can easily retrieve that history and avoid repeating itself. Without memory, agents are forced to rediscover the world every time the context resets. With memory, they accumulate experience.

Still, having memory alone doesn't really fix the direction problem. The system also has to find a way to figure out what it wants to accomplish as it moves through multiple steps.


## Planning - What gives an Agent *continuity*

Planning is where things start to look intentional. Not because the model suddenly develops insight, but because the system forces structure onto its outputs. In simple setups, planning is just another prompt phase where the model is asked to break a goal into steps. In intermediate setups planing is created and reviewed with user instructions and is stored in memorey so that the orchestrator can follow it. In more advanced setups, planning is dynamic. The agent updates its plan as new information arrives, revises steps when tools fail, and reorders priorities based on intermediate results.

It’s not the complexity of the plan that’s important but that it externalize structure beyond a single forward pass of the model. A plan doesn’t need to be deep, clever, or optimal to be useful. Even a very simple plan like "First retrieve data → then analyze → then summarize" can be powerful. That is because it turns a single-shot model into a multi-step system.

> In ai agentic systems, a bad but explicit plan is often better than having no plan at all or implicit step-by-step improvisation inside one prompt.

Planning separates "what should happen" from "how it happens." And this separation matters a lot in agents because without planning the model mixes intent and execution in one pass, decisions are entangled with actions and long-horizon coherence is weak. With planning the system gains a control layer and the execution becomes structured. This way the tools can be used by the orchestrator more reliably.
A useful theoretical model is to think of planning as a simple state machine. The agent moves through states like understanding the goal, gathering information, executing tasks, validating results, and finishing. Each state influences which tools are available and which prompts are used. Without it the agents respond to each need individually without a sense of progress or direction.

Of course, planning introduces its own problems. Planning is not just "good structure" but more like a structured prediction about an uncertain future. This could make them become outdated quickly as new information emerges or conditions change (something that seems accurate initially may be invalid at a later stage). Also models can overcommit to early assumptions (commitment-bias or anchoring bias), making them less likely to revise an initial course of action even with contradictory evidence.
For these reasons, modern agent architectures typically treat plans as soft guidance rather than rigid execution graphs.
What this really mean is that agents instead of following a predetermined sequence of actions, they continuously adapt their plans based on observations, tool outputs, and changes in the environment. This balance between planning and replanning enables both long-term coherence and the flexibility required for robust execution.


## The Loop - What Holds Everything Together

At the center of every agent system is a loop. These ***iterations*** are quite simple in structure, but powerful in effect.
The loop typically looks like this:

![Agent Loop Diagram](assets/images/llm_arch3.png)

This loop does not execute itself, tt is driven by an ***orchestrator***. This layer of system code is in charge of determining when the model is called, how context is constructed, when tools are executed, and how results are fed back into the next iteration. The orchestrator is where most of the real engineering complexity lives. It decides when to stop, when to retry, how to handle errors, how to format context, how to prioritize memory, and how to manage tool responses. The model acts as a decision function inside this system, while everything else is just infrastructure around execution. This way, agents can scale beyond single interactions. The loop transforms a static model into a dynamic process that can operate for minutes or even hours, gradually improving its output through interaction with the environment.

However, it also introduces a new type of failure. Instead of a single bad response, you might experience drift, where the agent gradually strays from its initial goal due to accumulating mistakes. Or you could see looping behavior, where it keeps repeating similar actions without making any real progress. Handling these failure modes is one of the toughest challenges when it comes to building reliable agents.


## Limitations - What Still Holds Agents Back

Even with all these components, agents are far from perfect. Context windows still create hard limits on what can be actively processed at any time. When it comes to long-running tasks, we often need to summarize or compress memory, which can lead to losing some crucial details. Tool calls might fail without a sound or return results that are a bit ambiguous. Retrieval systems can sometimes dig up context that’s irrelevant or misleading. And the models themselves can occasionally hallucinate or misinterpret what the tools are outputting.

We also need to think about a deeper issue which is *reliability*. Agents don’t have guarantees of correctness. They operate probabilistically, which means that even well-designed systems will occasionally produce incorrect sequences of actions. This is why human oversight is essential, particularly in high-stakes environments such as production systems, financial workflows, or critical infrastructure.
Another limitation is *cost and latency*. Each loop iteration involves model inference, tool execution, and context updates. Complex agents can become expensive quickly, especially when they run multiple reasoning steps per task.

Ultimately, this leads us to a practical understanding. Agents are certainly powerful, but they are not *autonomous* in the sense people often imagine. Instead, they're more like assistants who follow structured workflows with some flexibility, rather than independent systems that could take over the roles of developers or engineers.


## How everything fits together

If you strip away the terminology, an AI agent is just a system that wraps a model in a controlled environment where it can act, observe, and persist state across time.

The model provides reasoning. Tools provide action. Memory provides continuity. Planning provides direction. The orchestration loop binds everything together and handles the messy reality of execution.
None of these parts are optional if you want something that behaves like an agent rather than a chatbot. Remove tools and it becomes passive. Remove memory and it becomes forgetful. Remove planning and it becomes reactive. Remove the loop and it becomes static.

The interesting part is not any single component, but how they interact all together. Small changes in orchestration can completely change the behavior of the system, even when the underlying model stays the same. And that is the part that tends to surprise most engineers when they first build or use one of these systems. The intelligence is not concentrated in a single place. It emerges from the structure around the model.

Once that mental model clicks, the next question becomes unavoidable: how do you actually build one of these systems from scratch, starting with nothing more than a model API and a few tools, and turn it into something that can reliably carry out real work over time.


### References

- [Ibm - Context Window](https://www.ibm.com/think/topics/context-window)
- [Choosing a vector store for a RAG experiment](https://www.ibm.com/docs)
- [aman.ai agent-skills](https://aman.ai/primers/ai/agent-skills/)
- [memory-in-agentic-ai](https://medium.com/@kishie-tech-ai/memory-in-agentic-ai-short-term-long-term-and-episodic-memory-51548a937131)
