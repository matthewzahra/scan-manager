# Scan Manager Web App - developed by James Housden, Matthew Zahra, and Samiksha Vanga for Go Reply.

This application was developed on Google Cloud Platform during an internship at Go Reply and presented back to the rest of the company and its partners.

It serves as a maanger for medical information, allowing patients to upload text, DICOM or NIfTI files.

For text and NIfTI files, it employs Generative AI to give useful sumamries/insights, while for DICOM files, users can use the state of the art OHIF-viewer to look through the 3D representation of the scans.

The application also enforces proper authentication with the Google accounts of users by invoking Firebase.

Below is the setup guide that was given to Go Reply in order to setup the application themselves - it requires a working Google Cloud Project with Billing enabled.

## TO SETUP APPLICATION:

### Folder Structure (for implemented project):
```
flask-app/
│
├── static/
│   ├── images/
│   ├── keys/
│   │   └── key.json       # key needed for authentication - generated from within the service account for firebase
│   └── style.css
│
├── templates/              # html files
│   ├── add-patient.html
│   ├── base.html
│   ├── index.html
│   ├── inspect-file-nifti.html
│   ├── inspect-file-text.html
│   ├── patient-scans.html
│   ├── patient-search.html
│   ├── select-user-type.html
│   └── upload-scan.html
│
├── app.py                  # main script
├── app_helper_functions.py # helper functions
├── temp_scan_file.nii.gz   # Temporary file location used during upload
└── temp_text_file.txt      # Temporary file location used during upload
```
Don't forget to generate the key for the firebase service accouny - we need this to enable the authentication 

### Firestore
Create a Firebase project and then a firestore database. Each time it gets referenced it uses firestore credentials - **these will need to be updated with your specific credentials**. This is where scan and user data gets stored. 
Create 2 collections in it called 'users' and 'scans'

Also, importnat that you add the domain where the application is hosted to the list of "Authorised Domains" in firebase - this is to allow the login functionality to work.

### Google Cloud Storage
We need to initialise 3 buckets for storing text, NIfTI and DICOM files.
3 buckets must be initialisded - in our code, we assume that the location is London (europe-west2) and we name our buckets 'mri_test_bucket', 'dicom_bucket_store' and 'text_file_store'

In the cloud functions, the names of the buckets will need to be changed to reflect the URL of your buckets - this will contain your Google Cloud Project name, so will be unique to you

### DICOM Store
Enable the healthcare api, create a new dataset called 'dicom-dataset' and then create a datastore within that dataset called 'dicom-datastore'
You will need to update the URL to this datastore in your cloud functions as it is unique to your project.
This is where the .dcm fiels get stored.

### Cloud Functions
1) Create google cloud functions corresponding to each file in the cloud_functions/ folder
2) Enter the generated URL for each cloud function into the appropriate lines in 'scan-manager-flask-app/flask-app/url_config.json'

the cloud function 'delete-patient' makes use of the cloud function 'delete-files' - so it is important that the code for the former includes the link to the latter

IMPORTANT: ensure that all changes (deteailed above) necessary to make Firestore, Google Cloud Storage (buckets) and the DICOM Store working have been made to each cloud function

NB: Ensure that the requirements.txt file for each cloud function contains the relevant libraries

NB: Certain cloud functions are multipurpose e.g. download_data is used for downloading either scans or scan metadata depending on the arguments passed to it; upload_scan_and_patient is used for either uploading a scan or a patient depending on the arguments passed to it

NB: upload-scan-and-patient data needs a timeout of at least 2 minutes - as moving the files into the DICOM store can be slow

NB: some cloud funtions have 512 MiB of memory allowed (default is 256 MiB) - set them all to this if unsure (any that handle files may need it for larger inputs)

### OHIF Viewer
1) Set up OHIF viewer application by following the README_goreply file located in the repo at https://github.com/jameshousdenreply/Viewers. 
2) Enter the URL for the OHIF viewer in the appropriate line (labelled OHIF_VIEWER) in 'scan-manager-flask-app/flask-app/url_config.json'

---

### scan-manager-flask-app/
1) Ensure URLs of cloud functions and OHIF viewer service have been added to 'scan-manager-flask-app/flask-app/url_config.json' (see above)
2) Run the web app:
- To run locally with flask for debugging and testing, navigate to 'scan-manager-flask-app/flask-app', then run 'flask run' in terminal
- Alternatively to build docker image, the Dockerfile is contained in the 'scan-manager-flask-app' directory

### Chatbot
It is created in DialogflowCX and is trained on the text files 'chatbot_knolwedge.txt' and 'app_usage.txt'

### General
Any other changes should be detailed at the top of each file in comments - read each one and see what variable values you need to change to reflect your unique project instance

NB: NIfTI viewing from multiple instances causes conflict issues. The scan displayed tends to default to whichever scan was present in the repository at the time of building the docker image. The exact cause of the issue hasn't been identified, but is of low priority since DICOM format has been prioritised during development.
