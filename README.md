Google and Meta Authentication by Priyanshi Singh- DS, GGITS'28

import os
from django.core.files.storage import FileSystemStorage
from django.shortcuts import render, redirect
from django.http import JsonResponse
import google.generativeai as genai
from docx import Document
from PyPDF2 import PdfReader
from django.conf import settings
from django.core.files.base import ContentFile

# Configure Generative AI API
genai.configure(api_key="AIzaSyAMHEgaEy6cAJK0bfziGAcLohnbECPyQE8")
model = genai.GenerativeModel("gemini-1.5-flash")

# Utility to read file content
def read_file_content(file_path, file_type):
    content = ""
    if file_type == "pdf":
        reader = PdfReader(file_path)
        for page in reader.pages:
            content += page.extract_text()
    elif file_type in ["docx", "doc"]:
        doc = Document(file_path)
        for para in doc.paragraphs:
            content += para.text + "\n"
    elif file_type == "txt":
        with open(file_path, 'r') as file:
            content = file.read()
    else:
        raise ValueError("Unsupported file format.")
    return content

# Views
def index(request):
    return render(request, 'index.html')

def suggest_resume(request):
    if request.method == 'POST' and request.FILES['resume']:
        # Save the uploaded resume
        resume_file = request.FILES['resume']
        fs = FileSystemStorage()
        filename = fs.save(resume_file.name, resume_file)
        file_path = fs.path(filename)

        # Determine file type
        _, file_extension = os.path.splitext(filename)
        file_type = file_extension.lower().strip('.')

        try:
            # Read file content
            resume_content = read_file_content(file_path, file_type)

            # Generate suggestions using Generative AI
            response = model.generate_content([
                "Suggest improvements for this resume:", resume_content
            ])
            suggestions = response.text
        except Exception as e:
            return render(request, 'suggest.html', {"error": f"Error processing file: {str(e)}"})

        return render(request, 'suggest.html', {"suggestions": suggestions})

    return render(request, 'suggest.html', {"error": "Please upload a valid file."})

def convert_to_cv(request):
    if request.method == 'POST' and request.FILES['resume']:
        # Save the uploaded resume
        resume_file = request.FILES['resume']
        fs = FileSystemStorage()
        filename = fs.save(resume_file.name, resume_file)
        file_path = fs.path(filename)

        # Determine file type
        _, file_extension = os.path.splitext(filename)
        file_type = file_extension.lower().strip('.')

        try:
            # Read file content
            resume_content = read_file_content(file_path, file_type)

            # Create a CV using python-docx
            doc = Document()
            doc.add_heading('Curriculum Vitae', level=1)
            doc.add_paragraph(resume_content)

            # Save the CV to the server
            cv_filename = f"CV_{os.path.splitext(filename)[0]}.docx"
            cv_path = os.path.join(settings.MEDIA_ROOT, cv_filename)
            doc.save(cv_path)

        except Exception as e:
            return render(request, 'cv_converter.html', {"error": f"Error processing file: {str(e)}"})

        return render(request, 'cv_converter.html', {"cv_url": f"/media/{cv_filename}"})

    return render(request, 'cv_converter.html', {"error": "Please upload a valid file."})



                                                                                                                                                                                                                                                                            

import asyncio
import aiohttp
from django.shortcuts import render
from django.views import View
from asgiref.sync import sync_to_async

class USAJobsView(View):
    # Asynchronous method to fetch jobs from an API
    async def fetch_jobs(self, api_key, params):
        API_URL = 'https://data.usajobs.gov/api/search'

        headers = {
            'Host': 'data.usajobs.gov',
            'User-Agent': 'utkarshkushwaha246@gmail.com',
            'Authorization-Key': api_key
        }

        async with aiohttp.ClientSession() as session:
            try:
                async with session.get(API_URL, headers=headers, params=params, timeout=10) as response:
                    response.raise_for_status()  # Raise exception for errors
                    jobs = await response.json()
                    return jobs.get('SearchResult', {}).get('SearchResultItems', [])
            except Exception as e:
                return None

    # Asynchronous method to run both API requests in parallel
    async def get_jobs_data(self):
        params = {
            'Keyword': 'internship', 
            'LocationName': 'remote',  
            'ResultsPerPage': 10,  
        }

        api_key_1 = '3WGgHaa+SeqrF14ap76WXFHc1znhD8stg7bEJhkC6Y8='  # First API key
        api_key_2 = '2af21edf-fc6b-4d13-9cd8-17bba4a7ce1d'  # Second API key

        # Run both API calls concurrently
        job_data_1, job_data_2 = await asyncio.gather(
            self.fetch_jobs(api_key_1, params),
            self.fetch_jobs(api_key_2, params)
        )

        return job_data_1, job_data_2

    # Synchronous Django view method that will await async job fetching
    async def get(self, request):
        # Await the asynchronous task to fetch job data
        job_data_1, job_data_2 = await self.get_jobs_data()

        job_listings = []
        # Combine the results from both APIs if they are available
        if job_data_1:
            job_listings.extend([
                {
                    'title': job['MatchedObjectDescriptor']['PositionTitle'],
                    'organization': job['MatchedObjectDescriptor']['OrganizationName'],
                    'location': job['MatchedObjectDescriptor']['PositionLocationDisplay'],
                    'url': job['MatchedObjectDescriptor']['PositionURI'],
                }
                for job in job_data_1
            ])
        
        if job_data_2:
            job_listings.extend([
                {
                    'title': job['MatchedObjectDescriptor']['PositionTitle'],
                    'organization': job['MatchedObjectDescriptor']['OrganizationName'],
                    'location': job['MatchedObjectDescriptor']['PositionLocationDisplay'],
                    'url': job['MatchedObjectDescriptor']['PositionURI'],
                }
                for job in job_data_2
            ])
        
        return render(request, 'jobintern.html', {'jobs': job_listings})


from django.shortcuts import render
from .utils import Scrap_Internshala

def fetch_internships_view(request):
    if request.method == "POST":  
        skill = request.POST.get('skill', '')
        if skill:
            base_url = "https://internshala.com"
            try:
                internships_df = Scrap_Internshala(base_url, skill)
                internships = internships_df.to_dict(orient='records')  
                return render(request, 'internships.html', {'internships': internships, 'skill': skill})
            except Exception as e:
                return render(request, 'internships.html', {'error': str(e)})
        else:
            return render(request, 'internships.html', {'error': 'No skill provided'})
    return render(request, 'internships.html')  


from django.shortcuts import render
from .utils import scrap_jobs

def fetch_jobs_view(request):
    if request.method == "POST":  
        skill = request.POST.get('skill', '')
        if skill:
            base_url = "https://internshala.com"
            try:
                jobs_df = scrap_jobs(base_url, skill)
                jobs = jobs_df.to_dict(orient='records')  
                return render(request, 'jobs.html', {'jobs': jobs, 'skill': skill})
            except Exception as e:
                return render(request, 'jobs.html', {'error': str(e)})
        else:
            return render(request, 'jobs.html', {'error': 'No skill provided'})
    return render(request, 'jobs.html')





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
        model="gpt-4-turbo",
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
                return render(request, "ijobs.html", {"error": str(e)})

    return render(request, "ijobs.html", {"jobs": jobs, "skill": skill})




