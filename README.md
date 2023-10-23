# frameSmith

<img src="./image/frameSmith_logo_banner.png" alt="frameSmith logo"/>

***frameSmith*** scrapes information from the web and YouTube videos, and uses an OpenAI LLM to generate a framework of your chosing. It comes preconfigured to generate a [Lean Canvas](https://www.leancanvas.com/), which summarizes a product or company's strategy.

<div align="center">
<img src="./image/top.png" alt="API Key Example" width="80%" />
</div>

## Setup
1. Clone and navigate into the directory.
```
gh repo clone tmk1221/lean_canvas_bot
cd lean_canvas_bot
```

2. Create a virtual environment, and install Python dependencies.
```
virtualenv venv
source venv/bin/activate
pip install -r requirements.txt
```

3. Create a file in the root directory called `.env`, and add your OpenAI API key as shown below. No quotes necessary.

    <img src="./image/api_key_example.png" alt="API Key Example" width="80%" />

4. Update the variables within `./config.json` to match your product/company.
    1. `product`: Product/company name as its referred to in the sources you provide

    2. `openai_model`: OpenAI model used to generate the Lean Canvas

        - Note: At the time of writing, the most common options are: "gpt-3.5-turbo" or "gpt-4". GPT-4 is a more powerful model, but will cost more to use. For up-to-date information about available models, see [OpenAI's Model Overview](https://platform.openai.com/docs/models/overview)

    3. `news_urls`: Websites, blog posts and/or news articles specific to your product/company

        - Note 1: You need to provide at least 1 URL here for the bot to run.

        - Note 2: BeautifulSoup is used to scrape text from these websites. The text then goes through a cleaning step. You may want to print the texts afterwards to ensure that the texts were scraped and cleaned in the way you expected. See line 36 in `src/lean_canvas_generator.py` for what I mean.

    4. `youtube_urls`: YouTube videos specific to your product/company

        - Note 1: You do not need to provide any YouTube videos for the bot to run. If you do not want to use any YouTube videos, then replace the brackets with `None` in the `./config.json`.

        - Note 2: Transcript texts are captured via YouTube's API. Some YouTube videos don't have transcripts. If this is the case, it's okay, the API will just return an empty string.

## Usage
1. Run the bot

    Note: This takes several minutes to complete. Progress indicators are printed to the command line.

```
python3 ./src/generate.py
```

2. Find your lean canvas in `./lean_canvas_output`

## Adaptation Considerations
1. **Change data sources**

    Currently the Lean Canvas Bot is setup to scrape and clean data from blogs and news websites, and retreive transcripts from YouTube videos. Thus, data is loaded via URLs, which is limited considering the diversity of data sources in existence.

    Langchain supports [100+ document loaders](https://python.langchain.com/docs/integrations/document_loaders) with which you can load data from specific sources. You can look into replacing lines 16-39 in `./lean_canvas_generator.py` to do this.

<div align="center">
<img src="./image/data.png" alt="Data" width="80%" />
</div>

2. **Replace Lean Canvas with any framework**

    If the Lean Canvas framework doesn't suit your needs, this bot can be used to generate any framework. Think about the questions you would need to ask your knowledge base (i.e. vectorstore) in order to generate each section of your framework. Currently, the Lean Canvas Bot asks 9 questions of the knowledge base - one question for each section of a Lean Canvas. Replace the questions on lines 75-85 in `./lean_canvas_generator.py` with your own questions. Of course, you will still need to load in information relevant to the framework you desire.

<div align="center">
<img src="./image/framework.png" alt="Framework" width="80%" />
</div>

3. **Tradeoffs with Document Splitting**
    
    Document splitting refers to how entire documents are split into smaller documents. This is necessary because usually entire documents exceed LLM context windows (e.g. 32k tokens for GPT-4). The current implementation is to split the scraped web texts and transcripts into documents of 1000 character chunks.

    There is a tradeoff between larger and smaller documents. Larger documents will provide more context to the LLM; however, the embeddings are less specific for larger documents. This has the effect of "diluting" the semantic meaning of the documents, which makes it more difficult to return relevant documents. Smaller documents, on the other hand, have more specific embeddings, but provide less context to the LLM. If documents are too small, they might only return part of the answer, where the rest gets cutout and placed in subsequent documents.

    There are workarounds for this. See Langchain's [Parent Document Retriever](https://js.langchain.com/docs/modules/data_connection/retrievers/how_to/parent-document-retriever), or for a more general discussion, see Pinecone's [Chunking Strategies for LLM Applications](https://www.pinecone.io/learn/chunking-strategies/).

    In any case, if your Bot performance is suffering, document splitting is one of the first places I would look to optimize things. It plays a big part in the quality and relevance of the documents that get passed into the LLM prompt.