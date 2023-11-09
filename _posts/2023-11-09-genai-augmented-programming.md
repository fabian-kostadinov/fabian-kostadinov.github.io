---
layout: post
title: Generative AI-Augmented Programming
comments: true
tags: [Prompt Engineering, Prompt Design Patterns, Generative AI, Large Language Models]
---
The use of Generative AI allows a completely new programming paradigm. I call this paradigm __Generative AI-Augmented Programming (GAP)__. In very brief, GAP is a programming model that mixes regular software code with output from GenAI models. The idea is best illustrated with an example. Imagine you have a list of animals: ["horse", "cat", "whale", "goldfish", "bacteria", "dove"]. You do not have any further information than those animals. Now you are asked to sort this list according to the animal's expected or average weight in descending order. By applying common sense reasoning a human returns this ordered list: ["whale", "horse", "cat", "dove", "goldfish", "bacteria"]. Assume we implement a (e.g. Python) function <code>sort(list_to_sort: List[str], sort_criterion: str) -> List[str]</code> such that the sorting happens entirely automatically behind the scenes by calling a Large Language Model. Input is a List of strings and a sort criterion description as a string parameter. Both are being sent as a prompt to the LLM, which computes the result. The output is again same list, just ordered according to said criterion. Wild, isn't it? Welcome to _Generative-AI Augmented Programming_!
<span class="more"></span>

# A simple example: Sorting with GAP
Let's have a look at how such a sort function could be implemented.

First, we define a class _SortPrompt_. As its name indicates it represents a prompt to an LLM. The constructor takes arguments such as a list of items to sort, a sort order (ascending or descending) of type enum, a textual description of which criterion to apply, and it defines the LLM's system role ("You are an expert in..."). After having created an instance of the class, we can call the <code>to_str()</code> function to obtain a string representation of the prompt.
If we wanted to go a little more fancy here we could also have applied an OOD factory pattern.
```python
from typing import List, Callable
from dataclasses import dataclass
from enum import Enum
import textwrap

class SortOrder(Enum):
    ascending = 1
    descending = -1

@dataclass
class SortPrompt:
    system_role: str
    items: List[str]
    order: SortOrder
    criterion: str

    def to_str(self) -> str:
        items = ", ".join(self.items)
        prompt = """
            ### Instructions
            You are {system_role}.
            Your task is to sort below list of items in {order} order according to {criterion}.
            Once you have sorted them, provide a reasoning for the sort order you have selected.
            Return the output as a JSON object in the format:
            {% raw %}{{
                "result": ["item 1", "item 2", ..., "item n"],
                "sort_order": "EITHER ascending OR descending",
                "sort_criterion": "the criterion applied to sort",
                "reason": "description why you sorted all items accordigly"
            }}{% endraw %}
            ### Example
            List of items to sort: ["house", "racing horse", "bicyle", "pizza"]
            Sort order: ascending
            Sort criterion: purchasing price
            Expected output:
            {% raw %}{{
                "result": ["pizza", "bicycle", "racing horse", "house"],
                "sort_order": "ascending",
                "sort_criterion": "purchasing price",
                "reason": "A house is more expensive than a racing horse. A racing horse is more expensive than a bicycle. A bicycle is more expensive than a pizza."
            }}{% endraw %}
            ### Input
            {items}""".format(system_role=self.system_role, order=self.order.name, criterion=self.criterion, items=items)  
        return textwrap.dedent(prompt).strip()

```
As you can see, the _SortPrompt_ class is a simple container for data required to create the entire prompt. I am using the <code>@dataclass</code> decorator to save myself from creating an <code>__init__()</code> constructor function as well as getters and setters. The <code>to_str()</code> function assembles the entire prompt and returns it, so we can feed it into our sort function later on.

The prompt uses various means to (hopefully) improve the LLM's response quality.
1. It defines the expected output of the LLM as a JSON object.
2. It provides one (or even few-shot) examples.
3. It asks the LLM to provide a reasoning for its response.

Maybe there are even more tricks we could apply here, but we are right now not interested in [prompt design best practices](https://en.wikipedia.org/wiki/Prompt_engineering).

Next, we feed our prompt as a string into a simple sort function.
```python
def sort(prompt: str, completion: Callable, print_prompt=False) -> List[str]:
    if print_prompt:
        print(prompt)
    
    try:
        openai_response = completion(prompt)
        json_str = openai_response["choices"][0]["message"]["content"] # type: str
        json_response = json.loads(json_str) # type: dict
        response = json_response["result"] # type: list
    except Exception as e:
        print(f"sort: Error: {e}")

    return response
```
For simplicity I have designed the sort function to take the prompt as a string input. If a developer does not want to use the suggested _SortPrompt_ class for any reason she is free to write her own prompt entirely from scratch. In this way we are not technically enforcing usage of _SortPrompt_, it's just a suggestion.
The sort function also takes a Callable as a second function parameter that we call _completion_. The completion function is of course that function which makes the actual call to the LLM.
The return type is a list of strings, i.e. simply the ordered version of the list that we provided as part of the prompt string.

Here's a simple implementation for the completion function using the OpenAI API:

```python
import openai

# Don't forget to set openai.api_key = "...your key..." before calling:
# openai.api_key = os.getenv("OPENAI_API_KEY")

def openai_completion(prompt: str) -> any:
    try:
        openai_response = openai.ChatCompletion.create(
            model = "gpt-3.5-turbo-1106",
            messages = [
                {"role": "user", "content": prompt},
            ],
            temperature = 0.00000001
        )
    except Exception as e:
        print(f"openai_completion: Exception: {e}")
        
    return openai_response
```

I've set the temperature to a very low value (or possibly 0.0) to make the LLM's output as deterministic as possible. In my experience though at least OpenAI's gpt-3.5-turbo is not fully deterministic, even if the temperature is set to 0.0.

Okay, let's put everything together:

```python
sort_params = SortPrompt(
    system_role = "a helpful assistant",
    items = ["cat", "rat", "mouse", "elephant", "fly", "tiger", "bacteria", "goldfish"],
    order = SortOrder.descending,
    criterion = "their physical weight"
)

prompt = sort_params.to_str()
sorted_list = sort(prompt, openai_completion, print_prompt=False)
print(sorted_list)
```
Expected output: <code>["elephant", "tiger", "cat", "rat", "mouse", "goldfish", "fly", "bacteria"]</code>.

# A plethora of possibilities
Generative AI-augmented programming offers an almost endless number of such possibilities. Sorting is just one possibility. Let's look at some examples to illustrate.

## List functions
* A <code>filter</code> function that takes a list of strings and filters out those that don't fulfill a given criterion. Example: Take a list of animals but return only those that are carnivores. We do not define upfront what the concept of "carnivore" actually means, instead we leave this to the LLM to figure out.
* A <code>map</code> function that takes a list of strings and applies some other function (described in the prompt) to each element in the list. Example: Take a list of animals and always return the male name of the animal if available (_stallion_ instead of _horse_, _tomcat_ instead of _cat_, and so on).
* A <code>find</code> function that takes a list of strings and a matching criterion and returns the first element encountered where the matching criterion is given or <code>None</code> if no matching element exists. As an extension, it might also  be run in reverse starting with the last element. Example: Please find the first element that is a city in France: ["Washington", "Rome", "Lyon", "Stockholm", "Singapore", "Moscow", "Canberra", "Marseille"].
* A <code>find_all</code> function that does same as the _find_ function but matches all elements and returns a list of strings.
* A <code>contains</code> function that takes a list of strings and a match criterion (for example an element circumscribed with different words) and returns _True_ if the list contains such an element, or _False_ otherwise. Example: Does the following list contain any mammals: ["jellyfish", "spider", "trout", "toad", "hare", "eagle"]? Note that we don't provide any clear explanation what constitutes a "mammal" here, that's the LLM's task to find out.

And many more.

## Boolean functions
* A <code>condition</code> function that takes a given statement and returns <code>True</code> or <code>False</code> depending on whether the statement is true or false. Example: The statement "Planet Earth is the largest planet in the solar system." is clearly false. Hence, the condition function returns <code>False</code>.
* An <code>equal</code> or perhaps <code>synonym</code> function that takes two elements plus an equality comparator criterion. It then compares both elements according to the criterion given and returns _True_ if both are equal, and _False_ otherwise. Example: Are "ability" and "capability" equal?
* An <code>is_part_of</code> function that takes two items and returns _True_ if the first item is a part of the second, or _False_ otherwise. Example: Is "wheel" part of "car"?

And many more.

## General knowledge functions
* A <code>more_details</code> function that takes a concept and returns more details on it. If the LLM is allowed to use a plugin (e.g. web search or search of a specific database) then it can extend the knowledge base to draw from to the internet. Example: Input is "Hotels in Amsterdam", the LLM is allowed to do a search to a third party website with an overview on hotels. The LLM triggers the search and ultimately returns results.

And many more.

## User input callback functions
We could also reverse the entire approach and ask a user for feedback. Imagine that we don't fully trust the output of an LLM. We want to build in a user check. So, our functions above are enriched with a callback option. For example, once the LLM returned a response the program flow makes a call to a user interface and asks the user to validate the LLM's response. However, unless the user is expected to sit in front of the computer we could also add a callback function such that the user's response is captured asynchronously at a later point in time. (I'm sure this can probably be engineered in a better way, but I just wanted to make the point nevertheless.)

# GAP as a paradigm
I would bet that many readers think that the entire approach is both fascinating and also completely weird at the same time. Or simply completely irrelevant. Why on earth should anyone use an LLM for such things given the imprecision, vagueness and straight-out hallucinations that LLMs are plagued with these days?
Another objection might be that in many situations there does not even exist a proper concept to rely on in the first place. For example: Is a whale heavier than a horse? Well, that might depend on the type of whale we are referring to! A very small (or very young) whale might possibly be lighter than a big horse. So, sorting a list of animals according to their weight would result in a bunch of nonsense, no?

Both arguments are of course true. And yet, both arguments totally miss the point. Because: We are talking about _common sense reasoning_ here. If we wanted to have mathematical precision we could simply have used traditional programming in the first place! It is exactly because the LLMs are vague, imprecise and so on that they are of interest to us. The novelty is the "productive lack of precision" that is the game changer here. LLMs are as dirty in their knowledge as our own! If I ask a friend whether a whale is heavier than a horse, she might say something along the lines: "Well, generally speaking yes, but you could refer to a really small whale, and then it would not be the case anymore." And that is exactly the beauty of it! __GAP is not about achieving 100% correctness or accuracy. Instead, it is about coming to the most reasonable conclusion in a given situation.__ GAP is not programming for correctness and flawless execution logic like traditional programming paradigms. Instead it is more or less mimicking the reasoning that humans are doing: full of imperfection, full of contradictions, but frequently just reasonable.

I hear some readers saying: Oh, I get that, but what the heck should this be useful for?

Well, that is a question I'll still have to figure out myself. I have some preliminary ideas, but nothing to share here for the moment. For now, just imagine that you are sitting in front of a very weird 15 year old. This youth has an enourmously broad knowledge, but makes many mistakes and sometimes gets even very basic facts about the world completely wrong. The person never gets hungry and thirsty and never needs sleep. How would you make best use of such a person around you? Certainly, you would not expect the person to solve all mysteries of life. Also, you would not expect the person to know anything about a set of documents you have stored on your harddrive to which nobody in the world except yourself have access. Instead, the person would be very useful for offboarding and automating relatively simple workflows that require a common sense world knowledge. You would not have to teach the person that rain falls from the sky (rather than in the reverse direction), the person would tacitly know such stuff. What if you gave that person something more complicated to do, like reading a legal contract of a few dozen pages? Well, surely the person would not get everything right. Understanding contracts is complicated. But then again, it probably would get most simple questions right. And it would never be tired, no matter how many questions you throw at it. If you explained a few extra points (e.g. by extending your prompt) to the person then the person might even learn enough about how to interpret a contract that from that point onwards the responses would improve.

Compare that to what you have toda: Another person (i.e. your computer) who always gets everything 100% right - but only in an extremely narrowly defined area, and only under ideal circumstances. If you forget to tell a single bit of information it just stares at you with a blank expression.

Generative AI-augmented programming is about combining both types of expertise in an intelligent manner: the precision and flawlessness of formal logic and programming, but also its narrowness and limitedness. And the imprecision and opaqueness of Generative AI / LLMs, but also their breadth of knowledge and capabilities.

And that allows a breathtaking new way of how to create applications of a kind we have never seen before.
