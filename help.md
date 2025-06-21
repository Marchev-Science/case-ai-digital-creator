# Hackathon Challenge: AI-Driven Content Generator with n8n & GPT

## Introduction  
In this hackathon challenge, you’ll build an **automated content generation and dissemination system** using **n8n** (a no-code workflow tool) and **GPT/LLM** technology. The goal is to automatically create short **vertical videos** (for Instagram Reels, TikTok, YouTube Shorts, etc.) on trending topics, complete with voiceover, music, visuals, captions, and even automated posting. This guide will walk you through a step-by-step plan – from picking a hot topic to publishing the finished video – using mostly **free or open-source tools**. By the end, you’ll have a workflow that can turn a simple topic idea into an engaging video *and* share it on social platforms with minimal manual effort.

**What You’ll Learn:** 
- Setting up n8n locally and creating an end-to-end automation workflow.  
- Integrating GPT (Large Language Models) to generate scripts and text content.  
- Using Text-to-Speech (TTS) for voiceovers and free audio libraries for background music.  
- Incorporating images or video clips (via free APIs or AI generation) as visuals.  
- Using **FFmpeg** for video assembly: combining voice, music, visuals, and captions into a 9:16 video.  
- (Optionally) Automating the posting of videos to social media and optimizing posts with titles/hashtags.

This challenge is **beginner-friendly** (no coding required aside from a few configurations) but also leaves room for intermediate participants to extend it with more advanced features. Let’s dive in!

## Challenge Overview and Requirements  
Your automated system should perform the following key tasks (we’ll address each in detail):

1. **Select Trending Topics Autonomously:** Choose engaging or trending content topics in your chosen domain. *First, you as participants will manually pick a general domain or niche (e.g. tech news, fitness tips, travel). The system will then find trending or popular specific topics within that domain.*  
2. **Generate a Script with GPT:** Use a GPT-based Large Language Model to write a short script or narrative based on the chosen topic. This will be the “content” of your video (the narration/captions).  
3. **Convert the Script to a Vertical Video:** Turn the script into a video by generating:
   - **Voiceover audio** (spoken version of the script via TTS).  
   - **Background music** (using royalty-free tracks or AI-generated tools).  
   - **Visuals** (relevant stock images, AI-generated images, or short clips/animations).  
   - **On-screen captions** (text from the script, so viewers can read along).  
   All of these elements will be automatically combined into a video format suitable for mobile viewing.  
4. **Format for Reels/Shorts:** Ensure the video is in a **9:16 vertical format** (e.g. 1080×1920 resolution) with proper encoding, so it’s optimized for Instagram Reels, TikTok, YouTube Shorts, Facebook, etc.  
5. **Publish (Semi-Automated):** Prepare the content for posting on social platforms. Due to API limitations, full auto-posting may be tricky, but we’ll discuss options – from simulating a post to using third-party tools or platform APIs to upload the video.  
6. **Optimize for Engagement:** Before posting, generate an attention-grabbing title or description and relevant **hashtags** for the video. Also consider the best posting times or scheduling if applicable. GPT can assist in creating catchy captions and hashtag suggestions to boost visibility.

Throughout the guide, we’ll highlight **free and open-source tools** to accomplish each step, and mention optional paid services that could enhance your results. We encourage creativity – participants can modify prompts, try different APIs, or add features – but we’ll ensure the core workflow is feasible on a typical local machine with limited resources (no need for a supercomputer or expensive subscriptions).

## Tool Setup and Key Technologies  

Before building the workflow, make sure you have the following ready:

- **n8n (self-hosted):** Set up n8n on your local machine. You can install it via npm (`npm install -g n8n`) and then start it with `n8n start` to access the editor in your browser. Alternatively, use the n8n Desktop app or a Docker image. **Note:** You should run n8n locally (or on a VPS) because n8n Cloud won’t allow executing FFmpeg for video processing. *Ensure that FFmpeg is installed on your system and accessible*, since the workflow will invoke FFmpeg commands for video creation.  
- **OpenAI API (or another LLM API):** Sign up for an API key to use GPT-3.5 or GPT-4 through OpenAI’s API. n8n has a built-in OpenAI node which you can configure with your API key. (If you prefer not to use OpenAI, you could use an open-source LLM via an API, but for simplicity we’ll assume OpenAI’s API.)  
- **Text-to-Speech tool:** For voiceover, choose a TTS solution. Free options include:
  - **Voice RSS API** – a free web service that converts text to speech and returns an audio file. You just need to sign up for a free API key and make an HTTP request with parameters like language and text. (We’ll show an example using this.)  
  - **Offline TTS** – if you prefer, you could use an offline tool like eSpeak or Festival via a command-line call, but voice quality is robotic.  
  - **Optional (Paid)**: **ElevenLabs** provides very natural voices (with a free trial tier), or Google Cloud Text-to-Speech/AWS Polly (both have free quotas). These require API credentials and integration but can be used for higher-quality voice.  
- **Audio Source for Music:** Get at least one background music track. You can use any royalty-free music. For automation:
  - Use a **free music library** like Pixabay, Free Music Archive, or YouTube Audio Library (these might require manual download unless they have an API). 
  - Or use a simple **AI music generator**. (One example: *Mubert* offers an API for AI-generated music, but it might not be entirely free. For this hack, having a static music file might be easiest.)  
  - Alternatively, you could skip music or just keep the voiceover if adding music proves too complex – but adding a subtle music bed can make the video feel more polished.  
- **Images/Video for Visuals:** We’ll automate fetching visuals:
  - **Pexels API (Free)** – Pexels provides free stock photos and videos via API. You can request a developer API key (instant approval) and use an HTTP node in n8n to search their library. For example, a GET request to `https://api.pexels.com/videos/search?query=nature&orientation=portrait&per_page=1` (with your API key in the header) will return a JSON with a vertical video related to “nature”. You can specify `orientation=portrait` to ensure the video is vertical (9:16). Similarly, `search?query=YOUR_TOPIC` can get relevant images. We’ll use Pexels for demonstration since it’s free and has no cost.  
  - **Alternative Free Sources:** Unsplash (photos API), Pixabay API, or even a Google Images search via a scraping API. Any source that can provide an image URL or video URL for a given topic will work.  
  - **AI-Generated Images (Optional):** If you want specific or stylized visuals, you could generate images via an AI API (e.g., OpenAI’s DALL-E or Stability AI’s API for Stable Diffusion). This can be integrated in n8n as well, but it may be slower or require credits. For speed, using stock images is fine.  
- **FFmpeg:** This powerful open-source command-line tool will be the workhorse for video assembly. Ensure it’s installed and added to your PATH so n8n can invoke it. We will use n8n’s **Execute Command** node to run FFmpeg commands to create the video from our media assets. (No need for a separate video editor – FFmpeg can handle combining video clips or images, layering audio, and adding text overlays in one go.)  
- **Social Media Accounts/Keys (Optional):** If you plan to actually auto-publish:
  - YouTube API – to upload videos (requires OAuth setup in Google Developer Console, and n8n has a YouTube node or you can use an HTTP request to YouTube’s upload endpoint).  
  - Instagram/Facebook – Meta’s Graph API allows posting Reels to Instagram or videos to Facebook pages, but requires app setup and might be read-only for personal profiles.  
  - TikTok API – exists but is limited; likely you won’t fully automate TikTok uploads in a short hackathon due to approval needed.  
  - **Third-party multi-uploader (Optional):** The n8n community suggests services like **upload-post.com**, which can take your video and programmatically post to multiple platforms. This can simplify the publishing step if you have credentials for it. For this guide, we’ll focus on preparing content and simulate publishing, but you can choose one platform (like YouTube) to automate as a proof of concept.

Now that everything is set up, let’s break down the workflow into steps and build it up.

## Step 1: Selecting Trending Content Topics  
The first step is to get a *trending or engaging topic* in your chosen domain. For example, if your domain is “health & fitness,” a trending topic might be “High-Intensity Interval Training benefits” or “Top 5 Keto Diet Myths”. We want this to be autonomous if possible, so the system can pick something timely that people care about.

**How to find trending topics?** Here are a few approaches:

- **Google Trends:** Google provides an RSS feed of daily trending search topics. By fetching the Google Trends RSS, you can get a list of currently popular search queries. In n8n, you can use an **HTTP Request** node to GET the feed URL (e.g. for daily trends in the US: `https://trends.google.com/trends/trendingsearches/daily/rss?geo=US`). The response can be parsed (n8n has an XML node or you can parse XML in code) to extract trending keywords. *Note:* You can filter these by your domain or have GPT evaluate which trend from the list fits your chosen domain. For example, if your domain is tech, you could have GPT or a function filter the trends for tech-related terms.  
- **Social Media Trends:** If an API is available, you could query Twitter trending topics or Reddit’s popular topics. However, Twitter’s API is no longer free, and scraping it is non-trivial. Reddit has an JSON feed for popular posts which could be filtered by subreddit. As a simpler approach, you might skip direct social trends and rely on Google Trends or a news API.  
- **News APIs:** Using a general news API (like NewsAPI or Bing News search) with your domain as a query could return current headlines. For instance, search “latest fitness trends 2025” or similar and take a top result’s topic. This requires an API key (NewsAPI has a free tier).  
- **Manual fallback:** Since this is a hackathon, you can also allow a manual input in n8n (use an **Inject** node or simply type a topic into a Google Sheet/n8n UI) to choose the topic if the automation is tricky. The challenge encourages autonomy, but it’s okay to seed the system with a topic if needed.

**Implementing in n8n:**  
1. *Choose Domain:* You might start with a **Set** node or a simple **Webhook Trigger** where the domain or a broad category is provided (e.g., you set a variable `domain = "technology"`).  
2. *Fetch Trends:* Use an **HTTP Request** node to fetch a trend source. For Google Trends RSS, no auth is needed. For example, the n8n template for Google Trends shows retrieving trending keywords along with related news links. The HTTP node would get the RSS XML.  
3. *Parse/Select Topic:* Add a **Function** (Code) node or an **OpenAI** node to process the fetched trends. One idea is to feed the list of trending topics into GPT and ask: *“Given these trending topics, which one could yield interesting content in the domain of {your domain}? Return just the chosen topic.”* GPT can intelligently pick one that matches your domain niche. Alternatively, if you have structured data (like a list of topics with some scores), you could simply pick the first or a random one that matches certain keywords.  
4. *Output:* The result of this step should be a **single topic/title** that will drive the content generation. For example: *“10 Benefits of HIIT workouts”* or *“AI Advances in Self-Driving Cars”* – depending on domain.

**Note:** If using GPT to select the topic, ensure you don’t accidentally pick a too-general or uninteresting topic. You can refine the prompt with criteria (e.g., “choose a topic that is currently trending and will be engaging to a general audience interested in X domain”). If using Google Trends directly, you might already get very hot topics (but sometimes unrelated to your domain). Feel free to experiment here – this part can be as simple or complex as you want. For the hackathon, even a static topic or a random pick from a prepared list is acceptable if time is short.

## Step 2: Generating a Script with GPT  
Now that we have a topic, the next step is to generate the **script or narration** for our short video. This will be a textual content piece – usually a few paragraphs or bullet points that will be spoken and shown as captions.

**What makes a good script for Reels/Shorts?** It should be **short (30-60 seconds)**, engaging from the start (hook the viewer in the first few seconds), and possibly include a call-to-action or conclusion. It can be informative or entertaining, depending on your content style.

**Using GPT for script writing:**  
In n8n, you can utilize the **OpenAI node** (or an HTTP node calling the OpenAI API endpoint) to have GPT-3.5 or GPT-4 create the script. Here’s how: 

- **Prompt Design:** Craft a prompt that provides context and instructions to GPT. For example: *“Write a concise, engaging script for a 30-second vertical video about {Topic}. Start with a hook to grab attention, then include 2-3 key points, and end with a catchy conclusion. Write in a friendly, conversational tone. Make sure the script is around 100 words (suitable for ~30 seconds).”* You can also ask for a **title** or **hashtags** in the same prompt or separate prompts if you like (though we’ll cover hashtags in a later step).  
- **OpenAI Node Configuration:** In n8n’s OpenAI node, select the **Chat** or **Completion** mode (ChatGPT models work well). Provide your prompt and the topic variable as input. Choose the model (e.g., `gpt-3.5-turbo` for cost-efficiency or `gpt-4` if available and you want better quality). Set temperature (maybe around 0.7 for creativity or lower if you want a more factual script). The node will return the AI’s response text.  
- **Output Processing:** Capture the GPT output which should be the script text. You might want to do a quick check or trim any extra length. Typically, GPT might produce a few paragraphs. Ensure it's not too long – you can include in the prompt a word limit or ask it to keep it within 4-5 sentences. In our case, GPT will act as our “script writer” for the video content. 

**Example:** If the topic was “Benefits of HIIT workouts”, GPT might return something like: *“**Hook**: Tired of long workouts? Here’s why 15 minutes of HIIT can outdo an hour of jogging! **Point 1**: HIIT burns calories even after you’re done… **Point 2**: It boosts your heart health... **Conclusion**: Ready to sweat smarter, not longer? Give HIIT a try! #fitness #HIIT”*. This is an example of a script with a clear beginning and end and even a couple of hashtags. You can refine the style as needed.

Remember to **preserve this script text** in the workflow for subsequent nodes. In n8n, the text can be passed along as JSON fields (e.g., `{ "script": "text here..." }`). The upcoming steps (TTS, captioning) will use this.

## Step 3: Converting the Script into a Video  
This is the most involved step – we take the script and produce a video complete with voiceover, visuals, music, and captions. We’ll break this into sub-tasks:

### 3.1 Voiceover Generation (Text-to-Speech)  
To give our video a voice, we need to convert the script text to spoken audio. We’ll use a Text-to-Speech API for this. 

**Using Voice RSS API (Free method):** VoiceRSS is a straightforward choice for quick TTS:
- **API Call:** VoiceRSS requires a GET or POST request with your API key and text. For example:  
```

GET [http://api.voicerss.org/?key=YOUR\_API\_KEY\&hl=en-us\&c=MP3\&src=Hello](http://api.voicerss.org/?key=YOUR_API_KEY&hl=en-us&c=MP3&src=Hello), world!

```
This would return an MP3 audio saying “Hello, world!” in English. We specify `c=MP3` to get an MP3 file, and `hl=en-us` for language/accent. You can also choose a voice (`&v=` parameter) if multiple voices are available for the language.  
- **In n8n:** Add an **HTTP Request** node after the script is generated. Configure it to call the VoiceRSS endpoint, method GET, and include query parameters for `src` (set this to the script text, or if the text is long, you might send one sentence at a time – more on that shortly), `hl` (language code, e.g. en-us), `c` (codec, use MP3), and your `key`. Mark the node to expect a binary response (since it will return audio data). n8n can store this binary (e.g., as `data.mp3`).  
- **Handling length:** The free VoiceRSS API might have a character limit (e.g. 300 characters per request). If your script is longer, you have two options: (a) split the script into chunks (sentences) and call the API for each chunk, then later concatenate the audio; or (b) ask GPT to output a very concise script under the limit. For simplicity, try to keep the script short enough, or consider splitting it into maybe 2 parts if needed. Merging audio parts can be done with FFmpeg if necessary (e.g., using a simple concat command).  

**Alternative TTS options:**  
- If using **Google Cloud TTS**: You’d set up a service account and use an HTTP node to POST to Google’s TTS API with your text and voice settings (Google returns base64 audio data). This yields great voices but requires more setup.  
- If using **ElevenLabs (optional)**: They have a simple HTTP API too – you POST the text and voice ID and get audio. Quality is excellent (e.g., realistic human voices), so if you want to impress and have an API key, you could use it. Just mind the character limits and rate limits (free tier allows a few thousand characters per month).  
- If you absolutely cannot use an API, a hacky approach is to use **gTTS (Google Translate TTS)** by running a small Python function (n8n has a Code node that can execute JavaScript, but not Python directly – you’d need to call an external script or service for that). Probably not worth the trouble in a short hack; stick to a web API.  

At the end of this step, you should have an **audio file** (MP3 or WAV) of the voiceover saved in n8n. This will serve as the narration for the video. 

💡 *Tip:* Use a neutral or cheerful voice that matches the content. VoiceRSS’s default might be okay, but test a snippet. Some APIs let you choose male/female voices or different accents. Also, ensure the spoken speed is okay – some APIs allow adjusting speed (`&r=` in VoiceRSS). The default is usually fine for a short script.

### 3.2 Background Music Integration  
Adding background music can greatly enhance the video’s appeal, making it more dynamic. Since we plan to automate, we need a source for music that doesn’t require manual selection each time (unless you just use one track universally).

**Options for music:**
- Use a **pre-downloaded track**: The simplest path is to include a royalty-free music file in your project and reference its path in the FFmpeg command. For example, a calm instrumental loop or an upbeat jingle depending on your content vibe. You can just use the same music for all generated videos in the hackathon context.  
- Use an API to fetch a music clip: Free music APIs are not common. One creative idea: use the **Freesound API** to fetch a random ambient sound or music loop by keyword (requires free API key, and the clips might be short or variable). But due to time, probably stick with a local file.  
- **AI Music generation (optional):** If someone is adventurous, they could try a service like Mubert (which provides short AI music via API with a free tier) or even generate MIDI and synthesize – but that’s beyond our scope.  

**How to integrate in n8n:**  
If you have a music file ready (say `background.mp3`), you can do one of two things: 
1. Store it locally and ensure FFmpeg can access it by file path. For example, if n8n is running locally, you might reference a path like `/home/user/music/bg.mp3`. (If using Docker, you’d mount a volume.)  
2. Or host it somewhere accessible via URL and use an HTTP Request node to download it as binary in the workflow (similar to images). 

During the FFmpeg assembly, we will mix this music audio with the voiceover audio. Typically, you might want the music at a lower volume so it doesn’t overpower the voice. FFmpeg can adjust volume of inputs easily (we’ll show the command later).

For now, confirm you have a music track and know its location or URL. We won’t actually process it until the assembly step.

### 3.3 Visuals and Imagery  
Visuals are the background of your video – what the viewer sees as the narration plays and text captions appear. Depending on your approach, you can use **static images**, a **slideshow of images**, or even short **video clips**. 

For a beginner-friendly route, we recommend using one or a few **static images** or illustrations that relate to the script. More advanced participants could use multiple images or a video clip.

**Using Pexels for images/videos:**  
As mentioned, Pexels API is a great resource. Here’s how you might use it:
- After generating the script (or you could even feed the script to GPT to get image ideas), decide on a keyword for visuals. You could use the main topic or a key theme. For example, if the topic is HIIT workouts, keywords might be “exercise” or “gym”. If the topic is about self-driving cars, keyword “self-driving car” or “futuristic car”.  
- Use an **HTTP Request** node to call the Pexels **Photo search** or **Video search** endpoint with that query. For example:  
```

GET [https://api.pexels.com/v1/search?query=exercise%20fitness\&orientation=portrait\&per\_page=1](https://api.pexels.com/v1/search?query=exercise%20fitness&orientation=portrait&per_page=1)

```
(for photos) or  
```

GET [https://api.pexels.com/videos/search?query=exercise\&orientation=portrait\&per\_page=1](https://api.pexels.com/videos/search?query=exercise&orientation=portrait&per_page=1)

```
for videos, along with the required `Authorization: API_KEY` header. We add `orientation=portrait` to get vertical-friendly content. The response will contain an array of results with URLs.  
- Parse the response (the HTTP node will output JSON). For photos, you’ll get an image URL (usually multiple sizes; the `original` or `large` size URL is good). For videos, you get video files (different resolutions). Pick one – preferably a short clip. Pexels video API returns clips typically 15-30 seconds which is perfect, and provides links to MP4 files.  
- Once you have the URL, use another HTTP node (or set the first one to **Download** the file) to fetch the binary data of the image/video. In n8n, you can toggle the HTTP node to return binary and it will download the file. Now you have the visual asset in the workflow (e.g., an image file stored in memory).

**Choosing static vs multiple visuals:**  
For simplicity, you can proceed with one background image or one video clip that covers the entire script’s duration. Many motivational or quote videos simply show one nice background or a looping clip while the text overlays. If you want, you could fetch several images (e.g., 3 images) related to different parts of the script and then later arrange them sequentially (like a slideshow). However, doing multi-scene video requires concatenating clips or switching images at certain times, which complicates the FFmpeg command (you’d have to split the audio and align each image with a segment). As a beginner/intermediate, you can skip that complexity and just use one visual – the captions and voiceover will carry the content.

**AI-generated visuals (optional):**  
Another fun twist: you could input the script or topic to an AI image generator. For example, use DALL·E via OpenAI (the OpenAI node can generate images if you have credits), or an API like Stability’s Stable Diffusion. The workflow could say “create an illustration for [topic]” and then use that image. This adds cost (OpenAI image generation isn’t free) and time (few seconds more per image). Pollinations.ai is mentioned in an n8n template as a source for creative visuals. This is optional – a stock image is usually faster and totally free. If you do try it, ensure you handle the image output (the API will give a URL or base64 image).  

At this point, you should have either:
- A background **image file** (e.g., `background.jpg`) and its resolution. If it’s not already 1080x1920, don’t worry – FFmpeg can scale it to fit.  
- Or a **video clip file** (e.g., `clip.mp4`). If the clip is longer than your voiceover, we’ll trim it; if shorter, we can loop it or freeze-frame it (again, FFmpeg can handle that with `-stream_loop` or image looping).

### 3.4 Adding On-Screen Captions  
Captions (subtitles) are crucial for engagement on silent autoplay platforms. We want the spoken words of the script to also appear as text on the video. There are a few ways to do this, varying in complexity:

**Basic static caption:** Easiest method is to just put all the script text on screen for the entire video. However, this is not ideal if the script is long – it would appear as a big block of text. But if your script is just one or two short sentences (like a quote), it can work (as seen in the n8n quote video template which overlays a quote and author on the video). For more than a couple of lines, it’s better to time the text.

**Segmented captions:** A better approach is to split the script by sentence or phrase and display each at the right time as the voiceover speaks it. This is essentially creating subtitles. In a perfect scenario, you’d have timestamps for each word (like from a speech-to-text alignment or from the TTS if it provides timing). Some TTS services do provide timestamp info (e.g., Amazon Polly can give viseme timestamps). If not, you can approximate or manually split evenly.

For hack purposes, here’s a reasonable strategy:
- Split the script into a list of sentences (you can do this in a Function node using something like `script.split(/[.?!]/)` to break at punctuation, for instance). Trim empty entries. Now you have, say, 3-5 sentence strings.  
- We know the total duration of the voiceover audio. If using FFmpeg later, we can get the duration by a quick analysis (or since you generated the audio, you might know length – some APIs give it, or you can run a quick FFmpeg metadata read). Let’s assume the audio is 30 seconds. If you have 3 sentences, you might allocate roughly 10 seconds per sentence, or weighted by their length. Alternatively, you could generate an SRT subtitle file with equal segments.  
- One simple way: when calling the TTS for each sentence separately (if you did chunk it for the TTS API limit), you actually have separate audio files per sentence. In that case, you *do* know durations of each chunk (FFmpeg can concatenate them, and you can overlay each text for that chunk’s duration). But concatenating audio seamlessly and syncing captions gets complicated in one FFmpeg command. 

To keep things simpler, we might choose to **overlay the entire script text at once** but stylize it and place it so it’s readable. Or overlay one sentence at a time with manual timing if possible.

**Using FFmpeg’s drawtext filter:**  
FFmpeg has a filter called `drawtext` which can burn text onto video frames. We can specify font, size, color, position, and even timing (using `enable` expressions to show/hide text at certain times). In the n8n Thai quote example, they used drawtext with a specific font to overlay the quote on a 9:16 video. They also ensured the chosen font supports the language characters. For our case:
- We’ll use a standard font (or you can specify a font file if you want a custom look; ensure the font file is accessible to FFmpeg). 
- We might set the text to appear line by line. If not doing timed, just input the whole script. If doing timed per sentence, we can create multiple drawtext filters each with an `enable='between(t,start_time,end_time)'` expression to only show during that interval (where `t` is the timestamp in seconds). For example, if sentence1 from 0-10s, sentence2 from 10-20s, etc. You’d supply those times manually or via variables.  
- Given the time constraints of a hackathon, you might choose to simply display text without precise per-word timing. That’s acceptable. It still meets the criteria of having captions, even if they’re not karaoke-style synced. 

We’ll proceed with a simple method: in the FFmpeg command, include one drawtext filter for the whole script or for the main points. We’ll position it in the **center bottom** or center of the video, and choose a readable color (often white text with a black semi-transparent box behind it works well). We can also add an outline or drop-shadow via drawtext options to ensure it’s legible over the background.

*(If you prefer not to burn in subtitles with FFmpeg, another option is to generate an .srt file and upload that alongside the video on platforms like YouTube. But Reels/TikTok don’t accept separate subtitle files at upload time, they need them burned in or you use their editor. So burning in is the way to go for multi-platform.)*

### 3.5 Assembling the Video with FFmpeg  
Now comes the “movie magic” – combining voiceover audio, background music, visuals, and text into the final video file. We will construct an FFmpeg command that does the following:
- Takes the background visual (image or video) as the video input.  
- If it’s an image, we tell FFmpeg to treat it as a video stream (e.g., loop a single image for the duration of the audio).  
- Takes the voiceover audio as the primary audio track.  
- Takes the music audio as a second audio track, at a lower volume, and mixes it with the voiceover.  
- Applies the `drawtext` filter to overlay our script text (or captions) on the video frames.  
- Ensures the output dimensions and encoding are correct for vertical format (1080x1920, H.264 video codec, AAC audio codec, MP4 container).  

Let’s break down parts of the command (note: you will assemble these in an **Execute Command** node in n8n):

**Video Input (background):**  
- If using an image (say `bg.jpg`): FFmpeg can use `-loop 1 -t <duration>` to loop the image. Alternatively, we can just use `-framerate 30 -i bg.jpg` and then `-t` to cut at the length of audio. A common approach: `ffmpeg -loop 1 -i bg.jpg -i voice.mp3 ... -shortest ...` where `-loop 1` means keep showing that image. We also might need to scale the image to 1080x1920. This can be done via a filter like `scale=1080:1920:force_original_aspect_ratio=crop`. If the image isn’t the same aspect, you can choose to crop or letterbox. For simplicity, we assume portrait-oriented images from Pexels (we requested orientation=portrait). They might not exactly be 1080x1920, but we can resize/crop minimally.  
- If using a video clip (`clip.mp4`): Input it normally. If it’s longer than needed, we can cut it with `-t` or use `-shortest` later to stop when audio ends. If it’s shorter, we can loop it: e.g., use the `-stream_loop -1` flag before the `-i clip.mp4` to loop it infinitely, then rely on `-shortest` to stop at audio end. This way, a 10-sec clip will repeat to fill a 30-sec voiceover.  

**Audio Inputs:** We have two audio sources – voice (narration) and music. We need to *mix* them into one track. FFmpeg’s filtergraph can mix audio. One way is the `amix` filter. Alternatively, if we want to side-chain (e.g., lower music when voice is present), that’s more complex. Simpler: just reduce music volume and mix equally. Example: `-filter_complex "[1:a][2:a]amix=inputs=2:duration=first:dropout_transition=0,volume=2[aout]"` can mix two audio inputs (assuming input 1 is voice, 2 is music) and output to a single audio stream labeled `[aout]`. But controlling volume of each can be done by inserting volume filters on each before mixing. For instance: `[2:a]volume=0.3[a2]` to make music 30%, then `[1:a][a2]amix...`. Alternatively, use the simpler approach: since voice should be dominant, we set music volume low. The quote video template mentioned customizing music volume in the FFmpeg command.  

**Drawtext (Captions):** The drawtext filter goes on the video stream. For example:
```

-vf "scale=1080:1920\:force\_original\_aspect\_ratio=cover, drawtext=fontfile=/path/to/font.ttf\:text='Your caption here'\:x=(w-text\_w)/2\:y=(h-text\_h)-100\:fontcolor=white\:fontsize=40\:box=1\:boxcolor=0x00000088"

```
This does: scale the video to 1080x1920 (cover means it will crop if needed to fill the frame), then drawtext with a given font. We position text centered horizontally (x=(w-text_w)/2) and a bit above bottom (y=(h-text_h)-100, which puts it 100px from bottom). We set white font, size 40 (adjust as needed), and enable a text box with semi-transparent black background (`0x00000088` – last 2 hex digits are alpha transparency). This makes the captions readable over the background. If our `text=` has quotes or special characters, it can be tricky; we might need to escape or use FFmpeg’s ability to draw from a file or from an argument. Since n8n allows constructing the command string dynamically, ensure the text is properly quoted. If the script is long, consider splitting lines with `\n` in the text.  

If doing multiple timed texts, it would look like: 
```

drawtext=text='First line'\:enable='lt(t,5)': ...,
drawtext=text='Second line'\:enable='between(t,5,10)': ...

````
But you have to include all in one `-vf` separated by commas. This can get messy. For the hack, a single block of text is fine. You could also just use the most important one-liner as caption rather than every word.

**Putting it together:**  
The general FFmpeg command might look like: 

For image background:  
```bash
ffmpeg -y -loop 1 -i bg.jpg -i voice.mp3 -i music.mp3 -filter_complex \
"[0:v]scale=1080:1920:force_original_aspect_ratio=cover[v]; \
 [2:a]volume=0.3[a2]; \
 [1:a][a2]amix=inputs=2:duration=first:dropout_transition=0[aout]; \
 [v]drawtext=fontfile=/path/Arial.ttf:text='YOUR SCRIPT TEXT': \
 x=(w-text_w)/2: y=(h-text_h)-50: fontcolor=white: fontsize=40: box=1: boxcolor=black@0.5: boxborderw=10[vo]" \
 -map "[vo]" -map "[aout]" -t <DURATION> -pix_fmt yuv420p -c:v libx264 -c:a aac output.mp4
````

Explanations:

* `-loop 1` loops the image.
* We then feed image to scale (cover to fill screen) as `[v]`.
* Lower music volume to 30% as `[a2]`.
* Mix voice `[1:a]` and music `[a2]` to `[aout]` using amix (duration=first means stop when the first input (voice) ends) so it will cut off extra music.
* Drawtext on video `[v]` to output `[vo]`.
* Map the final video and audio to outputs, set duration explicitly or rely on voice track length via `-shortest`.
* `-pix_fmt yuv420p` ensures compatibility (some players require this pixel format for H.264), `-c:v libx264` encodes video, `-c:a aac` encodes audio. `-y` overwrites output if exists.

For video background, you’d drop `-loop 1`, input a video, maybe still scale it, and you might not need `-t` if using `-shortest` with the voice to cut the video when voice ends. Use `-stream_loop -1` if video is shorter and you want it to loop.

It looks complex, but the pieces are doing exactly what we want: combine streams and overlay text in one go. The benefit of using one command is that it avoids having to do multiple rendering passes. If this is too daunting, you could simplify initially: *first*, try generating a video with just image + voiceover (no music, no text) to ensure FFmpeg works. Then add one thing at a time (music, then text).

**Executing in n8n:**
Use an **Execute Command** node (requires n8n to run on a system where FFmpeg is installed, which we have). In the node, you’d provide the command string. You might have to adjust how you reference file inputs:

* If you downloaded the image and audio in previous nodes, they might be stored as temporary files or binary data. n8n’s Execute Command can’t directly use in-memory binary; you might need to save them to disk first. There is a **Write Binary File** node that можно save a binary stream from n8n to a file path. For example, after downloading from Pexels, use Write Binary File to save `bg.jpg` to `/tmp/bg.jpg`. Do similarly for audio files (voice.mp3, music.mp3). Then FFmpeg command can reference those paths. (Ensure the container/OS user permissions allow writing to that location.)
* Alternatively, if n8n is running on your machine, you could call the Pexels API via curl inside the FFmpeg command using ffmpeg’s ability to read from URL. FFmpeg can read http URLs as input. So you could actually do `-i "<PEXELS_VIDEO_URL>"` directly in the FFmpeg command and skip downloading via n8n. That’s a neat trick: since Pexels URLs are public, FFmpeg will pull the media directly. Same with music if hosted. This saves steps. But it might be less reliable if network is slow. For hack, either way is fine.

When the Execute Command runs, if all goes well, you’ll get `output.mp4` generated at the specified path. You can then have n8n bring that back into workflow (you could use a **Read Binary File** node to read the file if you want to use it in subsequent nodes, or if the file is at a known path, you might directly use it in the next step of uploading).

**Result of Step 3:** A completed video file that contains the visual, plus embedded narration and music, with caption text overlay. Test play it locally to verify quality: Is the voice audible over music? Is text visible and not cut off? Adjust parameters if needed (e.g., text position or size, music volume).

*(Don’t be discouraged if FFmpeg’s learning curve is steep – even experienced devs tinker a lot. The n8n template for quote videos proves it’s doable: they generated 9:16 videos with text and music via FFmpeg on local n8n. Use their approach as inspiration and feel free to search for “FFmpeg overlay text video” for syntax help. Once it works the first time, you’re golden.)*

## Step 4: Formatting for Reels, TikTok, YouTube Shorts

By design, we’ve been targeting the vertical format all along. But let’s explicitly ensure we meet the requirements of the platforms:

* **Resolution & Aspect Ratio:** We used 1080x1920 (or similar). All major short video platforms accept that. TikTok and Reels prefer 9:16. YT Shorts uses 9:16 as well (or any vertical >1:1). Our FFmpeg scaling set that up.
* **Duration:** Try to keep the video length within 60 seconds to qualify as a “Short” on YT and to hold attention on IG/TikTok. Our script is \~30 seconds, which is perfect. (Instagram Reels can go longer now, but short and sweet is fine.)
* **File Size/Encoding:** H.264 MP4 with AAC audio is universally accepted. Ensure the bitrate is reasonable (FFmpeg will choose a default that should be fine for 720p/1080p short clips). If file size is an issue (some platforms have limits like 100 MB for TikTok upload via API), you can add `-crf 23` to FFmpeg output for quality factor or limit the bitrate. But likely a 30s 1080p video will be just a few MB.
* **Orientation Flag:** Rarely, videos might need an orientation flag set. Using libx264 and the correct width×height usually avoids any rotation issues.

One more thing: some platforms (Instagram) might auto-crop if the video isn’t exactly 9:16 or has black bars. We made sure to cover the frame with the image/video. If you ended up with a slightly different aspect, you could add padding. But as long as it’s close to 9:16 (1080x1920), you’re good.

In summary, if you followed the above, your output.mp4 is ready for all four platforms mentioned, with no further changes needed for each specifically. The same file can be posted to each platform (you might tailor the caption text per platform, but the video itself can be identical).

## Step 5: Publishing the Content (Posting Automation)

With the video created, the final step is to (at least partially) automate publishing it on social media. Full automation can be tricky due to API restrictions, but here are some ways to tackle it:

* **YouTube Shorts (API available):** YouTube Data API allows uploading videos to your channel. In n8n, you can use the **YouTube node** if available or an HTTP request to the upload endpoint. You’ll need to have OAuth2 credentials set up (YouTube API OAuth with the `youtube.upload` scope). If set up, you can supply video file (as binary) and metadata (title, description, tags) to YouTube. The Thai quote workflow demonstrates automatically uploading the video to YouTube and even writing back the YouTube URL to a Google Sheet. For hackathon, doing a YouTube upload is probably the most feasible *actual* posting to test, since Google’s API is well-documented.
* **Instagram Reels and Facebook:** Meta’s Graph API can post videos to Instagram Business accounts or Facebook pages. However, for Instagram, the account must be a Business/Creator account linked to a Facebook page, and you use the endpoint `/me/instagram_media` etc. It’s quite involved to set up in short time (requires an App review if posting to Instagram, unless using your own account in Dev Mode with limited audience). It might be beyond scope to implement fully. Instead, you could simulate posting by storing the video or sending a notification:

  * For example, use n8n to email the video to yourself or upload it to Google Drive, so you get it on your phone and then manually post. Or use the Discord node to send the video to a Discord channel as a way of “publishing” it for the hack demo.
* **TikTok:** TikTok has a Content Posting API, but access is restricted and mostly for approved partners. Probably skip automation here; manual upload will do.
* **Multi-Platform Tool (Optional):** As mentioned, a service like **upload-post.com** can take your content and distribute it to multiple platforms via their API. If you managed to get an account there, you could integrate it (the n8n template uses it as a node). That might simplify things like: one API call to upload-post with video and caption and it handles posting to TikTok, IG, YT, FB, LinkedIn. This is a paid service typically, so it’s optional.

**For the hackathon demonstration,** it’s perfectly acceptable to not fully automate actual posting due to time. Instead, focus on producing the ready-to-post content and maybe *simulate* the publish. Some ways to simulate/indicate publishing in your workflow:

* Post the video to a private **YouTube** channel (unlisted) just to show it gets out there.
* Or have a **Slack/Telegram** notification step that says “New video ready: {title}” with the video attached or a link. This shows the end-to-end flow works, and the “posting” can be a human step after.
* Or simply save the file to a known folder and print a message “Video saved and ready to post to IG/TikTok!” in the execution output.

If you do want to attempt one platform, YouTube is a good choice because it’s straightforward to obtain OAuth credentials in a hackathon setting, and n8n’s YouTube integration (or Google Drive + YouTube) is documented.

Make sure when posting (or simulating) to also include the title/description/hashtags we generate in the next step.

## Step 6: Optimizing Engagement (Titles, Hashtags, Scheduling)

Finally, to maximize the reach and engagement of your content, you should generate a catchy **title** or caption and some **hashtags**, and consider the posting schedule.

**Title & Hashtags:**
We can leverage GPT again here. For example, after the script is made (or using the script as input), ask GPT to: *“Provide a short, attention-grabbing title for the above video script, and 5 relevant hashtags.”* It might return something like: *Title: “Get Fit Fast: HIIT Explained”* and *Hashtags: #HIIT #Fitness #Workout #Health #Training*. You can do this in the same OpenAI node as the script by extending the prompt, or a separate node after script generation. In a separate node, you’d feed the final script or the topic and say “Generate 3 hashtags related to {topic} that are popular on social media.” The n8n template for multi-platform content shows that it automatically generates hashtags and even emojis for engagement, so this idea is proven.

Take the output and format it. For YouTube, you might include the hashtags in the description; for Instagram/TikTok, include them in the caption. You can also generate a one-sentence video description if needed (like a social post text separate from the script itself).

**Posting Schedule:**
While scheduling actual posts might be beyond the hack’s scope (since you’d need the platforms to support scheduling via API or use a social media scheduler tool), you can simulate scheduling by simply having n8n triggered at a certain time. For instance, use an **n8n Cron node** to run the whole workflow daily at 9 AM (or whenever engagement is high). In this hack context, you could at least mention that your workflow could be set on a schedule to regularly produce content (maybe daily trending topic video). In our challenge description, it was optional. If you want, set a Cron trigger to kick off everything once a day – just be mindful not to spam the APIs too much (OpenAI, etc., have usage limits/costs).

**Engagement tips:**
Encourage participants to think about *why* someone would watch or share this video. The first 2 seconds are crucial – the hook in the script, combined with maybe an eye-catching first visual frame, matters. Also, ending with a question or prompt can drive comments (for example, “What do you think? Let us know!”). These can be reflected in the script generation prompt if desired. Since this is a technical build, we won’t dive deep into content strategy, but it’s worth noting for a holistic solution.

## Building the n8n Workflow

Now, let’s outline how the entire workflow comes together in n8n. This will help you organize the nodes in a logical sequence. You can also create a simple flowchart for clarity (for example, each step we described becomes a group of nodes in n8n). Here’s one possible structure, with node types in parentheses:

1. **Trigger:** Could be a Manual trigger (to run on click) or a Cron trigger (to run periodically). For hackathon testing, use manual trigger.
2. **Set Domain/Context (Set node):** e.g., set a variable “domain” or some initial values if needed (not mandatory if domain is hardcoded in prompt).
3. **Fetch Trending Topic (HTTP Request node):** e.g., GET Google Trends RSS.
4. **Select Topic (Function or OpenAI):** parse RSS and pick one, or ask GPT to choose relevant topic from it. Output: `topic`.
5. **Generate Script (OpenAI node):** Prompt with the chosen `topic` to create the video script. Output: `script_text`.
6. **Generate Title/Hashtags (OpenAI node, optional):** Using `topic` or `script_text`, generate a `title` and `hashtags`. (Or this could be combined with step 5 using one prompt that returns both script and suggestions).
7. **Text-to-Speech (HTTP node to VoiceRSS):** Send `script_text` and get back audio binary. Output: `voice.mp3` (binary).
8. **Download Visual (HTTP node to Pexels API):** Request image/video for `topic`. Possibly add a short delay before this, as sometimes immediately hitting multiple APIs could be rate-limited. Output: `bg.jpg` or `clip.mp4` (binary).
9. **Write files (Binary -> File nodes):** Save the `voice.mp3`, `bg.jpg/clip.mp4`, and also prepare a `music.mp3` file (either you have it bundled or download it similarly if from a URL). We need file paths for FFmpeg. Use **Write Binary File** nodes for each. For example, save voice to `/tmp/voice.mp3`, image to `/tmp/bg.jpg`, music to `/tmp/music.mp3`. Then FFmpeg command can reference those paths.
10. **Execute FFmpeg (Execute Command node):** Construct the command as detailed, referencing the file paths. This node will execute and hopefully produce `/tmp/output.mp4`. Capture any stdout/stderr for debugging if needed. (Ensure n8n user has permission to execute ffmpeg and write file).
11. **Read output file (Read Binary File node):** Load the resulting `/tmp/output.mp4` so that it’s inside n8n data. Now you can use it in subsequent nodes, like uploading or emailing.
12. **Social Media Upload (Optional, e.g., YouTube node):** If configured, connect YouTube and create an “Upload Video” node, mapping the binary file data to the video content, and `title`/`description` fields to the AI-generated title/hashtags text. On execution, this will upload the video to your channel. You could do similar for other platforms if credentials allow (or skip to next step).
13. **Notification/Mock Publish:** If not actually uploading anywhere, you can use an **Email node** or **Discord node** to send yourself (or a channel) a message like: “New video created: {title}\n{hashtags}” and attach the video file. This serves as a confirmation that the workflow completed successfully and provides the output.
14. **Log or Store (Optional):** You might log the event in a Google Sheet or database – e.g., store the topic and timestamp, and maybe a link if uploaded (the advanced templates update a Google Sheet with status and links). This is optional but helps if you run this regularly and want to avoid duplicates or track performance.

Each of these can be a node or a group of nodes. You will connect them in order, handling the data between. Use n8n’s Execute Workflow or Sub-workflow if you want to keep things modular (not necessary for hack, but nice if splitting tasks).

**Error handling:** It’s wise to add a few checks, like if no trending topic is found, or the APIs return errors (maybe handle with a Stop and notify). But again, not mandatory for an MVP.

**Diagram (Mental Model):** *Trending Topic* → *GPT script* → *TTS audio* + *Images* + *Music* → *FFmpeg assembly* → *Video output* → *Publish*. This linear flow can have branches merging at the FFmpeg step (since we gather audio/image in parallel potentially). The n8n workflow might branch after script: one path handles generating audio, another handles image, then join to run FFmpeg. You can use the **Merge** node to wait for both branches (set to Merge by pass-through, combining the binary data). Or ensure the image fetch node runs after TTS node sequentially (less optimal but simpler). Up to you – n8n is flexible in routing data.

## Resources and Tips for Participants

* **n8n Documentation:** The official docs and community forum are extremely helpful. For example, refer to the OpenAI node docs for how to set credentials, or community posts about FFmpeg usage. The n8n blog and templates library have many workflows using AI and media – don’t hesitate to reference them for ideas (we’ve cited some in this guide).
* **FFmpeg Guides:** A handy FFmpeg filter guide or cookbook can save time. The FFmpeg official wiki and StackOverflow have many one-liners for common tasks (like adding text, audio mixing). We cited an n8n template that directly uses FFmpeg with drawtext for dynamic text overlay – you can model your command on that.
* **Free Asset Sources:** Pexels (for images/videos), Pixabay, Unsplash, Openverse – all are great for visuals. For music, look at Pixabay Music or Free Music Archive for quick picks. VoiceRSS for TTS as we used. These keep your workflow free of cost.
* **Optional Enhancements:** If participants want to push further:

  * Integrate **RunwayML** or similar for generating actual video segments (Runway’s Gen-2 can create short videos from text prompts – this could replace the need for stock footage if someone has API access, but it may be slow or have limits). RunwayML can also help with tasks like background removal or replacing green screens if needed. *In the n8n context, you’d call Runway’s API and treat it like any other HTTP integration.* The feasibility on local limited resources is low, so this is purely optional.
  * Use **AI agents or multiple LLM calls** to refine content – e.g., use one AI call to research facts (like using an API such as Perplexity.ai or SerpAPI to get current info), then another to write the script, to ensure accuracy and novelty. This could enrich your content, though at cost of complexity.
  * Multi-language support: You could adapt the workflow to create content in other languages (just change GPT prompt language and TTS language code). Ensure you have appropriate fonts for captions (as the Thai example did by using a Thai font for drawtext).
  * **Scaling up:** If time, you could make the workflow handle multiple topics in one go (batch generation) or even a chain of videos. Or integrate a queue (maybe pull topics from a Google Sheet and mark them done after posting, as some templates do). This is beyond the basic ask but showcases how you can expand the system.

## Conclusion and Next Steps

By completing this challenge, you’ve essentially built a mini **content factory** that can ideate, create, and distribute content automatically. This has massive potential – from automating marketing content, generating educational clips, to running an entire faceless content channel on autopilot.

During the hackathon, focus on getting a basic version working: even if that means one static image, one voiceover, and saving the video locally. Make sure each component works (test your GPT output, test the TTS audio by listening, test the FFmpeg command on sample files). **Iterate in small steps**, and use n8n’s execution logs to debug where data might not be passing correctly.

Be creative with the content – choose a domain你 find fun, and let the AI surprise you with script outputs. Encourage your team to try different prompts or add creative elements (maybe an intro/outro screen, or dynamic intro music for dramatic effect!). The system you built is modular, so each piece can be improved or swapped out (for instance, use a different TTS voice, or use multiple images for a slideshow).

Finally, remember to showcase the end-to-end flow in your hackathon presentation: start with how a trending topic is picked, then show how the script is generated, then perhaps play a sample of the voiceover, show the assembled video, and discuss how it would be posted with a catchy title and hashtags. The wow factor is seeing a piece of content go from *idea* to *ready-to-share video* with minimal human intervention.

Good luck, and enjoy hacking on this AI-powered content creator! We can’t wait to see the creative videos and clever workflows you come up with. 🚀

**References and Further Reading:** (for your implementation, not all need to be cited in presentation, but they back up the techniques we used)

* n8n Workflow Template: *“Automatically Create and Upload YouTube Videos with Quotes”* – shows using FFmpeg locally to overlay text on 9:16 videos.
* n8n Workflow Template: *“Fully Automated AI Video Generation & Publishing”* – an advanced multi-platform solution that inspired parts of our design.
* n8n Community Forums – threads on integrating FFmpeg, Google Trends, etc., e.g., using Google Trends RSS in n8n and discussions of video automation tips (some mentioned using RunwayML or other tools for advanced editing).
* Voice RSS API documentation – for TTS usage and examples.
* Pexels API documentation – free stock media API usage (we used it for images/videos).
* n8n Template: *“Automate Multi-Platform Social Media Content Creation”* – demonstrates GPT-4 content generation with hashtags and posting via APIs.
* Reddit Post: *“AI-Powered Automation for Long-Form Videos”* – an example of a sophisticated pipeline using n8n, GPT-4, image generation, and FFmpeg with complex scene editing – proving the sky is the limit if you want to push this project further after the hackathon!

```
```
