---
layout: post
title: Advanced Prompt Architectures - Querying and processing denormalized structured data
comments: true
tags: [Prompt Engineering, Prompt Design Patterns, Langchain, ChatGPT, plugins]
---
In the past I was focusing mentally a lot on how large language models (LLMs) allow new ways how to interact with text data. Recently, while talking to a colleague, it suddenly dawned on me that large language models actually also enable new ways how to "query" or "process" your structured tabular data. In this article I will compare four different approaches:

1. Querying structured data via APIs.
2. Querying structured data with text-to-sql or similar.
3. Processing structured data that was denormalized ("exploded") before feeding it into an LLM.
4. Querying the vector embeddings of structured data that was denormalized.
<span class="more"></span>

All approaches ultimately serve a similar need. Imagine you have tabular data such as a CSV file or a database table. Using the LLM you would like to "query" or "process" your dataset and get responses back. I am putting "querying" and "processing" here in double quotes because these terms can be interpreted in multiple ways.

# Four approaches to querying or processing structured data

## 1. LLM plus APIs: Querying your data through an API
This approach has already been popularized with [ChatGPT plugins](https://openai.com/blog/chatgpt-plugins). In brief, the LLM interacts with an API that hides the querying logic. The LLM is just responsible to provide the right querying parameters whereas the creation of the query and its execution is hidden behind the API. Also [Langchain supports this approach](https://python.langchain.com/docs/use_cases/apis).

## 2. Text-to-SQl
This is a very powerful approach. The LLM creates SQL (or any other formal query language like Sparql) from a  query description in natural language. An agent then runs the query against a database and returns the results to the LLM. The LLM reports the results in natural language again. This approach is described for example in more detail in [Langchain's documentation on question answering on structured data](https://python.langchain.com/docs/use_cases/qa_structured/sql?ref=blog.langchain.dev).

## 3. Processing denormalized structured data by feeding it into an LLM
While the previous two approaches are powerful to return highly accurate and formally correct results (as long as the query is correct), there are situations where correctness of the result is not the main goal. We could perhaps state that both approaches described previously are aimed at maximizing precision, and this third approach is aimed at maximizing recall. (This is not entirely accurate but hopefully illustrates the point I am about to make.)

Structured data are usually stored in a normalized format in a database. If you are working with LLMs, __it might make a lot of sense to denormalize your data first whenever you intend to feed it directly into your LLM.__ The denormalized data is much easier for the LLM to interpret, and hence you can expect a much higher quality of response from the LLM.

Here is an example. Imagine that you have a dataset that contains a column with a _gender_ attribtute. Data is stored as simple characters: An 'F' for women (females), an 'M' for men (males), an 'O' for others, and a 'U' for unknown or unspecified. As long as you are querying these data with a formal query language such as SQL you can perfectly make sense of this specification. The interpretation of what the single characters mean is somehow encoded in your application logic.
However, from the perspective of an LLM those letters do not carry a lot of meaning. For the LLM understanding the context of single characters is much harder than understanding entire words. Chances are the LLM does not fully grasp the meaning of the letters 'F', 'M', 'O', and 'U'. However, in the very moment you _denormalize_ your data and replace the single characters with actual words - 'female', 'male', 'other', 'unknown' - the LLM now has a lot more information it can work with.

But why should you even want to have the LLM with all its imprecision to query your data rather than something as precise as a SQL query? Simply because the LLM can actually _understand and interpret_ your data in a way the SQl query cannot. For example, the LLM might understand that is unlikely for children to have an income of 100'000 EUR per year, or that only women can give birth to a child. Consider this example:

```
I will give you now an excerpt from a CSV file. The columns are:
1. age in years
2. income in USD
3. level of education
4. gender

Your task is to find rows which have a combination that seems to be unreasonable. Output should be the line followed by an arrow -> and a very brief explanation.

Here is the CSV:
age,income,level_of_education,gender
46,100000,college,male
21,30000,high school,female
7,140000,university,male
62,65000,high school,male
```

Note the row of a 7-year old kid earning 140k USD with a university degree. Here is the actual answer using ChatGPT v3.5:

```
Certainly, here are the rows with combinations that seem to be unreasonable, followed by an arrow (->) and a brief explanation:

1. 7,140000,university,male -> It is highly unusual for a 7-year-old to have an income of $140,000 and a university-level education. This combination is likely incorrect or an outlier in the data.
```

Note how vaguely my prompt is formulated, all I am instructing is to find "unreasonable combinations" without giving any further hint.

Let's try a harder example.

```
I will give you now an excerpt from a CSV file. The columns are:
1. year of birth; note that we are in the year 2023 currently
2. income in USD
3. level of education, where: 0 means "no education", 1 means "basic education", 2 means "high school", 3 means "college", 4 means "higher education"
4. gender, where: 'M' means male, 'F' means female, 'O' means other, 'U' means unknown

Your task is to find rows which have a combination that seems to be unreasonable. Output should be the line followed by an arrow -> and a very brief explanation.

Here is the CSV:
age,income,level_of_education,gender
1977,100000,3,M
2002,30000,2,F
2016,140000,4,M
1961,65000,2,M
```

I have replaced the age with the birth year and the degree of education with a categorical value. Here's the answer:

```
Thank you for providing the CSV data. Here are the rows with combinations that seem to be unreasonable, followed by an arrow (->) and a brief explanation:

1. 1977,100000,3,M -> While the year of birth (1977) and income ($100,000) are reasonable, it's unusual for a male with a college-level education to have been born in 1977 and still be in the workforce, given that we are in the year 2023. This suggests the person might not have retired, which is less common.

2. 2016,140000,4,M -> An individual born in 2016 with a higher education level (4) and a high income of $140,000 is highly unlikely, as they would be very young and typically not have completed higher education yet. This combination seems unrealistic.

Please note that the explanations are based on the data provided, and they highlight inconsistencies or unusual aspects in the dataset.
```

As you can see, now the LLM starts to hallucinate. The second answer is correct, but the first one does not make any sense. A person born in the year 1977 would be 46 in the year 2023 and hence not be retired yet.

Takeaway message:

> __Denormalizing the data improves the likelihood for the LLM to be able to properly interpret the data.__

## 4. Querying denormalized structured data as an embedding vector
The fourth approach is to first denormalize the structured data as described previously, then to create an embedding vector, index all vectors, and finally use Retrieval Augmented Generative AI (RAG) to query all vectors. Obvsiouly, this will not yield results that are as precise as queries on structured data. Here, the intention is a different one. Imagine that your structured data is enriched with a little bit of unstructured data. Or the opposite, you have some unstructured data that you enrich additionally with some structured data such as metadata.

!["Querying denormalized structured data as an embedding vector"](/public/img/2023-10-30-querying-denormalized-structured-data-as-an-embedding-vector.png)

Querying those embedding vectors should now yield "semantically similar rows" compared to your original structured data. A formal query (e.g. SQL) on structured data is not prepared for identifying semantically similar rows in your data. Hence, this could constitute a way how to decrease precision and potentially increase recall in your information retrieval system.

# Summary of all four techniques

Here is a brief overview on all four approaches.

| | 1. Querying data through API | 2. Text-to-SQL | 3. Processing denormalized data | 4. Querying denormalized data as embedding vector|
|-----------------|-----------|-----------|--------|--------|
| _Optimized for_ | Precision | Precision | Recall | Recall |
| _Suited if_ | You need the precision of a structured query | You need the precision of a structured query | You want the LLM's power to "understand" and interpret your data | You want to enable semantic searching capabilities on your dataset |
| _Advantage_     | Non-technical people can query structured data | Non-technical people can query structured data | LLM can interpret results | Allows semantic searching |
| _Disadvantage_  | Complexity of interoperability between LLM and API; no integration with own APIs possible for commercially available products (e.g. OpenAI) | LLM may formulate wrong queries; danger of data deletion? | Requires data denormalization; LLM may hallucinate; restricted maximum prompt size | Results may be insufficiently precise |

