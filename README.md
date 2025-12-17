MLFLOW SETUP ON AWS EC2 (BEGINNER RUNBOOK)

=====================================
1. AWS PRE-REQUISITES
=====================================

1. Login to AWS Console
2. Create an IAM User
   - Attach policy: AdministratorAccess
3. Create Access Key for the IAM user

4. Create an S3 bucket
   Example:
   - Bucket name: mlflow-bucket-chinu
   - Region: same as EC2

5. Create EC2 instance
   - AMI: Ubuntu
   - Instance type: t2.micro / t3.small
   - Storage: default
   - Key pair: create/download

6. Configure Security Group
   Add inbound rule:
   - Type: Custom TCP
   - Port: 5000
   - Source: 0.0.0.0/0 (for learning only)

=====================================
2. CONNECT TO EC2
=====================================

ssh -i your-key.pem ubuntu@<EC2_PUBLIC_IP>

=====================================
3. SYSTEM SETUP ON EC2
=====================================

sudo apt update

sudo apt install -y python3-pip

sudo apt install -y pipenv

sudo apt install -y virtualenv

=====================================
4. PROJECT SETUP
=====================================

mkdir mlflow
cd mlflow

pipenv install mlflow
pipenv install awscli
pipenv install boto3

pipenv shell

=====================================
5. AWS CREDENTIAL CONFIGURATION
=====================================

aws configure

Provide:
- AWS Access Key ID
- AWS Secret Access Key
- Default region (example: us-east-1)
- Output format: json

Verify:
aws s3 ls

=====================================
6. FIRST ATTEMPT TO RUN MLFLOW (FAILED)
=====================================

mlflow server -h 0.0.0.0 --default-artifact-root s3://mlflow-bucket-chinu

Issue:
- Browser error:
  "Invalid Host header - possible DNS rebinding attack detected"

Reason:
- New MLflow uses FastAPI security middleware
- External hosts are blocked by default

=====================================
7. FIX: ALLOWED HOSTS & CORS
=====================================

IMPORTANT:
MLflow CLI options MUST use double dashes (--)

Correct command:

mlflow server \
  --host 0.0.0.0 \
  --port 5000 \
  --default-artifact-root s3://mlflow-bucket-chinu \
  --allowed-hosts "*" \
  --cors-allowed-origins "*"

=====================================
8. PORT ALREADY IN USE ERROR (FIX)
=====================================

If you see:
Address already in use (0.0.0.0:5000)

Run:

pkill -f mlflow

Then restart MLflow using the correct command above.

=====================================
9. SUCCESS CHECK
=====================================

Expected log:
Uvicorn running on http://0.0.0.0:5000

Open in browser:
http://<EC2_PUBLIC_IP>:5000

http://ec2-54-227-8-59.compute-1.amazonaws.com:5000 (example)

MLflow UI should load successfully.

=====================================
10. NOTES / LEARNINGS
=====================================

- MLflow now uses FastAPI (not Flask)
- Host validation is strict by default
- --allowed-hosts is mandatory for public EC2 access
- All CLI options require "--"
- "*" is OK for learning, NOT production
- Backend DB defaults to SQLite (mlflow.db)

=====================================
11. NEXT IMPROVEMENTS (OPTIONAL)
=====================================

- Move backend store to RDS (MySQL/Postgres)
- Add Nginx reverse proxy
- Enable HTTPS
- Dockerize MLflow
- Restrict allowed-hosts properly

=====================================
END OF FILE
=====================================
=========================================
12. DVC
==========================================
dvc init

dvc repro

=========================================
13. AWS Deployment 
#optinal

sudo apt-get update -y

sudo apt-get upgrade

#required

curl -fsSL https://get.docker.com -o get-docker.sh

sudo sh get-docker.sh

sudo usermod -aG docker ubuntu

newgrp docker

setting>actions>runner>new self hosted runner> choose os> then run command one by one
 connect your github with aws
   go to your repo and click on settings in action slide to runner and click on it.
   click on self new hosted runner and select linux machince and execute those commands one by one in ec2
   
   ON lefthand side sceret and variables add security keys go to action ADD AWS_ACCESS_KEY_ID
   AWS_SCERET_ACCESS_KEY,AWS_REGION,ECR_LOGIN_URI,ECR_REPOSITORY_NAME

395310663337.dkr.ecr.us-east-1.amazonaws.com/chinuteja/mlfproject (url for ecr)



