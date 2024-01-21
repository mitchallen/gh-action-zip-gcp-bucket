gh-action-zip-gcp-bucket
==

See article:

* [How to Copy to a Google Storage Bucket with GitHub Actions (GCP)](https://scriptable.com/cloud/github-actions-gcp-bucket/)

Use a github action to push a folder to a GCP bucket

An example of how to:

* zip a folder via github actions
* push it as a vtag (see below)
* this will copy it to a GCP bucket

## Push Tag

```sh
git checkout main
git tag 
git tag v1.0.1  
git push origin --tags
```

## Notes

Below you will find the original ChatGPT response when I asked how to do this.

It left out a few details, like:

* How to create the SA account
* How to create the bucket

So after the initial response, see responses after I asked for more details.

I also changed things up a bit to:

* only trigger on a tag push
* get GCP project ID from a secret (it could probably been ok as just a var?)

### ChatGPT first pass:

Here's the original response from ChatGPT:

Creating a GitHub Action to zip a folder and upload it to a Google Cloud bucket involves several steps. Here's a guide on how to do it:

### 1. Setting up Google Cloud Credentials

First, you'll need to create a service account in your Google Cloud project and download the JSON key file. This service account should have the necessary permissions to write to your bucket.

### 2. Adding Secrets to GitHub Repository

- Go to your GitHub repository.
- Navigate to `Settings` > `Secrets`.
- Add your Google Cloud service account key as a new secret. Name it `GCP_SA_KEY`.

### 3. Creating the GitHub Action

Create a new file in your repository: `.github/workflows/upload-to-gcp.yml` and add the following content:

```yaml
name: Zip and Upload to GCP Bucket

on: [push] # or any other event

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v2
    
    # Step to Zip the folder
    - name: Zip the folder
      run: zip -r my-folder.zip my-folder/
    
    # Step to Authenticate with GCP
    - name: Set up Cloud SDK
      uses: google-github-actions/setup-gcloud@master
      with:
        version: 'latest'
        service_account_key: ${{ secrets.GCP_SA_KEY }}
        project_id: YOUR_PROJECT_ID
    
    # Step to Configure Docker to use the gcloud command-line tool as a credential helper
    - run: |-
        gcloud auth configure-docker
    
    # Step to Upload the file to GCP Bucket
    - name: Upload to Google Cloud Storage
      run: gsutil cp ./my-folder.zip gs://YOUR_BUCKET_NAME/path/to/destination/
```

In the above file:

- Replace `my-folder/` with the path to your folder.
- Replace `YOUR_PROJECT_ID` with your Google Cloud project ID.
- Replace `YOUR_BUCKET_NAME` and `/path/to/destination/` with your Google Cloud Storage bucket name and the path where you want to store the file.

### 4. Triggering the Action

This action will trigger on every push to the repository. You can change the trigger event in the `on:` section based on your needs (e.g., on release, on pull request).

When this action runs, it will:

1. Check out your repository.
2. Zip the specified folder.
3. Authenticate with Google Cloud using the service account key.
4. Upload the zipped folder to the specified Google Cloud Storage bucket.

Make sure to test this workflow and ensure that your service account has the necessary permissions to perform these operations.


* * *

### ChatGPT after I asked for more details on the SA creation

Certainly! Setting up Google Cloud credentials properly is critical for your GitHub Action to interact with Google Cloud services. Here’s a detailed guide on how to do it:

### Step 1: Setting up Google Cloud Credentials

#### 1.1 Create a Service Account

A service account is a special kind of account used by applications, not people. It's intended to provide specific, scoped access to your GCP resources.

1. **Go to the IAM & Admin Page**: Log in to your Google Cloud Platform console and navigate to "IAM & Admin > Service accounts".
2. **Create a Service Account**: Click on "Create Service Account". Fill in the service account name and description, then click "Create".
3. **Grant this service account access to the project**: Assign the role needed to perform the tasks. For uploading files to a bucket, roles like "Storage Object Admin" or "Storage Object Creator" might be sufficient. Be careful with the permissions; adhere to the principle of least privilege.
4. **Create a key**: Click on "Create Key". Choose the JSON key type. This will download a JSON file containing your key. Secure this file, as it provides access to your Google Cloud resources.

#### 1.2 Secure the Service Account Key

The downloaded JSON file is what you’ll use to authenticate your GitHub Action with Google Cloud. However, you shouldn’t just put this file in your repository. Instead:

1. **Go to Your Repository on GitHub**: Open the repository where you want to use the GitHub Action.
2. **Store the JSON Key as a Secret**:
    - Navigate to "Settings > Secrets".
    - Click on "New repository secret".
    - Name the secret (e.g., `GCP_SA_KEY`) and paste the entire JSON file content into the value field.

#### 1.3 Use the Service Account Key in GitHub Actions

In your GitHub Actions workflow file (the `.yml` file you're setting up), you can use the secret as an environment variable or directly in steps. For instance:

```yaml
- name: Set up Cloud SDK
  uses: google-github-actions/setup-gcloud@master
  with:
    version: 'latest'
    service_account_key: ${{ secrets.GCP_SA_KEY }}
    project_id: YOUR_PROJECT_ID
```

In this snippet, `${{ secrets.GCP_SA_KEY }}` will be replaced by the content of the `GCP_SA_KEY` secret you set in your repository's settings, effectively authenticating your GitHub Action with Google Cloud.

#### 1.4 Additional Security Measures

- **Least Privilege**: Ensure the service account has only the permissions necessary to perform its tasks. Avoid using roles with extensive permissions unless absolutely necessary.
- **Audit and Rotate Keys**: Regularly check who has access to your service account keys and rotate them if necessary.

By following these detailed steps, your GitHub Action should be able to securely authenticate with Google Cloud and perform tasks like uploading files to a GCS bucket. Remember to handle your service account keys with care, as they provide direct access to your Google Cloud resources.

* * *

### ChatGPT on how to create the bucket and grant permission

Creating a bucket in Google Cloud Storage (GCS) and ensuring that a service account has the appropriate permissions to create objects in it involves a few steps. Here's a detailed guide:

### 1. Create a Bucket in Google Cloud Storage

#### 1.1. Using Google Cloud Console

1. **Open the Cloud Storage Browser**: Go to your Google Cloud Console, navigate to "Storage", and then click on "Browser".
2. **Create a Bucket**: Click on "Create bucket".
3. **Configure Your Bucket**:
   - Enter a unique name for your bucket.
   - Choose how your data will be stored (Standard, Nearline, Coldline, or Archive).
   - Choose a location where your bucket data will be stored.
   - Configure other settings as needed (e.g., access control, lifecycle rules).
4. **Create**: Click "Create".

#### 1.2. Using Command Line (gcloud CLI)

If you prefer using the command line, you can use the `gcloud` command. Ensure you have [Google Cloud SDK](https://cloud.google.com/sdk) installed and initialized.

```bash
gcloud storage buckets create [BUCKET_NAME] --project=[PROJECT_ID] --location=[LOCATION] --storage-class=[STORAGE_CLASS]
```

Replace `[BUCKET_NAME]`, `[PROJECT_ID]`, `[LOCATION]`, and `[STORAGE_CLASS]` with your bucket name, project ID, location, and storage class respectively.

### 2. Grant Permissions to the Service Account

After creating the bucket, you need to ensure that your service account has the necessary permissions to create objects in the bucket.

#### 2.1. Using Google Cloud Console

***MCA - I think this is already done by default?***

1. **Navigate to Your Bucket**: In the Cloud Storage browser in the Google Cloud Console, click on your bucket's name.
2. **Open Permissions Tab**: Click on the "Permissions" tab.
3. **Add Members**:
   - Click "Add".
   - In the "New members" field, enter the service account's email address.
   - In the "Role" dropdown, select a role like "Storage Object Admin" (for full control over objects) or "Storage Object Creator" (if it only needs to create objects).
   - Click "Save".

#### 2.2. Using Command Line (gcloud CLI)

Alternatively, you can use the `gcloud` command to assign permissions.

```bash
gcloud projects add-iam-policy-binding [PROJECT_ID] \
  --member=serviceAccount:[SERVICE_ACCOUNT_EMAIL] \
  --role=roles/storage.objectAdmin
```

Replace `[PROJECT_ID]` with your project ID and `[SERVICE_ACCOUNT_EMAIL]` with the service account's email address.

### 3. Verify the Permissions

Ensure that the service account has the necessary permissions by attempting to upload an object to the bucket using the service account credentials.

### Notes

- **Permissions Propagation**: Keep in mind that it may take a few minutes for permissions changes to propagate.
- **Security Best Practices**: Always follow the principle of least privilege. Grant only the permissions necessary for the service account to perform its tasks.

By following these steps, you should have a bucket ready in Google Cloud Storage, and your service account should have the necessary permissions to create objects in that bucket.

* * *

## References

* https://baconipsum.com/

