### Setup Kube-Virt
```
export VERSION=$(curl -s https://api.github.com/repos/kubevirt/kubevirt/releases | grep tag_name | grep -v -- '-rc' | sort -r | head -1 | awk -F': ' '{print $2}' | sed 's/,//' | xargs)
echo $VERSION
kubectl create -f https://github.com/kubevirt/kubevirt/releases/download/${VERSION}/kubevirt-operator.yaml
```
Create Resource Deployment to `kubevirt` namespace
```
kubectl create -f https://github.com/kubevirt/kubevirt/releases/download/${VERSION}/kubevirt-cr.yaml

https://kubevirt.io/user-guide/virtual_machines/templates/#templates
```
Install  Mock VM
```
oc get vms
virtctl start testvm
```
Check Prometheus Metrics
```
oc port-forward svc/kubevirt-prometheus-metrics -n kubevirt 8888:443

https://localhost:8888/metrics
```
https://kubevirt.io/user-guide/operations/component_monitoring/#integrating-with-the-prometheus-operator

https://www.redhat.com/sysadmin/openshift-terminating-state
```
kubevirt_vmi_memory_resident_bytes{name="vm-test-01",namespace="default",node="node01"} 2.5595904e+07
kubevirt_vmi_network_traffic_bytes_total{domain="default_vm-test-01",interface="vnet0",name="vm-test-01",namespace="default",node="node01",type="rx"} 8431
kubevirt_vmi_network_traffic_bytes_total{domain="default_vm-test-01",interface="vnet0",name="vm-test-01",namespace="default",node="node01",type="tx"} 1835
kubevirt_vmi_vcpu_seconds{domain="default_vm-test-01",id="0",name="vm-test-01",namespace="default",node="node01",state="1"} 19
```

Expose port
```
virtctl image-upload dv  ubuntu20-server-cloud-dv  --image-path="$PWD/ubuntu-20-server.img" --size=5Gi --insecure

virtctl expose vmi fedora --name=fedora-ssh-port --port=20222 --target-port=22

virtctl image-upload --pvc-name=win10-home --image-path="$PWD/win10.iso"  --size=5.5Gi  --insecure
```


ปัญหา
1. IP เปลี่ยนเอง
2. การจัดการ Disk หลายๆตัว
3. Migrate VM ไปอีก Node Hardware

ไฟล์ system มันไปเก็บใน pvc ของ Base ISO Image เลยซึ่งมันผิด# vm-kubevirt-integration
