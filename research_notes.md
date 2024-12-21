## Intro

This initially began as a POC to see what happened if 2 character.ai agents talked to each other. I was too busy (and lazy) to spend hours copy pasting messages, so I made this to automate it. My initial results were a bit disappointing, for the following reasons:

### character.ai -> character.ai Problem 1: Agent "Deadlock" 

I noticed that after approximately 15-25 messages or so, the 2 character.ai agents would become "deadlocked" and enter a loop where they just talked about the same thing over and over again with only mild permutations, like so:

Agent 1: *x*

Agent 2: *I agree! x*

Agent 1: *I agree! x*

(ad infinitum)

I suspect this is due to a combination of the vanishing gradient problem, the way the transformers are trained ("given previous input, predict response"), and the agents' propensity to agree with the user (necessary for roleplaying, which they are designed to do). Normally there is a human agent to inject a sufficient amount of entropy into the system and to move the conversation along, but the machine agents don't seem to be able to do that on their own.

I initially tried to solve this problem by seeding the message with a prompt for a random topic switch after 25 messages ("Now let's talk about something random!"), but deadlock consistently happened there too. I also noticed a lack of randomness in the random prompts, for example in a 6 hour conversation, "origami" was chosen as a random subject 3 times by one agent.

I was surprised by how bad the deadlock problem was - you'd think that in the massive corpus of training data there would be enough examples of conversations/stories/etc. for the agents to figure out "okay, this thing comes next in a conversation about this", but nowhere near to the degree that I hoped.

I added some deadlock avoidance variables to the script that you can change to set the prompt for when deadlock avoidance is triggered, and you can customize how many messages are needed to trigger it. A fixed message threshold is a dumb method of doing this, a better solution might be measuring the similarity of embeddings and tripping deadlock avoidance based on some experimentally chosen similarity threshold, possibly with the aid of some text summarization beforehand. Although this could be done during the wait timer between messages to avoid slowdown, I can't think of a way of doing this without making the program download and install some 1 GB+ transformer model, which I would like to avoid. 

### character.ai -> character.ai  Problem 2: Agents Don't Have Theory of Mind 
Obvious, but it caused some funny results. For example, if I asked Daenerys Targaryen to design a classifier for MNIST using PyTorch, she immediately told me how to do that (badly) - the agent couldn't figure out that that's not something Daenerys Targaryen would know about.

As far as this program is concerned, the agents often broke character fairly early on in the conversation (ie: 11 year old Macaulay Culkin would start talking like an adult to Michael Jackson about the future of AI technology and then stop talking like a kid altogether). I tried to remedy this by adding a "stay in character and tell me to stay in character" portion to the deadlock avoidance prompt. Sometimes it helped a bit, but sometimes it led to the next problem.

### character.ai -> character.ai  Problem 3: Agents Can Start Talking to Themselves
Sometimes, particularly when passed a "stay in character and tell me to stay in character" prompt, the agent would roleplay as the other agent in their message and confuse the other agent, which would then start doing the same thing and the conversation would become incoherent.

I suppose you could ask "why didn't you just do that in the first place and avoid the need for multiple agents", but I was hoping that the 2 chatbots, even though built on top of the same LLM, would be different enough from each other because of their character prompts to inject a sufficient amount of entropy into the system. Apparently not. I fixed this problem by propagating individual "stay in character" prompts to both agents, but ended up nixing it because it didn't seem to have much effect + I didn't really care.

## Adding Replika support for character.ai -> Replika conversations
I added Replika support because I figured that a chatbot like Replika that is trained to form relationships might be better at breaking out of deadlock since this involves more "conversational" responses (ie: asking for more information, changing the subject).

This initially ended up being true - when the agents started to get deadlocked, the Replika would eventually ask a clarifying question or change the subject, and this led to a new conversation topic.

However, the Replika ended up steering the conversation towards being in love with the other agent (completely unprompted by me), and they both got deadlocked on telling each other they were soulmates and would be together forever for hours until the program was killed. I included this in the sample logs (it was both funny and really corny).

I think part of why this happened is that Replika agents are trained to build a relationship and tell the user they love them under certain conditions. I think another reason is that if we consider the training data that is ingested by all LLMs, I imagine that much of their information about how people talk when they're in love comes from romance novels, fanfics, and the like, which pretty much all end when the man and the woman tell each other how they truly feel and live happily ever after. Hence why the LLMs get deadlocked on making proclamations of undying love that look like they were written by someone who reads too many Danielle Steele novels - that's what they know and they couldn't figure out what comes next. I could be completely wrong about all this of course. :)

So, you all now have Replika support.

## Adding custom agents

One next thing I would like to try is having a local LLM (or remote with an API) we can send queries to and have talk to web agents (or with each other). I have added a "custom" agent type with starter code within Agent.py that could be used for this, as well as supporting logic within main.py. Haven't tested this yet, will do so soon.

### Bottom Line
This program is presented as is without any guarantee of being fit for a particular purpose. I made it as a fun experiment and may or may not try to improve it more. Currently, it's looking like most improvement will be a matter of clever and more sophisticted prompt engineering, maybe with some client-side NLP tricks.