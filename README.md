Google and Meta Authentication by Priyanshi Singh- DS, GGITS'28


from django.urls import path
from . import views
from .views import USAJobsView

 path('suggest-resume/', views.suggest_resume, name='suggest_resume'),
    path('convert-to-cv/', views.convert_to_cv, name='convert_to_cv'),  # Route for resume upload and suggestions 
     path('api/jobs/', USAJobsView.as_view(), name='job-list'), 
    path('fetch-internships/', views.fetch_internships_view, name='fetch_internships'),
    path('fetch-jobs/',views.fetch_jobs_view,name='fetch_jobs'),
     path("fetch-ijobs/", views.job_search_view, name="fetch_ijobs"),




