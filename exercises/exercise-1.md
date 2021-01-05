[TOC]

# Exercise-1



## Prepare local development

https://rht-labs.github.io/enablement-docs/#/1-the-manual-menace/README



Clone project , and open it on VS Code:

- https://raw.githubusercontent.com/rht-labs/enablement-framework/main/do500-devfile.yml

- https://github.com/haithamshahin333/enablement-ci-cd



VS Code shortcut:

* Find in files: Ctrl + Shift + F
* Find by file name: Ctrl + P



## Ansible python bug

参见：

- https://github.com/ansible/ansible/issues/32554#issuecomment-572985680

```bash
env no_proxy="*" ansible-playbook apply.yml -e target=tools \  -i inventory/ \
  -e "filter_tags=nexus"

env no_proxy="*" ansible-playbook apply.yml -e target=tools \
-i inventory/ \
-e "filter_tags=mongodb"


env no_proxy="*" ansible-playbook apply.yml -e target=tools \
  -i inventory/ \
  -e "filter_tags=jenkins"

```

Add `no_proxy` env var:

```bash
export no_proxy="*"
```

```bash
source ~/.zshrc
```



## Password issue

Use double quote to include your password incase of some special characters.

```bash
SOURCE_REPOSITORY_PASSWORD="your_password"
```

## Burn and Recreate

Login OpenShift.



Delete projects:

```bash
oc delete project williamnew-ci-cd williamnew-dev williamnew-test
```



Receate everything:

```bash
ansible-playbook apply.yml -i inventory/ -e target=bootstrap
ansible-playbook apply.yml -i inventory/ -e target=tools
```



## Change Nexus password



Default password:

```bash
admin / admin123
```



New password:

```bash
admin / redhat
```



## Add a SonarQube persistent deployment to the `ci-cd-deployments` section



Download sonarqube-deploy.yml to templates folder.

https://raw.githubusercontent.com/rht-labs/openshift-templates/v1.4.17/sonarqube/sonarqube-deploy.yml



Fix a RoleBinding error:

```yaml
- apiVersion: rbac.authorization.k8s.io/v1
  kind: RoleBinding
  metadata:
    name: "${NAME}_view"
  roleRef:
    name: view
    apiGroup: rbac.authorization.k8s.io
    kind: ClusterRole #Role
  subjects:
  - kind: ServiceAccount
    name: "${NAME}"
```



Change kind from `Role`  to `ClusterRole`。





Error: secret "sonardb" not found

Need create a postgressql firstly.



`params/sonarqube`:

```bash
POSTGRESQL_USER=sonaruser
POSTGRESQL_PASSWORD=sonarpassword
POSTGRESQL_DATABASE=sonar
```



Ensure database name is `sonar`, default dabase service name is `postgresql`。



Modify `sonarqube-deploy.yml` , use `postgresql` as database service name, and use `sonar` as database name.

```bash
jdbc:postgresql://postgresql:5432/sonar
```





Install postgresql:

```bash
env no_proxy="*" ansible-playbook apply.yml -e target=tools \
-i inventory/ \
-e "filter_tags=postgresql"

```



Install sonarqube:

```bash
env no_proxy="*" ansible-playbook apply.yml -e target=tools \
  -i inventory/ \
  -e "filter_tags=sonarqube"
  
```





Reference:

- https://github.com/redhat-cop/openshift-templates/blob/master/sonarqube/sonarqube-deploy.yml
- https://jdbc.postgresql.org/documentation/head/connect.html



## Jenkins integrate with Slack

https://plugins.jenkins.io/slack/

https://www.jenkins.io/doc/pipeline/steps/slack/



Choose a slack channel, and add Jenkins CI integration.



https://do500.slack.com/services/B01HK36NQ5D?added=1





After it's installed, click on **Manage Jenkins** again in the left navigation, and then go to **Configure System**. Find the **Global Slack Notifier Settings** section and add the following values:

- **Team Subdomain:** `do500`
- **Integration Token Credential ID:** Create a secret text credential using `Fbuh5juEMJX3fyRRhm9qNmF0` as the value



```bash
Fbuh5juEMJX3fyRRhm9qNmF0
```





https://do500.slack.com/services/1597108772183?updated=1



Redploy Jenkins



```bash
env no_proxy="*" ansible-playbook apply.yml -e target=tools \
  -i inventory/ \
  -e "filter_tags=jenkins"
```



Create Slack credential:

```bash
Credential / Secret text / Slack token
```





Configure Slack globally:

Configure System / Slack

```bash

Username: jenkins

# strange, use "do500" as team workspace not work.
Override url: https://do500.slack.com/services/hooks/jenkins-ci/
```



Configure per job:

```bash
Username: jenkins

# strange, use "do500" as team workspace not work.
Override url: https://do500.slack.com/services/hooks/jenkins-ci/

# leave it to empty ? to use default channel ?
Channel: #hongkong-jan4-8-2021
```



Test connection after configure them.



References:

- https://www.cidevops.com/2018/05/jenkins-slack-integration-jenkins-push.html