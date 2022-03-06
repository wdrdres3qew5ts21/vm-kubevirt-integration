### Current KubeVirt  Version (0.50.x+++)
```json
Client Version: version.Info{GitVersion:"v0.44.1", GitCommit:"01805c72083902832ecbd08559f4d3aa88110ea1", GitTreeState:"clean", BuildDate:"2021-08-12T12:35:53Z", GoVersion:"go1.16.6", Compiler:"gc", Platform:"darwin/amd64"}

Server Version: version.Info{GitVersion:"v0.50.0", GitCommit:"7e034450c10ad2a879db960b9912be2dff7ed9ce", GitTreeState:"clean", BuildDate:"2022-02-10T13:39:37Z", GoVersion:"go1.17.5", Compiler:"gc", Platform:"linux/amd64"}
```
### Setup Kube-Virt
```
oc new-project kubevirt

export VERSION=$(curl -s https://api.github.com/repos/kubevirt/kubevirt/releases | grep tag_name | grep -v -- '-rc' | sort -r | head -1 | awk -F': ' '{print $2}' | sed 's/,//' | xargs)
echo $VERSION
kubectl create -f https://github.com/kubevirt/kubevirt/releases/download/${VERSION}/kubevirt-operator.yaml

# Not working anymore need to enable from KubeVirt CR only
# kubectl create configmap kubevirt-config -n kubevirt --from-literal debug.useEmulation=true

# Create Resource Deployment to `kubevirt` namespace

kubectl create -f https://github.com/kubevirt/kubevirt/releases/download/${VERSION}/kubevirt-cr.yaml

export VERSION=$(curl -s https://github.com/kubevirt/containerized-data-importer/releases/latest | grep -o "v[0-9]\.[0-9]*\.[0-9]*")

kubectl create -f https://github.com/kubevirt/containerized-data-importer/releases/download/$VERSION/cdi-operator.yaml

kubectl create -f https://github.com/kubevirt/containerized-data-importer/releases/download/$VERSION/cdi-cr.yaml

# Update KubeVirt CR to use Software Emulation
oc apply -f final-solution/kubevirt-cr/

```

# Kubevirt Software Emulation
Not working anymore need to enable from KubeVirt CR only
https://github.com/kubevirt/kubevirt/blob/main/docs/software-emulation.md

# Create Demo Cirros VM
https://kubevirt.io/user-guide/virtual_machines/templates/#templates

```
oc project kubevirt
kubectl apply -f https://kubevirt.io/labs/manifests/vm.yaml
```
CDI KubeVirt
```
export VERSION=$(curl -s https://github.com/kubevirt/containerized-data-importer/releases/latest | grep -o "v[0-9]\.[0-9]*\.[0-9]*")
kubectl create -f https://github.com/kubevirt/containerized-data-importer/releases/download/$VERSION/cdi-operator.yaml
kubectl create -f https://github.com/kubevirt/containerized-data-importer/releases/download/$VERSION/cdi-cr.yaml
```

# Start  Mock VM
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
oc new-project vm-images

oc project vm-images

virtctl image-upload dv  ubuntu20-iso-dv  --image-path="$PWD/ubuntu-20-server.iso" --size=5Gi --insecure

virtctl expose vmi fedora --name=fedora-ssh-port --port=20222 --target-port=22

virtctl image-upload --pvc-name=win10-home --image-path="$PWD/win10.iso"  --size=5.5Gi  --insecure

virtctl image-upload dv fedora35-iso-dv --image-path="$PWD/fedora-35-server.iso"  --size=5.5Gi  --insecure
```

### Need using IMG or QCOW2 format for automatic boot system

```
oc project vm-images

virtctl image-upload dv  ubuntu20-cloud-dv  --image-path="$PWD/ubuntu-20-server.img" --size=5.5Gi --insecure

virtctl image-upload dv fedora35-cloud-dv --image-path="$PWD/fedora-35-cloud.qcow2"  --size=5.5Gi  --insecure

oc new-project legacy-company

# Maybe not require anymore ?
oc adm policy add-scc-to-user  ibm-anyuid-hostaccess-scc  -z default
```

# Using `./tool` for Debug about Network to Virtual Machine Pod
Network Debug Pod Required Privileged to Access Linux Capabilities Feature
Inside nettool pod just paste SSH Private Key so you can login to VM
```
oc apply -f tool/
```
Login to VM with this Solution
```
oc exec -it nettool-fc8556c58-ktrs6    bash
## inside Nettool (Network Debug Pod)

ssh root@<target-vm-ip>
```
VM can be hardcode IP by setting Ethernet Interface at Virtual Machine CR

ในปัจจุบันถ้าเราใช้ Custom IP ที่สร้างขึ้นมาเองจาก Multus  Pod ที่ใช้ Custom IP ตัวนั้นจากสามารถ Ping เจอกันแค่อยู่ใน Host เดียวกันเท่านั้นเพราะว่ามันใช้ Bridge เดียวกัน สามารถลองดูตัวอย่างได้จาก Pod Frontend ของเรา ถ้าหากเราไม่ใช้ Custom IP จะไม่มี Bridge พิเศษสร้างขึ้นมา
โดยในปัจจุบัน Pod ที่มีการใช้ Custom IP คือ Fedora-Warehouse, Ubuntu-Database, Frontend todo หมายควาว่าถ้า Pod เหล่านี้ไปลงที่ Node ไหน Node เหล่านั้นจะต้องมี Bridge Custom ขึ้นมา
ใช้ MACVLAN แก้ปัญหาข้าม Node ได้

```yaml
apiVersion: kubevirt.io/v1
kind: VirtualMachine
metadata:
  generation: 1
  labels:
    kubevirt.io/os: linux
  name: fedora-warehouse-domain
  namespace: legacy-company
... Reduce Detail about this Example for more detail see ./final-solution/virtual-machine/fedora-warehouse-domain.yaml
networkData: |
            version: 2
            ethernets:
              eth0: 
                addresses:
                - 192.168.255.1/24
```


# Using `./final-solution` directory for create Infrastructure

```
oc new-project legacy-company
oc project legacy-company
oc apply -f final-solution/virtual-machine/
```

# Using `./dns` directory for demonstrate problem

```
yyyyyyyy
```

# Debug Kubernetes Node (Real Host that host pod)

```
oc debug node/<vm-ip-host>
```



Argo Kubevirt
```
virtualmachines.kubevirt.io "ubuntu" is forbidden: User "system:serviceaccount:thesis-virt:argocd-argocd-application-controller" cannot patch resource "virtualmachines" in API group "kubevirt.io" in the namespace "thesis-virt"
```

ปัญหา
1. IP เปลี่ยนเอง
2. การจัดการ Disk หลายๆตัว
3. Migrate VM ไปอีก Node Hardware

ไฟล์ system มันไปเก็บใน pvc ของ Base ISO Image เลยซึ่งมันผิด# vm-kubevirt-integration

Trigger Argo Hook

webook
https://argoproj.github.io/argo-cd/operator-manual/webhook/



Debug network


dig @dns.openshift-dns.svc.cluster.local -p 8053 infra.napzz-thesis-chula.com

http://www-inf.int-evry.fr/~hennequi/CoursDNS/NOTES-COURS_eng/syntax.html

https://cloud.redhat.com/blog/using-the-multus-cni-in-openshift

https://github.com/openshift/multus-cni/blob/master/docs/quickstart.md


### วิธ๊เข้าเครื่อง
เข้าไปใน pod debug network
```
ssh -i  id_rsa vm1@172.30.65.35
```


## Operator Requirement
Golang 1.16.x Version (only)

swtich GO version
https://stackoverflow.com/questions/58210941/unable-to-brew-switch-go-versions