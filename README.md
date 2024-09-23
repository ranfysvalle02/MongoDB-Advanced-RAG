
# RAG Based Chat-bot using Langchain and MongoDB Atlas
This starter template implements a Retrieval-Augmented Generation (RAG) chatbot using LangChain and MongoDB Atlas. RAG combines AI language generation with knowledge retrieval for more informative responses. LangChain simplifies building the chatbot logic, while MongoDB Atlas' Vector database capability provides a powerful platform for storing and searching the knowledge base that fuels the chatbot's responses.

## Setup 

### Step 1: Local Atlas Environment 

1. **Pull the Docker Image:**
   * **Latest Version:**
     ```bash
     docker pull mongodb/mongodb-atlas-local
     ```
   * **Specific Version:**
     ```bash
     docker pull mongodb/mongodb-atlas-local:<tag>
     ```
     Replace `<tag>` with the desired version.

2. **Run the Database:**
   ```bash
   docker run -p 27017:27017 mongodb/mongodb-atlas-local
   ```
   This command runs the Docker image, exposing port 27017 on your host machine to connect to the database.

3. **Connect to the Database:**
   Use `mongosh` to connect to the database:
   ```bash
   mongosh "mongodb://localhost/?directConnection=true"
   ```

## Creating an Atlas Vector Search Index Programmatically with mongosh

### Steps:

1. **Connect to the Atlas Cluster:**
   Use `mongosh` to connect to your Atlas cluster. Refer to the official documentation for more details on connecting via `mongosh`.

2. **Switch to the Database:**
   Select the database that contains the collection you want to index:

   ```javascript
   use <databaseName>
   ```

3. **Create the Index:**
   Execute the following command:

   ```javascript
   db.<collectionName>.createIndex(
       { "<fieldToIndex>": "vector" },
       {
           name: "<indexName>",
           dimensions: <numberOfDimensions>
       }
   );
   ```

   Replace the placeholders with your specific values:
   * **`<databaseName>`:** The name of your database.
   * **`<collectionName>`:** The name of your collection.
   * **`<fieldToIndex>`:** The field you want to index.
   * **`<indexName>`:** The name of your index (optional, defaults to `vector_index`).
   * **`<numberOfDimensions>`:** The number of vector dimensions.

### Example:
```javascript
use sample_mflixdb;
db.movies.createIndex(
    { "plot_vector": "vector" },
    {
        name: "plot_vector_index",
        dimensions: 128
    }
);
```

This example creates a vector search index on the `plot_vector` field of the `movies` collection in the `sample_mflix` database.

- Once completed, head to the QnA section to start asking questions based on your trained data, and you should get the desired response.

  ![image](https://github.com/utsavMongoDB/MongoDB-RAG-NextJS/assets/114057324/c76c8c19-e18a-46b1-834a-9a6bda7fec99)



## Reference Architechture 

![image](https://github.com/mongodb-partners/MongoDB-RAG-Vercel/assets/114057324/3a4b863e-cea3-4d89-a6f5-24a4ee44cfd4)


This architecture depicts a Retrieval-Augmented Generation (RAG) chatbot system built with LangChain, OpenAI, and MongoDB Atlas Vector Search. Let's break down its key players:

- **PDF File**: This serves as the knowledge base, containing the information the chatbot draws from to answer questions. The RAG system extracts and processes this data to fuel the chatbot's responses.
- **Text Chunks**: These are meticulously crafted segments extracted from the PDF. By dividing the document into smaller, targeted pieces, the system can efficiently search and retrieve the most relevant information for specific user queries.
- **LangChain**: This acts as the central control unit, coordinating the flow of information between the chatbot and the other components. It preprocesses user queries, selects the most appropriate text chunks based on relevance, and feeds them to OpenAI for response generation.
- **Query Prompt**: This signifies the user's question or input that the chatbot needs to respond to.
- **Actor**: This component acts as the trigger, initiating the retrieval and generation process based on the user query. It instructs LangChain and OpenAI to work together to retrieve relevant information and formulate a response.
- **OpenAI Embeddings**: OpenAI, a powerful large language model (LLM), takes centre stage in response generation. By processing the retrieved text chunks (potentially converted into numerical representations or embeddings), OpenAI crafts a response that aligns with the user's query and leverages the retrieved knowledge.
- **MongoDB Atlas Vector Store**: This specialized database is optimized for storing and searching vector embeddings. It efficiently retrieves the most relevant text chunks from the knowledge base based on the query prompt's embedding. These retrieved knowledge nuggets are then fed to OpenAI to inform its response generation.


This RAG-based architecture seamlessly integrates retrieval and generation. It retrieves the most relevant knowledge from the database and utilizes OpenAI's language processing capabilities to deliver informative and insightful answers to user queries.


## Implementation 

The below components are used to build up the bot, which can retrieve the required information from the vector store, feed it to the chain and stream responses to the client.

#### LLM Model 

        const model = new ChatOpenAI({
            temperature: 0.8,
            streaming: true,
            callbacks: [handlers],
        });


#### Vector Store

        const retriever = vectorStore().asRetriever({ 
            "searchType": "mmr", 
            "searchKwargs": { "fetchK": 10, "lambda": 0.25 } 
        })

#### Chain

       const conversationChain = ConversationalRetrievalQAChain.fromLLM(model, retriever, {
            memory: new BufferMemory({
              memoryKey: "chat_history",
            }),
          })
        conversationChain.invoke({
            "question": question
        })
