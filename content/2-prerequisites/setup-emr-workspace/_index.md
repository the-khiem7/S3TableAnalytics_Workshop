---
title: "Setup EMR Workspace"
date: 2026-07-31
weight: 3
chapter: false
pre: " <b>3.3. </b>"
---

{{% notice info %}}
This section covers setting up the EMR Workspace that will be used for processing data in S3 Tables during the workshop.
{{% /notice %}}

1. Navigate to the [EMR Console](https://console.aws.amazon.com/emr/home) page.
   - ![EMR console](/images/workshop/emr_console.webp)

2. Select 'EMR Serverless' from the left menu.

3. In the 'EMR Studio' section on the right, select 'workshop_emr_studio' and click the 'Manage applications' button.

4. Clicking this button will open the EMR Studio page in a new window.
   - You will see an EMR Serverless Application created with the name 'workshop_emr_application'.
   - ![EMR serverless application](/images/workshop/emr_application.png)

5. Click on 'Workspaces' in the left menu of EMR Studio.

6. Click the 'Create Workspace' button.

7. Enter 'workshop-workspace' for the 'Workshop name', then click the 'Create Workspace' button.
   - ![Create workspace](/images/workshop/create_workspace.png)

8. After creation, a new window with the JupyterLab page will open. (Allow your browser to open new windows.)
   - ![JupyterLab](/images/workshop/jupyterlab_notebook.webp)

9. Click the 'EMR Compute' icon on the left side of JupyterLab to attach the EMR Serverless Application to JupyterLab.
   - ![Attach EMR application](/images/workshop/attach_emr_application.png)
   - EMR Serverless application: Select `workshop_emr_application`
   - Interactive runtime role: Select `Workshop_Job_Role`

{{% notice warning %}}
The EMR workspace setup is now complete.
{{% /notice %}}
