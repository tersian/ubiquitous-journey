## pb-ci-cd

An end to end tutorial for using UJ in your own project. This example uses the Java Quarkus pet battle applications as examples.

### Bootstrap your CI/CD environment
Create a directory to hold your CICD tooling automation.
```bash
mkdir pb-ci-cd
```

Document all the things here:
```bash
cd pb-ci-cd && touch README.md
```

Clone UJ into your project:
```bash
git clone https://github.com/rht-labs/ubiquitous-journey
rm -rf ubiquitous-journey/.git/
rm -rf ubiquitous-journey/.github
```

Initialize git repository in the top level folder.
```bash
git init
git add README.md
git commit -m  "🐪 initial commit 🐪"
git branch -M main
git remote add origin git@github.com:eformat/pb-ci-cd.git
git push -u origin main
```

#### `ubiquitous-journey/ubiquitous-journey/values-tooling.yaml`

Edit this file and choose tools we want by enabling them as `true` (the rest we set to false). In this example we are using nexus, sonarqube and tekton:
```bash

enabled: true

- nexus
- sonarqube
- tekton
```

Set `sonarqube plugins values`
```yaml
    values:
      initContainers: true
      plugins:
        install:
          - https://github.com/checkstyle/sonar-checkstyle/releases/download/8.35/checkstyle-sonar-plugin-8.35.jar
          - https://repo1.maven.org/maven2/org/sonarsource/java/sonar-java-plugin/6.3.2.22818/sonar-java-plugin-6.3.2.22818.jar
          - https://repo1.maven.org/maven2/org/sonarsource/jacoco/sonar-jacoco-plugin/1.1.0.898/sonar-jacoco-plugin-1.1.0.898.jar
```

#### `ubiquitous-journey/ubiquitous-journey/values-extratooling.yaml`

Edit this file and set all of these to false (we dont need them for now).

#### `ubiquitous-journey/argo-app-of-apps.yaml`

Replace source with new git repo, github now uses main instead of master and fix path:
```bash
sed -i -e 's|rht-labs/ubiquitous-journey|eformat/pb-ci-cd|' ubiquitous-journey/argo-app-of-apps.yaml
sed -i -e 's|source_ref: master|source_ref: main|' ubiquitous-journey/argo-app-of-apps.yaml
sed -i -e 's|source_path: ubiquitous-journey|source_path: ubiquitous-journey/ubiquitous-journey|' ubiquitous-journey/argo-app-of-apps.yaml
sed -i -e 's|sync_policy_automated: false|sync_policy_automated: true|' ubiquitous-journey/argo-app-of-apps.yaml
```

Check all of it into git:
```bash
git add .
git commit -m 'initial commit'
git push
```

#### Bootstrap

Login to openshift as a user with `cluster-admin`
```bash
oc login
cd pb-ci-cd/ubiquitous-journey
```

Boostrap ArgoCD (run this command again if you see a `no matches for kind "ArgoCD" in version "argoproj.io/v1alpha1"` warning - this is fine, the CR was not found yet)
```bash
helm template bootstrap --dependency-update -f bootstrap/values-bootstrap.yaml bootstrap | oc apply -f-

namespace/labs-ci-cd unchanged
namespace/labs-pm unchanged
namespace/labs-cluster-ops unchanged
namespace/labs-dev unchanged
namespace/labs-test unchanged
namespace/labs-staging unchanged
serviceaccount/jenkins unchanged
secret/argocd-privaterepo configured
clusterrole.rbac.authorization.k8s.io/labs-ci-cd-argocd-application-controller unchanged
clusterrolebinding.rbac.authorization.k8s.io/labs-ci-cd-argocd-application-controller unchanged
argocd.argoproj.io/argocd created
operatorgroup.operators.coreos.com/labs-ci-cd unchanged
subscription.operators.coreos.com/prometheus-operator unchanged
subscription.operators.coreos.com/argocd-operator unchanged
rolebinding.rbac.authorization.k8s.io/labs-devs_edit_role unchanged
rolebinding.rbac.authorization.k8s.io/labs-admins_admin_role unchanged
rolebinding.rbac.authorization.k8s.io/jenkins_admin_role unchanged
rolebinding.rbac.authorization.k8s.io/labs-devs_edit_role unchanged
rolebinding.rbac.authorization.k8s.io/labs-admins_admin_role unchanged
rolebinding.rbac.authorization.k8s.io/jenkins_admin_role unchanged
rolebinding.rbac.authorization.k8s.io/labs-devs_edit_role unchanged
rolebinding.rbac.authorization.k8s.io/labs-admins_admin_role unchanged
rolebinding.rbac.authorization.k8s.io/jenkins_admin_role unchanged
rolebinding.rbac.authorization.k8s.io/labs-devs_edit_role unchanged
rolebinding.rbac.authorization.k8s.io/labs-admins_admin_role unchanged
rolebinding.rbac.authorization.k8s.io/jenkins_admin_role unchanged
rolebinding.rbac.authorization.k8s.io/labs-devs_edit_role unchanged
rolebinding.rbac.authorization.k8s.io/labs-admins_admin_role unchanged
rolebinding.rbac.authorization.k8s.io/jenkins_admin_role unchanged
rolebinding.rbac.authorization.k8s.io/labs-devs_edit_role unchanged
rolebinding.rbac.authorization.k8s.io/labs-admins_admin_role unchanged
rolebinding.rbac.authorization.k8s.io/jenkins_admin_role unchanged
```

You should see your pods and deployments spinning up:

![labs-ci-cd-boot.png](images/labs-ci-cd-boot.png)

Give me ALL THE TOOLS, EXTRAS & OPSY THINGS !
```bash
helm template -f argo-app-of-apps.yaml ubiquitous-journey/ | oc -n labs-ci-cd apply -f-

application.argoproj.io/ubiquitous-journey created
application.argoproj.io/uj-extras created
application.argoproj.io/uj-day2ops created
```

### Check out deployments in UI

Find the argocd Route in the labs-ci-cd namespace `argocd-server`
![argo-route.png](images/argo-route.png)

Browse to it and login via OpenShift
![argo-login.png](images/argo-login.png)

All the apps should be syncing and deploying (we set auto sync true):
![argo-uj-all.png](images/argo-uj-all.png)

If you select one of the `app-of-app` applications e.g. `ubiquitous-journey` we can see out CICD tooling deployed. You can further drill-down from here as well.
![argo-login.png](images/argo-app-of-apps-uj.png)