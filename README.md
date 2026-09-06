## Development and Deployment of a 'Chat with LLM' Application Using the Gradio Blocks Framework

### AIM:
To design and deploy a "Chat with LLM" application by leveraging the Gradio Blocks UI framework to create an interactive interface for seamless user interaction with a large language model.

### PROBLEM STATEMENT:

### DESIGN STEPS:

#### STEP 1: Load environment variables and initialize the Hugging Face FalconLM-instruct endpoint using the text_generation.Client to handle prompt-based text generation.

#### STEP 2: Create a Gradio interface to generate completions based on user input and a slider to control max_new_tokens dynamically.

#### STEP 3: Design a Gradio chatbot using message formatting to maintain history and return contextual responses, enabling continuous dialogue with FalconLM.

### PROGRAM: 
```python
import os
import gradio as gr
from dotenv import load_dotenv, find_dotenv
from text_generation import Client

load_dotenv(find_dotenv())

hf_api_key = os.environ["HF_API_KEY"]
endpoint = os.environ["HF_API_FALCOM_BASE"]

client = Client(
    endpoint,
    headers={"Authorization": f"Bearer {hf_api_key}"},
    timeout=120
)
```
```python
def generate(input_text, slider):
    try:
        result = client.generate(
            input_text,
            max_new_tokens=int(slider)
        )
        return result.generated_text
    except Exception as e:
        return f"Error: {str(e)}"
```
```python
def format_chat_prompt(message, chat_history):
    prompt = ""

    if chat_history is None:
        chat_history = []

    for user_message, bot_message in chat_history:
        prompt += f"\nUser: {user_message}\nAssistant: {bot_message}"

    prompt += f"\nUser: {message}\nAssistant:"

    return prompt
```
```python
def respond(message, chat_history):
    if chat_history is None:
        chat_history = []

    try:
        formatted_prompt = format_chat_prompt(
            message,
            chat_history
        )

        result = client.generate(
            formatted_prompt,
            max_new_tokens=256,
            stop_sequences=[
                "\nUser:",
                "<|endoftext|>"
            ]
        )

        bot_message = result.generated_text

    except Exception as e:
        bot_message = f"Error: {str(e)}"

    chat_history.append((message, bot_message))

    return "", chat_history

gr.close_all()

with gr.Blocks() as demo:

    with gr.Tab("Text Generation"):

        input_text = gr.Textbox(
            label="Prompt"
        )

        slider = gr.Slider(
            label="Max new tokens",
            minimum=1,
            maximum=1024,
            value=20
        )

        generate_btn = gr.Button(
            "Generate"
        )

        output_text = gr.Textbox(
            label="Completion"
        )

        generate_btn.click(
            generate,
            inputs=[
                input_text,
                slider
            ],
            outputs=output_text
        )

    with gr.Tab("Chatbot"):

        chatbot = gr.Chatbot(
            height=240
        )

        msg = gr.Textbox(
            label="Prompt"
        )

        btn = gr.Button(
            "Submit"
        )

        clear = gr.ClearButton(
            components=[
                msg,
                chatbot
            ],
            value="Clear"
        )

        btn.click(
            respond,
            inputs=[
                msg,
                chatbot
            ],
            outputs=[
                msg,
                chatbot
            ]
        )

        msg.submit(
            respond,
            inputs=[
                msg,
                chatbot
            ],
            outputs=[
                msg,
                chatbot
            ]
        )

demo.launch(share=True)
```

### OUTPUT:
<img width="1146" height="592" alt="image" src="https://github.com/user-attachments/assets/7b956ab5-5f08-4393-954c-152996bedd35" />

### RESULT:
Thus, The designing and deploying of a "Chat with LLM" application by leveraging the Gradio Blocks UI framework is executed successfully.
