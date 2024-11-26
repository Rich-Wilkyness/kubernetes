# 2 Types of accounts (service accounts and user accounts)
- Service accounts are run by robots, not people. They are used to run automated tasks, scripts, and other automated processes.
    - Prometheus is a service account. It is used to monitor the system.
    - Jenkins is a service account. It is used to run automated tests.
- User accounts are run by people. They are used to log in to the system and perform tasks.

## Commands
- to create a service account:
    - `kubectl create serviceaccount <service-account-name>`
- to get a list of service accounts:
    - `kubectl get serviceaccounts`
- tokens are used to authenticate service accounts. To get a token for a service account:
    - `kubectl get serviceaccount <service-account-name> -o jsonpath='{.secrets[0].name}'`
    - `kubectl get serviceaccount my-service-account -o jsonpath='{.secrets[0].name}'`
    - tokens are created automatically when a service account is created and stored in a secret.
        - this is no longer true in 1.24
        - to get the token, you need to create a secret and then extract the token from the secret. After creating a serviceaccount run: `kubectl create token <service-account-name>`
            - this token has a default expiration of 1 hour
    - to view the token:
        - `kubectl describe secret <secret-name>`
        - you can get the secret name by running:
            - `kubectl describe serviceaccount <service-account-name>`
- you can mount a service account token as a volume in a pod:
    - this is useful as it allows the pod to authenticate itself to the API server. 
    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
        name: mypod
    spec:
        serviceAccountName: my-service-account 
        containers:
        - name: mycontainer
            image: nginx
            volumeMounts:
            - name: token-volume
              mountPath: /var/run/secrets/kubernetes.io/serviceaccount
              readOnly: true
        volumes:
        - name: token-volume
          projected:
            sources:
            - serviceAccountToken:
                path: token
                expirationSeconds: 3600
    ```