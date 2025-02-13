//Google and Meta Authentication by Priyanshi Singh- DS, GGITS'28

#utils.py

def convert_docx_to_pdf(file):
    """
    Converts a DOCX file to PDF with full fidelity using docx2pdf.
    Also ensures COM initialization to prevent errors.
    """
    # Ensure COM is initialized
    pythoncom.CoInitialize()

    try:
        with tempfile.TemporaryDirectory() as temp_dir:
            # Save the uploaded DOCX file temporarily
            temp_docx_path = os.path.join(temp_dir, "uploaded.docx")
            with open(temp_docx_path, "wb") as temp_docx:
                temp_docx.write(file.read())

            # Convert the DOCX file to PDF
            temp_pdf_path = os.path.join(temp_dir, "converted.pdf")
            convert(temp_docx_path, temp_pdf_path)

            # Read the generated PDF and return as binary data
            with open(temp_pdf_path, "rb") as temp_pdf:
                pdf_content = temp_pdf.read()

        return pdf_content
    except Exception as e:
        raise Exception(f"Error processing DOCX file: {str(e)}")
    finally:
        pythoncom.CoUninitialize()  # Uninitialize COM after use
 
