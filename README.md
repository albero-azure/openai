## Building Interactive Enterprise Grade Applications with Open AI and Microsoft Azure

_Disclaimer. This is an opinion of the authors, and it does not necessarily reflect the recommendations or point of view of our current employer. This article is mainly focused on Data Engineering and Data Serving as part of the application development with Azure Open AI We are not covering AI models or components as such_ _😉_

_In this article we summarize our point of view and some early lessons learned when it comes to implementation of Azure Open AI backed applications. You will be guided through the main components that bring intelligent interactive applications into live and make them feedback-driven. While it is easier to visualize these considerations in the case of a conversational application, the details provided here are not limited to those scenarios and can be applied to other cases as well. This is a new field of course so please do not treat this article as a definite or prescriptive guidance – apply common sense where possible, experiment and test … ask Open AI if in doubt_ _😉_

Azure Open AI is clearly a hot topic now. Although AI models have been there for a while, it has got the attention of the general public only in a few weeks as a game changer in pretty much everything we do. Azure Open AI writes poems and generates oil-painted drawings, Copilot assists with presentations and code, Chat GPT helps us to understand problems and find solutions as well as verify our own understanding.

These are all great applications of general-purpose model and lots of companies started thinking on how to use Azure Open AI in their own applications and services. Starting from conversational banking and ending with deploying generated code directly into production.

The question we discuss with a vast number of customers these days is HOW? And in case with Azure Open AI and generative AI in general this "how" spans across multiple domains and goes far beyond technology.

The first thing which comes into our mind when talking about generative AI is the question of shared responsibility between AI Model, User and Application (Application Owners). We start this article by distinguishing responsibilities of the AI model from user responsibilities and continue with application responsibility before jumping to potential technical implementation.

**Responsibility Shared between User and AI Model**

Human beings usually initiate communication with some intent, in most cases such an intent shape the context and the style of communication. The context and style of communication are very important in conversational AI as they determine the effectiveness of the interaction between the AI system and the human user.

Here are a few reasons why:

·      The context of the conversation helps the AI system understand the intent of the user's message. This is crucial as it helps the AI system generate appropriate responses that are relevant to the user's needs.

·      Conversational AI systems are designed to engage users in a natural and free-flowing manner. By using an appropriate style of communication, the AI system can enhance engagement and create a more satisfying user experience.

In summary, the context and style of communication are critical components of conversational AI. By using appropriate language and tone AI systems can provide more effective and engaging interactions with users, ultimately leading to a more satisfying user experience.

The AI model is responsible for interpretation of your questions, performing basic validation and producing the responses (be it text, code, images, or something else). As mentioned earlier communication is a complex task involving different cultural aspects, personal experience where biases play a significant role.

If the user interacts with the model, obviously, it is up to the user to interpret the results obtained from this model. If the results produced by the model are unexpected or do not meet your criteria, you should first check for the style and context of the conversation. Adjusting one of these might be helpful.

Of course, to be able to adjust the context you should be allowed to do so by the application itself. In this sense we must distinguish between interactive applications and those with pre-defined use case and interaction flow (such as wizard-style applications).

What is less obvious is the verification of the results as it is up to the user to verify the results obtained from the AI model. These results can be cross-checked, verified by a simple search, or using one's knowledge and experience. In case verification fails, you should also review your style or context of conversation with the AI model.

One important aspect is that while interactive applications are typically equipped with feedback loop for those with pre-defined use case you need to ensure that some variation of a feedback loop is present. It can be arranged by direct feedback forms, voting for helpful answers and other techniques which are not covered in this article.

One last thing which is obviously in the hands of the user and falls entire under user responsibility is the utilization of these results.

![alt text](http://url/to/img.png)

**Role and Responsibilities of Enterprise Applications using AI Models**

In the last few weeks, we have witnessed some significant number of attempts of model misuse. People are attempting to get some information, code or pictures which can be considered unlawful or unethical in different circumstances. While model and platform (including Azure Open AI) adjust their mechanisms for filtering and control, people are becoming more creative.

For companies such a problem has another layer of complexity. Enterprises are typically under quite a strict regulation and policies preventing them from talking about competitors and providing outdated or misleading information, etc. Companies are there for making profits in a legal and ethical manner, so they are also concerned about customers getting equal, ethical, and effective treatment.

To answer some of these challenges we have created a small model (please keep in mind that this is mental exercise rather than definite guidance). It is used later in this article to help us define parts of the architecture for building enterprise applications with the use of AI.

![alt text](http://url/to/img.png)

In this model user of AI is responsible for his or her intent, context handling, hypothesis as well as asks and style of communication as inputs for the system. In cases where users were trying to deliberately misuse the model this is their responsibility in the first place. Of course, such cases should be taken into consideration and dealt with and for that we have two more areas of responsibility.

First is the responsibility of the application owners. They must ensure that ask posted by the user of generative AI application is proper in terms of the local laws, organizational policies and purpose of application. For instance, requests may be politely rejected if the user asks about competitors or tries to figure out some potential flaws in the processes. Alternative to rejecting an ask from user will be adjustments of an ask in accordance with organizational needs.

Depending on the request from the user it may require some additional capabilities. For instance, to reduce variability (control deviations in answers to different users) and ensure equality of treatment some of app owners may decide to use caching of answers or other means of answers reusability.

Obviously, since applications are modifying initial requests from users (according to laws, their own rules, and policies) they are responsible for logging of actions taken and reasoning behind such actions. In addition, application owners may require some basic forensics tools and processes.

Generative AI model is responsible for interpretation of an ask from / through applications, validation of an ask against its own rules (like validation which is available in Azure Open AI today) and providing a result back to the requestor (application).

After the request is processed by the model it becomes an application owner responsibility to validate results and adjust if needed. Such an adjustment may be required, for example, if an older or unsupported version of the product or service is mentioned, etc. Of course, it is an application task then to serve results back to the user.

In most of interactive use cases vast number of compliancy issues should be sorted out as a part of ask adjustment and orchestration rather than validation. During the validation and post processing two major things should occur:

- Lightweight validation of the results.
- Additional post processing such as enhancements with data, formatting, etc.

Ultimately user takes the responsibility for utilizing the results obtained from the application.

Using this model, we now can think on how such an approach may look when deployed on Azure.

**Potential Architecture for Enterprise Grade Applications based on Generative AI capabilities**

In our mental exercise the responsibility model in the communication process described above yields the following structural architecture.

**Principal Architecture**

Before diving into the description of the Architecture it is worth considering the goal of the process. Typically, the it is using all the relevant context from previous interactions or knowledge base to create a prompt for the endpoint generating an answer to the user. From this design perspective it can be defined as contextual API design. So, based on responsibility model and process described in the previous chapter, we have the following major capabilities which should be present in principal architecture:

- Caching and serving previous answers from cache.
- Filtering and modifying incoming requests in accordance with laws and company policies (including checking for illegal requests).
- Orchestrating request processing.
- Validating, filtering, and modifying responses to exclude potentially misleading or outdated information or enrich this information with data from other sources / other additional requests.
- Logging for all the actions and processing steps including modifications made, rationale behind these modifications and (ideally) supplementary links to laws, policies, etc.

Caching ( **Response Cache** ) plays an important role as it potentially allows us to reduce number of requests to the model and serve customers faster. Even more importantly, it can play an important role in reducing bias and ensuring equal treatment for various users. Of course, we will still have some similar answers for the same questions. Since AI models are compute-heavy caching of the requests will also add capability of sustainable AI model usage. Thus, we would need a capability (potentially an additional model – **Score Processor** ) which will allow us to assess best fitting results as well as check new results and their performance against once already cached.

Filtering of incoming requests ( **Request Modifier** ) protects company systems and processes from potential exposure of important information as well as potential fraudulent actions. It may also add additional level of clarity so model can serve customer request more precisely.

Orchestrating ( **Request Orchestrator** ) plays an important role in processing of complex requests. It may also be used to select an appropriate model based on precision of the request and other parameters. By the model here we can mean both one of the LLM models (such as Chat GPT or other Azure Open AI models) but also some proprietary or organization-owned models developed for specific purposes. Orchestration engine may also play a role in case we would like to allow users to perform actions / transactions through the same interface. In such case Orchestrator will be responsible for establishing communications with internal transactional systems.

Validation, filtering and (potentially) enriching response ( **Response Filter** ) allows us to control the quality and completeness of response. We should be aware that user will neither get the outdated information nor containing facts about competitors as well as simply wrong information. We would also like to potentially enrich our answer here by adding information from other models (including proprietary) or other systems. Such an enrichment process will also allow us to add mandatory information (such as legal disclaimers, etc.) **Response Filter** may also be used to enhance answers to the user with data from internal transactional or analytical systems.

One of the key capabilities we should have across all these components is to perform search either to match previous interactions or to identify the context. For this purpose, vector search may be a solution as it enables efficient and effective retrieval of relevant information. It includes representing data, such as text or images, as high-dimensional vectors in a vector space. These vectors capture semantic similarities and relationships between different data points. By using vector search, generative AI models can quickly locate similar vectors and generate responses or outputs based on the retrieved information, leading to more accurate and contextually relevant results.

Of course, we would need operations logging for each step of processing with all the history of changes, track of reasons for change, etc. In addition to this capability, we would need to have a set of forensics tools to be able to analyze the sequence of changes and perform validation of application and AI replies.

Naturally, user application will communicate with the whole system using some API ( **Application API Endpoint** ) where security, authentication as well as logic for communicating with user applications is implemented.

![alt text](http://url/to/img.png)

**Azure Components Used**

When it comes to components implementation and layout, we consider the further information. Most of the components in such a system should be optimized to work with data represented in a form of natural language. In addition, these components should have their own AI capabilities and be able to implement and run some skills. Typically, we use Search engines to fulfill these tasks.

We believe that the best choice for Response Cache, Score Processor, Request Modifier as well as Response filter will be Azure Cognitive Search. This is a PaaS service which has some great integration with predefined and custom AI models (in case they are used / should be used as a part of enhancements during post-processing or some pre-processing like labelling if it is required). It is also able to index native-language data and serve it using arbitrary queries.

_Note. During the final preparations for publishing this article we have come across recent article on_ [_Prompt Engineering_](https://lilianweng.github.io/posts/2023-03-15-prompt-engineering/) _from Lead Researcher in Azure Open AI Lillian Weng. All though it doesn't link directly to the topic of our article – it has a nice demonstration of how Search Engines can be used for prompting. Quite similar approach with self-prompting can be used for our purposes of serving replies from cache, filtering and request modification._

_This specific case it shows both how to achieve functionality mentioned and also emphasizes importance of usage data technologies tailored for natural language indexing and processing._

![alt text](http://url/to/img.png)

_Image Source_ [_Press et al. 2022_](https://arxiv.org/abs/2210.03350)

It is obvious that building a full new backend is not a simple task or one which someone want to perform. Luckily most of these tasks (including interaction with back end and all supporting activities) are already solved for you by developers of libraries. One of the most advanced libraries out there is [Langchain](https://python.langchain.com/en/latest/) which has all the necessary tools for you to effectively interact with you back end. Another useful library is coming from Microsoft itself and is called [Semantic Kernel](https://devblogs.microsoft.com/semantic-kernel/hello-world/).

There is no direct service match for Orchestrator at the moment so it should be implemented in a form of custom application and deployed on Azure AppService or Azure Kubernetes Service. API Endpoint can be also implemented as a custom application and exposed via Azure API Management.

![alt text](http://url/to/img.png)

Operation Logging can be implemented using a combination of Storage Engine for storing and Azure Data Explorer for querying and serving data. Of course, forensics tools, methods, and procedures as well as verification of correctness of user requests processing should be implemented as custom processing with your own tools of choice compatible with ADX.

**Workflow and Processing**

In case we implement this architecture, it will function as follows:

1)      **User** submits a request via application (for instance, chat bot).

2)      **AI API Endpoint Application** checks cache.

Such a cache can be implemented using databases supporting vector search. Selecting particular technology should be performed based on the access pattern and parameters of the vector.

a)      Operation is logged.

b)     In case request contains some follow up action to be performed (submitting a transaction, etc.) **Orchestrator** can perform such an action via synchronous or asynchronous exchange. We will cover this in more details later in this article.

c)      If there is a hit, response is returned.

3)     Else **AI API Endpoint Application** submits request to **Request Orchestrator** engine which performs initial evaluation (which precision is required / which extra rules to add).

4)     In case additional rules apply, **Orchestrator** submits request to **Request Modifier**. **Request Modifier** adds additional rules to the request to comply with laws, policies, etc. Such modifications may be required in case incoming request can potentially be considered as a violation of organizational policies or process rules or as a harmful or fraudulent action in the context of particular application. It also may imply specific meaning of the terms, abbreviations, etc.

This filtering does not necessary cover general cases as Open AI models probe against them.

Depending on complexity of the use case and request modifications are performed using some predefined rules or based on the organizational knowledge base created and maintained separately for different Azure Open AI instances.

a)      All modifications made are logged by **Request Modifier**.

5)      **Orchestrator** evaluates request to choose appropriate model (it also acts as a throttling engine and load management).

6)      **Orchestrator** submits a request to Azure Open AI. Some of the additional basic request checks (such as ethical checks, abuse, etc.) are pre-built into Azure Open AI models and performed by Azure Open AI model by default. And once result is received, it triggers a postformatting, validation and enhancement procedures implemented in **Response Filter**.

7)     Response is logged.

8)      **Response Filter** approves response or removes parts which are inappropriate / meet certain criteria (like unsupported product mentioned, etc.). Response Filter can also enrich responses with other results, data or submit additional requests to the model (add lineage, for instance or data quality metrics, etc.)

a)      Some minor but important tasks can include formatting aspects such as adding legal disclaimers, feedback and contact details, etc.

b)     One of the important aspects of the response filter is that in order to enhance replies with data and additional formatting we might need to split prompts into structured set and address them one-by-one.

c)      Obviously, in order to perform post processing rules as well as additional reply caches should be stored in **Response Filter**. The latter can be achieved by using various approaches depending on the complexity of the rules and actual tasks performed during this stage. At this stage we can also connect to backend data systems to enhance our reply with additional data points valuable for the user.

9)     Result is returned to **AI API Endpoint**.

a)      And served back to the **End User Application**.

10)   **End User Application** tracks user communication with the model (actions / resubmit of request / quality of follow up questions) and serves back to **AI API Endpoint**.

**11)  Responses and scoring are recorded and evaluated by Score Processor.**

12)  The best scored results are cached by **Score Processor** in **Response Cache**.

13)  Responses are feeded back to the model via **Reinforcement Learning Feedback Loop**.

![alt text](http://url/to/img.png)

**Some Notes on Implementing Orchestration**

A common pattern in integrating LLM in complex application scenarios relies on the concept of Orchestration of available skills and integrations (often referred to as "being Agentic" or creating a "Plan with the available skills").

One way or another, the foundational idea is to create a dynamic environment where AI model can enable a more goal-orientated approach by selecting the appropriate actions, considering the user's input, the availability of APIs, data sources, and potentially other boundaries to ground the answers to your enterprise data or policies.

To implement such a pattern, we always start with a **Request** (Ask) in mind that should be answered by a dynamically informed outcome the **Response** (Get), relying on the orchestration capabilities of the selected framework.

![alt text](http://url/to/img.png)

A **Request** triggers the orchestrator or **Kernel** that can use the **Planner** to break down a user interaction in steps available to the environment as **Resources** , like **Skills** , **Memories** or **Connectors**.

- **Skills** are enabling the execution of a function to perform a specific **Step**.
- **Memories** can provide additional context to fully covers any specific **Request.**
- **Connectors** provide out-of-the-box integration with different tools.

Thus, combining Skills, Memories and Connectors is possible to build quite a sophisticated orchestration and processing engines around LLM and other AI models. Such combination will allow you to build the logic of your **Orchestrator**.

Basically, you can build **Request Modifier** and **Response Filter** using a similar approach but depending on a case these two components may require additional concepts (such as **Rules** ). In the simplest scenarios both components may be introduced as a part of Orchestrator but it makes more sense to implement them as separate services.

**Connect AI-powered user facing apps to your transactional and analytical systems**

For the sake of simplicity we would refer components enabling interactions between End-User Applications and Azure Open AI or other models as a "Semantic Kernel". As stated above, such a group of the components may be implemented using either Microsoft Semantic Kernel, Langchain or other sets of libraries out there. So, higher level architecture may look like this.

![alt text](http://url/to/img.png)

It is quite obvious that AI backed applications do not exist in a void. We need / would like to connect it to the wider Data ecosystem for a variety of reasons:

1. Supply our analytical systems with data on interactions with users and their feedback on data utilization.
2. Provide our users with an ability to perform actions using the same interactive conversational interface.
3. Supply our users with data backing up statements made by generative AI (which is especially helpful in Enterprise applications). It might be also helpful in some cases where user expects application to communicate if there is any data relevant (booked flight, existing profile, etc.)

So, in order to connect such applications with wider ecosystem we can use several extension points.

![alt text](http://url/to/img.png)

**Collecting data on user behavior**

The first and most obvious one is how we supply our analytical systems with data from interactions with our users. On the diagram above we are performing quite some amount of logging operations. These logs in conjunctions with the logged results of scoring may be used later by our analytical systems.

This data can be instrumental in case we would like to feed factual suggestions to the users of our applications and services (like next best action, etc.)

**Connecting to transactional systems to perform actions**

An ability to supply users with a way to perform actions can be achieved by allowing the **Orchestrator** to write into an Action Queue (such as Azure Service Bus) or stream engine (like Azure Event Hub). You will also need a processor application which will perform the same actions as backend of your normal transactional applications.

You will also require a processor to have some form of notification so users can get confirmation that their request is processed and that it comes through AI App infrastructure. Such a notification process can utilize same Queue / Stream infrastructure. Obviously, these requests should also be logged.

Alternatively, orchestration engine can execute actions immediately by sending requests to application APIs (in case they exist) or submitting transactions directly to the transactional database. In such scenarios there should be separate modules within Orchestrator responsible to synchronous path. These same modules should also be equipped with validation procedures as well as rollback procedures.

Since user is communicating with the platform in interactive manner, executing synchronous calls might be risky in case no effective rollback procedure is present in the database or API.

Good example of this type of interaction will be booking or rebooking a flight ticket where users can not only ask to perform actions on their behalf but can also reconsider during the same conversation or start different flow (like asking for alternative options after the action was initiated).

![alt text](http://url/to/img.png)

**Enhancing responses with factual data from analytical systems**

Last but not least, is enhancing responses from AI models with factual data. This can be achieved by using the following approach.

Analytical or transactional data relevant for the use case and stored in Data Lake or Database can be indexed using Azure Cognitive Search (as a part of Response Filter or some additional component). There should be a set of prompts associated with the data which can be used to retrieve the data relevant for the request we are post-processing. Indexed Data can be added to the response. This approach is more relevant for data which is not changing often or is immutable in the context of user interactions (such as flights, routes, fares, etc.)

Ideally, analytical systems will be exposed as a number of properly governed and controlled Data Products. These Data Products may or may not be a part of Data Mesh implementation but they should represent each entity exposed to the user in consistent and coherent manner.

Let us think of another example. For instance, our user asks application what the current itinerary is recorded in the system or which contact details, loyalty programs, etc. are used. In such a situation we need to quickly retrieve this information from our internal systems. Quite obviously, building an index on top of quickly changing data is not an optimal solution. Instead, we can perform a lookup in some relational or non-relational database.

The challenge here is that in order to perform a lookup we need to have appropriate key as we cannot simply scan the whole dataset. Such a key may be found in internal systems and added as a session parameter during user authentication. The problem is that we should be prepared to perform lookups in various collections or tables (orders, itineraries, profiles, etc.) Thus, logically we have to have and maintain mapping table with one main key (which will be used as a session parameter) mapped to keys of other relevant collections. In this case our lookup will be in fact two lookups performed by Response Filter. First lookup against mapping table and second – against data table.

![alt text](http://url/to/img.png)

One last thing data should be also supplied with the feedback mechanisms. There might be a certain level of complexity associated with synchronizing data and responses from generative AI. Such gap could be potentially filled by introducing some of the specialized feedback mechanisms combining adjustment of prompts on Request Modifier and Response Filter as well as direct feedback regarding data quality, etc. At this moment in time working with data may require human intervention.

This is it, hope you enjoyed the article! Good luck with building your smart applications!
