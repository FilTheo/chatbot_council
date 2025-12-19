# Deployment Instructions

This guide explains how to deploy the **LLM Council** demo to **Vercel** and **Google Cloud Run**.

## 0. Prerequisites: Get your Hugging Face Key

Both deployments require a Hugging Face API key to access the models.

1.  Go to [huggingface.co/settings/tokens](https://huggingface.co/settings/tokens).
2.  Click **Create new token**.
3.  Name it (e.g., `council-demo`), select **Read** permissions, and create it.
4.  Copy the token (starts with `hf_...`). You will need this later.

---

## 1. Deploying to Vercel

Vercel is great for quick demos, but it has strict **timeout limits** (usually 10-60 seconds on the free tier). Since the Council takes time to "think" and vote, **it might timeout** before finishing.

### Steps
1.  **Push your code to GitHub**.
2.  Log in to [Vercel](https://vercel.com/) and click **Add New Project**.
3.  Import your GitHub repository.
4.  In the **Configure Project** screen:
    *   Find the **Environment Variables** section.
    *   Add a new variable:
        *   **Key**: `HF_TOKEN`
        *   **Value**: (Your Hugging Face token)
5.  Click **Deploy**.

---

## 2. Deploying to Google Cloud Run

**Why Cloud Run?**
Vercel's serverless functions can time out if the AI takes too long to answer. **Google Cloud Run** solves this by allowing requests to run for up to 60 minutes. It is better suited for long-running AI deliberations like this one.

### Step 1: Create a Dockerfile
Create a new file named `Dockerfile` (no extension) in your project folder and paste this content:

```dockerfile
# Use an official Python runtime as a parent image
FROM python:3.9-slim

# Set the working directory
WORKDIR /app

# Copy project files
COPY . .

# Install dependencies
# We install gunicorn explicitly for the production server
RUN pip install --no-cache-dir -r requirements.txt && pip install gunicorn

# Expose the port (Google Cloud Run expects port 8080 by default)
ENV PORT=8080

# Command to run the app using Gunicorn
# "app:app" means: look in file 'app.py' for the object named 'app'
CMD exec gunicorn --bind :$PORT --workers 1 --threads 8 --timeout 0 app:app
```

### Step 2: Deploy using the Command Line
Assuming you have the [gcloud CLI](https://cloud.google.com/sdk/docs/install) installed:

1.  **Login**:
    ```bash
    gcloud auth login
    ```

2.  **Deploy**:
    Run this command in your project folder. Replace `[YOUR_HF_TOKEN]` with your actual token.
    ```bash
    gcloud run deploy council-demo --source . --set-env-vars HF_TOKEN=[YOUR_HF_TOKEN]
    ```

3.  **Follow the prompts**:
    *   **Source code location**: Press Enter (current directory).
    *   **Region**: Select a region (e.g., `us-central1` or `europe-west1`).
    *   **Allow unauthenticated invocations?**: Type `y` (Yes) so students can access the website.

4.  **Done!**
    The terminal will show a **Service URL** (e.g., `https://council-demo-xyz.a.run.app`). Click it to view your app.

### Alternative: Manual Deployment (via Browser)
If you don't want to use the command line, you can deploy directly from the Google Cloud Console.

1.  **Push your code to GitHub**.
2.  Go to the [Google Cloud Console](https://console.cloud.google.com/).
3.  Search for **Cloud Run** in the top search bar and click it.
4.  Click **CREATE SERVICE**.
5.  Select **"Continuously deploy new revisions from a source repository"**.
6.  Click **SET UP WITH CLOUD BUILD**.
7.  Select **GitHub** as your Repository Provider and follow the prompts to connect your account and repository.
8.  In the **Build Configuration**:
    *   Branch: `main` (or whichever branch you are using).
    *   Build Type: **Dockerfile**.
    *   Source location: `/Dockerfile`.
9.  Back on the **Create Service** page:
    *   **Service name**: `council-demo`.
    *   **Region**: Select one (e.g., `us-central1`).
    *   **Authentication**: Select **"Allow unauthenticated invocations"**.
10. Expand **"Container(s), Volumes, Networking, Security"**:
    *   **Container Port**: Ensure it is set to `8080`.
    *   Go to the **VARIABLES** tab.
    *   Click **ADD VARIABLE**.
    *   **Name**: `HF_TOKEN`.
    *   **Value**: (Your Hugging Face token).
11. Click **CREATE**.
12. Wait for the build to finish. Once done, you will see a URL at the top of the page.

