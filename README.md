//Google and Meta Authentication by Priyanshi Singh- DS, GGITS'28

#Views.py

def register(request):
    if request.method == 'POST':
        username = request.POST['username']
        email = request.POST['email']
        password1 = request.POST['password1']
        password2 = request.POST['password2']

        if password1 != password2:
            messages.error(request, "Passwords do not match!")
            return redirect('register')

        if User.objects.filter(username=username).exists():
            messages.error(request, "Username already taken!")
            return redirect('register')

        if User.objects.filter(email=email).exists():
            messages.error(request, "Email already registered!")
            return redirect('register')

        # Create the user
        user = User.objects.create_user(username=username, email=email, password=password1)
        user.save()
        messages.success(request, "Registration successful! Please log in.")
        return redirect('login')
    return render(request, 'register.html')


def login_user(request):
    if request.method == 'POST':
        username = request.POST['username']
        password = request.POST['password']

        user = authenticate(request, username=username, password=password)
        if user is not None:
            login(request, user)
            messages.success(request, "Login successful!")
            return redirect('home')  # Replace with your dashboard or home page
        else:
            messages.error(request, "Invalid username or password!")
            return redirect('login')
    return render(request, 'login.html')



def logout_user(request):
    logout(request)
    messages.success(request, "You have been logged out.")
    return redirect('login')


@login_required
def upload_resume(request):
    if request.method == 'POST':
        uploaded_file = request.FILES.get('resume')  # Get the uploaded file

        if uploaded_file:
            file_type = uploaded_file.name.split('.')[-1].lower()

            if file_type not in ['pdf', 'docx', 'jpeg', 'jpg', 'png', 'tiff']:
                messages.error(request, "Only pdf, docx, jpeg, jpg, png, and tiff files are allowed.")
                return redirect('upload')

            # Convert DOCX to PDF if necessary
            if file_type == 'docx':
                try:
                    pdf_content = convert_docx_to_pdf(uploaded_file)
                except Exception as e:
                    messages.error(request, f"Error processing DOCX file: {e}")
                    return redirect('upload')
            elif file_type in ['jpeg', 'jpg', 'png', 'tiff']:
                try:
                    resume_instance = Resume()  # Create an instance of Resume (you don't need to save yet)
                    pdf_content = resume_instance.convert_itp(uploaded_file)
                except Exception as e:
                    messages.error(request, f"Error processing image file: {e}")
                    return redirect('upload')
            else:
                pdf_content = uploaded_file.read()  # Read PDF directly

            # Save the PDF content in the database
            Resume.objects.create(user=request.user, resume=pdf_content) 
            messages.success(request, "Resume uploaded successfully!")
            return redirect('upload')
        else:
            messages.error(request, "No file uploaded.")
    return render(request, 'upload.html')


def download_resume(request, resume_id):
    # Fetch the resume instance from the database
    resume_instance = get_object_or_404(Resume, id=resume_id)
    
    if resume_instance.resume:
        # Create a response object to serve the file
        response = HttpResponse(resume_instance.resume, content_type='application/pdf')
        response['Content-Disposition'] = f'attachment; filename="resume_{resume_id}.pdf"'
        return response
    else:
        return HttpResponse("No file found.", status=404)



def resume_editor(request):
    return render(request, 'resume_editor.html')


@login_required
def save_resume_pdf(request):
    if request.method == "POST":
        resume_text = "hello world "  # Replace with actual text from request.POST

        if not resume_text:
            return JsonResponse({"message": "Error: No resume text provided."}, status=400)

        try:
            # Create PDF using FPDF
            pdf = FPDF()
            pdf.set_auto_page_break(auto=True, margin=15)
            pdf.add_page()
            pdf.set_font("Arial", size=12)
            pdf.multi_cell(0, 10, resume_text)

            # Get binary data of PDF
            pdf_binary = pdf.output(dest='S').encode('latin1')  # Properly get binary content

            # Save to database
            filename = f"{request.user.username}_resume.pdf"
            resume = Resume.objects.create(user=request.user, resume=pdf_binary, filename=filename)

            return JsonResponse({"message": "Resume saved successfully!", "filename": filename})

        except Exception as e:
            return JsonResponse({"message": f"Error: {str(e)}"}, status=500)

    return render(request, 'save_resume_pdf.html')








