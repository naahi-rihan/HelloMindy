# HelloMindy
Developing AI-powered Chatbot for Mental Health Support
import openai
import gradio as gr

# Set your OpenAI API key
openai.api_key = 'YOUR_OPENAI_API_KEY'

# Function to generate responses using GPT
def generate_response(prompt):
    response = openai.Completion.create(
        engine="text-davinci-003",  # Use the appropriate model
        prompt=prompt,
        max_tokens=150,
        n=1,
        stop=None,
        temperature=0.9,
    )
    return response.choices[0].text.strip()

# Chatbot function
def chatbot(user_input, chat_history=[]):
    # Generate the prompt
    history_text = '\n'.join([f'User: {u}\nBot: {b}' for u, b in chat_history])
    prompt = f'{history_text}\nUser: {user_input}\nBot:'
    
    # Get the chatbot response
    bot_response = generate_response(prompt)
    
    # Update chat history
    chat_history.append((user_input, bot_response))
    
    return bot_response, chat_history

# Gradio UI
def ui(chat_history, user_input):
    response, chat_history = chatbot(user_input, chat_history)
    return gr.update(value="", interactive=True), chat_history + [(user_input, response)]

with gr.Blocks() as demo:
    gr.Markdown("# Mental Health Support Chatbot")
    chat_history = gr.State([])
    with gr.Column():
        chatbot_output = gr.Chatbot(elem_id="chatbot").style(height=500)
        with gr.Row():
            user_input = gr.Textbox(show_label=False, placeholder="Type your message here...").style(container=False)
            user_input.submit(ui, [chat_history, user_input], [user_input, chatbot_output])
    
    gr.Examples(
        examples=[
            "I'm feeling really anxious today.",
            "Can you provide me with some coping strategies?",
            "I need help with managing stress.",
        ],
        inputs=user_input,
    )

demo.launch()
