The following page described the infrastructural setup of the mlflow deployment 

## OIDP Plugin Configuration
While the official plugin documentation is not particularly extensive (https://github.com/mlflow-oidc/mlflow-oidc-auth/blob/main/docs/configuration.md) , local tests confirmed the purpose of each of the environment variables configured for the OIDP plugin deployment in MLflow. Here's a list with a small explaination:

- OIDC_DISCOVERY_URL -> *{keycloak_url}/realms/m{mlflow_realm}/.well-known/openid-configuration*
- OIDC_CLIENT_ID -> client for mlflow (e.g. *mlflow_client*)
- OIDC_CLIENT_SECRET -> secret of the client above
- OIDC_REDIRECT_URI -> *{url_of_mlflow_server}/callback
- OIDC_USERS_DB_URI -> connection string used by mlflow to connect to PostgreSQL (e.g. *postgresql://.../mlflow_auth), assuming the existence of a DB called *mlflow_auth*
- OIDC_SCOPE -> *openid email profile* (Keycloak does not support comma separated scopes)
- OIDC_GROUP_NAME -> name of the group of standard MLflow users configured in Keycloak, with a '/' before the name (needed for Keycloak specifically): the OIDP plugin will verify if the logged in User is part of this group to allow access (or the admin group). Example: */mlflow_users*
- OIDC_ADMIN_GROUP_NAME -> name of the group of admin MLflow users configured in , with a '/' before the name (needed for Keycloak specifically). Same mechanism as above. Example: */mlflow_admins*. 
- OIDC_PROVIDER_DISPLAY_NAME -> string appearing in the login button of the plugin (e.g. *Login with KeyCloak*)
- SECRET_KEY -> a secret key used by the Flask deployment of the plugin (needed to avoid weird authorization error after Users' login)

Not connected to the plugin, but other env variables needed to the mlflow deployment are:
- MLFLOW_BACKEND_STORE_URI=postgresql://mlflow:mlflow@postgres:5432/mlflow_tracking

  

POSTGRES_USER=mlflow

POSTGRES_PASSWORD=mlflow

  

LOG_LEVEL=WARNING


## References

- MLflow guide on self hosting: https://mlflow.org/docs/latest/self-hosting/
- MLflow guide on security setup https://mlflow.org/docs/latest/self-hosting/security/network/
- MLflow plugin to support OIDC: https://github.com/mlflow-oidc/mlflow-oidc-auth
- Docker compose sample using MLflow, PostgreSQL and MinIO: docker-compose sample @ https://github.com/mlflow/mlflow/blob/master/docker-compose/README.m -