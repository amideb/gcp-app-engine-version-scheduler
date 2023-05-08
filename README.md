
## GCP App Engine Version Scheduler

To automatically shut down and start a Google Cloud Platform (GCP) App Engine version at specific times, you can use Cloud Scheduler to trigger Cloud Functions. Here's a step-by-step guide to accomplish this:

1.  Enable the required APIs: Enable the following APIs on your GCP project if not already enabled:

-   App Engine Admin API
-   Cloud Functions API
-   Cloud Scheduler API

2.  Create a Cloud Function to start and stop the App Engine version: Create a Python-based Cloud Function that takes the action (start/stop), project ID, service, and version as input and performs the specified action on the App Engine version. To do this, follow these steps:

a. In the GCP Console, navigate to Cloud Functions and click "Create Function."

b. Give your function a name, e.g., "app_engine_version_controller" and choose an appropriate region.

c. Set the runtime to "Python 3.9" or a version of your choice.

d. In the "Function to execute" field, input "control_version."

e. In the "Inline editor," replace the contents of `main.py` with the following code:

```
import os
from google.cloud import appengine_v1

def control_version(request):
    action = request.args.get('action')
    project_id = request.args.get('project_id')
    service = request.args.get('service')
    version = request.args.get('version')
    
    appengine_client = appengine_v1.ServicesClient()
    
    if action == 'start':
        appengine_client.update_traffic(project_id, service, {version: 1})
    elif action == 'stop':
        appengine_client.update_traffic(project_id, service, {version: 0})
    else:
        return 'Invalid action specified'
    
    return f'{action.capitalize()} request for {service}/{version} completed'
  ```

f. Replace the contents of `requirements.txt` with the following:

```
google-cloud-appengine==1.0.0
```
g. Click "Create" to deploy the Cloud Function.

3.  Create Cloud Scheduler jobs to start and stop the App Engine version:

a. In the GCP Console, navigate to Cloud Scheduler and click "Create Job."

b. Give your job a name, e.g., "stop_app_engine_version" and choose an appropriate region.

c. Set the frequency (Cron expression) to `30 15 * * *` (This means the job will run at 3:30 PM daily).

d. Choose "HTTP" as the target type and set the HTTP method to "GET."

e. In the "URL" field, input the Cloud Function URL with the required parameters:

```
https://REGION-PROJECT_ID.cloudfunctions.net/app_engine_version_controller?action=stop&project_id=PROJECT_ID&service=SERVICE_NAME&version=VERSION_NAME
```

Replace `REGION`, `PROJECT_ID`, `SERVICE_NAME`, and `VERSION_NAME` with the appropriate values for your setup.

f. Click "Create" to create the Cloud Scheduler job.

g. Repeat the same process to create another Cloud Scheduler job with the name "start_app_engine_version" and a frequency (Cron expression) of `40 15 * * *` (This means the job will run at 3:40 PM daily). Update the `action` parameter in the URL to `start`.

Now, your App Engine version will shut down at 3:30 PM and start at 3:40 PM daily.
