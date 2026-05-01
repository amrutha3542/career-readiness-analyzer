# 🚀 Career Readiness Analyzer

## 📌 Description
Career Readiness Analyzer is an AI-based web application that analyzes a user's resume and provides insights about career readiness, strengths, and improvement areas.

## 🛠️ Tech Stack
- Python
- Streamlit
- pdfplumber
- matplotlib

## ✨ Features
- Upload resume (PDF)
- Analyze skills
- Show improvement suggestions
- Visual output (graphs)

## ▶️ How to Run
1. Install dependencies:
   pip install -r requirements.txt

2. Run the app:
   streamlit run app.py

## 📷 Screenshots
import streamlit as st
import pdfplumber
import matplotlib.pyplot as plt
import numpy as np

# ---------------------------------
# PAGE CONFIG
# ---------------------------------
st.set_page_config(page_title="AI Career Analyzer Pro", layout="wide")

# ---------------------------------
# BACKGROUND
# ---------------------------------
def set_bg():
    st.markdown("""
    <style>
    .stApp {
        background-image: url("https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcS_ioLM8c1BDf96SMLzjumY51hCBz9e1fyqhQ&s");
        background-size: cover;
        background-attachment: fixed;
    }
    .block-container {
        background-color: rgba(0,0,0,0.70);
        padding: 2rem;
        border-radius: 10px;
    }
    h1,h2,h3,p,label {color:white;}
    </style>
    """, unsafe_allow_html=True)

set_bg()

st.title("🚀 AI Career Readiness Analyzer PRO")

# ---------------------------------
# LOGIN
# ---------------------------------
if "logged_in" not in st.session_state:
    st.session_state.logged_in = False

if not st.session_state.logged_in:
    st.subheader("🔐 Login")
    u = st.text_input("Username")
    p = st.text_input("Password", type="password")

    if st.button("Login"):
        if u == "admin" and p == "1234":
            st.session_state.logged_in = True
            st.success("Login Success")
            st.rerun()
        else:
            st.error("Invalid")

    st.stop()

if st.button("Logout"):
    st.session_state.logged_in = False
    st.rerun()

# ---------------------------------
# JOB DATABASE
# ---------------------------------
jobs = {
    "Data Scientist": {
        "skills": ["python","sql","machine learning","statistics","pandas"],
        "growth": 40
    },
    "AI Engineer": {
        "skills": ["python","machine learning","deep learning","tensorflow"],
        "growth": 45
    },
    "Software Developer": {
        "skills": ["python","java","dsa","sql"],
        "growth": 38
    },
    "Data Analyst": {
        "skills": ["python","sql","excel","pandas"],
        "growth": 28
    }
}

courses = {
    "python":"Python Course",
    "sql":"SQL Mastery",
    "machine learning":"ML Basics",
    "deep learning":"DL with TensorFlow",
    "statistics":"Statistics for Data Science",
    "java":"Java Programming",
    "dsa":"DSA Practice",
    "excel":"Excel Advanced",
    "pandas":"Pandas Data Analysis"
}

# ---------------------------------
# CHATBOT (RULE BASED)
# ---------------------------------
def chatbot(q):
    q = q.lower()
    if "ai engineer" in q:
        return "Learn Python + ML + DL + TensorFlow"
    elif "data scientist" in q:
        return "Focus on ML + Stats + Python + SQL"
    elif "career" in q:
        return "Upload resume to get best career match"
    return "I can help only with career-related queries"

# ---------------------------------
# FILE UPLOAD
# ---------------------------------
file = st.file_uploader("Upload Resume (PDF)", type=["pdf"])

if file:

    text = ""
    with pdfplumber.open(file) as pdf:
        for page in pdf.pages:
            if page.extract_text():
                text += page.extract_text().lower()

    # ---------------------------------
    # SKILL EXTRACTION
    # ---------------------------------
    found = [s for s in courses.keys() if s in text]

    st.subheader("🧠 Extracted Skills")
    st.success(found)

    # ---------------------------------
    # ATS SCORE FUNCTION
    # ---------------------------------
    def ats_score(job_skills):
        matched = len([s for s in job_skills if s in found])
        return (matched / len(job_skills)) * 100

    scores = {}

    st.subheader("📊 Job Analysis")

    for job, data in jobs.items():

        score = ats_score(data["skills"])
        scores[job] = score

        st.markdown(f"## 💼 {job}")
        st.write(f"Growth: {data['growth']}%")

        st.progress(int(score))
        st.write(f"ATS Score: {round(score,2)}%")

        missing = [s for s in data["skills"] if s not in found]

        st.write("❌ Skill Gap:", missing)

        for m in missing:
            st.write("➡ Learn:", courses.get(m,m))

        st.write("---")

    # ---------------------------------
    # TOP 3 CAREERS
    # ---------------------------------
    top3 = sorted(scores.items(), key=lambda x: x[1], reverse=True)[:3]

    st.subheader("🏆 Top Career Recommendations")

    for i,(k,v) in enumerate(top3):
        st.success(f"{i+1}. {k} - {round(v,2)}%")

    # entities (highlight important roles)
    st.write("Recommended Path:")
    st.write("👉 " , " → ".join([t[0] for t in top3]))

    # ---------------------------------
    # BEST CAREER
    # ---------------------------------
    best = max(scores, key=scores.get)
    st.success(f"🔥 Best Career: {best}")

    # ---------------------------------
    # CAREER ROADMAP
    # ---------------------------------
    st.subheader("🗺 6-Month Roadmap")

    roadmap = [
        "Month 1: Learn Python + SQL",
        "Month 2: Core Machine Learning",
        "Month 3: Build Projects",
        "Month 4: Deep Learning Basics",
        "Month 5: Resume + Internship Apply",
        "Month 6: Interview Prep"
    ]

    for r in roadmap:
        st.write("📌", r)

    # ---------------------------------
    # RESUME RANK
    # ---------------------------------
    rank = np.mean(list(scores.values()))

    st.subheader("📈 Resume Ranking")
    st.metric("Rank Score", f"{round(rank,2)}/100")

    # ---------------------------------
    # CERTIFICATION SUGGESTIONS
    # ---------------------------------
    st.subheader("🎓 Recommended Certifications")

    certs = set()
    for job, data in jobs.items():
        for s in data["skills"]:
            if s not in found:
                certs.add(courses.get(s))

    for c in list(certs)[:5]:
        st.write("🏅", c)

    # ---------------------------------
    # PREDICTION GRAPH
    # ---------------------------------
    st.subheader("📊 Future Demand Prediction")

    years = [2026, 2027, 2028]

    fig, ax = plt.subplots()
    for job in jobs:
        base = jobs[job]["growth"]
        trend = [base, base+5, base+10]
        ax.plot(years, trend, label=job)

    ax.set_title("Job Growth Forecast")
    ax.legend()
    st.pyplot(fig)

    # ---------------------------------
    # PIE CHART
    # ---------------------------------
    st.subheader("📊 Skill Match Distribution")

    labels = list(scores.keys())
    values = list(scores.values())

    report = f"""
CAREER REPORT

Skills: {found}

Top Career: {best}

Scores: {scores}

Top 3: {top3}
"""


## 👩‍💻 Author
Amrutha
