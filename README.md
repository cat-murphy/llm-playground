# llm-playground

Using the green "Use this template" button on the top right, create a copy of this repository under your GitHub account, giving it the same name. Open that repository in a codespace, then run the following commands in the terminal:

### Setup

```bash
bash setup.sh
```

That will install llm, a Python tool for interacting with different LLMs, llm-groq, which specifically connects to Groq, and llm-gemini, which connects to Google's models. You will need to generate an API key at https://console.groq.com/keys. Copy that key, then do the following command in the terminal:

```bash
llm keys set groq
```

You will paste your copied key in the terminal (it won't appear, and you might get a small popup with the word "paste" in grey. Wait a couple of seconds and you'll be able to click on it.). To make sure it worked, run this command in the terminal:

```bash
llm -m groq-llama-3.3-70b "10 names for a pet aardvark"
```

Now create an API key for Gemini here: https://aistudio.google.com/apikey. Copy that key, then do the following command in the terminal:

```bash
llm keys set gemini
```

You will paste your copied key in the terminal (it won't appear, and you might get a small popup with the word "paste" in grey. Wait a couple of seconds and you'll be able to click on it.). To make sure it worked, run this command in the terminal:

```bash
llm -m gemini-2.5-flash "10 names for a pet turtle"
```

You can see all of the models you have access to using llm with the following command:

```bash
llm models
```

### Text Models

The llm library allows us to chat with LLMs, but also to pass things to those LLMs _and_ to log our conversations in a SQLITE database. Here's one way we can start: asking it to summarize the specific contents of a web page that we'll fetch using a tool called newspaper4k that allows us to reduce the amount of tokens we send to the LLM (they have limits):

```bash
python -m newspaper --url=https://cnsmaryland.org/2025/08/02/in-the-deep-woods-a-little-owl-teaches-big-lessons/ -of=text | llm -m groq/openai/gpt-oss-120b "summarize this story in 3 paragraphs"
```

Read the story: https://cnsmaryland.org/2025/08/02/in-the-deep-woods-a-little-owl-teaches-big-lessons/ and then compare that to the summary. Is the summary a good description? Why or why not?

PUT YOUR ANSWER HERE: 
    
    OK, so when I first read over Groq’s summary in class — before reading Sade’s story, I should mention — I thought “Yeah, OK, seems fine.” But when I went back and read the original story, I noticed Groq’s summary is … um … a little “off.” For example, Groq says Longeveld’s saw-whet records provide “vital insight into their ecology and informing conservation strategies.” Ecology, yes. Conversation strategies, no. It guessed. It didn’t make it up. But it definitely tried to fill in some blank there, because the article simply doesn’t say that. (If I had to guess, I think it made that leap based on the fact that scientists used to think they were rare, and the article focuses on the importance of the dataset. The article never says anything about conservation, but I can see where it’s getting it.) Groq’s output has some other weird quirks, too. It spelled “banding” with a hyphen — as in, “band‑ing station” — the first time, then never again. It said scientists originally believed the birds were “sedentary,” which isn’t quite the right word. It said that “a first‑time bander such as student Grace” got to release the bird — also not quite the right syntax. That’s just it. In a lot of ways, the summary is fine. Generally speaking, it’s relatively accurate — it’s just weird. It’s slightly off. It’s like it’s almost there, but not quite. I mean, if you’ve read the article, you can tell this is AI. So that begs the question, how do we avoid that in our beat books? Is it a matter of better prompting? The training itself? Something else? No clue. Just spitballing here.

Now it's your turn: find a story that you wrote or are very familiar with that's published online, and try the same process, altering the command and then evaluating the result.

```bash
python -m newspaper --url=https://quchronicle.com/85766/featured/quinnipiac-hedge-funds-cayman-islands/ -of=text | llm -m groq/openai/gpt-oss-120b "summarize this story in 3 paragraphs" > chron_summary.txt
```

YOUR EVALUATION HERE.

    See, this summary was great. I wrote the article, and honestly, I don’t know that I could have written a better one. Groq highlighted the important details and explained the more complicated aspects of the story (quite well, actually), but it did so without droning on or adding interpretations. I didn’t love that it used the word “critics” to describe who was calling this out, but otherwise it used language that I thought matched the tone of the original text. I have a theory about this: My story was about as straight news as you can get. Very few quotes, basically just pure analysis and explanation. The one quote is from our PR guy, and Groq did a good job of basically being like, “the university claims that it doesn’t directly invest in fossil fuels, but these funds ensure it benefits from the industry, even if indirectly.” This is kind of the opposite of what Groq did with Sade’s story. But Sade’s profile was a soft news story and therefore included a lot more color. More quotes, more descriptions, more subjectivity. And we didn’t specify that Groq should prioritize the more concrete information, and I think that’s why the summary was a little “off.” So, going forward, I wonder if we have to devise a way to prompt the LLMs to summarize the story content separately from what people are saying.

### Restructuring Information

Text models can do more than summarize; they can restructure information to make it more useful. We'll take a recent document describing sanctions of Maryland attorneys (the sanctionsfy25.txt file in this repository) and send it to Llama 3.3 using the `cat` command, asking the LLM to produce JSON data of some of the contents.

```bash
cat sanctionsfy25.txt | llm -m groq-llama-3.3-70b "produce only an array of JSON objects based on the text with the following keys: name, sanction, date, description. The date should be in the yyyy-mm-dd format. No yapping." 
```

That "no yapping" thing? That's one of the ways you get LLMs to stop narrating their output and just give you what you specify.

Now it's your turn: find a short (3 pages or less) news story that contains unstructured text, save it as a text file and drop it into the list of files on the left side. Then create a prompt like the one above that turns some elements of it into structured data. Put your prompt in the space below, and below that tell me how the LLM did.

```bash
cat dhs_foias.txt | llm -m groq-llama-3.3-70b "produce a CSV file based on the text with the following columns: case number, date, requester name, requester agency, type of records requested, full description. the date should be in the yyyy-mm-dd format. no yapping." > dhs_foias_data.csv
```

PUT YOUR EVALUATION HERE.

### Vision Models

One of the options we have with Groq is to use a "vision" model that can describe the information in certain types of images. Let's test that out using an image from this Maryland physician discipline report: https://www.mbp.state.md.us/BPQAPP/orders/D002592202.265.pdf. The specific vision model we're using only accepts jpg, gif and png files, so I've created a PNG from the first page. Run this command in the terminal:

```bash
llm -m gemini-2.5-flash "what is the license number from this image?" -a md_doc.png
```

You should get something like: "The license number in the image is D25922." as a response.

Now it's your turn: find an image _that you would be ok showing to your grandmother_ and make sure it's a PNG, GIF or JPG file. Drag it into the codespace so you can see it in your list of files, then change the command below so that it refers to your image and asks a different question. Edit the README to do that so I can see your work. How did the LLM do?

```bash
llm -m gemini-2.5-flash "YOUR QUESTION" -a YOUR_FILE 
```

PUT YOUR EVALUATION HERE.


### Audio Models

Another option is to use models that transcribe audio. Listen to [this mp3 file](https://dare.wisc.edu/audio/south-carolina-desegregating-edisto-state-park/) from the Dictionary of American Regional English project at the University of Wisconsin. Click the download link and save the .mp3 file to your computer, then drag it to the file to the list of files on the left. Then, using llm, have the `whisper-large-v3-turbo` model from Groq provide a transcription using that file name.

```bash
llm -m whisper-large-v3-turbo -a YOUR_FILE 
```

How did the LLM do compared to the original transcript?

PUT YOUR EVALUATION HERE.

### How You Could Use AI

Thinking about news archives, write a few examples of how you might use LLMs to help (beyond writing code). Be specific: don't say that you'll use it to accomplish a larger goal. Instead, say how it could perform or improve specific tasks. I encourage you to think big.

PUT YOUR ANSWERS HERE.

### Finishing Up

When you are finished, add, commit and push your work to GitHub and submit the URL of the repository in ELMS.