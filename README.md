================================================================================
Instructions:

Install Ansible
    YUM http://docs.ansible.com/ansible/latest/intro_installation.html#latest-release-via-yum
    APT http://docs.ansible.com/ansible/latest/intro_installation.html#latest-releases-via-apt-ubuntu

Create a security key on AWS
    # Check https://us-west-2.console.aws.amazon.com/ec2/v2/home?region=us-west-2#KeyPairs:sort=keyName

Download the pem file to your ~/.ssh/
    chmod 400 ~/.ssh/<your_key>.pem
    ssh-add ~/.ssh/<your_key>.pem


Export necessary environment variables.

    export ANSIBLE_HOST_KEY_CHECKING=False
    export AWS_ACCESS_KEY_ID='XXXXXXXXXXXXXXXXXXXX'
    export AWS_SECRET_ACCESS_KEY='xxxxxxxxxxxxxxxxx/xxxxxxxxxxxxxxxxxxxxxx'
    
Get the code from github
    cd ~
    git clone https://github.com/gnumoreno/NdbenchAWS.git

Clone the ndbench repository from Moreno ## Enabling discovery through config file
    cd ~
    git clone https://github.com/gnumoreno/ndbench.git
    
Download ec2.py and ec2.ini
    cd ~/NdbenchAWS
    curl -O https://raw.githubusercontent.com/ansible/ansible/devel/contrib/inventory/ec2.py
    curl -O https://raw.githubusercontent.com/ansible/ansible/stable-1.9/plugins/inventory/ec2.ini
    # Test it
    ./ec2.py --list

Provision your Scylla cluster
    # https://www.scylladb.com/download/amazon/
    Add the following tag key/value on the last step:
          Cluster: Scylla 
    
Provision your instances on AWS 

    Ubuntu 16.04
    Recommended instance: m5.4xlarge
    Number of instances: (whatever is needed for your test, start with 3 if you don't know)
    Storage: General Purpose SSD (GP2) 50GB
    Add the following tag key/value:
            Cluster: Ndbench                              

Setup the application.properties file with your configs - very important to setup the scylla node (ndbench.config.cass.host)
    cd ~/NdbenchAWS
    vi application.properties

Run the automation
    cd ~/NdbenchAWS
    ansible-playbook -i ec2.py -u ubuntu ndbench.yml
    ansible-playbook -i ec2.py -u ubuntu ndbench_start.yml
    # It will display a list of nodes at the end. Pick one and open in the browser
    
    
Useful commands
    # Check Tomcat logs
    ansible -i ec2.py "tag_Cluster_Ndbench" -u ubuntu -m command -a "tail /var/log/tomcat8/catalina.out"
    
    # Restart all tomcat nodes
    ansible -i ec2.py "tag_Cluster_Ndbench" -u ubuntu -b -m command -a "systemctl restart tomcat8"
    
    # Bootstrap ansible on the scylla cluster by installing python2 and simplejson
    # http://docs.ansible.com/ansible/latest/intro_installation.html#managed-node-requirements
    ansible -i ec2.py "tag_Cluster_Scylla" -u centos --sudo -m raw -a "yum install -y python2 python-simplejson"

    # Check your node
    ansible -i ec2.py "tag_Cluster_Scylla" -u centos -m command -a "nodetool status"
    
    # Check stats for your table
    ansible -i ec2.py "tag_Cluster_Scylla" -u centos -m command -a "cqlsh -e 'desc schema;'"
    
    # Check schema
    ansible -i ec2.py "tag_Cluster_Scylla" -u centos -m command -a "nodetool tablestats ndbench.scylla"
    
    # Check Schema -- Does not work
    ansible -i ec2.py "tag_Cluster_Scylla" -u centos -m command -a "cqlsh -e 'select * from ndbench.scylla where key = \'T2461142\'';"
    
    # Start major compaction on all nodes
    ansible -i ec2.py "tag_Cluster_Scylla" -u centos -m command -a "nodetool compact"
    
    # Check all the nodes public hostnames optionally using awscli
    sudo apt install python-pip -y
    sudo pip install awscli
    aws ec2 describe-instances --filters "Name=tag:Cluster,Values=Ndbench" --query 'Reservations[*].Instances[*].PublicDnsName'
    aws ec2 describe-instances --filters "Name=tag:Cluster,Values=Scylla" --query 'Reservations[*].Instances[*].PublicDnsName'
    
================================================================================    
