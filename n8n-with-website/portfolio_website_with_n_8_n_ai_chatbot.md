# Portfolio Website with n8n-Powered AI Chatbot

This project is a personal portfolio website integrated with **n8n** to power an AI chatbot. The chatbot can answer questions about me (skills, projects, experience, contact info, etc.) by querying structured data and AI workflows.

## 🚀 Overview

The goal of this project is to create an interactive portfolio where visitors don’t just read static content—they can **ask questions** and get instant, accurate answers about me through a chatbot embedded directly in the website.

Example questions users can ask:
- “Who are you?”
- “What technologies do you work with?”
- “Show me your recent projects”
- “How can I contact you?”

The chatbot responses are handled by **n8n workflows**, making the system flexible, automatable, and easy to extend.

## 🧠 How It Works

1. The user types a question into the chatbot on the website.
2. The website sends the query to an **n8n webhook**.
3. n8n processes the request:
   - Understands the intent of the question
   - Fetches relevant information about me (from predefined data, files, or APIs)
   - Uses AI (optional) to generate a natural-language response
4. n8n sends the response back to the website.
5. The chatbot displays the answer in real time.

## 🛠 Tech Stack

- **Frontend**: HTML
- **Backend Automation**: n8n
- **Chatbot Integration**: n8n Webhooks + AI node
- **Hosting**: Any static hosting (Vercel, Netlify, GitHub Pages, etc.)
- **AI (Optional)**: OpenAI / other LLM via n8n


## 🔗 n8n Integration

- Webhook node receives chatbot questions
- Logic nodes route and process queries
- AI node generates human-like answers
- Response node sends data back to the frontend

This setup allows easy updates—no frontend changes needed when adding new answers or logic.

## ✨ Features

- Interactive AI chatbot inside the portfolio
- Answers questions specifically about me
- Easy to extend with new workflows in n8n
- Clean and modern portfolio layout
- Real-time responses

## 📌 Future Improvements

- Add memory to the chatbot (conversation context)
- Connect to a database for dynamic content
- Support multiple languages
- Admin panel to update personal data without code

## 📬 Contact

You can contact me using the following details:

- **Gmail /**: hayatadil300@gmail.com

If you’d like to collaborate or have questions about this project, feel free to reach out through the contact section on the website.

---

**Built to showcase not just who I am—but how I think and build.**
