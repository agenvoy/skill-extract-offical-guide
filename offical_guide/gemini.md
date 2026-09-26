# Gemini prompt design strategies

Retrieved 2026-09-27 from https://ai.google.dev/gemini-api/docs/prompting-strategies
Converted from the devsite HTML `<article>` region (no .md variant is served).

---

Gemini 3.8 Flash is now available. Try it out.

  - Home

  - Gemini API

  - Docs

# Prompt design strategies

Prompt design is the process of creating prompts, or natural language requests,
that elicit accurate, high quality responses from a language model.

This page introduces basic concepts, strategies, and best practices to get you
started designing prompts to get the most out of Gemini AI models.

Note: Prompt engineering is iterative. These guidelines and templates are
starting points. Experiment and refine based on your specific use cases and
observed model responses.

## Topic-specific prompt guides

Looking for more specific prompt strategies? Check out our other prompting guides
on:

- Prompting with media files

- Prompting for image generation

- Prompting for video generation

You can find other sample prompts in the prompt gallery
meant to interactively showcase many of the concepts shared in this guide.

## Clear and specific instructions

An effective and efficient way to customize model behavior is to provide it with
clear and specific instructions. Instructions can be in the form of a question,
step-by-step tasks, or as complex as mapping out a user's experience and mindset.

### Input

Input is the required text in the prompt that you want the model to provide a
response to. Inputs can be a question that the model
answers (question input), a task the model performs (task input), an entity the
model operates on (entity input), or partial input that the model completes or
continues (completion input).

     Input type
     Prompt
     Generated output

    Question

    Task

    Entity

#### Partial input completion

Generative language models work like an advanced auto completion tool. When you
provide partial content, the model can provide the rest of the content or what
it thinks is a continuation of that content as a response. When doing so, if you
include any examples or context, the model can take those examples or context
into account.

The following example provides a prompt with an instruction and an entity input:

  Prompt:

  Response:

  (gemini-2.5-flash)

While the model did as prompted, writing out the instructions in natural language
can sometimes be challenging and it leaves a lot to the model's interpretation.
For example, a restaurants menu might contain many items. To reduce the size of
the JSON response, you probably want to omit the items that weren't ordered. In
this case, you can give an example and a response prefix and let the model
complete it:

  Prompt:

  Response:

  (gemini-2.5-flash)

Notice how "cheeseburger" was excluded from the output because it wasn't a part
of the order.

While you can specify the format of simple JSON response objects using prompts,
we recommend using Gemini API's
structured output feature when specifying
a more complex JSON Schema for the response.

### Constraints

Specify any constraints on reading the prompt or generating a response. You can
tell the model what to do and not to do. For example, you can specify a constraint
in the prompt on how long you want a summary to be:

    Prompt:

    Response:

    (gemini-2.5-flash)

### Response format

You can give instructions that specify the format of the response. For example,
you can ask for the response to be formatted as a table, bulleted list, elevator
pitch, keywords, sentence, or paragraph. The following system instruction tells
the model to be more conversational in its response:

  System instruction

  Prompt

  Response:

  (gemini-2.5-flash)

#### Format responses with the completion strategy

The completion strategy can also help format the response.
The following example prompts the model to create an essay outline:

  Prompt:

  Response:

  (gemini-2.5-flash)

The prompt didn't specify the format for the outline and the model chose a format
for you. To get the model to return an outline in a specific format, you can add
text that represents the start of the outline and let the model complete it based
on the pattern that you initiated.

  Prompt:

  Response:

  (gemini-2.5-flash)

## Zero-shot vs few-shot prompts

You can include examples in the prompt that show the model what getting it right
looks like. The model attempts to identify patterns and relationships from the
examples and applies them when generating a response. Prompts that contain a few
examples are called few-shot prompts, while prompts that provide no
examples are called zero-shot prompts. Few-shot prompts are often used
to regulate the formatting, phrasing, scoping, or general patterning of model
responses. Use specific and varied examples to help the model narrow its focus
and generate more accurate results.

We recommend to always include few-shot examples in your prompts. Prompts without
few-shot examples are likely to be less effective. In fact, you can remove
instructions from your prompt if your examples are clear enough in showing the
task at hand.

The following zero-shot prompt asks the model to choose the best explanation.

  Prompt:

  Response:

  (gemini-2.5-flash)

If your use case requires the model to produce concise responses, you can include
examples in the prompt that give preference to concise responses.

The following prompt provides two examples that show preference to the shorter
explanations. In the response, you can see that the examples guided the model to
choose the shorter explanation (Explanation2) as opposed to the longer
explanation (Explanation1) like it did previously.

  Prompt:

  Response:

  (gemini-2.5-flash)

### Optimal number of examples

Models like Gemini can often pick up on patterns using a few examples, though
you may need to experiment with the number of examples to provide in the prompt
for the best results. At the same time, if you include too many examples,
the model may start to overfit
the response to the examples.

### Consistent formatting

Make sure that the structure and formatting of few-shot examples are the same to
avoid responses with undesired formats. One of the primary objectives of adding
few-shot examples in prompts is to show the model the response format. Therefore,
it is essential to ensure a consistent format across all examples, especially
paying attention to XML tags, white spaces, newlines, and example splitters.

## Add context

You can include instructions and information in a prompt that the model needs
to solve a problem, instead of assuming that the model has all of the required
information. This contextual information helps the model understand the constraints
and details of what you're asking for it to do.

The following example asks the model to give troubleshooting guidance for a router:

  Prompt:

  Response:

  (gemini-2.5-flash)

The response looks like generic troubleshooting information that's not specific
to the router or the status of the LED indicator lights.

To customize the response for the specific router, you can add to the prompt the router's
troubleshooting guide as context for it to refer to when providing a response.

  Prompt:

  Response:

  (gemini-2.5-flash)

## Break down prompts into components

For use cases that require complex prompts, you can help the model manage this
complexity by breaking things down into simpler components.

- Break down instructions: Instead of having many instructions in one
prompt, create one prompt per instruction. You can choose which prompt to
process based on the user's input.

- Chain prompts: For complex tasks that involve multiple sequential steps,
make each step a prompt and chain the prompts together in a sequence. In this
sequential chain of prompts, the output of one prompt in the sequence becomes
the input of the next prompt. The output of the last prompt in the sequence
is the final output.

- Aggregate responses: Aggregation is when you want to perform different
parallel tasks on different portions of the data and aggregate the results to
produce the final output. For example, you can tell the model to perform one
operation on the first part of the data, perform another operation on the rest
of the data and aggregate the results.

## Experiment with model parameters

Each call that you send to a model includes parameter values that control how
the model generates a response. The model can generate different results for
different parameter values. Experiment with different parameter values to get
the best values for the task. The parameters available for
different models may differ. The most common parameters are the following:

- Max output tokens: Specifies the maximum number of tokens that can be
generated in the response. A token is approximately four characters. 100
tokens correspond to roughly 60-80 words.

- Temperature: The temperature controls the degree of randomness in token
selection. The temperature is used for sampling during response generation,
which occurs when topP and topK are applied. Lower temperatures are good
for prompts that require a more deterministic or less open-ended response,
while higher temperatures can lead to more diverse or creative results. A
temperature of 0 is deterministic, meaning that the highest probability
response is always selected.
Note: The `temperature`, `top_p`, and `top_k` parameters control how the model
generates responses. Although you can modify these parameters, we strongly
recommend keeping them at their default values for Gemini 3.x models. Changing
these parameters (for example, setting the temperature below 1.0) can cause
unexpected behavior, such as looping or degraded performance, particularly in
complex mathematical or reasoning tasks.

- topK: The topK parameter changes how the model selects tokens for
output. A topK of 1 means the selected token is the most probable among
all the tokens in the model's vocabulary (also called greedy decoding),
while a topK of 3 means that the next token is selected from among the 3
most probable using the temperature. For each token selection step, the
topK tokens with the highest probabilities are sampled. Tokens are then
further filtered based on topP with the final token selected using
temperature sampling.

- topP: The topP parameter changes how the model selects tokens for
output. Tokens are selected from the most to least probable until the sum of
their probabilities equals the topP value. For example, if tokens A, B,
and C have a probability of 0.3, 0.2, and 0.1 and the topP value is 0.5,
then the model will select either A or B as the next token by using the
temperature and exclude C as a candidate. The default topP value is 0.95.

- stop_sequences: Set a stop sequence to
tell the model to stop generating content. A stop sequence can be any
sequence of characters. Try to avoid using a sequence of characters that
may appear in the generated content.

## Prompt iteration strategies

Prompt design can sometimes require a few iterations before
you consistently get the response you're looking for. This section provides
guidance on some things you can try when iterating on your prompts:

- Use different phrasing: Using different words or phrasing in your prompts
often yields different responses from the model even though they all mean the
same thing. If you're not getting the expected results from your prompt, try
rephrasing it.

- Switch to an analogous task: If you can't get the model to follow your
instructions for a task, try giving it instructions for an analogous task
that achieves the same result.

This prompt tells the model to categorize a book by using predefined categories:

  Prompt:

  Response:

  (gemini-2.5-flash)

The response is correct, but the model didn't stay within the bounds of the
options. You also want to model to just respond with one of the options instead
of in a full sentence. In this case, you can rephrase the instructions as a
multiple choice question and ask the model to choose an option.

  Prompt:

Response:

(gemini-2.5-flash)

- Change the order of prompt content: The order of the content in the prompt
can sometimes affect the response. Try changing the content order and see
how that affects the response.

## Fallback responses

A fallback response is a response returned by the model when either the prompt
or the response triggers a safety filter. An example of a fallback response is
"I'm not able to help with that, as I'm only a language model."

If the model responds with a fallback response, try increasing the temperature.

## Grounding and code execution

Gemini is able to use tools to avoid hallucinations in scenarios where it might
otherwise produce incorrect responses.

Grounding with Google Search connects the
Gemini model to real-time web content, and should be enabled whenever the model
may need to know obscure or recent facts.

Gemini's code execution tool enables the
model to generate and run Python code, and should be enabled whenever the model
needs to perform any kind of arithmetic, counting, or calculation.

## Gemini 3

Gemini 3 models are designed for advanced
reasoning and instruction following.
They respond best to prompts that are direct, well-structured, and clearly
define the task and any constraints. The following practices are recommended for
optimal results with Gemini 3:

### Core prompting principles

- Be precise and direct: State your goal clearly and concisely. Avoid
unnecessary or overly persuasive language.

- Use consistent structure: Employ clear delimiters to separate different
parts of your prompt. XML-style tags (e.g., <context>, <task>) or
Markdown headings are effective. Choose one format and use it consistently
within a single prompt.

- Define parameters: Explicitly explain any ambiguous terms or parameters.

- Control output verbosity: By default, Gemini 3 models provide direct and
efficient answers. If you need a more conversational or detailed response,
you must explicitly request it in your instructions.

- Handle multimodal inputs coherently: When using text, images, audio, or
video, treat them as equal-class inputs. Ensure your instructions clearly
reference each modality as needed.

- Prioritize critical instructions: Place essential behavioral
constraints, role definitions (persona), and output format requirements in
the System Instruction or at the very beginning of the user prompt.

- Structure for long contexts: When providing large amounts of context
(e.g., documents, code), supply all the context first. Place your specific
instructions or questions at the very end of the prompt.

- Anchor context: After a large block of data, use a clear transition
phrase to bridge the context and your query, such as "Based on the
information above..."

### Gemini 3 Flash strategies

- Current day accuracy: Add the following clause to the system
instructions to help the model pay attention to the current day being in 2026:

- Knowledge cutoff accuracy: Add the following clause to the system
instructions to make the model aware of its knowledge cutoff:

- Grounding performance: Add the following clause to the system
instructions (with edits where appropriate) to improve the model's ability
to ground responses in provided context:

### Enhancing reasoning and planning

Gemini 2.5 and 3 series models automatically generate internal "thinking" text
to improve reasoning performance. As such, it's generally not necessary to have
the model outline, plan, or detail reasoning steps in the returned response
itself. For problems that require heavy reasoning, simple requests like "Think
very hard before answering" can improve performance, though at the cost of
extra thinking tokens.

See the Gemini thinking documentation for more
detail.

### Structured prompting examples

Using tags or Markdown helps the model distinguish between instructions,
context, and tasks.

XML example:

Markdown example:

### Example template combining best practices

This template captures the core principles for prompting with Gemini 3. Always
make sure to iterate and modify for your specific use case.

System Instruction:

User Prompt:

## Agentic workflows

For deep agentic workflows, specific instructions are often required to control how the model reasons, plans, and executes tasks. While Gemini provides strong general performance, complex agents often require you to configure the trade-off between computational cost (latency and tokens) and task accuracy.

When designing prompts for agents, consider the following dimensions of behavior that you can steer in the agent:

### Reasoning and strategy

Configuration for how the model thinks and plans before taking action.

- Logical decomposition: Defines how thoroughly the model must analyze constraints, prerequisites, and the order of operations.

- Problem diagnosis: Controls the depth of analysis when identifying causes and the model’s use of abductive reasoning. Determines if the model should accept the most obvious answer or explore complex, less probable explanations.

- Information exhaustiveness: The trade-off between analyzing every available policy and document versus prioritizing efficiency and speed.

### Execution and reliability

Configuration for how the agent operates autonomously and handles roadblocks.

- Adaptability: How the model reacts to new data. Determines whether it should strictly adhere to its initial plan or pivot immediately when observations contradict assumptions.

- Persistence and Recovery: The degree to which the model attempts to self-correct errors. High persistence increases success rates but risks higher token costs or loops.

- Risk Assessment: The logic for evaluating consequences. Explicitly distinguishes between low-risk exploratory actions (reads) and high-risk state changes (writes).

### Interaction and output

Configuration for how the agent communicates with the user and formats results.

- Ambiguity and permission handling: Defines when the model is permitted to make assumptions versus when it must pause execution to ask the user for clarification or permission.

- Verbosity: Controls the volume of text generated alongside tool calls. This determines if the model explains its actions to the user or remains silent during execution.

- Precision and completeness: The required fidelity of the output. Specifies whether the model must solve for every edge case and provide exact figures or if ballpark estimates are acceptable.

### System instruction template

The following system instruction is an example that has been evaluated by researchers to improve performance on agentic benchmarks where the model must adhere to a complex rulebook and interact with a user. It encourages the agent to act as a strong reasoner and planner, enforces specific behaviors across dimensions listed above and requires the model to proactively plan before taking any action.

You can adapt this template to fit your specific use case constraints.

## Next steps

- Now that you have a deeper understanding of prompt design, try writing your
own prompts using Google AI Studio.

- To learn about multimodal prompting, see
Prompting with media files.

- To learn about image prompting, see the Nano Banana prompt guide.

- To learn about video prompting, see the Veo prompt guide.