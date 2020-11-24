# Beyond Kubernetes Certification - Challenges

To keep myself updated and involved with K8S, I will be exploring K8S beyond the certification topics and create challenges here on my findings. These challenges are `good-to-know` and might be `overkill`  for CKA/CKAD based certifications.

I just mentioned few tips and nothing else for `CKA,CKAD` certifications as the internet is flooded with many different blogs, repos, videos, training, exercises to prepare for all 3 Kubernetes certifications. 

If you are looking for `CKS` resources, scroll to the bottom of this page.

**Note** - Please feel free to make a pull request if there's something wrong, should be added, or updated.

## Sections
 1. [CKA and CKAD - Beyond Certification Challeneges](https://github.com/sidd-harth/kubernetes#cka-ckad-challenges)
 2. [CKA and CKAD Exam Tips](https://github.com/sidd-harth/kubernetes#cka-ckad-exam-tips)
    - [Using aliases](https://github.com/sidd-harth/kubernetes#using-aliases)
    - [VIM editor changes](https://github.com/sidd-harth/kubernetes#vim-editor-changes)
    - [Bookmarks](https://github.com/sidd-harth/kubernetes#bookmarks)
 3. [CKS Resources](https://github.com/sidd-harth/kubernetes#cks-resources)

## CKA CKAD Challenges
- Challenge 1 - [Encrypting Secret Data at Rest](https://github.com/sidd-harth/kubernetes/blob/main/challenges/Encrypting%20Secret%20Data%20at%20Rest.md)
- Challenge 2 - [Expanding PVC Storage Size](https://github.com/sidd-harth/kubernetes/blob/main/challenges/Expanding%20PVC%20Storage%20Size.md)
- Challenge 3 - [Startup Probe](https://github.com/sidd-harth/kubernetes/blob/main/challenges/Startup%20Probe.md)

## CKA CKAD Exam Tips
Kubectl `aliases`
```
alias k=kubectl
alias kn='k config set-context --current --namespace '
alias kd='k -o yaml --dry-run=client'
alias kall='k get all -o wide --show-labels'
alias kc='k config get-contexts'
```
#### Using `aliases`
In the exam, every question has a `context` given, we need to switch over to that context. Some questions are expected to work on specific `namespaces`. Sometimes we tend to forget adding `-n` argument to create resources in a specific namespace. 

These `aliases` will help in quickly changing the `namespace` and also checking the `current context`  before answering/debugging the questions.

- Example -
  - Create a Deployment name `nginx-frontend` 
  - Expose it using a Service named `nginx-svc` 
  - Write the output of all Service `Endpoints` to /opt/INC002/endpoints.txt
  - Everything needs to be done in `rs67` namespace.

`Without aliases`
```
k create deploy nginx-frontend --image nginx -n rs67
k expose deploy nginx-frontend --name nginx-svc --port 80 -n rs67
k get ep -n rs67 > /opt/INC002/endpoints.txt
```
`With aliases`
```
kn rs67         # changing context to use rs67 namespace
kc              # shows the current context and the namespace details

k create deploy nginx-frontend --image nginx
k expose deploy nginx-frontend --name nginx-svc --port 80
k get ep > /opt/INC002/endpoints.txt

kn default      # I feel it is a good practice to switch back to default namespace after every question
```

#### `VIM` Editor changes - 
These two additions were enough for me to edit/create `YAMLs` using VI
```
sudo vi /etc/vim/vimrc
set number
set paste
```

#### Bookmarks
During the exam, you can keep only one other browser tab open to refer to official documentation. I have uploaded the bookmarks which I have used for 1.19version. These bookmarks can be used for both CKA/CKAD.
| Name | Resource |
| ---- | ------ |
| Bookmark | [Kubernetes-Chrome-Bookmarks](https://github.com/sidd-harth/k8s/blob/main/Kubernetes-Chrome-Bookmarks.html) |

## CKS Resources
  - [Walid Shaari](https://github.com/walidshaari/Certified-Kubernetes-Security-Specialist)
  - [ibrahim Jelliti](https://github.com/ibrahimjelliti/CKSS-Certified-Kubernetes-Security-Specialist)
  - [Kim Wuestkamp](https://wuestkamp.medium.com/kubernetes-cks-full-course-simulator-3893120baa1d)
  - [Kubernetes CKS 2020 Complete Course + Simulator](https://www.udemy.com/course/certified-kubernetes-security-specialist/)
  
