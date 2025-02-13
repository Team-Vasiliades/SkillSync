Google and Meta Authentication by Priyanshi Singh- DS, GGITS'28

import openai
import requests
from django.shortcuts import render

# API Keys (Replace with your actual keys)
SERPAPI_KEY = "b1b74ec92d0858703b2a9bb9b5e0ac21746fd3bae2b9359dc6cedd581008242e"
OPENAI_API_KEY = "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"

def fetch_jobs_from_google(skill):
    """Fetch job listings from Google using SerpAPI"""
    url = "https://serpapi.com/search"
    params = {
        "engine": "google_jobs",
        "q": f"{skill} jobs",
        "hl": "en",
        "api_key": SERPAPI_KEY,
    }

    response = requests.get(url, params=params)
    data = response.json()
    
    jobs = []
    if "jobs_results" in data:
        for job in data["jobs_results"]:
            jobs.append({
                "title": job.get("title", "N/A"),
                "company": job.get("company_name", "N/A"),
                "location": job.get("location", "N/A"),
                "link": job.get("link", "#")
            })

    return jobs

def refine_jobs_with_chatgpt(jobs):
    """Use OpenAI API to refine and structure job data"""
    openai.api_key = OPENAI_API_KEY
    prompt = f"Format the following job listings into a structured table with site link:\n\n{jobs}"

    response = openai.ChatCompletion.create(
        model="gpt-3.5-turbo",
        messages=[{"role": "user", "content": prompt}]
    )

    return response["choices"][0]["message"]["content"]

def job_search_view(request):
    jobs = []
    skill = ""

    if request.method == "POST":
        skill = request.POST.get("skill", "")
        if skill:
            try:
                jobs = fetch_jobs_from_google(skill)
                formatted_jobs = refine_jobs_with_chatgpt(jobs)
            except Exception as e:
                return render(request, "jobs.html", {"error": str(e)})

    return render(request, "jobs.html", {"jobs": jobs, "skill": skill})






