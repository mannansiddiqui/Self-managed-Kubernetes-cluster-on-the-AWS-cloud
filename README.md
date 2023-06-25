# Self-managed Kubernetes cluster on the AWS cloud

#### Description:

- Launch three EC2 instances.
- Configure the first instance as a master node and the other two instances as slave nodes.
- Deploy a WordPress website using the deployment kind.
- Connect an RDS instance as the database for the WordPress site.
- Migrate the data to the new database.

#### Step-1: Launch three EC2 instances.

Firstly, log in to the AWS management console.

![Img-1](https://user-images.githubusercontent.com/74168188/178555843-f062573f-166c-4b06-b947-d2d11da46507.png)

After login, navigate to the search bar, type EC2, and select EC2

![Img-2](https://user-images.githubusercontent.com/74168188/178555883-5e169bfd-d205-4b99-9992-8dca7e957ff2.png)

Now, the EC2 dashboard will appear. Click Instances

![Img-3](https://user-images.githubusercontent.com/74168188/178555921-6213742a-726c-4532-923e-3edc4c4d1413.png)

Click the Launch instances button.

![Img-4](https://user-images.githubusercontent.com/74168188/178555939-529e74f0-6ece-43dc-aa1c-98d6b7f2deda.png)

To launch an EC2 instances, a few details are required, i.e., number of instances, instance name, OS image (AMI), instance type, etc.

![5](https://user-images.githubusercontent.com/74168188/184777042-222e173c-7249-4bae-bd7a-ecdf9763e52d.png)

Select a key pair to attach with the instance to log in with that key. If you do not have key pair, then create a new key pair by clicking Create a new key pair. Moreover, according to usage, download key pair in .pem or .ppk format.
  
![6](https://user-images.githubusercontent.com/74168188/184779694-2ccccb1e-3a81-473e-a048-c1b821a3ba81.png)

Use by default network settings

![7](https://user-images.githubusercontent.com/74168188/184779392-a1f1b432-58c2-4a89-a709-b4ea0bff8168.png)

We are good to go to launch an instances. Click the Launch instance button. If everything is correctly set up, you will find a success message on the screen.

![8](https://user-images.githubusercontent.com/74168188/184798593-a26a1bf9-4fc4-4fa1-9ffb-b4b0cfd21645.png)

Click on the instance id to navigate to the EC2 dashboard and edit their names.

![9](https://user-images.githubusercontent.com/74168188/184804077-b27ccb53-b74b-4b84-b933-0d5a1368b526.png)

#### Step-2: Configure the first instance as a master node and the other two instances as slave nodes.

Let's configure master node

Login to master node via ssh:

![10](https://user-images.githubusercontent.com/74168188/184804512-c9668ecc-bac9-4fd5-b990-5309b69297ce.png)

Install docker as Kubernetes service like kube-apiserver, kube-controller-manager, kube-scheduler, etc will run in containers.

```
sudo yum install docker -y
```

![11](https://user-images.githubusercontent.com/74168188/184805347-69492a98-f8d1-4710-901a-5f42ac022124.png)
![12](https://user-images.githubusercontent.com/74168188/184805355-0d4be293-4ef8-46cb-bcb3-738a1ffc0777.png)

After installation done enable docker service so that after os reboot docker service will be in running state.

```
sudo systemctl enable docker --now
```

![13](https://user-images.githubusercontent.com/74168188/184805626-89637ce9-58ac-489e-b543-207d6018a94b.png)

Now we have to configure yum so that yum can able to download and install kubeadm and kubelet.

For this create a repo file with any name inside **/etc/yum.repos.d/** using ```sudo vim /etc/yum.repos.d/kubernetes.repo```

![14](https://user-images.githubusercontent.com/74168188/184807368-1fd74a42-f94f-4e90-b0ee-2f4866aa2b51.png)

Write below content inside file:
```
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-\$basearch
enabled=1
gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
```
![15](https://user-images.githubusercontent.com/74168188/184808758-5cdf6eac-6a73-4665-a628-950f14502837.png)

Now, press Esc key to exit from insert mode then type :wq!

Install kubeadm that will create a k8s cluster. Also, kubectl and kubelet will also be installed as dependency
```
sudo yum install kubeadm -y
```
![16](https://user-images.githubusercontent.com/74168188/184811383-b7990e83-05f8-4ef9-afd8-2c63d8d98b25.png)
![17](https://user-images.githubusercontent.com/74168188/184811393-f41c9a69-2100-401a-b720-a7f9fe828205.png)
![18](https://user-images.githubusercontent.com/74168188/184811405-150b1b66-3344-4e96-8df9-5e01af7a9a5d.png)
![19](https://user-images.githubusercontent.com/74168188/184811416-5ce838b3-42ab-4d40-8339-49be5ce6d388.png)

Now, download all docker images required for each service like kube-apiserver, kube-controller-manager, kube-scheduler, etc using:
```
sudo kubeadm config images pull
```
![20](https://user-images.githubusercontent.com/74168188/184812988-8da6ec78-bfe6-4f69-aa3e-f42bc8929a4b.png)

Enable kubelet using ```sudo systemctl enable kubelet --now```

![21](https://user-images.githubusercontent.com/74168188/184814761-6fcd50dc-0beb-4cac-b470-00fc42454c5f.png)

Now, install **iproute-tc** using ```sudo yum install iproute-tc -y```

![22](https://user-images.githubusercontent.com/74168188/184815177-1220f851-89c4-4e98-a0ef-4389938edc55.png)

Let's create kubernetes cluster using ```sudo kubeadm init --pod-network-cidr=10.244.0.0/16 --ignore-preflight-errors=NumCPU --ignore-preflight-errors=Mem```

Here in my case, i used t2.micro instance type which provides 1 CPU and 1 GB RAM but At least 2 CPU and 2 GB RAM is recommended by Kubernetes. So to work with t2.micro instance type we passed **--ignore-preflight-errors=NumCPU --ignore-preflight-errors=Mem**

![23](https://user-images.githubusercontent.com/74168188/184825791-e0770197-442a-47f2-a4d9-2bb9ccc4d4f9.png)
![24](https://user-images.githubusercontent.com/74168188/184825805-beaaa6d1-0b7d-4a3a-8049-c931a6525260.png)

To access K8S cluster we have two approaches:

### Approach-1: Accessing K8S cluster from master node

For this, run below commands from master node:
```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
![25](https://user-images.githubusercontent.com/74168188/184832655-02c8f444-bf14-4cde-98ac-89d8cc7bcb49.png)

To check kubectl working fine run ```kubectl get pods```

![26](https://user-images.githubusercontent.com/74168188/184833154-19c4d180-c240-4f3f-8b4f-9c56de7a7a61.png)

### Approach-2: Accessing K8S cluster from a different node

For this, run below commands from that node from which you want to access K8S cluster.

```
mkdir -p $HOME/.kube
sudo vim $HOME/.kube/config
```
Copy the /etc/kubernetes/admin.conf file from master node and paste it inside $HOME/.kube/config file of a node from which you want to access K8S cluster.

![27](https://user-images.githubusercontent.com/74168188/184837093-6971f056-90c7-4df2-ab83-e1f1b149c479.png)

![28](https://user-images.githubusercontent.com/74168188/184837253-f2bcb780-b736-4b45-8dac-494691435f54.png)
![29](https://user-images.githubusercontent.com/74168188/184837612-346f77b9-37de-4073-aeb1-d0722c01e36b.png)
 
 Selected IP is changed by publicIP of master node then press Esc key to exit from insert mode and press :wq to save and quit.
 
 Now try to run ```kubectl --insecure-skip-tls-verify get pods```
 
 ![30](https://user-images.githubusercontent.com/74168188/184838244-7a6bd261-2562-4ba2-9637-c9eec33e6474.png)
 
 Lastly, run below command to provide connectivity between pods
 ```
 kubectl apply -f https://raw.githubusercontent.com/flannel-io/flannel/master/Documentation/kube-flannel.yml
```

![31](https://user-images.githubusercontent.com/74168188/184840090-a878265e-4fdd-4973-9ae9-81f60211d6bd.png)

Now, time to configure slave/worker node

First login to slave/worker node via ssh:

![32](https://user-images.githubusercontent.com/74168188/184840916-65312975-35d1-4119-8183-6bca092bd74d.png)

Install docker as Kubernetes requires a container runtime and will use docker

```
sudo yum install docker -y
```

![33](https://user-images.githubusercontent.com/74168188/184841699-2695ebb9-e74f-42c9-aa28-9a5e14633816.png)
![34](https://user-images.githubusercontent.com/74168188/184841720-146478cd-2f36-4163-b67a-1c2f2077f5df.png)

After installation done enable docker service so that after os reboot docker service will be in running state.
```
sudo systemctl enable docker --now
```
![35](https://user-images.githubusercontent.com/74168188/184842263-b2edd22b-9417-4e2e-9181-0617618d1b5d.png)

Now we have to configure yum so that yum can able to download and install kubeadm and kubelet.

For this create a repo file with any name inside **/etc/yum.repos.d/** using ```sudo vim /etc/yum.repos.d/kubernetes.repo```

![36](https://user-images.githubusercontent.com/74168188/184842863-ffd2b1a1-fe85-46be-885d-5ea52906aba5.png)

Write below content inside file:
```
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-\$basearch
enabled=1
gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
```
![37](https://user-images.githubusercontent.com/74168188/184843276-96f2c87c-8ca5-4240-b337-9233d1af46b0.png)

Now, press Esc key to exit from insert mode then type :wq!

Install kubeadm that will join slave to master node. Also, kubectl and kubelet will also be installed as dependency
```
sudo yum install kubeadm kubelet -y
```
![38](https://user-images.githubusercontent.com/74168188/184844418-06184952-ae02-434c-b81d-bd3b722c5127.png)
![39](https://user-images.githubusercontent.com/74168188/184844435-624eec7d-2958-459b-a096-e7cb854bccaf.png)
![40](https://user-images.githubusercontent.com/74168188/184844444-013c6f3a-c00f-4ec9-9c25-0c2910ffc890.png)
![41](https://user-images.githubusercontent.com/74168188/184844460-37f57756-f6ab-4846-9ec7-0359ba569fdf.png)

Enable kubelet using ```sudo systemctl enable kubelet --now```

![42](https://user-images.githubusercontent.com/74168188/184844748-659c9856-572f-49e3-9137-f193f934e589.png)

Now, install **iproute-tc** using ```sudo yum install iproute-tc -y```

![43](https://user-images.githubusercontent.com/74168188/184844958-549483cc-4e36-417c-bbb8-1243a82782e8.png)

Lastly, join slave to master node using ```sudo kubeadm join 172.31.39.152:6443 --token nlk5qe.7tj1wedsmxdyycio --discovery-token-ca-cert-hash sha256:38353362ba17b9079be482d2c118880b5ae35eb9f4deed6af0a378ab8ab1fda3```

![44](https://user-images.githubusercontent.com/74168188/184845441-40b3a3ef-0eeb-4949-928c-a5b271037c46.png)

Similarly, configure another slave node

Now run ```kubectl --insecure-skip-tls-verify get nodes -o wide``` from where you want to manage K8S cluster or ```kubectl get pods -o wide``` from master (in my case as we setup this in Approach-1)

From master node:
![45](https://user-images.githubusercontent.com/74168188/184846221-65506fde-3c2e-4956-974f-cab786cab9df.png)

From different node:
![46](https://user-images.githubusercontent.com/74168188/184846541-48a006b1-b48d-4570-be9d-4fb82cef97e3.png)

#### Step-3: Deploy a WordPress website using the deployment kind.

Before creating deployment we need to create Database because Wordpress requires WORDPRESS_DB_HOST, WORDPRESS_DB_USER, WORDPRESS_DB_PASSWORD, WORDPRESS_DB_NAME else Wordpress container will exit again and again.

So, First create a RDS(MySQL) DB from AWS console

![47](https://user-images.githubusercontent.com/74168188/184855550-f856ce44-dc3f-4697-aecb-bd94de0a58e0.png)

Click **Create database**

![48](https://user-images.githubusercontent.com/74168188/184857456-844dcb17-e8f1-450e-94cc-cefb6c1513a0.png)
![49](https://user-images.githubusercontent.com/74168188/184857467-fe07c2ae-a94e-48d6-b89f-412d8f14a5a3.png)
![50](https://user-images.githubusercontent.com/74168188/184857481-1ca2cfc3-28fe-42e2-abf5-a598e53e4022.png)
![51](https://user-images.githubusercontent.com/74168188/184857496-f926cd4a-cf86-4500-bc30-d190f3b08d21.png)
![52](https://user-images.githubusercontent.com/74168188/184857511-19ef9be2-b2c7-480d-829f-bf9a44ddf7bc.png)

Click **Create database**

Now, time to create deployment using YAML file

![53](https://user-images.githubusercontent.com/74168188/184859776-0dfb7a21-d31b-4b55-8ec9-1071e8802424.png)

Write below content inside YAML file
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: wordpress-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: wordpress
  template:
    metadata:
      labels:
        app: wordpress
    spec:
      containers:
      - name: wordpress
        image: wordpress:latest
        ports:
        - containerPort: 80
        env:
        - name: WORDPRESS_DB_HOST
          value: database-2.cexbgrjuvozl.ap-south-1.rds.amazonaws.com
        - name: WORDPRESS_DB_USER
          value: admin
        - name: WORDPRESS_DB_PASSWORD
          value: Mannan123#
        - name: WORDPRESS_DB_NAME
          value: mydb
```

![54](https://user-images.githubusercontent.com/74168188/184859964-7a5b7494-0541-42f1-81d9-9473e232443d.png)

In env, WORDPRESS_DB_HOST is the endpoint of database (get this from AWS console)

![55](https://user-images.githubusercontent.com/74168188/184859116-00076653-138c-4f1e-98a6-9347cb22a362.png)

Now, create deployment using ```kubectl create -f <FILENAME.yml>```

![53](https://user-images.githubusercontent.com/74168188/184861501-7ac27e1e-6c10-4ce4-97fa-4702027d827d.png)

To check pods is running successfully or not run ```kubectl get pods -o wide```

![57](https://user-images.githubusercontent.com/74168188/184860988-69212b0c-9f28-498e-a7fd-43b2724a4879.png)

To describe pods run ```kubectl describe pods <POD_NAME>```

![58](https://user-images.githubusercontent.com/74168188/184861102-e989e4d7-2215-436f-9214-97295db2c45b.png)
![59](https://user-images.githubusercontent.com/74168188/184861118-5005a08d-0591-4c34-8eca-3fa73937f135.png)

Now, to expose this wordpress app create a service for this create service.yml file

![60](https://user-images.githubusercontent.com/74168188/184861743-882a7b49-6f2f-40d7-ac1c-85cc27a18825.png)
![61](https://user-images.githubusercontent.com/74168188/184862339-236a002f-f1d6-4c07-94d7-71fad6633f16.png)

again create service using ```kubectl create -f <FILENAME.yml>```

![62](https://user-images.githubusercontent.com/74168188/184862584-a609015f-acc4-4cd3-abbd-488dc63248c2.png)

To check service is running or not run ```kubectl get service -o wide```

![63](https://user-images.githubusercontent.com/74168188/184863446-1270ff51-4398-4db9-b92c-f167064db89d.png)

Time to hit **InstanceIP:30838**

![64](https://user-images.githubusercontent.com/74168188/184876055-1d70c434-9852-40f5-b87b-7441f42ff79b.png)

We get the desired page. Now, configure wordpress and create posts.

![65](https://user-images.githubusercontent.com/74168188/184883182-8c999fe0-60a0-4650-b7b3-db2cb7f09bd4.png)
![66](https://user-images.githubusercontent.com/74168188/184883417-c1fd2501-1cb7-48e9-a133-e56283d27019.png)
![67](https://user-images.githubusercontent.com/74168188/184883492-a5dbb719-d1fb-4790-8aaa-fb1b6e1b1e00.png)
![68](https://user-images.githubusercontent.com/74168188/184883648-12a9c6a9-1631-4d19-84d2-393c22176e05.png)

Now, create page

![69](https://user-images.githubusercontent.com/74168188/184883953-1c5af7ca-8912-4a14-9f44-a0c32218eb75.png)
![70](https://user-images.githubusercontent.com/74168188/184884136-23454ed4-42f7-4f65-aedb-8f366265481f.png)
 
 Then click **Publish** and get the url and try to hit that
 
 ![71](https://user-images.githubusercontent.com/74168188/184884339-f9d66f1a-a135-44f4-898e-77f422792424.png)
 
 #### Step-4: Connect an RDS instance as the database for the WordPress site.
 
 Already done in step-3
 
 #### Step-5: Migrate the data to the new database.
 
 Now take a dump of mysql db from any node using ```mysqldump -h <ENDPOINT> -u <USERNAME> -p <DB_NAME>```
 
 ![72](https://user-images.githubusercontent.com/74168188/184885660-455fa729-d9d3-4a70-90f7-297c5eb34cd4.png)
 
 Now delete db using ```drop database <DB_NAME>```
 
 ![73](https://user-images.githubusercontent.com/74168188/184887976-ebe2203f-f054-49fe-8122-25f9a7f2aa7e.png)

Now try to hit **InstanceIP:30838**

![74](https://user-images.githubusercontent.com/74168188/184888280-16f08d01-2bbd-41c2-a8a9-7a95cdf773b6.png)

Wordpress is not able to access db as db is deleted. So, create a new db with same name and put dump inside this db.

![75](https://user-images.githubusercontent.com/74168188/184888818-57a961f4-2cb5-44eb-a4d9-a6456593dc37.png)
![76](https://user-images.githubusercontent.com/74168188/184889121-38085179-f209-499c-a8ed-2c8b79699b62.png)

Now again try to hit **InstanceIP:30838**

![77](https://user-images.githubusercontent.com/74168188/184889475-30c6cc32-a1d1-4866-8f2f-ac05097b0caa.png)

Now, wordpress again able to access db and older data also shown.

### Thank you...
