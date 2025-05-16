import streamlit as st
from supabase import create_client
import edge_tts
import asyncio
import uuid

SUPABASE_URL = "https://pjpkkuikhgpvmunypawv.supabase.co"
SUPABASE_KEY = "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6InBqcGtrdWlraGdwdm11bnlwYXd2Iiwicm9sZSI6ImFub24iLCJpYXQiOjE3NDczOTIyMjQsImV4cCI6MjA2Mjk2ODIyNH0.x-Hb-RvCXNXC_GaP817lHQYFwSLmOogN9b30kplfp5Q"
supabase = create_client(SUPABASE_URL, SUPABASE_KEY)

if "authenticated" not in st.session_state:
    st.session_state.authenticated = False
if "page" not in st.session_state:
    st.session_state.page = "login"
if "user_email" not in st.session_state:
    st.session_state.user_email = None

async def generate_tts(text, voice="en-US-Neural2-A", filename="output.mp3"):
    communicate = edge_tts.Communicate(text, voice)
    await communicate.save(filename)

def tts(text, voice="en-US-Neural2-A"):
    filename = f"{uuid.uuid4()}.mp3"
    asyncio.run(generate_tts(text, voice, filename))
    return filename

def show_signup():
    st.title("Sign Up")
    email = st.text_input("Email")
    password = st.text_input("Password", type="password")
    if st.button("Create Account"):
        try:
            res = supabase.auth.sign_up({"email": email, "password": password})
            if res.user:
                st.success("Account created. Please log in.")
                st.session_state.page = "login"
                st.experimental_rerun()
        except Exception as e:
            st.error(f"Signup Error: {str(e)}")
    if st.button("Back to Login"):
        st.session_state.page = "login"
        st.experimental_rerun()

def show_login():
    st.title("Login")
    email = st.text_input("Email")
    password = st.text_input("Password", type="password")
    if st.button("Login"):
        try:
            res = supabase.auth.sign_in_with_password({"email": email, "password": password})
            if res.user:
                st.session_state.authenticated = True
                st.session_state.user_email = email
                st.session_state.page = "dashboard"
                st.experimental_rerun()
        except Exception as e:
            st.error(f"Login Error: {str(e)}")
    if st.button("Create New Account"):
        st.session_state.page = "signup"
        st.experimental_rerun()

def show_dashboard():
    st.title("Health Reminder Dashboard")
    st.markdown(f"Welcome **{st.session_state.user_email}**!")

    st.subheader("Set a Health Reminder")
    reminder_text = st.text_area("Reminder Text")
    reminder_time = st.time_input("Time for Reminder")
    repeat_daily = st.checkbox("Repeat Daily?")
    language = st.selectbox("Choose Language", ["English", "Marathi"])
    gender = st.radio("Voice", ["Male", "Female"])

    voices = {
        ("English", "Male"): "en-US-GuyNeural",
        ("English", "Female"): "en-US-JennyNeural",
        ("Marathi", "Male"): "mr-IN-NeerajNeural",
        ("Marathi", "Female"): "mr-IN-AarohiNeural"
    }
    selected_voice = voices[(language, gender)]

    if st.button("Save Reminder"):
        try:
            supabase.table("reminders").insert({
                "email": st.session_state.user_email,
                "reminder": reminder_text,
                "time": str(reminder_time),
                "repeat": repeat_daily,
                "language": language,
                "voice": selected_voice
            }).execute()
            st.success("Reminder saved.")
        except Exception as e:
            st.error(f"Failed to save reminder: {str(e)}")

    if st.button("Play Reminder Voice"):
        if reminder_text:
            mp3_file = tts(reminder_text, selected_voice)
            st.audio(mp3_file, format="audio/mp3")
        else:
            st.warning("Please enter reminder text.")

    st.subheader("Your Reminders")
    try:
        data = supabase.table("reminders").select("*").eq("email", st.session_state.user_email).execute()
        reminders = data.data
        if reminders:
            for r in reminders:
                col1, col2 = st.columns([4, 1])
                with col1:
                    st.markdown(
                        f"**{r['reminder']}**  \n"
                        f"`{r['time']}` | {'Daily' if r['repeat'] else 'Once'} | {r['language']} ({'Male' if 'Guy' in r['voice'] or 'Neeraj' in r['voice'] else 'Female'})"
                    )
                with col2:
                    if st.button("Delete", key=r["id"]):
                        supabase.table("reminders").delete().eq("id", r["id"]).execute()
                        st.success("Reminder deleted.")
                        st.experimental_rerun()
        else:
            st.info("No reminders found.")
    except Exception as e:
        st.error(f"Could not fetch reminders: {str(e)}")

    if st.button("Logout"):
        st.session_state.authenticated = False
        st.session_state.page = "login"
        st.experimental_rerun()

if st.session_state.page == "login":
    show_login()
elif st.session_state.page == "signup":
    show_signup()
elif st.session_state.page == "dashboard":
    if st.session_state.authenticated:
        show_dashboard()
    else:
        st.warning("Please login to access dashboard.")
        st.session_state.page = "login"
        st.experimental_rerun()
