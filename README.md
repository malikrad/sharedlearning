# Cloudformation Setup

See aws-linux.yaml under cloudformation folder. 

## Prerequisites
- Domain &  ACM SSL certificate
- CLI Access with MFA
- Set cname record in route 53 for DNS name of the loadbalancer
- Install IDE with cloudformation linter for development

## Deployment
Example:
aws cloudformation create-stack --stack-name gitlabloadbalancer --template-body "file://C:/Users/martin.raabe/Desktop/gitlab shared learning/cicd-sharedlearning-aws/cloudformation/gitlabloadbalancer.yaml" --profile dev-mfa


## Usage

Open: https://gitlab.myopenshiftdemo.com/
Create user and inform admin to get the approval ( Admin tbd)


## next steps

- SSO?
- Compliance Tools?
- Setup from clean AMI?
- Review / finalize the Cloudformation script
    - Delete stack and create from scratch
    - HTTp -> HTTPS  redirect with cloudformation
    - Do we need an SLL Policy for compliance?
    - add Tags
    - Add 302 code as healty for load balancer target group

# CI/CD demo server setup
The servers are setup using ansible (2.4) and docker containers downloaded from the internet. All requested components for the build pipeline are provided in docker containers. These components are to date:
* gitlab
* gitlab-runner

The components are provided as roles in the recommended ansible directory structure (following best practices https://docs.ansible.com/ansible/latest/user_guide/playbooks_best_practices.html )

General software required for software operation is managed by the common role `roles/common/tasks/main.yml`. The list of necessary packages for operation is maintained in the variables file `roles/common/vars/main.yml`. The `hosts` file defines which component is deployed to which server. At this point in time it is the localhost only.

# Operation of the installed server software
## Start all server components

`ansible-playbook -i hosts site.yml`

## Start a  role (for example Gitlab)

`ansible-playbook -i hosts gitlab.yml`

# Roles
All roles are sharing the same directory structure. For example:
```
├── roles
│   ├── artifactory
│   │   ├── tasks
│   │   │   ├── filesystem.yml
│   │   │   ├── groups.yml
│   │   │   ├── install.yml
│   │   │   ├── main.yml
│   │   │   └── users.yml
```
To maintain the execution order, tasks are orderd in the `main.yml` (for example: you can't add an user account when the configured primary group does not exist). When there is a communication required between roles the containers are linked using docker links. For example Sonarqube `roles/sonarqube/tasks/install.yml`
```
- name: Install and manage sonarqube
  docker_container:
    name: sonarqube
    image: sonarqube:lts-alpine
    state: started
    pull: true
    env_file: /root/sonarqube.env
    links:
     - postgres
    volumes:
    - "/srv/sonarqube/conf:/opt/sonarqube/conf"
    - "/srv/sonarqube/data:/opt/sonarqube/data"
    - "/srv/sonarqube/logs:/opt/sonarqube/logs"
    - "/srv/sonarqube/extensions:/opt/sonarqube/extensions"
    ports:
    - "9000:9000"
```

