```console
git clone https://github.com/ntap-johanneswagner/tridentdemo
cd /root/tridentdemo
./prework.sh
```

This will take some minutes...
```console
helm repo add netapp-trident https://netapp.github.io/trident-helm-chart
```

```console
helm install trident netapp-trident/trident-operator --version 100.2510.0 --create-namespace --namespace trident --set tridentAutosupportImage=registry.demo.netapp.com/trident-autosupport:25.10.0,operatorImage=registry.demo.netapp.com/trident-operator:25.10.0,tridentImage=registry.demo.netapp.com/trident:25.10.0,tridentSilenceAutosupport=true,windows=true,imagePullSecrets[0]=regcred
```

```console
kubectl get pods -n <trident-namespace>
```

## :trident: Scenario 02 - Configure Trident
```console
cd /root/tridentdemo/scenario02
```
### Backends

backend-ontap-nas.yaml is an example for using the ontap-nas driver.  
backend-ontap-nas-eco.yaml is an example for using the ontap-nas-economy driver.  
backend-ontap-san.yaml is an example for using the ontap-san driver using iSCSI.  
backend-ontap-san-eco.yaml is an example for using the ontap-san-economy driver using iSCSI.  

```console
kubectl apply -f secret-svm.yaml
kubectl apply -f backend-ontap-nas.yaml
kubectl apply -f backend-ontap-nas-eco.yaml
kubectl apply -f backend-ontap-san.yaml
kubectl apply -f backend-ontap-san-eco.yaml 
```


```console
kubectl get tbc -n trident
```

### StorageClass

sc-nas.yaml is the StorageClass definition for the ontap-nas driver.  
sc-nas-eco.yaml is the StorageClass definition for the ontap-nas-economy driver.  
sc-san.yaml is the StorageClass definition for the ontap-san driver.  
sc-san-eco.yaml is the StorageClass definition for the ontap-nas-economy driver.

```console
kubectl apply -f sc-nas.yaml
kubectl apply -f sc-nas-eco.yaml
kubectl apply -f sc-san.yaml
kubectl apply -f sc-san-eco.yaml 
```

```console
kubectl get sc
```

### VolumeSnapshotClass

```console
kubectl apply -f volumesnapshotclass.yaml
```

## :trident: Scenario 03 - Testing Trident with the first applications
```console
cd /root/tridentdemo/scenario03
```

# App for all Storageclasses

```console
kubectl apply -f allstorageclasses.yaml
```

```console
kubectl get pods,pvc -n allstorageclasses
```

```console
kubectl exec -n allstorageclasses $(kubectl get pod -n allstorageclasses -o name) -- sh -c 'echo "Hello little Container! Trident will care about your persistent Data that is written to a pvc utilizing the ontap-nas driver!" > /nas/test.txt'
kubectl exec -n allstorageclasses $(kubectl get pod -n allstorageclasses -o name) -- sh -c 'echo "Hello little Container! Trident will care about your persistent Data that is written to a pvc utilizing the ontap-nas-economy driver!" > /naseco/test.txt'
kubectl exec -n allstorageclasses $(kubectl get pod -n allstorageclasses -o name) -- sh -c 'echo "Hello little Container! Trident will care about your persistent Data that is written to a pvc utilizing the ontap-san driver!" > /san/test.txt'
kubectl exec -n allstorageclasses $(kubectl get pod -n allstorageclasses -o name) -- sh -c 'echo "Hello little Container! Trident will care about your persistent Data that is written to a pvc utilizing the ontap-san-economy driver!" > /saneco/test.txt'
```

```console
kubectl exec -n allstorageclasses $(kubectl get pod -n allstorageclasses -o name) -- more /nas/test.txt
kubectl exec -n allstorageclasses $(kubectl get pod -n allstorageclasses -o name) -- more /naseco/test.txt
kubectl exec -n allstorageclasses $(kubectl get pod -n allstorageclasses -o name) -- more /san/test.txt
kubectl exec -n allstorageclasses $(kubectl get pod -n allstorageclasses -o name) -- more /saneco/test.txt
```

# App for all ontap-nas driver

```console
kubectl apply -f nasapp.yaml
```

```console
kubectl get pods,pvc -n nasapp
```

```console
kubectl exec -n nasapp $(kubectl get pod -n nasapp -o name) -- sh -c 'echo "Hello little Container! Trident will care about your persistent Data that is written to a pvc utilizing the ontap-nas driver!" > /nas/test.txt'
```

```console
kubectl exec -n nasapp $(kubectl get pod -n nasapp -o name) -- more /nas/test.txt
```

# App for all ontap-nas-economy driver

```console
kubectl apply -f nasecoapp.yaml
```

```console
kubectl get pods,pvc -n nasecoapp
```

```console
kubectl exec -n nasecoapp $(kubectl get pod -n nasecoapp -o name) -- sh -c 'echo "Hello little Container! Trident will care about your persistent Data that is written to a pvc utilizing the ontap-nas driver!" > /naseco/test.txt'
```

```console
kubectl exec -n nasecoapp $(kubectl get pod -n nasecoapp -o name) -- more /naseco/test.txt
```

# App for all ontap-san driver

```console
kubectl apply -f sanapp.yaml
```

```console
kubectl get pods,pvc -n sanapp
```

```console
kubectl exec -n sanapp $(kubectl get pod -n sanapp -o name) -- sh -c 'echo "Hello little Container! Trident will care about your persistent Data that is written to a pvc utilizing the ontap-nas driver!" > /san/test.txt'
```

```console
kubectl exec -n sanapp $(kubectl get pod -n sanapp -o name) -- more /san/test.txt
```

# App for all ontap-san-economy driver

```console
kubectl apply -f sanecoapp.yaml
```

```console
kubectl get pods,pvc -n sanecoapp
```

```console
kubectl exec -n sanecoapp $(kubectl get pod -n sanecoapp -o name) -- sh -c 'echo "Hello little Container! Trident will care about your persistent Data that is written to a pvc utilizing the ontap-nas driver!" > /saneco/test.txt'
```

```console
kubectl exec -n sanecoapp $(kubectl get pod -n sanecoapp -o name) -- more /saneco/test.txt
```


## :trident: Scenario 04 - Backup anyone? Installation of Trident protect
```console
cd /root/tridentdemo/scenario04
```

```console
kubectl create ns trident-protect
kubectl create secret docker-registry regcred --docker-username=registryuser --docker-password=Netapp1! -n trident-protect --docker-server=registry.demo.netapp.com
```

```console
helm repo add netapp-trident-protect https://netapp.github.io/trident-protect-helm-chart/
helm registry login registry.demo.netapp.com -u registryuser -p Netapp1!

helm install trident-protect netapp-trident-protect/trident-protect --set clusterName=lod1 --version 100.2510.0 --namespace trident-protect -f trident_protect_helm_values.yaml
```

After a very short time you should be able to see Trident protect being installed successfully. 
```console
kubectl get pods -n trident-protect
```

Trident Protect CR can be configured with YAML manifests or CLI.  
Let's install its CLI which avoids making mistakes when creating the YAML files:  
```console
cd
curl -L -o tridentctl-protect https://github.com/NetApp/tridentctl-protect/releases/download/25.10.0/tridentctl-protect-linux-amd64
chmod +x tridentctl-protect
mv ./tridentctl-protect /usr/local/bin

curl -L -O https://github.com/NetApp/tridentctl-protect/releases/download/25.02.0/tridentctl-completion.bash
mkdir -p ~/.bash/completions
tridentctl-protect completion bash > ~/.bash/completions/tridentctl-completion.bash
echo 'source ~/.bash/completions/tridentctl-completion.bash' >> ~/.bashrc
source ~/.bashrc
```


The CLI will appear as a new sub-menu in the _tridentctl_ tool.  
```console
tridentctl-protect version
```

## :trident: Scenario 05 - Trident protect initial configuration

# create appvault
```console
cat /root/tridentdemo/ansible_S3_SVM_result.txt
```

```console
set -priv advanced
vserver object-store-server user show -vserver svm_s3
```


```console
BUCKETKEY=<youraccesskey>
BUCKETSECRET=<yoursecretkey>
```

```console
kubectl create secret generic -n trident-protect s3-creds --from-literal=accessKeyID=$BUCKETKEY --from-literal=secretAccessKey=$BUCKETSECRET
```
 
```console
tridentctl-protect create appvault OntapS3 ontap-vault -s s3-creds --bucket s3lod --endpoint 192.168.0.230 --skip-cert-validation --no-tls -n trident-protect
```
Verify the creation:
```console
tridentctl-protect get appvault -n trident-protect
```

```console
cd
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip -q awscliv2.zip
./aws/install
rm -rf aws

mkdir ~/.aws
cat > ~/.aws/credentials << EOF
[default]
aws_access_key_id = $BUCKETKEY
aws_secret_access_key = $BUCKETSECRET
EOF
```

## :trident: Scenario 06 - Protecting an application
## A. App creation 


```console
tridentctl-protect create app nasapp --namespaces 'nasapp(app=busybox)' -n nasapp
```
You can verify the status with the following command:
```console
tridentctl-protect get app -n nasapp
```


## B. Snapshot creation  


```console
tridentctl-protect create snapshot nasappsnap --app nasapp --appvault ontap-vault -n nasapp
```

```console
tridentctl-protect get snap -n nasapp
```

```console
kubectl get vs -n nasapp
```


Browsing through the bucket, you will also find the content of the snapshot (the metadata):  
```console
SNAPPATH=$(kubectl get snapshot nasappsnap -n nasapp -o=jsonpath='{.status.appArchivePath}')
aws s3 ls --no-verify-ssl --endpoint-url http://192.168.0.230 s3://s3lod/$SNAPPATH --recursive  
```

## C. Backup creation  


```console
tridentctl-protect create backup nasappbkp1 --app nasapp --snapshot nasappsnap --appvault ontap-vault  -n nasapp
tridentctl-protect get backup -n nasapp
```

```console
APPPATH=$(echo $SNAPPATH | awk -F '/' '{print $1}')
aws s3 ls --no-verify-ssl --endpoint-url http://192.168.0.230 s3://s3lod/$APPPATH/
```


## D. Scheduling  
```console
cd /root/tridentdemo/scenario06
```
```console
kubectl apply -f nasapp-sched.yaml
```

```console
tridentctl-protect get schedule -n nasapp
```

## :trident: Scenario 07 - Restoring an application
## A. In-place partial snapshot restore  

Let's first delete the content of one of the volume mounted on the pod (_nas_).  
```console
kubectl exec -n nasapp $(kubectl get pod -n nasapp -o name) -- rm -f /nas/test.txt
kubectl exec -n nasapp $(kubectl get pod -n nasapp -o name) -- more /nas/test.txt
```
```console
tridentctl-protect create sir nasappsir1 --snapshot nasapp/nasappsnap --resource-filter-include='[{"labelSelectors":["volume=volnas"]}]' -n nasapp
```

```console
tridentctl-protect get sir -n nasapp; kubectl -n nasapp get pod,pvc
```

# check result
```console
kubectl exec -n nasapp $(kubectl get pod -n nasapp -o name) -- ls /nas/
kubectl exec -n nasapp $(kubectl get pod -n nasapp -o name) -- more /nas/test.txt
```

## B. In-place restore of a backup 
```console
kubectl delete -n nasapp deploy busybox
kubectl delete -n nasapp pvc --all
```
```console
tridentctl-protect create bir nasappbir -n nasapp --backup nasapp/nasappbkp1
```
```console
tridentctl-protect get bir -n nasapp
```

```console
kubectl -n nasapp get po,pvc
```
```console
kubectl exec -n nasapp $(kubectl get pod -n nasapp -o name) -- more /nas/test.txt
```

