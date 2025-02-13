//Google and Meta Authentication by Priyanshi Singh- DS, GGITS'28

#Views.py
from django.shortcuts import render, redirect, get_object_or_404
from django.contrib.auth.models import User
from django.contrib.auth import authenticate, login, logout
from django.contrib import messages
from django.contrib.auth.decorators import login_required
from .models import Resume
from .utils import convert_docx_to_pdf  # Assuming you already have this function for DOCX
from django.http import HttpResponse, JsonResponse
from django.views.decorators.csrf import csrf_exempt
from django.utils.decorators import method_decorator
import json
from fpdf import FPDF
import io
from .models import Resume





