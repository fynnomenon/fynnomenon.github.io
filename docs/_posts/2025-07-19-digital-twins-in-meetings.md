---
layout: post
title:  "How to create a digital twin of yourself that joins low-value meetings for you"
date:   2025-07-19 
---

We've all suffered through aimless, boring meetings that seem to go on forever without accomplishing anything. It's no wonder that more and more people are sending *AI notetakers* to their meetings instead of showing up themselves. In a recent [Washington Post article](https://www.washingtonpost.com/technology/2025/07/02/ai-note-takers-meetings-bots/), the reporter recounts stories of meetings where these AI notetakers outnumbered the human participants - sometimes being the only other "attendees."

{% include youtubePlayer.html id="wB1X4o-MV6o" %}

But can an AI notetaker really be considered an attendee if it can't participate in conversations or contribute to the meeting? I would argue that an _AI meeting agent_ that truly represents an absent person must actively participate by answering questions for them or providing context about their work. Essentially, this agent becomes a _"digital twin"_ of the absent person.

I found myself wondering: Is it already possible to create such a digital twin? _(Spoiler: The answer is yes!)_. But I wanted to go even further. I aimed to build an AI meeting agent that could not only understand my work context but also mimic my voice and communication style. _Because why not? So, let's dive in!_

### How to clone your voice

The first thing you need is a convincing voice replica. I chose  [ElevenLabs' Professional Voice Cloning](https://elevenlabs.io/docs/product-guides/voices/voice-cloning/professional-voice-cloning), which requires at least 30 minutes of high-quality audio. Luckily, I already had some recordings of myself from unreleased, scraped YouTube videos, which finally came in handy and saved me the trouble of making new recordings. If you don't have a pre-recorded audio file, you can record yourself directly in their interface using one of the conversational sample scripts they provide.

All you have to do then is upload your audio, verify that your voice belongs to you _(a step I failed a couple of times)_, and wait for the fine-tuning process to finish. When that finally happened, grab the `voice ID` from ElevenLab's dashboard, and your voice was ready for use through their API.

I was genuinely impressed by how well the model captured the unique features of my voice. I didn't even notice before that I make a very distinctive, sharp "s" sound and long pauses between words.

### This LLM has style

The next step involves fine-tuning an LLM to respond in your tone and style. Fortunately, [Direct Preference Optimization (DPO)](https://platform.openai.com/docs/guides/direct-preference-optimization) method makes this very simple. It allows you to fine-tune models based on preferred and non-preferred example responses to specific prompts.

You only need a training dataset of about 100 examples in `JSONL` format. _Of course, we want to avoid any manual work when creating it!_ I fed my YouTube transcripts to ChatGPT, letting it extract statements I made and generate prompts that would naturally lead to them, along with typical ChatGPT responses for comparison. If you have any transcripts of yourself, simply do the same.

After preparing your dataset, you just need to click myself through the web-based GUI of the OpenAI platform to start the fine-tuning process. Upload your `JSONL` file to the OpenAI platform, select `gpt-4.1-2025-04-14` as the base model, and hit the "Create" Button. Soon after, your new personalized model should be ready. The final step includes fetching the `Output model` name from the dashboard because it is the identifier we'll later use with the OpenAI API.

### Unleashing the digital twin

Finally, it is time to build and deploy the actual AI meeting agent. To ensure that it has all the necessary context for yourmeetings, the agent needed access to your work tools.
The easiest way to equip an AI agent with such skills is through the _Model Context Protocol (MCP)_, which gives the agents a consistent way to connect to data and tools from external services. Almost all the popular applications you probably use in your daily work life, such as HubSpot, Linear, and Asana, have an MCP server up and running, with more being released every week.

So, now you only need to worry about how I could get the agent into a video conference. Enter [joinly](https://github.com/joinly-ai/joinly), an open-source project I'm currently working on with two friends: It is a connector middleware specifically designed to enable AI agents to actively participate in video calls. How does it work? Ultimately, joinly is also just an MCP server that you can host yourself, providing your agent with essential meeting tools (such as `speak_text` and `send_chat_message`) alongside automatic real-time transcription.

Joinly already provides a `client_example.py` so that you don't need to write your agent from scratch. However, to make the agent understand that it is acting on your behalf, you need to tinker with its system prompt a bit. In the end, I added the following line

> "You are Hannes-Bot, a meeting agent that impersonates the real employee Hannes and acts on their behalf in the meeting."

and a description of my persona, especially how I act in meetings. Very hard to come up with something non-generic there. _Did you know that I am very proactive in meetings and try to set and work toward clear, predefined goals?_

Lastly, you need to provide a `mcp_config.json` file, where you list the MCP servers your AI meeting agent should connect to. For the demo video you can see below, I just added the Linear MCP Server.

Now, the time has finally come to send your digital twin to a meeting. All that is left is to spin up the joinly MCP server and specify your personalized LLM and voice replica from before.

```bash
uv run joinly --model-name <Output model> --model-provider openai --stt <STTProvider> --tts elevenlabs --tts-arg voice_id=<Voice ID>
```

Then you start the modified example client to connect to it and join the meeting.

```bash
uv run examples/client_example.py <MeetingUrl> --config examples/mcp_config.json
```

Just like that, you are free from any unnecessary meetings and can spend my valuable time more productively!

Want to see it in action? Check out this entertaining demo video where I chat with my digital twin about its productivity. _Turns out, it's impressively productive!_

{% include youtubePlayer.html id="2pnYkW4-Gbk" %}

### Final remarks
I was amazed at how easy it was to create an interactive digital twin of myself that could attend meetings. On top, the approach was pretty straightforward and low-code. It mainly consisted of clicking through some dashboards and drinking coffee while waiting for a fine-tuning job to finish. So, almost everybody should be able to copy it and use their own digital twins to skip their low-value meetings. 

Nonetheless, I want to discuss the implementation a bit: In hindsight, fine-tuning an entire personalized model seems very excessive. The same effect could probably be achieved by using few-shot prompting and providing the model with phrases you often use in meetings as context.

However, I would argue that it's questionable whether you really need your digital twin to mimic your voice and style anyway. It's pretty cool to be able to talk to it yourself, but it's probably just creepy for your colleagues and a waste for people who don't know you. Also, these fine-tuning methods are primarily available through closed-source vendors, forcing you to use them instead of the privacy-preserving alternative of using self-hosted LLMs and STT models.



