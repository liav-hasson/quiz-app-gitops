# GitOps Repository - Quiz-app

This repository contains GitOps methodology implementation using ArgoCD, and an App-of-apps pattern.
Below i will try to explain the logic of each app, the dependencies ordering using sync waves, and the troubleshooting i did when implementing.

---

## About The Quiz-app Project

The Quiz-app is a DevOps learning platform build by a DevOps student.
The app lets the user select a category, a sub-category and a difficulty, then generates a question about a random keyword in that subject. The user then answers the question, recieves a score, and short feedback.

All the code is fully open source, and contains 5 main repositories:
- **[Frontend repository](https://github.com/liav-hasson/quiz-app-frontend.git)** - React frontend that runs on Nginx.
- **[Backend repository](https://github.com/liav-hasson/quiz-app-backend.git)** - Flask Python backend logic.
- **[GitOps repository](https://github.com/liav-hasson/quiz-app-gitops.git) << You are here!** - ArgoCD App-of-app pattern.
- **[IaC repository](https://github.com/liav-hasson/quiz-app-iac.git)** - Terraform creates oll the base infrastructure, on AWS.
- **[Mini-version repository](https://github.com/liav-hasson/quiz-app-mini.git)** - Allows you to self-host localy, or on AWS.

## Sync Waves Explenations

Sync waves provide structured control over deployment steps, using designated hooks to define specific actions at each phase of the sync process. 
This helps in handling dependencies, such as creating a ConfigMap before a Deployment that relies on it, or ensuring that critical infrastructure is ready before application components are rolled out. ([Read more](https://codefresh.io/learn/argo-cd/understanding-argo-cd-sync-waves-with-examples/))

### Sync wave -50, Prerequisits

The first thing that ArgoCD deploys to the cluster are the dependencies.
These contain TargetGroupBinding for ArgoCD and Grafana, and volumes preperations for Mongodb and Grafana.

Why Target group binding?
What volume preperation?
