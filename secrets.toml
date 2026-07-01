import streamlit as st
import google.generativeai as genai
from PIL import Image
import yaml
from yaml.loader import SafeLoader
import os

# -------------------------------
# Streamlit Page Config
# -------------------------------
st.set_page_config(
    page_title="Electronics Troubleshooting Chatbot",
    page_icon="🤖",
    layout="centered"
)

# -------------------------------
# Load Config
# -------------------------------
CONFIG_FILE = "config.yaml"

def load_config():
    with open(CONFIG_FILE) as file:
        return yaml.load(file, Loader=SafeLoader)

def save_config(config):
    with open(CONFIG_FILE, "w") as file:
        yaml.dump(config, file, default_flow_style=False)

config = load_config()

# -------------------------------
# Authentication and Registration
# -------------------------------
def authenticate():
    # Initialize auth session state
    if 'authenticated' not in st.session_state:
        st.session_state['authenticated'] = False

    if st.session_state['authenticated']:
        return True

    # Only show login/register if NOT authenticated
    auth_choice = st.sidebar.radio("Choose Action:", ["Login", "Register"])

    if auth_choice == "Register":
        st.subheader("📝 Create a New Account")

        with st.form("register_form"):
            new_username = st.text_input("Username")
            new_name = st.text_input("Full Name")
            new_email = st.text_input("Email")
            new_password = st.text_input("Password", type="password")
            confirm_password = st.text_input("Confirm Password", type="password")
            register_btn = st.form_submit_button("Register")

        if register_btn:
            if not new_username or not new_password or not new_email:
                st.error("⚠️ Please fill in all required fields.")
            elif new_password != confirm_password:
                st.error("❌ Passwords do not match.")
            elif new_username in config['credentials']['usernames']:
                st.error("❌ Username already exists. Choose another.")
            else:
                config['credentials']['usernames'][new_username] = {
                    "email": new_email,
                    "name": new_name,
                    "password": new_password
                }
                save_config(config)
                st.success("✅ Registration successful! Please login from the sidebar.")

    elif auth_choice == "Login":
        st.subheader("🔑 Login")

        with st.form("login_form"):
            username = st.text_input("Username")
            password = st.text_input("Password", type="password")
            login_btn = st.form_submit_button("Login")

        if login_btn:
            if username in config['credentials']['usernames'] and config['credentials']['usernames'][username]["password"] == password:
                st.session_state['authenticated'] = True
                st.sidebar.success(f"✅ Logged in as {username}")
                st.rerun()
            else:
                st.error("❌ Invalid Username or Password")

    return False

# -------------------------------
# Main Chatbot Interface
# -------------------------------
def chatbot_interface():
    # -------------------------------
    # API Key Config
    # -------------------------------
    if "GOOGLE_API_KEY" in st.secrets:
        api_key = st.secrets["GOOGLE_API_KEY"]
    else:
        api_key = st.text_input("Enter your Google API Key:", type="password")

    if not api_key:
        st.warning("Please provide your Google API Key to continue.")
        st.stop()

    # Configure Gemini client
    genai.configure(api_key=api_key)

    # -------------------------------
    # System Prompt
    # -------------------------------
    SYSTEM_PROMPT = """
    You are an expert in electronics troubleshooting.

    I face the following issue:
    \"{question}\"

    Analyze the image (if provided), identify any visible issues, and provide:
    1. A diagnosis based on the description and/or image.
    2. Explanation of the problem.
    3. Suggested fix or faulty component.
    """

    # -------------------------------
    # Function to Get Response
    # -------------------------------
    def get_gemini_response(user_question, image=None):
        model = genai.GenerativeModel("gemini-2.5-flash")
        final_prompt = SYSTEM_PROMPT.format(question=user_question if user_question else "No text provided")

        if image:
            response = model.generate_content([final_prompt, image])
        else:
            response = model.generate_content(final_prompt)

        return response.text.strip()

    # -------------------------------
    # Chatbot UI
    # -------------------------------
    st.title("🔧 Conversational Image Recognition")
    st.markdown("Upload an image of your electronic circuit/device and ask a troubleshooting question.")

    # ✅ Single-line-looking text_area with Shift+Enter support
    user_question = st.text_area(
        "🔎 Describe your issue or question:",
        height=50,  # looks like a single line
        key="input"
    )

    uploaded_image = st.file_uploader("📷 Upload an image (optional)", type=["jpg", "jpeg", "png"])

    image_obj = None
    if uploaded_image:
        image_obj = Image.open(uploaded_image)
        # 👇 Centered & smaller image preview
        st.markdown("<div style='text-align:center;'>", unsafe_allow_html=True)
        st.image(image_obj, caption="Uploaded Image", width=250)
        st.markdown("</div>", unsafe_allow_html=True)

    if st.button("🚀 Analyze & Troubleshoot"):
        with st.spinner("Analyzing... please wait"):
            try:
                response = get_gemini_response(user_question, image_obj)
                st.subheader("✅ AI's Diagnosis & Suggestion:")
                st.write(response)
            except Exception as e:
                st.error(f"Error: {e}")

# -------------------------------
# Sidebar with Logout (Fixed)
# -------------------------------
def sidebar_logout():
    if st.session_state.get('authenticated', False):
        if st.sidebar.button("Logout"):
            st.session_state.clear()
            st.session_state['authenticated'] = False
            st.rerun()

# -------------------------------
# Main Flow
# -------------------------------
if authenticate():
    sidebar_logout()
    chatbot_interface()
else:
    sidebar_logout()
