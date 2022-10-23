### gcloud cli - Gmail Account Login

**Note:** Certain steps work only for locally installed gcloud cli. Remember to restart local terminal to enable auto-completion after install.

| Step                  | Command                                 | Output                                          | Notes                                                        |
| --------------------- | --------------------------------------- | ----------------------------------------------- | ------------------------------------------------------------ |
| 1. Check if logged in | ```gcloud auth list```                  | ```No credentialed accounts.``` (for local cli) | This step is for local cli. There error output is expected as we have not tied local terminal with any account. Cloud shell in Google console website is already tied with current credentials, and wont show this error. |
| 2. Login              | ```gcloud auth login {gmail-address}``` | This opens a browser for authn                  | This will open browser for OAuth2 flow authentication. After authentication, use first command to see successfully logged in account (Credentialed Accounts) |
| 3. Do an operation    | ```gcloud compute instances list```     | ```Listed 0 items.```                           | This is a sample command to list compute instances (VM) on our project. For local CLI, it may be necessary to setup the project beforehand, under which resources have to be listed. ```gcloud config set project {your project id here}``` |
| 4. Logout             | ```gcloud auth revoke```                | ```Revoked credentials: {your gmail here}```    | This only works for local cli. For cloud shell, it gives this expected error: ```ERROR: (gcloud.auth.revoke) Cannot revoke GCE-provided credentials.``` |

### gcloud cli - Service Account Login (with openssl)

Generate cert:  (ref: [link](https://cloud.google.com/iam/docs/creating-managing-service-account-keys#uploading))

```bash
openssl req -x509 -nodes -newkey rsa:2048 -days 365 \
>     -keyout ~/studygroup_private_key.pem \
>     -out ~/studygroup_public_key.pem \
>     -subj "/CN=unused"
```

Convert cert to p12 (as gcloud login uses p12 format). Make sure set a password (gcloud auth doesnt work without it)

```
openssl pkcs12 -export -inkey studygroup_private_key.pem -in studygroup_public_key.pem -out studygroup_public_key.p12
```

| Step     | Command                                                      | Descr                                            |
| -------- | ------------------------------------------------------------ | ------------------------------------------------ |
| Login #1 | ```gcloud auth activate-service-account --key-file=studygroup_public_key.p12 --password-file=pass studygroup@myproject.iam.gserviceaccount.com``` | Login with password in a file (for automation). Create clear text file with password|
| Login #2 | ```gcloud auth activate-service-account --key-file=studygroup_public_key.p12 --prompt-for-password studygroup@myproject.iam.gserviceaccount.com``` | Login with password prompt for interactive login |
| ...      | Do rest of the operations as normal. Note revoke will generate warning, but will removed credentials. |                                                  |

Command ref: https://cloud.google.com/sdk/gcloud/reference/auth/activate-service-account

