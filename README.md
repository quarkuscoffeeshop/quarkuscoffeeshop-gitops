# Quarkus Cafe on ACM
![](../images/acm-quarkus-cafe-app.png)

## Provision RHPDS Enviornment 
* OCP4 ACM Hub
* OCP4 ACM Managed
* OCP4 ACM Managed

## Prerequisites for deployment 

## Install kustomize
[kustomize](https://kubernetes-sigs.github.io/kustomize/installation/)
```
$ curl -s "https://raw.githubusercontent.com/\
kubernetes-sigs/kustomize/master/hack/install_kustomize.sh"  | bash
$ sudo mv kustomize /usr/local/bin/
```

## Administrator Tasks On Target Cluster (OCP4 ACM Managed)


### Run ansible playbook to install AMQ and Postgres on target clusters.

#### Install Postgres Operator
[install-postgres-operator](https://github.com/quarkuscoffeeshop/quarkuscoffeeshop-helm/wiki#install-postgres-operator)

#### Install Amq Streams and Configure Postgres DB
```
$ ansible-galaxy collection install community.kubernetes
$ ansible-galaxy install tosin2013.quarkus_cafe_demo_role
$ export DOMAIN=ocp4.example.com
$ export OCP_TOKEN=123456789
$ export POSTGRES_PASSWORD=123456789
$ export STORE_ID=ATLANTA
$ cat >deploy-quarkus-cafe.yml<<YAML
- hosts: localhost
  become: yes
  vars:
    openshift_token: ${OCP_TOKEN}
    openshift_url: https://api.${DOMAIN}:6443
    insecure_skip_tls_verify: true
    default_owner: ${USER}
    default_group: ${USER}
    project_namespace: quarkuscoffeeshop-demo
    delete_deployment: false
    skip_amq_install: false
    skip_configure_postgres: false
    skip_mongodb_operator_install: true
    skip_quarkuscoffeeshop_helm_install: true
    domain: ${DOMAIN}
    postgres_password: "${POSTGRES_PASSWORD}"
    storeid: ${STORE_ID}
  roles:
    - tosin2013.quarkus_cafe_demo_role
YAML
$ ansible-playbook  deploy-quarkus-cafe.yml
```

## Install ACM Managed and Configure a HUB

Import an existing cluster
![](https://i.imgur.com/IFdi3Ez.png)

Run Genereated command on target machine 
![](https://i.imgur.com/6inP821.png)

View Cluster status
![](https://i.imgur.com/YwLk7w4.png)



## Configure OpenShift client context for cluster admin access 

**Login into hub cluster**
```
oc login -u admin -p XXXX --insecure-skip-tls-verify https://api.YOURCLUSTER1.DOMAIN:6443
```

**Set the name of the context for hub cluster**
```
oc config rename-context $(oc config current-context) hubcluster
```

**Login into 1st cluster (A environment)**
```
oc login -u admin -p XXXX --insecure-skip-tls-verify https://api.YOURCLUSTER1.DOMAIN:6443
```

**Set the name of the context for (A environment)**
```
oc config rename-context $(oc config current-context) cluster1
```

**Login into 2nd cluster (B environment) ::: OPTIONAL**
```
oc login -u admin -p XXXX --insecure-skip-tls-verify https://api.YOURCLUSTER2.DOMAIN:6443
```

**Set the name of the context (B environment)**
```
oc config rename-context $(oc config current-context) cluster2
```

**Login into 3rd cluster (C environment) ::: OPTIONAL**
```
oc login -u admin -p XXXX --insecure-skip-tls-verify https://api.YOURCLUSTER3.DOMAIN:6443
```

**Set the name of the context (C environment)**
```
oc config rename-context $(oc config current-context) cluster3
```

## Test the different cluster contexts
```
# Switch to hub
oc config use-context hubcluster

# Switch to cluster1
oc config use-context cluster1

# List the nodes in cluster1
oc get nodes

# Switch to cluster2
oc config use-context cluster2

# List the nodes in cluster2
oc get nodes

# Switch to cluster3 ::: Optional 
oc config use-context cluster3

# List the nodes in cluster3 ::: Optional 
oc get nodes

# Switch back to cluster1
oc config use-context cluster1
```

## Deploy Quarkus cafe Application on ACM Hub
**From ACM**  
Login to ACM Managed cluster (OCP4 ACM Hub)

**Fork quarkuscoffeeshop-gitops github repo**

**Git clone your forked repo to server**

**Optional: Create a personal access token for forked repo**  
[Creating a personal access token](https://docs.github.com/en/github/authenticating-to-github/creating-a-personal-access-token)

**Update acm_configs/02_channel.yaml replace with your repo**
```
pathname: 'https://github.com/quarkuscoffeeshop/quarkuscoffeeshop-gitops.git'
```

### Configure Services using kustomize
## Edit the following for each microservice
* imageTag is located under the kustomiztion.yaml in each directory 
* enviornment varaibles are located in each directory under patch-env.yaml

### Collect the quarkus cafe endpoints for cluster1 
```
$ echo "Cluster 1: $(oc --context=cluster1 get ingresses.config.openshift.io cluster -o jsonpath='{ .spec.domain }')"

$ ENDPOINT=$(oc --context=cluster1 get ingresses.config.openshift.io cluster -o jsonpath='{ .spec.domain }')

$ sed -i "s/apps.ocp4.example.com/${ENDPOINT}/g" clusters/overlays/cluster1/quarkuscoffeeshop-web/patch-env.yaml

$ cat clusters/overlays/cluster1/quarkuscoffeeshop-web/patch-env.yaml

$ sed -i "s/apps.ocp4.example.com/${ENDPOINT}/g" clusters/overlays/cluster1/quarkuscoffeeshop-customermocker/patch-env.yaml

$ cat clusters/overlays/cluster1/quarkuscoffeeshop-customermocker/patch-env.yaml
```

### Collect the quarkus cafe endpoints for cluster2 
```
$ echo "Cluster 2: $(oc --context=cluster2 get ingresses.config.openshift.io cluster -o jsonpath='{ .spec.domain }')"

$ ENDPOINT=$(oc --context=cluster2 get ingresses.config.openshift.io cluster -o jsonpath='{ .spec.domain }')

$ sed -i "s/apps.ocp4.example.com/${ENDPOINT}/g" clusters/overlays/cluster2/quarkuscoffeeshop-web/patch-env.yaml

$ cat clusters/overlays/cluster2/quarkuscoffeeshop-web/patch-env.yaml

$ sed -i "s/apps.ocp4.example.com/${ENDPOINT}/g" clusters/overlays/cluster2/quarkuscoffeeshop-customermocker/patch-env.yaml

$ cat clusters/overlays/cluster2/quarkuscoffeeshop-customermocker/patch-env.yaml
```

### Collect the quarkus cafe endpoints for cluster 3 
```
$ echo "Cluster 3: $(oc --context=cluster3 get ingresses.config.openshift.io cluster -o jsonpath='{ .spec.domain }')"

$ ENDPOINT=$(oc --context=cluster3 get ingresses.config.openshift.io cluster -o jsonpath='{ .spec.domain }')

$ sed -i "s/apps.ocp4.example.com/${ENDPOINT}/g" clusters/overlays/cluster3/quarkuscoffeeshop-web/patch-env.yaml

$ cat clusters/overlays/cluster3/quarkuscoffeeshop-web/patch-env.yaml

$ sed -i "s/apps.ocp4.example.com/${ENDPOINT}/g" clusters/overlays/cluster3/quarkuscoffeeshop-customermocker/patch-env.yaml

$ cat clusters/overlays/cluster3/quarkuscoffeeshop-customermocker/patch-env.yaml
```

### Update Postgres passwords
```
POSTGRES_CLUSTERONE='CHANGEPASSWORD'
POSTGRES_CLUSTERTWO='CHANGEPASSWORD'
POSTGRES_CLUSTERTHREE='CHANGEPASSWORD'

sed -i "s|'changepassword'|'${POSTGRES_CLUSTERONE}'|g" clusters/overlays/cluster1/quarkuscoffeeshop-counter/patch-env.yaml
cat clusters/overlays/cluster1/quarkuscoffeeshop-counter/patch-env.yaml

sed -i "s|'changepassword'|'${POSTGRES_CLUSTERTWO}'|g" clusters/overlays/cluster2/quarkuscoffeeshop-counter/patch-env.yaml
cat clusters/overlays/cluster2/quarkuscoffeeshop-counter/patch-env.yaml

sed -i "s|'changepassword'|'${POSTGRES_CLUSTERTHREE}'|g" clusters/overlays/cluster3/quarkuscoffeeshop-counter/patch-env.yaml
cat clusters/overlays/cluster3/quarkuscoffeeshop-counter/patch-env.yaml
```

### Update routes for Quarkus Cafe Application
```
cp  clusters/overlays/cluster1/quarkuscoffeeshop-web/route.yaml.backup clusters/overlays/cluster1/quarkuscoffeeshop-web/route.yaml

cp  clusters/overlays/cluster2/quarkuscoffeeshop-web/route.yaml.backup  clusters/overlays/cluster2/quarkuscoffeeshop-web/route.yaml

cp  clusters/overlays/cluster3/quarkuscoffeeshop-web/route.yaml.backup  clusters/overlays/cluster3/quarkuscoffeeshop-web/route.yaml

# Define the variable of `ROUTE_CLUSTER1`
ROUTE_CLUSTER1=quarkuscoffeeshop-web-quarkuscoffeeshop-demo.$(oc --context=cluster1 get ingresses.config.openshift.io cluster -o jsonpath='{ .spec.domain }')

# Replace the value of changeme with `ROUTE_CLUSTER1` in the file `route.yaml`
sed -i "s/changeme/${ROUTE_CLUSTER1}/" clusters/overlays/cluster1/quarkuscoffeeshop-web/route.yaml

# Verify Change
cat clusters/overlays/cluster1/quarkuscoffeeshop-web/route.yaml

# Define the variable of `ROUTE_CLUSTER2`
ROUTE_CLUSTER2=quarkuscoffeeshop-web-quarkuscoffeeshop-demo.$(oc --context=cluster2 get ingresses.config.openshift.io cluster -o jsonpath='{ .spec.domain }')

# Replace the value of changeme with `ROUTE_CLUSTER2` in the file `route.yaml`
sed -i "s/changeme/${ROUTE_CLUSTER2}/" clusters/overlays/cluster2/quarkuscoffeeshop-web/route.yaml

# Verify Change
cat clusters/overlays/cluster2/quarkuscoffeeshop-web/route.yaml

# Define the variable of `ROUTE_CLUSTER3`  ::: OPTIONAL
ROUTE_CLUSTER3=quarkuscoffeeshop-web-quarkuscoffeeshop-demo.$(oc --context=cluster3 get ingresses.config.openshift.io cluster -o jsonpath='{ .spec.domain }')

# Replace the value of changeme with `ROUTE_CLUSTER3` in the file `route.yaml`  ::: OPTIONAL
sed -i "s/changeme/${ROUTE_CLUSTER3}/" clusters/overlays/cluster3/quarkuscoffeeshop-web/route.yaml

# Verify Change
cat clusters/overlays/cluster3/quarkuscoffeeshop-web/route.yaml
```


## Review cluster configuration
**Cluster1**
```
kustomize build clusters/overlays/cluster1 | less
```

**Cluster2** 
```
kustomize build clusters/overlays/cluster2 | less
```

**Cluster3**
```
kustomize build clusters/overlays/cluster3 | less
```

**Commit and push changes to Github**
```
git add clusters/

git commit -m "Updating Variables for Deployment"

git push 
```

**Use context hubcluster**
```
oc config use-context hubcluster
```

## Deploy using tekton pipelines
[Quarkus Cafe Deployment  on ACM using tekton pipelines](tekton-demo-deployment.md)

## Deploy via CLI
**Create namespace for subscription**
```
oc create -f acm-configs/01_namespace.yaml
```

**Create a Channel**
```
oc create -f acm-configs/02_channel.yaml
```

**Create application**
```
oc create -f acm-configs/03_application_webapp.yaml
```

**Confirm Clusters are properly labeled**
*  `clusterid=cluster1`
*  `clusterid=cluster2`
*  `clusterid=cluster3`
*  `clusterid=...`

**Create placement rules**
```
oc create -f acm-configs/04_placement_cluster1.yaml
oc create -f acm-configs/04_placement_cluster2.yaml
oc create -f acm-configs/04_placement_cluster3.yaml
```

**Create subscription**
```
oc create -f acm-configs/05_subscription_cluster1.yaml
oc create -f acm-configs/05_subscription_cluster2.yaml
oc create -f acm-configs/05_subscription_cluster3.yaml
```

**Verify the deployments have been created on all the clusters.**
```
# cluster 1 
oc config use-context cluster1
oc get pods -n quarkuscoffeeshop-demo

# cluster 2
oc config use-context cluster1
oc get pods -n quarkuscoffeeshop-demo

# cluster 3
oc config use-context cluster1
oc get pods -n quarkuscoffeeshop-demo
```


**Access the cluster URls**
```
# Test against cluster 1
ROUTE_CLUSTER1=quarkuscoffeeshop-web-quarkuscoffeeshop-demo.$(oc --context=cluster1 get ingresses.config.openshift.io cluster -o jsonpath='{ .spec.domain }')
echo http://$ROUTE_CLUSTER1/cafe

# Test against cluster 2
ROUTE_CLUSTER2=quarkuscoffeeshop-web-quarkuscoffeeshop-demo.$(oc --context=cluster2 get ingresses.config.openshift.io cluster -o jsonpath='{ .spec.domain }')
echo http://$ROUTE_CLUSTER2/cafe

# Test against cluster 3 ::: OPTIONAL
ROUTE_CLUSTER3=quarkuscoffeeshop-web-quarkuscoffeeshop-demo.$(oc --context=cluster3 get ingresses.config.openshift.io cluster -o jsonpath='{ .spec.domain }')
echo http://$ROUTE_CLUSTER3/cafe
```

### Todo 
* Add HAproxy Global load balancer support
* Ansible Tower ACM Integration
* Create Bootstrap Script

##  To update Policies on ACM use after the quarkus cafe has been deployed.

[Quarkus Cafe Policies on ACM](acm-policys.md)