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

    This is awesome. I already use LLMs to reformat unstructured data, but being able to do it directly from the command line is considerably more efficient. I just grabbed the text from some random DHS .pdf with an unstructured list of FOIA requests. It was something I found the other day while I was f*cking around with the other command line exercise with the ICE FOIA data. The name that appeared in that data — Colin Aamot — appeared 19 times in this 25-line CSV. Listen, I just really like public records. But this was the basis for ProPublica's story about conservative organizations flooding the federal government with FOIAs. I love weird records stories like this (I mean, my dream is to work for ProPublica), so I could see this being really helpful — especially considering that the original .pdf is much, much longer. I presume ProPublica did something similar, because these kinds of records would be a huge PITA to analyze and cross-reference unless you converted them to structured data — and this would make that process so much easier. 

### Vision Models

One of the options we have with Groq is to use a "vision" model that can describe the information in certain types of images. Let's test that out using an image from this Maryland physician discipline report: https://www.mbp.state.md.us/BPQAPP/orders/D002592202.265.pdf. The specific vision model we're using only accepts jpg, gif and png files, so I've created a PNG from the first page. Run this command in the terminal:

```bash
llm -m gemini-2.5-flash "what is the license number from this image?" -a md_doc.png
```

You should get something like: "The license number in the image is D25922." as a response.

Now it's your turn: find an image _that you would be ok showing to your grandmother_ and make sure it's a PNG, GIF or JPG file. Drag it into the codespace so you can see it in your list of files, then change the command below so that it refers to your image and asks a different question. Edit the README to do that so I can see your work. How did the LLM do?

```bash
llm -m gemini-2.5-flash "what is this a social commentary on?" -a double_edged_sword.png
```

RESPONSE:

This meme is a social commentary on the **disruptive and transformative power of Artificial Intelligence, specifically large language models like ChatGPT, on traditional human knowledge, skills, and ways of learning and working.**

Here's a breakdown:

1.  **"5000 YEARS OF ACCUMULATED HUMAN KNOWLEDGE AND KNOWHOW" (the burning house):** This represents the vast body of information, expertise, skills, and established methods that humanity has developed over millennia. The burning house symbolizes this accumulated knowledge and the traditional ways of acquiring and using it being rapidly challenged, potentially rendered obsolete, or undergoing a radical overhaul. It suggests that established systems and practices are being "destroyed" or fundamentally altered.

2.  **"CHATGPT" (the girl with the mischievous/smug look):** This represents the advanced AI itself. The girl's expression, characteristic of the "Disaster Girl" meme, implies that ChatGPT is either directly causing this "destruction" or is a powerful, almost gleeful, witness to the upheaval it brings. It signifies the AI's immense capability to process, generate, and synthesize information in ways that challenge human-centric models.

**In essence, the meme suggests:**

*   **Obsolescence of traditional skills:** Many jobs, tasks, and skills that previously required extensive human knowledge and effort (e.g., writing, research, basic coding, data synthesis) might be heavily impacted or even replaced by AI.
*   **A paradigm shift in learning:** The way we learn, research, and understand information is changing as AI can provide answers, explanations, and content instantly.
*   **The speed of change:** The burning house highlights the rapid and overwhelming nature of this technological shift.
*   **Uncertainty and anxiety:** While AI offers incredible potential, the meme also taps into the underlying anxiety about what this rapid change means for human value, employment, and the future of traditional institutions.

It's a humorous yet pointed observation about the perceived revolutionary (and somewhat threatening) impact of AI on the very foundations of human intellectual and practical achievement.

```bash
llm -m gemini-2.5-flash "whose eyebrow is this?" -a eyebrow.png
```
RESPONSE:

This is a close-up of **Steve Jobs'** eye and eyebrow, wearing his signature semi-rimless glasses.

```bash
llm -m gemini-2.5-flash "did i do this math problem correctly?" -a sophomore_math_hw.png
```
You did the math problem correctly, and your final answer for the equation `y = -2(x + 9)^2 + 9` is correct.

However, there's a slight handwriting ambiguity in one step that could be confusing:

*   You wrote: `-153 = 812 + 9`

It looks like the `a` (the variable you are solving for) was written as a digit `2`, making it look like `812` instead of `81a`.

If we interpret `812` as `81a`, your steps are perfect:
1.  `y = a(x - h)^2 + k` (Correct vertex form)
2.  `y = a(x + 9)^2 + 9` (Correctly substituted vertex `(-9, 9)`)
3.  `-153 = a(0 + 9)^2 + 9` (Correctly substituted the point `(0, -153)`)
4.  `-153 = a(9)^2 + 9` (Correctly simplified `0 + 9`)
5.  `-153 = 81a + 9` (This is the line where `81a` was written as `812`. `9^2 = 81` is correct)
6.  `-153 - 9 = 81a` (Subtract 9 from both sides)
7.  `-162 = 81a` (Correct subtraction)
8.  `a = -162 / 81` (Correct division)
9.  `a = -2` (Correct value for `a`)
10. `y = -2(x + 9)^2 + 9` (Correct final equation)

So, **yes, you did it correctly!** Just be careful with how you write your variables to avoid it looking like a number.

```bash
llm -m gemini-2.5-flash "what answer did i select for question 32? which three questions are crossed out? what kind of dog might this be?" -a dog_on_hw.png
```

PUT YOUR EVALUATION HERE.

    OK, I wanted to give it a meme (obviously). I tried to rickroll it, but it said it couldn’t do .gifs, which is a travesty — Google should really work on that. When that didn’t work, I decided to have the AI analyze a meme making fun of AI. The irony was too much to bear to not do it. And I mean, it’s a pretty good analysis. Gemini definitely explains — and over-explains, one might say — the meme as a commentary on the “disruptive and transformative power of Artificial Intelligence, specifically large language models like ChatGPT.” But my lord was it long — 444 words long, to be specific. And it wasn’t funny, which was disappointing but to be expected. Definitely should have included a no yapping prompt. But yeah, overall, it’s pretty impressive (to me, anyway) that it can analyze and contextualize an image as though it can “see” it.

    But that was too easy. So I decided to test its identification abilities. I grabbed an older photo of Stephen Colbert (from when he was doing the Colbert Report), and gave the LLM a cropped version showing just his eye, arched eyebrow and some of his glasses. When I asked who it was, it said — drumroll, please — Steve … Jobs? Mhm, so close, but yet so far. Wrong Steve. Gemini was really confident about it, too. 

    Alright, new task: Read my handwritten high school math homework. I went searching through my camera roll and found some old photos I took of my math homework in high school. One is just a piece of lined paper with an equation written out. The other shows my dog lying on my homework, but some of the page is still visible. Mind you, I had really weird handwriting in high school. I wrote “a” with a serif top because I wanted to be quirky and special — if you think I’m bad now, high school Cat was a nightmare — and if I was writing fast it sometimes looked like a 2. I knew as soon as I found the picture that it might have an issue with my “a,” so I asked it if I solved the equation correctly. And GODDAMN. First, yes, it went through each individual step of the equation and confirmed that I did the problem right (a feat in and of itself). It’s impressive enough that it can analyze a blurry, poorly lit picture of a handwritten assignment. But it also told me that the way I wrote “a” might be confusing. So not only did it identify the handwriting ambiguity, it figured out what I meant and still determined that I was correct. I mean, shit, that’s impressive. The other photo was definitely harder to read, and it did give the wrong answer for what I put for question 32. I put “C” and it said “A,” but that makes sense because the difference between a “c” and a lowercase “a” is one line, and it’s a terrible photo. However, it did still correctly determine which three questions were crossed out, and, hilariously, it correctly identified what breed of dog Tommy is. Genuinely super impressed overall. Being able to analyze handwritten documents would be exceptionally helpful if you were analyzing, say, notes that a world leader wrote — something featured in the JFK museum in Hyannis, Massachusetts — or declassified documents containing handwritten annotations. The possibilities here are endless, particularly in the context of older records. 

### Audio Models

Another option is to use models that transcribe audio. Listen to [this mp3 file](https://dare.wisc.edu/audio/south-carolina-desegregating-edisto-state-park/) from the Dictionary of American Regional English project at the University of Wisconsin. Click the download link and save the .mp3 file to your computer, then drag it to the file to the list of files on the left. Then, using llm, have the `whisper-large-v3-turbo` model from Groq provide a transcription using that file name.

```bash
llm -m groq/whisper-large-v3-turbo -a speech_audio.mp3
```

How did the LLM do compared to the original transcript?

PUT YOUR EVALUATION HERE.
   
    Mhm, I’m neither here nor there about this one. It’s not terrible, especially given the quality and the accent, and I don't blame it for small things like the name of the state park being wrong — that happens with a lot of transcription services. The main thing is that I still think Otter would have done better. The other thing that made me skeptical — and would make me nervous about using AI for audio transcription — is that some of the mistakes changed the meaning. There’s a pretty big difference between a “moot” case and a “new“ case. It also really struggled with the parts where he misspoke and corrected himself, which made his point about members of the Black community being OK with the state park’s shutdown because it was a blanket policy — meaning no one could enjoy it. But Gemini tripping up on his mention of the golf course made that unclear in the transcript. And it didn’t transcribe filler words like “uh,” which I don’t love — most of these words can, for clarity’s sake, be cut from most quotes in most stories, but I want to be the one making that call. I also need to know how much it changes the quote to take those words out, and whether an ellipsis is necessary. Because sometimes those filler words and repetitions and flubs and delays are contextually important. So, overall, meh. I’d guess it would be more helpful for transcribing higher-quality audio, but I don’t know if I’d choose it over Otter. The only thing I’ll concede is that it would be super helpful in a situation where you need to transcribe multiple recordings in order to do a text analysis. 

### How You Could Use AI

Thinking about news archives, write a few examples of how you might use LLMs to help (beyond writing code). Be specific: don't say that you'll use it to accomplish a larger goal. Instead, say how it could perform or improve specific tasks. I encourage you to think big.

PUT YOUR ANSWERS HERE.

    Hm. OK. Outside of what I already mentioned, I think this would really help with tasks that involve large files or in one way or another require analyzing *multiple* things. That could be one giant .csv or long .pdf or multiple documents of the same type or of different types — whatever it may be, I think using an LLM through the command line would make that process much more efficient. I’m thinking along the lines of a lengthy Senate bill on deadline, multiple years of legislative audits, several years’ worth of tax records, etc. My personal desire would be to use it to analyze media coverage. Like, capture the front pages of specific news outlets — or grab them from the Wayback Machine lol — and analyze what’s being covered, what’s not being covered, the staying power of certain stories vs. stories that drop of the national consciousness. Another one I’d be interested in would be analyzing government email and Slack messages obtained through a records request. Because those records would all be unstructured and not uniformly formatted, so I think AI would be helpful for tagging and reformatting messages into a structured dataset with columns like date, sender, receiver(s), time, type, subject, people mentioned, full text. Same goes for tweets. I’d love to use AI to turn a bunch of X posts from influential figures into structured data that could be analyzed for themes, tone, reach, subject, etc. I’d want to test this out on tax filings with the aim of extracting data for multiple non-profit organizations and analyzing it for trends in investments, salaries, endowments, gifts, travel policies, etc. Ooh, and it would be great for putting in multiple versions of a proposed regulation or law in order to identify the substantive changes. Realistically, I’d want to try this with everything — my thing is public records, but going through them records hand is often an error-prone and time-consuming process. Obviously, for our purposes, the ideal would be to use it to group articles en masse by topic, then analyze the content of those articles to create a structured dataset, categorized by sub-topic, that includes summaries of the issues at hand, timelines, descriptions of the key players involved, each side’s arguments and follow-up angles. And again, the command line-AI pipeline makes this task seem slightly less impossible because this process will allow us to quickly analyze and compile datapoints from multiple articles simultaneously without having to switch back and forth between the documents, a chat interface, a spreadsheet, R, etc. 

### Finishing Up

When you are finished, add, commit and push your work to GitHub and submit the URL of the repository in ELMS.