# Security Primitives
- Authentication and Authorization are two of the most fundamental security primitives in Kubernetes. 
- Note: Storing user credentials in a file is not recommended. It is taught here for educational purposes only.
    - Use an external system like LDAP, Kerberos, etc.

- Who can access?
    - This is set up by Authentication mechanisms. 
        - Files - Usernames and Passwords
        - Files - Usernames and Tokens
        - Certificates
        - External Systems - LDAP, Kerberos, etc.
        - Service Accounts

- What can they do?
    - This is set up by Authorization mechanisms.
        - Role-Based Access Control (RBAC)
        - Attribute-Based Access Control (ABAC)
        - Webhook Mode
        - Node Authorization

- All communication between the Kubernetes components is secured using TLS certificates.

# Authentication
- End users are authenticated via the app itself and not Kubernetes.
    - so we wont talk about that here.

- Kubernetes DOES NOT manage user (Admin / Developer) accounts. 
    - It is assumed that the user accounts are managed by an external system like LDAP, Kerberos, etc.
- Kubernetes DOES manage Service Accounts.
    - `kubectl create serviceaccount my-service-account`

- We will focus on User Accounts
    - All access is managed through the kube-apiserver.
        - It authenticates the user before processing the request.
        - auth mechanisms 
            - static password files
                - csv file
                    ```csv
                    username, password, uid, groups(optional)
                    ```
                - pass the file as an option to the kube-apiserver (in the kube-apiserver.yaml file or as a flag)
                    ```bash
                    --basic-auth-file=/path/to/password.csv
                    ```
                    ```yaml
                    apiVersion: v1
                    kind: Pod
                    metadata:
                        name: kube-apiserver
                        namespace: kube-system
                    spec:
                        containers:
                        - name: kube-apiserver
                          image: k8s.gcr.io/kube-apiserver:v1.21.0
                          command:
                          - kube-apiserver
                          - --basic-auth-file=/path/to/password.csv
                    ```
                - must restart the kube-apiserver to apply changes

            - static token files
                - csv file
                    ```csv
                    token, username, uid, groups(optional)
                    ```
                - curl -v -k https://master-node-ip:6443/api/v1/pods --header "Authorization: Bearer <token>"
            - certificates
            - external systems like LDAP, Kerberos, etc.


# Kube Config
- The kubeconfig file is used to configure access to Kubernetes clusters.
- The file is located at ~/.kube/config by default. (if you use a different location, you need to set the KUBECONFIG environment variable to point to the file) 
    - `~/ = /home/<username>/`
- 3 sections:
    - clusters (Dev, Prod, Google Cloud, etc.)
        - cluster name
        - server
        - certificate-authority
        - `--server my-kube-playground:6443`
    - users (Admins, Developers, Prod Users, etc.)
        - user name
        - client-certificate
        - client-key
        - `--client-key admin-key`
        - `--client-certificate admin-cert`
        - `--certificate-authority ca-cert`
    - contexts (Admin@Dev, etc.)
        - context name
        - cluster
        - user
        - namespace
- contexts marry users and clusters together. (they point to a cluster and a user) 
- Example:
    ```yaml
    apiVersion: v1
    kind: Config
    current-context: my-context # defines which context to use
    clusters:
    - name: my-cluster
      cluster:
        server: https://my-kube-playground:6443
        certificate-authority: /path/to/ca.crt # usually looks like /etc/kubernetes/pki/ca.crt
        # can also use certificate-authority-data: pass in the base64 encoded (need to manually encode) cert directly 
    contexts:
    - name: my-context
      context:
        cluster: my-cluster # use the same name as the cluster name above
        user: my-kube-admin # use the same name as the user name below
        namespace: my-namespace # optional, defines priveleges to be for a specific namespace
    users:
    - name: my-kube-admin
      user:
        client-certificate: /path/to/admin.cert # usually looks like /etc/kubernetes/pki/admin.crt
        client-key: /path/to/admin.key # usually looks like /etc/kubernetes/pki/users/admin.key
    ```

- to view the current config: `kubectl config view`
    - to see the current context being used 
- to change the current context: `kubectl config --kubeconfig=/root/my-kube-config use-context <context-name>`
    - kubeconfig is optional if you are not using the default location for the kubeconfig file at ~/.kube/config 
- use `kubectl config --help` for more options

- remember to view hidden files: `ls -a`
    - the .kube directory is hidden (the file starts with a ".")

