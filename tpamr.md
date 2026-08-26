

export KUBECONFIG=~/.kube/tpsc05-config
kubectl config use-context bbox-context-kub1

k describe application bbox -n tpsc05busybox
k get pods,pvc -n tpsc05busybox
kubectl exec $(kubectl get pod -o name) -- more /data1/file.txt
kubectl exec $(kubectl get pod -o name) -- more /data2/file.txt
APPID=$(kubectl get application bbox -o=jsonpath='{.metadata.uid}' -n tpsc05busybox) && echo $APPID

kubectl config use-context bbox-context-kub2

kubectl get -n tpsc05busyboxdr pvc,pods

cat << EOF | kubectl apply -f -
apiVersion: protect.trident.netapp.io/v1
kind: AppMirrorRelationship
metadata:
  name: bboxamr1
  namespace: tpsc05busyboxdr
spec:
  desiredState: Established
  destinationAppVaultRef: ontap-vault
  namespaceMapping:
  - destination: tpsc05busyboxdr
    source: tpsc05busybox
  recurrenceRule: |-
    DTSTART:20240901T000200Z
    RRULE:FREQ=MINUTELY;INTERVAL=5
  sourceAppVaultRef: ontap-vault
  sourceApplicationName: bbox
  sourceApplicationUID: $APPID
  storageClassName: sc-nfs
EOF

tridentctl-protect get amr -n tpsc05busyboxdr --context bbox-context-kub2

ONTAP - snapmirror show

kubectl get -n tpsc05busyboxdr pvc,pods

kubectl config use-context bbox-context-kub1

kubectl delete deploy busybox && kubectl delete pvc --all

kubectl config use-context bbox-context-kub2

kubectl patch amr bboxamr1 -n tpsc05busyboxdr --type=merge -p '{"spec":{"desiredState":"Promoted"}}'

tridentctl-protect get amr -n tpsc05busyboxdr --context bbox-context-kub2

kubectl get -n tpsc05busyboxdr pod,pvc

kubectl exec -n tpsc05busyboxdr $(kubectl get pod -n tpsc05busyboxdr -o name) -- more /data1/file.txt

kubectl exec -n tpsc05busyboxdr $(kubectl get pod -n tpsc05busyboxdr -o name) -- more /data2/file.txt
