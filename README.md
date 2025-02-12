Google and Meta Authentication by Priyanshi Singh- DS, GGITS'28


# forms.py
from django import forms

class InternshipSearchForm(forms.Form):
    keyword = forms.CharField(max_length=100, required=False, label="Search for Internship")
    location = forms.CharField(max_length=100, required=False, label="Location")



from django import forms

class JobSearchForm(forms.Form):
    keyword = forms.CharField(max_length=100, required=True, label="Search for Job")
    location = forms.CharField(max_length=100, required=False, label="Location")




from django import forms
from .models import UserProfile

class ResumeUploadForm(forms.ModelForm):
    class Meta:
        model = UserProfile
        fields = ['resume']





