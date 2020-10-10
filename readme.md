# Kubernetes cluster on AWS EC2 instances (Amazon Linux 2 AMI)

Do you want to train with k8s on AWS EC2? 
I provided easy steps to install k8s cluster there.
- You can use single-node cluster node, so you need only one EC2 machine for it. I provide manual for 2 EC2 instances.
- I have chosen Amazon Linux 2 AMI machine type as the most performance.

## Requirements

- Add EC2 instance type Amazon Linux 2 AMI with security group, where you specify inbound rules (protocol, port range, source)
- Download key.pem from your EC2 instance(s) to ~/.ssh/aws_key.pem
- You can set source everywhere 0.0.0.0/0
- Master node ports and protocols:
    * tcp: 22 (for ssh), 53 (for dns pods), 443 (for https your pods), 2379 - 2380, 6443 (for worker connection), 9153, 10248, 10250 - 10252
    * upd: 53 (for dns pods)
- Worker node ports and protocols:
    * tcp: 22, 443 (for https your pods), 30000 - 32767, 10250
- Install ansible
- Configure ~/.ansible/hosts file:
```
[aws]
192.168.0.1 ansible_ssh_private_key_file=~/.ssh/aws_key.pem
192.168.0.2 ansible_ssh_private_key_file=~/.ssh/aws_key.pem

[aws:vars]
ansible_user=ec2-user

[aws1]
192.168.0.1
```
Where 
- 192.168.0.1 and 192.168.0.2 are external ips;
- aws and aws1 are names for ansible playbooks;
- ec2-user is your EC2 machine user.

## Let's init the cluster

#### You need to clone repo:
```
git clone https://github.com/msergei/k8s-cluster-aws-ec2.git && cd k8s-cluster-aws-ec2
```

#### Prepare all aws nodes:
```
ansible-playbook prepare_nodes.yml
```

#### Init master node:
```
 ansible-playbook master_init.yml
```
After you will need to copy 'kubeadm join' with 'token' and 'discovery-token-ca-cert-hash sha256' from console print.
If you forget to do it, you need to connect to your aws1 host and execute:
```
kubeadm token create --print-join-command
```

#### Add more workers
Connect to other host(s) and execute kubeadm join command from previous step, command example:
```
sudo kubeadm join 192.168.0.1:6443 --token huihdsf.87y8h68f7fv5 --discovery-token-ca-cert-hash sha256:iupg7t869gvg67f79gitd7diyvcyi5dycvyi5di
```
