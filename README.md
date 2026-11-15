# Google-Prompt-Engineering-for-Video-Generation

# Prompt Engineering for Video Generation with Gemini & Veo

A practical Jupyter notebook demonstrating how to improve AI-generated video by moving from a **simple text prompt** to a structured, multimodal **image-to-video prompt engineering workflow** using Google Gemini and Veo on Vertex AI.

The notebook uses a short educational video about the **Pythagorean theorem** to demonstrate how prompt structure, visual references, camera direction, sound and dialogue can influence generated video.

---

## What this notebook demonstrates

This notebook explores four techniques for AI video generation:

1. **Simple text-to-video generation**
2. **Creating an input image for image-to-video generation**
3. **Using an LLM to enhance a video prompt**
4. **Generating video from an image plus a detailed cinematic prompt**

The overall workflow is:

**Idea → Simple video → Reference image → Structured keywords → Enhanced prompt → Image-to-video generation**

---

## Models used

The notebook uses three generative AI models:

```python
IMAGE_MODEL_ID = "gemini-3.1-flash-image"
TEXT_MODEL_ID = "gemini-3-flash-preview"
VIDEO_MODEL_ID = "veo-3.1-fast-generate-001"
```

Each has a different role:

| Model | Purpose |
|---|---|
| `gemini-3.1-flash-image` | Create the visual starting frame |
| `gemini-3-flash-preview` | Expand structured keywords into a richer video prompt |
| `veo-3.1-fast-generate-001` | Generate the final video |

> Model IDs and preview endpoints can change. Check the current Vertex AI documentation before running the notebook.

---

# How the notebook works

## 1. Simple text-to-video prompt

The notebook starts with a minimal prompt:

```python
prompt="A person explaining the Pythagorean theorem."
```

This is sent directly to Veo:

```python
operation = client.models.generate_videos(
    model=VIDEO_MODEL_ID,
    prompt="A person explaining the Pythagorean theorem.",
    config=types.GenerateVideosConfig(
        aspect_ratio="16:9",
        number_of_videos=1,
        duration_seconds=8,
        resolution="720p",
        person_generation="allow_adult",
        generate_audio=True,
    ),
)
```

The result is an **8-second, 16:9, 720p video with generated audio**.

It is saved as:

```text
original.mp4
```

This provides a useful baseline for comparing a simple prompt with a more deliberately engineered one.

---

## 2. Create an input image

The next step introduces greater visual control.

The notebook starts with:

```text
slide-image.png
```

This represents the educational slide that should appear within the video.

Gemini is then asked to generate a classroom environment around that slide:

```python
"""Generate a professor in a classroom full of students in
front of the provided slide image."""
```

The generated image is saved as:

```text
video-image.png
```

This becomes the starting image for the final video.

The workflow is therefore:

```text
slide-image.png
      ↓
Gemini image generation
      ↓
video-image.png
      ↓
Veo
      ↓
enhanced.mp4
```

---

# Prompt engineering

Rather than manually writing one long video prompt, the notebook separates the desired video into several components.

```python
subject = "a professor"

action = "giving a lecture on the Pythagorean theorem"

scene = "in a classroom, presenting in front of students"

style = "photorealistic"

camera_angle = "eye-level shot"

camera_movement = "zoom in"

sound_effects = "clicking of computer keys"

dialogue = """
When you square a side length, like a or b, you are literally calculating
the area of a square built perfectly off that side.
"""
```

These elements provide more deliberate control over both the visual and audio characteristics of the generated video.

---

## Video prompt framework

The notebook effectively uses the following structure:

```text
Subject
+
Action
+
Scene
+
Visual style
+
Camera angle
+
Camera movement
+
Sound effects
+
Dialogue
```

For this example:

| Prompt element | Value |
|---|---|
| Subject | A professor |
| Action | Giving a lecture on the Pythagorean theorem |
| Scene | Classroom with students |
| Style | Photorealistic |
| Camera angle | Eye-level shot |
| Camera movement | Zoom in |
| Sound | Clicking computer keys |
| Dialogue | Explanation of squared side lengths |

This structured approach makes prompts easier to design, modify and reuse.

---

# Prompt enhancement with Gemini

The individual elements are collected into a list:

```python
keywords = [
    subject,
    action,
    scene,
    style,
    camera_angle,
    camera_movement,
    sound_effects,
    dialogue
]
```

Gemini is then instructed to transform them into a single high-fidelity video prompt:

```python
gemini_prompt = f"""
Your task is to expand the following keywords into a single, high-fidelity,
descriptive prompt for video generation.

Every single keyword MUST be included.

Synthesize them into a single, cohesive, and cinematic instruction.

Do not add any new core concepts.

Output ONLY the final prompt string, without any introduction or explanation.

Mandatory Keywords:
{",".join(keywords)}
"""
```

The result becomes:

```python
video_prompt = response.text
```

This separates two tasks:

**Human:** define the intent and constraints.

**Gemini:** turn those constraints into a detailed video-generation instruction.

**Veo:** generate the video.

---

# Image-to-video generation

The final step combines the enhanced prompt with the generated starting image.

```python
operation = client.models.generate_videos(
    model=VIDEO_MODEL_ID,
    prompt=video_prompt,
    image=types.Image.from_file(
        location="video-image.png"
    ),
    config=types.GenerateVideosConfig(
        aspect_ratio="16:9",
        number_of_videos=1,
        duration_seconds=8,
        resolution="720p",
        person_generation="allow_adult",
        generate_audio=True,
    ),
)
```

The generated video is saved as:

```text
enhanced.mp4
```

You can then compare:

```text
original.mp4
```

with:

```text
enhanced.mp4
```

to see how structured prompting and image guidance affect the final result.

---

# Handling asynchronous video generation

Video generation takes longer than text or image generation.

Veo therefore returns an asynchronous operation:

```python
operation = client.models.generate_videos(...)
```

The notebook checks the operation every 15 seconds:

```python
while not operation.done:
    time.sleep(15)
    operation = client.operations.get(operation)
```

Once generation finishes, the resulting video is written to disk and displayed inside the notebook.

```python
def show_video(video, filename):
    with open(filename, "wb") as out_file:
        out_file.write(video)

    display(
        Video(
            filename,
            embed=True,
            width=600
        )
    )
```

---

# Generation settings

The example uses:

```python
types.GenerateVideosConfig(
    aspect_ratio="16:9",
    number_of_videos=1,
    duration_seconds=8,
    resolution="720p",
    person_generation="allow_adult",
    generate_audio=True,
)
```

### Key parameters

| Parameter | Example | Purpose |
|---|---|---|
| `aspect_ratio` | `"16:9"` | Widescreen video |
| `number_of_videos` | `1` | Generate one result |
| `duration_seconds` | `8` | 8-second clip |
| `resolution` | `"720p"` | Output resolution |
| `person_generation` | `"allow_adult"` | Allows adult people to appear |
| `generate_audio` | `True` | Generate audio alongside video |

---

# Requirements

The notebook was developed with:

- Python 3.11
- Jupyter Notebook
- Google Gen AI Python SDK
- Google Cloud Vertex AI
- IPython

The key imports are:

```python
import time
from IPython.display import Image, Markdown, display, Video
from google import genai
from google.genai import types
```

Install project dependencies using:

```bash
pip install -r requirements.txt
```

---

# Google Cloud setup

You will need access to a Google Cloud project configured for Vertex AI and the models used by the notebook.

The project uses a helper function for authentication:

```python
from helper import authenticate

credentials, project_id = authenticate()
```

The Gemini client is then configured using:

```python
client = genai.Client(
    project=project_id,
    location="global",
    credentials=credentials,
    http_options=types.HttpOptions(
        base_url=os.getenv("GOOGLE_VERTEX_BASE_URL")
    )
)
```

Depending on your environment, you may also need to configure:

```text
GOOGLE_VERTEX_BASE_URL
```

---

# Repository structure

A typical repository structure could look like this:

```text
.
├── README.md
├── prompt-engineering-video-generation.ipynb
├── helper.py
├── requirements.txt
└── slide-image.png
```

After executing the notebook, additional files are generated:

```text
video-image.png
original.mp4
enhanced.mp4
```

Giving you:

```text
.
├── README.md
├── prompt-engineering-video-generation.ipynb
├── helper.py
├── requirements.txt
├── slide-image.png
├── video-image.png
├── original.mp4
└── enhanced.mp4
```

---

# Running the notebook

## 1. Clone the repository

```bash
git clone <repository-url>
cd <repository-name>
```

## 2. Create a virtual environment

```bash
python -m venv .venv
```

### macOS/Linux

```bash
source .venv/bin/activate
```

### Windows

```powershell
.venv\Scripts\Activate.ps1
```

## 3. Install dependencies

```bash
pip install -r requirements.txt
```

## 4. Configure Google Cloud authentication

Ensure that your environment can authenticate to the Google Cloud project being used with Vertex AI.

## 5. Add the starting image

Make sure this file exists:

```text
slide-image.png
```

before running the image-generation section.

## 6. Start Jupyter

```bash
jupyter notebook
```

Open the notebook and run the cells sequentially.

---

# Why structured video prompting matters

Video prompts involve more dimensions than static image prompts.

Instead of describing only **what should appear**, you can describe:

### Who or what

```text
a professor
```

### What happens

```text
giving a lecture on the Pythagorean theorem
```

### Where

```text
a classroom in front of students
```

### Visual treatment

```text
photorealistic
```

### Camera placement

```text
eye-level shot
```

### Camera movement

```text
zoom in
```

### Sound environment

```text
clicking of computer keys
```

### Spoken content

```text
When you square a side length...
```

Thinking about these dimensions separately can provide substantially more control than trying to invent one large prompt from scratch.

---

# From image prompting to video prompting

This notebook builds naturally on an image-generation workflow.

Image generation focuses heavily on:

```text
Subject
Scene
Composition
Lighting
Style
```

Video adds another layer:

```text
Movement
Camera movement
Timing
Sound
Dialogue
```

A reusable mental model is therefore:

```text
WHAT
Subject + Action

WHERE
Scene + Environment

LOOK
Style + Composition

CAMERA
Angle + Movement

AUDIO
Sound + Dialogue
```

This is a useful framework when designing prompts for generative video systems.

---

# Important notes

## Generation takes time

Video generation is significantly more computationally intensive than text generation.

The notebook polls the generation operation every:

```text
15 seconds
