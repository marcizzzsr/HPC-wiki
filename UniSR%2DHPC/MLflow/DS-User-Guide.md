## Introduction

This page describes how a Data Scientist can access the on-premise deployment of MLflow to track their experiments on the workstations.

## Browser Usage

- The instance of mlflow on premise is reachable (via San Raffaele network or behind VPN) from regular browser at https://mlflow-server.ihsr.ai-hub.it/.
After reaching the page, click on *Login with KeyCloak*
![image.png](/.attachments/image-97bd59cd-b8a9-4883-8c8c-d3e709cd7b5a.png)

- You will be redirected to a KeyCloak login page, where you can click on *EntraID HSR* to login with your HSR Entra account

![Keycloak_MLflow.png](/.attachments/Keycloak_MLflow-5ef61a74-938a-4cec-ae43-68273c9761ee.png)

- If everything is setup correctly, you should see the interface of mlflow. If you encounter any error (e.g. an error message stating that the user is not allowed to access the application) please reach out to the IT Team in SRACE (Alan Zambello, Alex Collini, Samuele Conti).

- Once logged in, you should see (depending on the deployment configuration) the interface of either the OIDP MLflow plugin or the standard Mlflow UI. Please note that you can switch between the two by clicking on the dedicated tab on the top right of the screen

![Keycloak_MLflow_2.png](/.attachments/Keycloak_MLflow_2-ad3cd047-1622-4525-a468-b1d779d31e0d.png)

![Keycloak_MLflow_3.png](/.attachments/Keycloak_MLflow_3-eff9dd43-508f-4ab1-bacf-2a25b3b9dbd2.png)

- If you want to use the usual mlflow interface, you can simply press the *MLflow* button on the top right on the screen. The *Permissions* tab is automatically added by the OIDP plugin, and allows you to still see the experiments of mlflow by checking the *Experiments* tab on the left. Still, for practical purposes, switching to the usual mlflow view is advised for day-to-day activities.

- The main reason to access the *Permission* page is to create a personal access token that you will need to use to connect to the mlflow server from the workstation on premise: in order to do that, you can press *Create Access Token*, set an expiration date to the token and create a new one. Please store the token safely.

![Keycloak_MLflow_4.png](/.attachments/Keycloak_MLflow_4-10047b0f-0a69-4b88-8f5b-35540847ff62.png)

- The other feature of mlflow are exactly as standard mlflow distributions.

## Code Usage

Please note that you will need to log in to the mlflow interface via browser at least one time before being able to connect also purely via code (see the section above to follow the necessary steps).

When developing Python code directly from the workstation (via the SSH connection), you can connect to the mlflow server on premise by adding the following lines to your code. You will need the email used to login via KeyCloak in the browser with the HSR Entra ID and the token created in your personal page in the mlflow permissions page.

```
mlflow.set_tracking_uri("https://mlflow-server.ihsr.ai-hub.it/")
os.environ["MLFLOW_TRACKING_USERNAME"] = "hsr_email_address_used_for_keycloak_login_in browser"
os.environ["MLFLOW_TRACKING_PASSWORD"] = "token_created_via_browser"
```

After that, all usual mlflow commands are supported (e.g. `mlflow.set_experiment`, `mlflow.start_run`, etc.): you will be able to run the code and check the results directly via the mlflow interface in the browser.
