## Introduction
The following page describes the infrastructural setup of the mlflow deployment on premise.

## Architecture
The deployment of the mlflow service is based on three main services:
- an mlflow deployment on premise, storing artifacts on local storage and connecting to a PostgreSQL DB to store metadata information about experiments and run. The mlflow deployment is based on mlflow with the addition of a public OIDC plugin (https://github.com/mlflow-oidc/mlflow-oidc-auth/tree/main) to support an authentication and authorization mechanism on top of the standard mlflow features.
Please note that the server should also be reachable from a machine outside on premise (e.g. a DS computer) behind VPN or hospital network.
- a PostregSQL deployment on premise, reachable from mlflow. Two databases are required to support mlflow and the OIDP integration: *mlflow_auth* and *mlflow_tracking* (empty databases only are sufficient in the initialization phase)
- A keycloak deployment on cloud, with a dedicated realm for mlflow (see dedicated section for configuring it).

## MLflow Dockerfile
The mlflow deployment is based on a Dockerfile taking a *ghcr.io/mlflow/mlflow* image; a standard set of requirements to support mlflow is defined in the *requirements.txt* following a freeze of the libraries *mlflow-oidc-auth[full]* and *psycopg2* (to connect to PostgreSQL).

To avoid issues with the compatibility between mlflow and the database, a minimum requirements.txt should comprise *mlflow-oidc-auth* and all the *mlflow* dependencies that are installed with it (*mlflow*, *mlflow-skinny*, *mlflow-tracing*). Not freezing the mlflow dependencies may cause errors when upgrading the mlflow-oidc-auth library (errors mainly due to change of database schema in mlflow between releases).

Additionally, a custom code to implement a proper logout mechanism was added, by replacing a specific *.py* module of the original plugin after the pip install stage. This module will need to be evaluated to support any possible library update.

You can see the code details here: https://dev.azure.com/HSREMIC/DevOps/_git/MLflow

## OIDP Plugin Configuration
While the official plugin documentation is not particularly extensive aside the list of supported env variables (https://github.com/mlflow-oidc/mlflow-oidc-auth/blob/main/docs/configuration.md) , local tests confirmed the purpose of each of the environment variables configured for the OIDP plugin deployment in MLflow. Here's a list with a small explanation:

- OIDC_DISCOVERY_URL -> *{keycloak_url}/realms/{mlflow_realm}/.well-known/openid-configuration*
- OIDC_CLIENT_ID -> client for mlflow (e.g. *mlflow_client*)
- OIDC_CLIENT_SECRET -> secret of the client above
- OIDC_REDIRECT_URI -> *{url_of_mlflow_server}/callback*
- OIDC_USERS_DB_URI -> connection string used by mlflow to connect to PostgreSQL (e.g. *postgresql://.../mlflow_auth*), assuming the existence of a DB called *mlflow_auth*
- OIDC_SCOPE -> *openid email profile* (Keycloak does not support comma separated scopes)
- OIDC_GROUP_NAME -> name of the group of standard MLflow users configured in Keycloak, with a '/' before the name (needed for Keycloak specifically): the OIDP plugin will verify if the logged in User is part of this group to allow access (or the admin group). Example: */mlflow_users*
- OIDC_ADMIN_GROUP_NAME -> name of the group of admin MLflow users configured in , with a '/' before the name (needed for Keycloak specifically). Same mechanism as above. Example: */mlflow_admins*. 
- OIDC_PROVIDER_DISPLAY_NAME -> string appearing in the login button of the plugin (e.g. *Login with KeyCloak*)
- SECRET_KEY -> a secret key used by the Flask deployment of the plugin (needed to avoid weird authorization error after Users' login)

Not connected to the plugin, but other env variables needed to the mlflow deployment are:
- MLFLOW_BACKEND_STORE_URI -> connection string to PostgreSQL, e.g. *postgresql://.../mlflow_tracking*, assuming the existence of a DB named *mlflow_tracking* (name required by mlflow) used to store metadata about mlflow experiments.
- POSTGRES_USER -> user used by mlflow to connect to PostgreSQL
- POSTGRES_PASSWORD -> password of the user above
- LOG_LEVEL -> log level of the OIDP plugin (*WARNING* should be a good compromise)
- DEFAULT_LANDING_PAGE_IS_PERMISSIONS -> optional -> can be set to *False* if Users want to land to the standard mlflow page instead of the Permission one of the plugin.

## Keycloak configuration

The following configuration are needed in Keycloak to support a correct functioning of the OIDP mlflow plugin on premise:

- a dedicated realm (e.g. *mlflow*)
- a client in the realm (e.g. *mlflow_client*) configured for authentication, authorization, standard flow and direct access grants.

![image.png](/.attachments/image-7c6dfc39-c089-4a84-bedf-1c54bb890cbf.png)

- the client should configure *{mlflow_url}/callback* as a valid redirect URI and *{mlflow_url}/oidc/ui/auth* as a valid post logout redirect URI
- the dedicated client scope of the client (which can be found under the Client Scopes tab of the client) should be configure to have a Groups mapper, to make the Groups of a User be available to mlflow during the login process. To add it, click on *Configure a new mapper* (or *Add Mapper* and *By Configuration*) and select the *Group Membership* mapper. Leave *groups* as token claim.
![image.png](/.attachments/image-421714d7-4a8c-44ba-98bb-54e487f57755.png)
- Create two groups that will be used by the mlflow OIDP plugin to verify if a User is authorized to access mlflow (either as a standard user or as an admin). Please note that the names of the groups should be then configured accordingly as env vars in the mlflow deployment (with an additional '/' before the name).

![image.png](/.attachments/image-9290f649-7290-46da-ab4a-ba7e9baded0a.png)

- If an Identity provider is also added to support a login via Entra ID (e.g. via HSR Entra ID), it is advised to add a Mapper that allows Users belonging to a specific Azure Group to be automatically added to the mlflow Users group defined in the step before. Add a Mapper with *Force* as sync mode override, *Advanced Claim to Group* as mapper type and use *groups* as the key of the claim and the Group Object ID as the value. Select the group defined in the previous step as target group of the mapping.

![image.png](/.attachments/image-88e39e34-cf53-4801-87fe-f18a6d6c684b.png)

If everything was configured correctly, Users should be able to log in with Entra, be automatically mapped to one of the Keycloak Groups and be allowed to access the mlflow UI.


## References

- MLflow guide on self hosting: https://mlflow.org/docs/latest/self-hosting/
- MLflow guide on security setup https://mlflow.org/docs/latest/self-hosting/security/network/
- MLflow plugin to support OIDC: https://github.com/mlflow-oidc/mlflow-oidc-auth
- Docker compose sample using MLflow, PostgreSQL and MinIO: docker-compose sample @ https://github.com/mlflow/mlflow/blob/master/docker-compose/README.m -