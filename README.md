# mental-health-chatbot
import tkinter as tk
from tkinter import scrolledtext
import threading
import time
import random
from datetime import datetime

class MentalHealthChatbot:
    def _init_(self, root):
        self.root = root
        self.root.title("Mental Health Companion - Your Friendly Advisor")
        self.root.geometry("600x700")
        self.root.configure(bg="#f0f8ff")

        self.setup_ui()

    def setup_ui(self):
        title_label = tk.Label(self.root, text="💙 Mental Health Companion 💙",
                               font=("Arial", 16, "bold"), bg="#f0f8ff", fg="#4169e1")
        title_label.pack(pady=10)

        self.chat_display = scrolledtext.ScrolledText(
            self.root,
            wrap=tk.WORD,
            width=70,
            height=25,
            font=("Arial", 11),
            bg="#ffffff",
            fg="#333333",
            state=tk.DISABLED,
            relief=tk.SUNKEN,
            borderwidth=2
        )
        self.chat_display.pack(pady=10, padx=20, fill=tk.BOTH, expand=True)

        input_frame = tk.Frame(self.root, bg="#f0f8ff")
        input_frame.pack(pady=10, padx=20, fill=tk.X)

        self.user_input = tk.Text(input_frame, height=3, font=("Arial", 11),
                                  bg="#ffffff", relief=tk.SUNKEN, borderwidth=2)
        self.user_input.pack(side=tk.LEFT, fill=tk.BOTH, expand=True, padx=(0, 10))
        self.user_input.bind("<Return>", self.on_enter)

        self.send_button = tk.Button(input_frame, text="Send 💌",
                                     command=self.send_message,
                                     font=("Arial", 11, "bold"),
                                     bg="#4169e1", fg="white",
                                     relief=tk.RAISED, bd=3,
                                     padx=20)
        self.send_button.pack(side=tk.RIGHT)

        disclaimer = tk.Label(self.root,
                              text="I'm here to listen and support you. I'm not a doctor - please seek professional help if needed.",
                              font=("Arial", 9),
                              bg="#f0f8ff", fg="#666666")
        disclaimer.pack(pady=5)

        self.add_message("bot", "Hello! I'm your Mental Health Companion 💙
I'm here to listen, support you, and help you feel better. You can share anything that's on your mind. How are you feeling today?")

    def on_enter(self, event):
        # Prevent default newline on Enter and instead send message
        self.send_message()
        return "break"

    def send_message(self):
        user_message = self.user_input.get("1.0", tk.END).strip()
        if user_message:
            self.add_message("user", user_message)
            self.user_input.delete("1.0", tk.END)
            threading.Thread(target=self.process_message, args=(user_message.lower(),), daemon=True).start()

    def add_message(self, sender, message):
        self.chat_display.config(state=tk.NORMAL)

        timestamp = datetime.now().strftime("%H:%M")
        if sender == "user":
            self.chat_display.insert(tk.END, f"[{timestamp}] You: {message}

", "user")
        else:
            self.chat_display.insert(tk.END, f"[{timestamp}] Companion: {message}

", "bot")

        self.chat_display.config(state=tk.DISABLED)
        self.chat_display.see(tk.END)

        self.chat_display.tag_config("user", foreground="#4169e1", font=("Arial", 11, "bold"))
        self.chat_display.tag_config("bot", foreground="#2e5c8a", font=("Arial", 11))

    def process_message(self, user_input):
        # Simulate typing delay
        time.sleep(random.uniform(1, 2))
        response = self.generate_response(user_input)
        self.root.after(0, self.add_message, "bot", response)

    def generate_response(self, user_input):
        validation_responses = [
            "I hear you, and it's completely okay to feel that way.",
            "Your feelings are valid, and I'm here with you.",
            "That sounds really tough. You're doing great just by talking about it.",
            "I can see this is weighing on you. Thank you for sharing.",
            "It's brave of you to express how you're feeling."
        ]

        responses = {
            ("sad", "depressed", "down", "empty", "hopeless", "cry", "tears"): [
                "I'm so sorry you're feeling sad right now. 💙 Sometimes just sharing helps lighten the load.",
                "It's okay to feel down sometimes. Would you like to tell me more about what's making you feel this way?",
                "You're not alone in this. Even the darkest days have light waiting to break through."
            ],

            ("anxious", "worry", "stress", "nervous", "panic", "overwhelmed"): [
                "Anxiety can feel so overwhelming. Let's try a simple breathing exercise together: breathe in for 4, hold for 4, out for 4.",
                "You're safe here with me. What if we focus on one thing at a time?",
                "Stress is your body's way of saying it needs a break. What usually helps you relax?"
            ],

            ("angry", "mad", "frustrated", "rage", "irritated"): [
                "It's completely normal to feel angry sometimes. What happened?",
                "Your anger is valid. Sometimes writing it down or taking deep breaths can help release it.",
                "You're allowed to feel strong emotions. Let's find a healthy way to work through this."
            ],

            ("lonely", "alone", "isolated", "no one", "miss"): [
                "Loneliness can feel so heavy. I'm right here with you now 💕",
                "You're not truly alone - I'm listening and there are people who care about you.",
                "Sometimes reaching out, even in small ways, can help. Who’s someone you feel safe talking to?"
            ],

            ("hate myself", "ugly", "failure", "stupid", "worthless"): [
                "You are worthy of love and kindness, especially from yourself. 💜",
                "Those thoughts don't define who you are. You have so much value.",
                "Try saying one kind thing about yourself right now. I'll wait."
            ],

            ("hurt myself", "self harm", "cut", "suicide", "kill myself", "end it"): [
                "I'm here and I care about you deeply. Please reach out to someone you trust right now.",
                "Your life matters. Can you call a crisis hotline or emergency services? You're not alone.",
                "Please stay safe. In the US: 988 (Suicide & Crisis Lifeline). Tell me your location and I'll help find resources."
            ],

            ("good", "happy", "great", "thank", "grateful"): [
                "That's wonderful to hear! What made you feel good today?",
                "Your happiness makes me happy too! Keep nurturing those positive moments.",
                "Thank YOU for sharing. Moments like these are worth celebrating!"
            ],

            ("tired", "sleep", "exhausted", "insomnia"): [
                "Rest is so important. Try a 4-7-8 breathing pattern: in 4, hold 7, out 8.",
                "Create a wind-down routine: dim lights, no screens, gentle stretching.",
                "Your body knows what it needs. Be gentle with yourself tonight."
            ],
        }

        # Check critical keywords first
        critical_keywords = ("hurt myself", "self harm", "cut", "suicide", "kill myself", "end it")
        if any(keyword in user_input for keyword in critical_keywords):
            return random.choice(responses[critical_keywords])

        # Match input to categories
        for keywords, resp_list in responses.items():
            if any(keyword in user_input for keyword in keywords):
                return random.choice(resp_list)

        # Default fallback responses
        default_responses = [
            "Thank you for sharing that with me. How does that make you feel?",
            "I'm listening carefully. Tell me more if you'd like.",
            "You're safe here. Whatever you're feeling is okay to share.",
            "That sounds important to you. I'm here to support you through it.",
            "Take your time. I'm not going anywhere."
        ]
        return random.choice(default_responses)

    def run(self):
        self.root.mainloop()


if _name_ == "_main_":
    root = tk.Tk()
    app = MentalHealthChatbot(root)
    app.run()
