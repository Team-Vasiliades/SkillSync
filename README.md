//Google and Meta Authentication by Priyanshi Singh- DS, GGITS'28

import io
from django.db import models
from django.contrib.auth.models import User
from django.core.exceptions import ValidationError
import fitz  # PyMuPDF
from fpdf import FPDF
from io import BytesIO
from PIL import Image
from .utils import convert_docx_to_pdf  # Import the function from utils.py
