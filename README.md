# NdBench AWS for Scylla

**This is an ansible automation to deploy NdBench on multiple nodes in AWS**

## Instructions

### Install Ansible
 * [YUM](http://docs.ansible.com/ansible/latest/intro_installation.html#latest-release-via-yum)
 * [APT](http://docs.ansible.com/ansible/latest/intro_installation.html#latest-releases-via-apt-ubuntu)

### Create a security key on AWS
 * [Where's my secrect access key?](https://aws.amazon.com/blogs/security/wheres-my-secret-access-key/)

### Download the pem file to your ~/.ssh/

    chmod 400 ~/.ssh/<your_key>.pem
    ssh-add ~/.ssh/<your_key>.pem


### Export necessary environment variables.

    export ANSIBLE_HOST_KEY_CHECKING=False
    export AWS_ACCESS_KEY_ID='XXXXXXXXXXXXXXXXXXXX'
    export AWS_SECRET_ACCESS_KEY='xxxxxxxxxxxxxxxxx/xxxxxxxxxxxxxxxxxxxxxx'
    
### Get the code from github

    cd ~
    git clone https://github.com/gnumoreno/NdbenchAWS.git

### Clone the ndbench repository

    cd ~
    git clone https://github.com/gnumoreno/ndbench.git
    
### Download ec2.py and ec2.ini

    cd ~/NdbenchAWS/ndbench
    curl -O https://raw.githubusercontent.com/ansible/ansible/devel/contrib/inventory/ec2.py
    curl -O https://raw.githubusercontent.com/ansible/ansible/stable-1.9/plugins/inventory/ec2.ini
    chmod +x ec2.py
    ./ec2.py --list 

### Provision your Scylla cluster

[How to provision a Scylla cluster in AWS](https://www.scylladb.com/download/amazon/)
 * To make your life easier, add the following tag key/value on step 5:
 
        Cluster: Scylla  
    
### Provision your instances on AWS 

 * Ubuntu 16.04
 * Choose instance
 * Number of instances: (whatever is needed for your test, start with 1 if you don't know)
 * Storage: General Purpose SSD (GP2) 50GB
 * Add the following tag key/value to make your life easier:
 
       Cluster: Ndbench                              

### Change the application.properties file with your configs

 * Do not forget your seed node ip (ndbench.config.cass.host)
 

    cd ~/NdbenchAWS/ndbench
    
    vi application.properties

### Run the automation

    cd ~/NdbenchAWS/ndbench
    ansible-playbook -i ec2.py -u ubuntu ndbench.yml
    ansible-playbook -i ec2.py -u ubuntu ndbench_start.yml
 * It will display a list of nodes at the end. Pick one and open in the browser
    
    
### Useful commands

 * Check Tomcat logs
 
   ansible -i ec2.py "tag_Cluster_Ndbench" -u ubuntu -m command -a "tail /var/log/tomcat8/catalina.out"
 
 * Restart all tomcat nodes
 
   ansible -i ec2.py "tag_Cluster_Ndbench" -u ubuntu -b -m command -a "systemctl restart tomcat8"
    
 * [Bootstrap ansible on the scylla cluster](http://docs.ansible.com/ansible/latest/intro_installation.html#managed-node-requirements)
 
   ansible -i ec2.py "tag_Cluster_Scylla" -u centos --sudo -m raw -a "yum install -y python2 python-simplejson"

 * Check your node
 
   ansible -i ec2.py "tag_Cluster_Scylla" -u centos -m command -a "nodetool status"
    
 * Check stats for your table
 
   ansible -i ec2.py "tag_Cluster_Scylla" -u centos -m command -a "cqlsh -e 'desc schema;'"
    
 * Check schema
 
   ansible -i ec2.py "tag_Cluster_Scylla" -u centos -m command -a "nodetool tablestats ndbench.scylla"
    
 * Check Schema
 
   ansible -i ec2.py "tag_Cluster_Scylla" -u centos -m shell -a "cqlsh -e 'DESC SCHEMA;'"
    
 * Start major compaction on all nodes
 
   ansible -i ec2.py "tag_Cluster_Scylla" -u centos -m command -a "nodetool compact"
    
 * Check all the nodes public hostnames optionally using awscli
 
   sudo apt install python-pip -y
   sudo pip install awscli
   aws ec2 describe-instances --filters "Name=tag:Cluster,Values=Ndbench" --query 'Reservations[*].Instances[*].PublicDnsName'
   aws ec2 describe-instances --filters "Name=tag:Cluster,Values=Scylla" --query 'Reservations[*].Instances[*].PublicDnsName'
