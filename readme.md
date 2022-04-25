### Current KubeVirt  Version (0.50.x+++)

| ปัญหาที่พบ | คำอธิบาย | วิธีแก้ป้ญหา |
| --- | ----------- |-----------|
| KubeVirt ไม่รองรับ Macvlan ทำให้ไม่สามารถวาง VM แล้วเชื่อมต่อข้าม Node ได้ | เพราะ CNI ต้องการให้ IP กับ MacAddress นั้นตรงกันแต่ MacVlan ทำให้ มี MacAddress ของ VM ถูกย้ายไปที่ระดับ Pod แทนทำให้ Traffic เจ้าไปไม่ถึง|https://github.com/kubevirt/user-guide/pull/528 ทาง KubeVirt แนะนำว่าใช้ NMState แล้วเปลี่ยนไปใช้ Bridge CNI จากนั้นให้ NMState เชื่อมทุกๆ Bridge ของทุกๆ Kube Node เข้าด้วยกัน ก็น่าจะทำให้ Network สามารถข้าม Node ได้ แต่ NMState อาจจะเพิ่ม Scope ใหญ่เกินไปเดี่ยวขอพักไว้ก่อนครับเดี่ยวทำ Node เดียวให้ได้ก่อนค่อยขยับไปหลาย Node https://github.com/kubevirt/kubevirt/issues/5483 https://github.com/nmstate/kubernetes-nmstate  |
|ถ้าใช้ Multus กับ VM KubeVirt เหมือนจะไม่สามารถ FIX IP ได้เหมือนตอนทำกับ Pod สำหรับ VM ที่ถูก Migrate ขึ้นมา [แก้ได้แล้ว]| https://github.com/kubevirt/kubevirt/issues/4564 ถ้าเป็น Pod ปกติหรือ Interface แรกสามารถใช้ Cloud Init Fix IP ได้แต่ปัญหาคือมันจะไมไ่ด้ Interface Multus เข้าไปและทำให้เชื่อมต่อกับ Pod อื่นหรือออกไปยัง Internet ข้างนอกไมไ่ด้ | ทดลองใช้ FIX NetworkAttchMent ไปก่อน แล้วเชื่อม Bridge เองทีหลัง เพราะว่า IP อยู่ใน วงเดียวกันทำ manual routing เองอาจจะทำได้ |
|ip ไม่ยอมคืนหลงัใช้งาน pod[แก้ได้แล้ว] |https://www.cni.dev/plugins/current/ipam/host-local/ จากทฤษฎีนี้  พบว่าจริงๆแล้วไม่ต้องคืน IP เพราะเรา FIX IP เสมอ https://devopstales.github.io/home/multus/| https://www.cni.dev/plugins/current/ipam/host-local/



Repo: https://github.com/wdrdres3qew5ts21/vm-kubevirt-integration (Structure/Assets)
https://github.com/wdrdres3qew5ts21/coredns-integration-operator (Golang Coding)


#### สิ่งที่ CRD ควรจะทำได้

ความสามารถ
คำอธิบาย
Auto Discovery Pod Virtual Machine (feature หลักต้องทำ)
ด้วยการที่ Pod ใน Kubernetes นั้นสามารถถูกปิดและเกิดใหม่เรื่อยๆและ IP ของ Interface หลักอาจจะเปลี่ยนไปเรื่อยๆตัว DNS จึงแตกต่างจาก DNS Record ปกติที่ IP ไม่มีการเปลี่ยน (หรือไม่ได้เปลี่ยนเรื่อยๆบ่อยๆ) จึงน่าจะใช้เทคนิคเดียวกับ Kubernetes Service ปกติคือกาใช้ Operator ไปต่อกับ Kubernetes API เพื่อดึง Label นำ IP ของ Pod เพื่อทำการ Mapping กับ Interface ของ IP จาก Multus
Automate Configure Network Topology
(feature รองแต่มีประโยชน์)
สำหรับกรณีที่ Subnet ต้องมีการทำ Routing ข้ามระหว่าง Subnet ควรจะมี Feature ที่ CRD สามารถไปสร้าง Network ที่จำเป็นให้ได้เพื่อลดงาน Admin ที่ดูแลระบบ



#### ความคืบหน้าปัจจุบัน
สามารถสร้างโครง Operator ด้วย Golang และควบคุม Resource เพื่อทำ Reconcile ใน Kubernetes ได้ว่ามี Object หรือ Resource ที่ต้องสร้างใดบ้าง
ขั้นตอนต่อไป: ต้องสามารถสร้าง CoreDNS ที่ Mapping กับ Pod Virtual Machine เพื่อทำเป้น DNS Record โดยผู้ดูแลระบบไม่ต้อง Config ผ่าน Linux โดยตรงแต่ควรสามารถ Config จาก CRD ได้และ Operator ทำการแก้ไขอัพเดท DNS  Record ที่ Map กับ Pod Virtual machine เอง (feature หลักต้องทำ)
สามารถสร้าง Pod ที่มีหลาย Network Interface ได้และเชื่อมต่อข้ามกันระหว่าง Subnet ได้ผ่าน Pod Router แต่กระบวนการเชื่อม Routing ยังทำ manual ด้วยการ exec เข้าไปใน pod 
ขั้นตอนต่อไป: ผู้ดูแลระบบไม่ควรที่จะต้องเข้าไป manual configure คำสั่ง linux ตรงๆใน Pod เพื่อทำ Routing เอง แต่ควรจะสามารถ Config ผ่านคำสั่ง High Level ผ่าน ได้ง่ายๆจาก CRD ได้ (feature รองแต่มีประโยชน์ที่จะทำเพื่อให้โปรเจคสมบูรณ์ใช้ได้จริง - แต่ในโครงร่างไม่ได้ระบุเน้นเรื่อง Subnet ที่แตกต่างกันครับเขียนแค่ Subnet Network ในวงเดียวกัน)
 มีตัวอย่างร่างของ Pod CoreDNS ที่ Mapping Private DNS กับ Pod ได้ ด้วยวิธี Manual จากเทอมที่แล้วที่อยู่ในเล่มโครงร่างครับ
 ขั้นตอนต่อไป: เชื่อมข้อ 1) และ 2) เข้าด้วยกันเพื่อให้สามารถทำได้ผ่านเครื่อง Virtual machine Pod จริงๆมากกว่าแค่ Pod Kubernetes ปกติ (ปัจจุบันทดสอบบน Pod ปกติเป็นหลัก)


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

# Create Resource Deployment to kubevirt namespace

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

virtctl image-upload dv fedora35-iso-dv   -n vm-images --image-path="$PWD/fedora-35-server.iso"  --size=5.5Gi  --insecure
```
### Image ใช้จริงอยู่ตรงนี้
### Need using IMG or QCOW2 format for automatic boot system

```
oc project vm-images

virtctl image-upload dv  database-server-vm-dv  -n vm-images --image-path="$PWD/data-server-sql.img" --size=25Gi --insecure

virtctl image-upload dv  frontend-server-vm-dv   -n vm-images --image-path="$PWD/frontend-server.img" --size=25Gi --insecure

virtctl image-upload dv  backend-server-vm-dv   -n vm-images --image-path="$PWD/backend-server-docker.img" --size=25Gi --insecure

virtctl image-upload dv  ubuntu20-cloud-dv   -n vm-images --image-path="$PWD/ubuntu-20-server.img" --size=5.5Gi --insecure

virtctl image-upload dv fedora35-cloud-dv  -n vm-images --image-path="$PWD/fedora-35-cloud.qcow2"  --size=2Gi  --insecure
virtctl image-upload pvc fedora35-cloud-dv  -n vm-images --image-path="$PWD/fedora-35-cloud.qcow2"  --size=2Gi  --insecure

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
https://developers.redhat.com/blog/2019/11/12/using-the-red-hat-openshift-tuned-operator-for-elasticsearch

Mount Capability net เข้ามาให้ forward ได้ผ่าน sysctl init contaienr
https://cloud.ibm.com/docs/openshift?topic=openshift-kernel#worker

การทำให้ Network สามารถ Routing ข้าม Subnet กันได้ทำได้จากวิธีการเพิ่ม Default GW Routing ข้ามไปยังอีก Kubernetes Node หนึ่งซึ่งทำเฉพาะตัวที่เป็น Router ก็พอแล้ว

sudo apt-get install openjdk-11-jdk

sysctl -w net.ipv4.ip_forward=1

kubelet --allowed-unsafe-sysctls 'net.ipv4.ip_forward'

ls -alZ /proc/sys/net/ipv4/ip_forward

เวลา forward package มันจะวิ่งตรงไปเลยไม่มีการแปลง IP ที่ตรงกลางตัวอย่างเช่นจาก Public ไปหา Private Subnet ที่เครื่อง Router ก็จะเห็น IP ถูก Forward ตรงๆไปเลย
```
18:33:53.749498 IP 192.168.15.26 > 192.168.16.17: ICMP echo request, id 55, seq 122, length 64
18:33:53.749523 IP 192.168.16.17 > 192.168.15.26: ICMP echo reply, id 55, seq 122, length 64
18:33:54.749449 IP 192.168.15.26 > 192.168.16.17: ICMP echo request, id 55, seq 123, length 64
18:33:54.749479 IP 192.168.16.17 > 192.168.15.26: ICMP echo reply, id 55, seq 123, length 64
```

```
<Subnet>
ip route add 192.168.16.0/24 via 192.168.15.20 dev net1

```
ใช้ `TCP Dump` ตรงกลางที่ Router ซึ่งมี Interface สองขาคือ Subnet Public กับ Subnet Private
```
tcpdump -i <interface-name>
```
ตัวอย่างเช่น public interface วิ่งด้วย net1 เชื่อมไปหา router ด้วย net1 ก็ต้อง dump ด้วย net1
ซึ่งระหว่างที่มี packet วิ่งมาที่ router ตรงกลาง traffic นั้น forward จาก Interface net1 ไปหา net2 เราจึงต้องทำการเปิด sysctl routing `net.ipv4.ip_forward` ด้วย และก็ใช้ NAT ในการแปลงระหว่าง Network ที่อยู่คนล่ะวง


ซึ่งจะต้องการสิทธิของ Linux Routing `RTNETLINK answers: Operation not permitted` จึงจะสามารถเพิ่ม routing ได้ จึงต้องเพิ่มสิทธิ
`NET_ADMIN` เข้าไปด้วย
https://blog.pentesteracademy.com/linux-security-understanding-linux-capabilities-series-part-i-4034cf8a7f09

https://www.youtube.com/watch?v=5qHjNRKQh3M&ab_channel=LinuxEssentials


# Using `./final-solution` directory for create Infrastructure

ถ้าใช้ VM-Images ข้าม Namespace แล้วจะเกิด Error ดั่งนี้ ต้องให้สิทธิในการ Clone DataVolume เลยต้องทำการให้สิทธิด้วยนั่นเอง
```
oc apply -f final-solution/virtual-machine/
Error from server (Invalid): error when creating "final-solution/virtual-machine/fedora-crm-domain.yaml": admission webhook "virtualmachine-validator.kubevirt.io" denied the request: Authorization failed, message is: User system:serviceaccount:legacy-company:default has insufficient permissions in clone source namespace vm-images
```

```
oc new-project legacy-company
oc project legacy-company
oc apply -f final-solution/rbac
oc apply -f final-solution/virtual-machine/
```

# Using `./dns` directory for demonstrate problem
จะสร้างตัวอย่างของ DNS Middleware Layer ที่ทำให้ Pod ใน Kubernetes สามารถ Resolve Private Domain ใหม่ได้ทันทีโดยไม่ต้องแก้ Config อะไรในตัว Pod เลยเพราะว่าตัว Pod นั้นมา Query CoreDNS ของ Cluster หลักแล้ว Traffic ก็ถูกไปถาม DNS Server ตัวใหม่ที่เราสร้างจากตรงนี้ได้ด้วยเช่นกันนั่นเอง
```
oc apply -f dns
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
x

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

### Local Virtual Machine Snapshot
192.168.18.5
data-server (ubuntu 20.04)
username: supakorn
password: lnwza007

192.168.18.4
backend-server (ubuntu 20.04)
username: supakorn
password: lnwza007

192.168.18.6
frontend-server (ubuntu 20.04)
username: supakorn
password: lnwza007

ต้อง dump vmdk format ออกมาด้วยคำสั่งนี้ 
```
VBoxManage clonehd  --format VMDK data-server.vmdk data-server.img
VBoxManage clonehd  --format VMDK data-server.vmdk data-server.vmdk
```

https://blogs.oracle.com/cloud-infrastructure/post/running-kvm-and-vmware-vms-in-container-engine-for-kubernetes

KubeVirt มีปัญหากับ Macvlan ต้องใช้ Bridge แทน
https://github.com/kubevirt/kubevirt/issues/5483#issuecomment-1077625725


Interface Network ที่ใช้งานประเภทความต่างระหว่าง Hardware Emulation กับ para Virtual ซึ่งเราต้องเปลี่ยน Interfacte ลองให้ใช้เป้น Virt-o ไปเลย และก็ configure ตัว VM ด้วย NetworkManager แทนที่จะใช้ NetworkD โดยตรงถ้าเป็น Ubuntu ให้ไปเปลี่ยนใน Netplan ใช่เป้น backend Network Manger

https://www.nakivo.com/blog/virtualbox-network-setting-guide/

https://askubuntu.com/questions/1290471/ubuntu-ethernet-became-unmanaged-after-update

our /etc/netplan/*.yaml file should look like:
```
network:
  version: 2
  renderer: NetworkManager
```
ตามด้วยคำสั่ง
```
sudo netplan generate

sudo netplan apply

reboot
```
### Virtual Machine Detail 

ติดตั้ง mysql ลงใน ubuntu data-server
https://www.digitalocean.com/community/tutorials/how-to-install-mysql-on-ubuntu-20-04

sudo apt install mysql-server

sudo systemctl start mysql.service
### Local Virtual Machine Snapshot

192.168.18.4
backend-server (ubuntu 20.04)
username: supakorn
password: lnwza007


192.168.18.5
data-server (ubuntu 20.04)
username: supakorn
password: lnwza007



192.168.18.7
crm-server (ubuntu 20.04)
username: supakorn
password: lnwza007



### SQL Script on Data-Server
https://www.digitalocean.com/community/tutorials/how-to-create-a-new-user-and-grant-permissions-in-mysql
วิธี login เข้าระบบ mysql ผ่าน socket authneticatio root user

```
sudo mysql
```
แก้ไฟล์ Config ใน /etc/mysql/mysql.conf.d
```
select user,host from mysql.user;

CREATE USER 'supakorn'@'192.168.18.4' IDENTIFIED BY 'lnwza007';
GRANT all privileges ON *.* TO 'supakorn'@'192.168.18.4';
flush privileges

create database napzz;
use napzz
create table customer(id int primary key,firstname varchar(100),lastname varchar(100),age int,)
inser ข้อมูลลงไป
```

ต่อ mysql แบบ remote ด้วยคำสั่ง
```
mysql -h 192.168.18.5 -P 3306 -u supakorn -p
```

ในเครื่อง Fedora ทดลองเปลี่ยน IP Address

```
nmcli show connection

nmcli con mod enps03 ipv4.addresses 192.168.18.5/24
```

ตอนนี้ตัว Bridge Router Pingไปหา Ip VM KubeVirt แต่ว่ามันไปไม่ได้เพราะมันใช้ MacVlan
https://github.com/kubevirt/kubevirt/issues/5483



### Debug Bridge Network
แสดง Link ที่เชื่อมอยู่ใน Bridge นั้นๆและ debug Network Namespace
```
brctl show

# ใช้คำสั่งนี้แทน brctl ได้ในการดู bridge
ip link show type bridge

ip link show master brse

ip netns list

545: vetha3ba3899@if8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master brse state UP mode DEFAULT group default 
    link/ether fa:39:1a:4b:ab:30 brd ff:ff:ff:ff:ff:ff link-netnsid 43
553: veth1ccc0e6e@if6: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master brse state UP mode DEFAULT group default 
    link/ether e2:2d:1e:2f:44:cb brd ff:ff:ff:ff:ff:ff link-netnsid 44
839: veth7d1146c3@if6: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master brse state UP mode DEFAULT group default 
    link/ether 8a:71:24:5b:c4:0e brd ff:ff:ff:ff:ff:ff link-netnsid 45
```

### Static IP For KubeVirt
ถ้าเกิดเราใช้ IPAM แบบปกติมันจะ Fix IP ของ Multus ไมไ่ด้เราต้องใช้ Static NetworkAttachment มาเสียบลงไปแล้วใช้ Bridge เดียวกันกับก่อนหน้าที่เราใช้ ให้ดูที่สองไฟล์นี้เป็นตัวอย่างจะใช้ CNI ประเภท Bridge แทน macvlan เพื่อแก้ปัญหา Network จาก Kubevirt https://github.com/kubevirt/kubevirt/issues/5483
และใช้ Bridge ชื่อเดียวกันทำให้ plugin สำเร็จและ Routing ได้

```
oc apply -f final-solution/bridge-virtual-machine/ubuntu-second-domain.yaml

oc apply -f final-solution/bridge-virtual-machine/bridge-static.yaml 
```

เครื่องจริงที่ใช้คือเครื่องที่โดน Import VM โดยตรงก็ทำงาไนด้เช่นเดียวกันโดยการใช้เทคนิค FIX IP

```
oc apply -f final-solution/bridge-virtual-machine/database-server.yaml

oc apply -f final-solution/bridge-virtual-machine/bridge-server.yaml 
```


### การใช้ NMState เชื่อม Bridge ข้าม Node
https://kubevirt.io/2020/Multiple-Network-Attachments-with-bridge-CNI.html

ลง Network Add-on Manager CRD
```
kubectl apply -f https://github.com/kubevirt/cluster-network-addons-operator/releases/download/v0.74.0/namespace.yaml
kubectl apply -f https://github.com/kubevirt/cluster-network-addons-operator/releases/download/v0.74.0/network-addons-config.crd.yaml
kubectl apply -f https://github.com/kubevirt/cluster-network-addons-operator/releases/download/v0.74.0/operator.yaml
```

จะไปสร้าง DaemonSet namespace `cluster-network-addons` สร้าง Resource Network AddOns 
```
oc apply -f final-solution/02-nmstate/manifest
oc apply -f final-solution/02-nmstate/mmstate.yaml

```

ลง Kubernetes NMState CRD ซึ่งจะใชเในการจัดการ Network Manager บนเครื่อง Host ให้ Clone git ลงมาก่อนแล้วติดตั้งลงไป
https://github.com/nmstate/kubernetes-nmstate

```
oc apply -f final-solution/02-nmstate/bundle/
```







### บั้คเกี่ยวกับ JVM รายละเอียดปลีกย่อย

BackendServer
ssh supakorn@localhost -p 2200



docker run -p 8080:8080 quay.io/linxianer12/todoapp-frontend:1.0.0

docker run -p 8080:9090  quay.io/linxianer12/subject-reservation-service:b0dff8a-dirty

Java Virtual Machine มีปัญหาเมื่อนำ VM จาก Mac ไปใช้
```
2022-04-11 16:01:07.278  INFO [hello-world,,,] 1 --- [           main] trationDelegate$BeanPostProcessorChecker : Bean 'org.springframework.cloud.autoconfigure.ConfigurationPropertiesRebinderAutoConfiguration' of type [org.springframework.cloud.autoconfigure.ConfigurationPropertiesRebinderAutoConfiguration$$EnhancerBySpringCGLIB$$91bfcc6f] is not eligible for getting processed by all BeanPostProcessors (for example: not eligible for auto-proxying)
[CodeBlob (0x00007f0bf42ccb50)]
Framesize: 84
Runtime Stub (0x00007f0bf42ccb50): ic_miss_stub
Could not load hsdis-amd64.so; library not loadable; PrintAssembly is disabled
#
# A fatal error has been detected by the Java Runtime Environment:
#
#  Internal Error (sharedRuntime.cpp:842), pid=1, tid=0x00007f0c04fd9ae8
#  fatal error: exception happened outside interpreter, nmethods and vtable stubs at pc 0x00007f0bf42ccc06 
#
# JRE version: OpenJDK Runtime Environment (8.0_171-b11) (build 1.8.0_171-b11)
# Java VM: OpenJDK 64-Bit Server VM (25.171-b11 mixed mode linux-amd64 compressed oops)
# Derivative: IcedTea 3.8.0
# Distribution: Custom build (Wed Jun 13 18:28:11 UTC 2018)
# Core dump written. Default location: //core or core.1
#
# An error report file with more information is saved as:
# //hs_err_pid1.log
#
# If you would like to submit a bug report, please include
# instructions on how to reproduce the bug and visit: 
#   http://icedtea.classpath.org/bugzilla
#
```
บั้ค JVM Instruction
https://github.com/elastic/elasticsearch/issues/49178

https://bugzilla.redhat.com/show_bug.cgi?id=1328503

https://bugzilla.redhat.com/show_bug.cgi?id=1897563

https://askubuntu.com/questions/710392/java-8-oracle-1-8-0-66-problem-with-printassembly-could-not-load-hsdis-amd64



### Forward ไปหา Private Network
ต้องใช้ domian internal เหมือนเดิมเพราะว่า Certificate นั้นผูกไว้กับ Domain ที่เรียก เราเลยต้องแก้ที่ `/etc/hosts` แทนเพื่อให้รักษา Certificate ที่ Signining Domain เอาไว้

semanage fcontext -a -t container_file_t  "$(pwd)/nginx(/.*)?"
restorecon -Rv  nginx/

podman run -p 3030:3030 -d -v /home/lab-user/nginx/:/etc/nginx/conf.d docker.io/nginx:1.21.6-perl

podman run  -p 3030:3030  -d quay.io/linxianer12/proxy-openshift-kubevirt-lab:0.0.1



-v /home/lab-user/nginx/:/etc/nginx/conf.d

sudo su

firewall-cmd --add-port=3030/tcp --permanent

firewall-cmd --reload

export KUBECONFIG=/home/lab-user/scripts/ocp/auth/kubeconfig

export KUBECONFIG=/Users/supakorn.t/ProjectCode/kubevirt-thesis/kubeconfig

docker build -t quay.io/linxianer12/proxy-openshift-kubevirt-lab:0.0.1


บั้ค NMState ทำ Network หลุด
https://bugzilla.redhat.com/show_bug.cgi?id=2037411


ทดสอบบน Openshift 4.7 ด้วย RHEL 8.3 สามารถโหลดได้
ลง 
```
pip3 gdown

gdown https://drive.google.com/uc?id=1L7yh-lSS-fMLoF_VaVcMh7oW1ZF0MFnV

gdown https://drive.google.com/uc?id=11hL6vi57S7poek7PTtR53ZTMBEANQPmz

gdown https://drive.google.com/uc?id=1kQjbCmCdlxi4gI5RIT3ugHkq0uOSq-N-
```


```
NAME                       STATUS   ROLES    AGE    VERSION
master-0.ocp.example.com   Ready    master   416d   v1.20.0+5fbfd19
master-1.ocp.example.com   Ready    master   416d   v1.20.0+5fbfd19
master-2.ocp.example.com   Ready    master   416d   v1.20.0+5fbfd19
worker-0.ocp.example.com   Ready    worker   416d   v1.20.0+5fbfd19
worker-1.ocp.example.com   Ready    worker   416d   v1.20.0+5fbfd19
worker-2.ocp.example.com   Ready    worker   416d   v1.20.0+5fbfd19
[lab-user@provision Downloads]$ oc get nncp
NAME        STATUS
brse-ens5   SuccessfullyConfigured
[lab-user@provision Downloads]$ oc get nnce
NAME                                 STATUS
master-0.ocp.example.com.brse-ens5   NodeSelectorNotMatching
master-1.ocp.example.com.brse-ens5   NodeSelectorNotMatching
master-2.ocp.example.com.brse-ens5   NodeSelectorNotMatching
worker-0.ocp.example.com.brse-ens5   SuccessfullyConfigured
worker-1.ocp.example.com.brse-ens5   SuccessfullyConfigured
worker-2.ocp.example.com.brse-ens5   SuccessfullyConfigured
[lab-user@provision Downloads]$ uname -a
Linux provision 4.18.0-240.15.1.el8_3.x86_64 #1 SMP Wed Feb 3 03:12:15 EST 2021 x86_64 x86_64 x86_64 GNU/Linux
[lab-user@provision Downloads]$ cat /etc/os-release 
NAME="Red Hat Enterprise Linux"
VERSION="8.3 (Ootpa)"
ID="rhel"
ID_LIKE="fedora"
VERSION_ID="8.3"
PLATFORM_ID="platform:el8"
PRETTY_NAME="Red Hat Enterprise Linux 8.3 (Ootpa)"
ANSI_COLOR="0;31"
CPE_NAME="cpe:/o:redhat:enterprise_linux:8.3:GA"
HOME_URL="https://www.redhat.com/"
BUG_REPORT_URL="https://bugzilla.redhat.com/"

REDHAT_BUGZILLA_PRODUCT="Red Hat Enterprise Linux 8"
REDHAT_BUGZILLA_PRODUCT_VERSION=8.3
REDHAT_SUPPORT_PRODUCT="Red Hat Enterp
```