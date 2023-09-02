# GitOps Project

#### In this project, we'll combine Jenkins, Git, GitHub, Docker, Kubernetes, and ArgoCD to deploy our Python web application onto a Kubernetes cluster.

## PipeLine Flow

`1. Developer will push the code to github `

`2. Jenkins will clone the repository to jenkins environment ,`

`3. jenkins will Build docker image using Dockerfile that was attached with code,`

`4. Jenkins will push that image to container registory Dockerhub,`

`5. Jenkins will retag the image with new image tag on github deployment file`

`6. Argocd will deploy deployments/pods and service from that manifests files to the kubernetes cluster `

![alt text](https://github.com/Meet-S0ni/kubernetescode/blob/main/flow.png)

## Steps 

**Step: 1** 
- create a kubernetes cluster and virtual machine ( this will be our environment )
- connect cluster with vm ( you need to be able to access cluster from vm )

**Step: 2** 
- install docker, openjdk-11 and jenkins in virtual machine as per your operating system 

**Step: 3** 
- start jenkins server and install this  plugins to jenkins

    -- SSH  \
    -- docker \
    -- groovy   \
    -- github, git

**Step: 4** ( global credentials )

- add credential of dockerhub `id: dockerhub` \
    `username= <username-of-dockerhub>` \
    `password= <password-of-dockerhub>`

- generate personal access token with scope `repo, admin:repo_hook, notification`

- add credential of github `id: github` \
    `username= <username-of-github>` \
    `password= <personal-access-token>`


( `personal access token` should have sufficiant permissins to push code on your github repo, admin:repo_hook, notification, )

**Step: 5** 
- create a pipeline named "buildimage" in jenkins
    - at Build Triggers section select \
        `GitHub hook trigger for GITScm polling`
    - at pipeline section select: `Pipeline script from SCM ( Git )` \
      `repository : your repository with Dockerfile, code, and Jenkinsfile` \
        ( provide credential if your repo is privet ) 

**Step: 6**
- create pipeline named "updatemanifest" in jenkins 
    - select `This project is parameterized` \
      `name: DOCKERTAG`  \
      `default value: latest`
    - at pipeline section select: `Pipeline script from SCM ( Git )` \
      `repository : your repository with Dockerfile, code, and Jenkinsfile` \
        ( provide credential if your repo is privet ) 

( note: Your jenkins vm should have Public IP  because we are using webhooks)

**Step: 7**
- at your github kubernetescode repository \
  `settings > Webhooks > Add webhook`  \
  `url : <your-jenkins-url/github-webhook/` \
  `example url : http://20.12.17.101:8080/github-webhook/`  
    ( makeshure your github repo should be upto date )

- now try to build first "buildimage" pipeline  \
    `( it should automatically run updatemanifest pipeline )`
   
**Step: 8**
- Now we are going to install argocd in our kubernetes cluster using this commands

```bash
  kubectl create namespace argocd
  kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```
- change argocd service to LoadBalancer or NodePort for accessing argocd  `service name : service/argocd-server`

```bash
kubectl edit service/argocd-server -n argocd
```
- default user for argocd will be `admin`
- to get password for login to argocd use command `linux`
```bash
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d; echo
```

( you can also use argocd cli )

**Step: 9**
- create app in argocd  
  provide this details
    - application name: <argocdapp1>
    - Project: `default`
    - sync policy: as per your requirement `Automatic`
    - Source:   
      Repository URL: <https://github.com/Meet-S0ni/kubernetesmanifest.git>  
      Path: <provide path in your repo> `( ./ )`
  - click `create`
  - first argocd will deploy application as per github manifests 
      
**Step: 10**
- install git-github to your vm
- authanticate with your repo using pat
- clone kubernetescode repo 
- make changes in your code and push it to your repository
- check that it is deploying ( updating ) application or not

### Feedback and suggestions are highly appreciated

Feel free to reach out with any comments or improvements you might have. Thank you for taking the time to explore our GitOps project!"  
https://github.com/Meet-S0ni#connect-with-me

## 

Here are some related Repos to get started 

- [Argocd doccumentation](https://argo-cd.readthedocs.io/en/stable/getting_started) 
- [Kubernetescode](https://github.com/Meet-S0ni/kubernetescode)
- [Kubernetesmanifest](https://github.com/Meet-S0ni/kubernetesmanifest)
 
