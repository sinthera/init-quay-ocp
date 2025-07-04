# Ansible playbooks to create and configure new ocp projects and quay organizations

######
######
######
######
###### Date last revision: 18/06/2025

    These playbooks is used to create and configure new openshift projects. 
    It initially creates quay organization in which it creates the necessaried robot account used in CI/CD pipeline (bamboo) and security checks (ACS). 
    Meanwhile it handles the creation and configuration of the associated openshift namespace, label it with argocd label if necessary and create the specified resource quota. 
    Finally, it creates the openshift secret containing robot account token to pull the images from openshift.

    :)


### Create the token to allow the playbook to access to the quay registry and launch aap resources creation
1. Log in to the Quay Container Registry web UI.
2. Use an existing organization or create a new one.
3. In the organization, create an application.
4. In the application, select the Generate Token menu.
5. Select the permissions to associate to the token. To be able to use all the modules in the collection, select Administer Organization, Administer Repositories, Create Repositories, Super User Access, and Administer User.
6. Click "Generate Token" and copy it.
7. Create a project with awx/configure-aap.yaml template with a survey with "registry_url" and "registry_token" var name to ask, example:
    registry_url: https://quay-registry.apps.cluster.it
    registry_token: abc123def456lkj987poi765hgf
8. Launch the template.

### Ansible vault
1. Create encrypted file with ansible-vault in controller (password protected)
```bash
[user@host]# ansible-vault create vault-secrets.yaml
``` 
```YAML
ocp_password: secretpassword
ssh-key: |
  --------BEIGIN PRIVATE KEY....ecc
  ..........END PRIVATE KEY--------
```
2. Put the generated file (vault-secrets.yaml) inside the playbook directory and ensure it is included in loop.yaml.
3. Create AAP credential containing the password to decrypt vault file (vault_password).
4. Associate the credential to the AAP template "create-openshift-resources".

### Playbook run
Input variables:
- project: the project to create
- ocp_username: your ocp username
- gitops: yes/no if gitops scoped
- a list of administrators users for svil cluster
- for each cluster:
- requestsCpu: for resourceQuota
- requestsMem: for resourceQuota
- limitsCpu: for resourceQuota
- limitsMem: for resourceQuota