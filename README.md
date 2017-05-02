# JupyterMxNet

## Configuring Jupyter On Amazon Deep Learning AMI

Using this tutorial you will be able to install and run Jupyter using https on a Amazon Deep Learning AMI. AWS now also offers a Ubuntu version of the Deep Learning AMI, most of the following steps are compatible with both versions of the Deep Learning AMI.
# Launch an EC2 Instance With The Amazon Deep Learning AMI

    Launch an instance using the Amazon Deep Learning AMI using the AWS console: https://aws.amazon.com/marketplace/pp/B01M0AXXQB

    Navigate to the AWS Management Console:

    Or Launch the Amazon Deep Learning AMI from a CloudFormation Script: https://github.com/awslabs/deeplearning-cfn/blob/master/cfn-template/deeplearning.template

    Choose an instance type sutiable for deep learning. Common choices are GPU backed instances such as g2 and p2 instance types, both of which have dedicated GPU's.



from IPython.display import Image
from IPython.core.display import HTML 
Image(url="https://s3-us-west-2.amazonaws.com/mxnetjupytersetup/Untitled.jpg")


Next, we are going to create a security group. This security group contains some specific rules to allow you to access the Jupyter server from your browser with HTTP/HTTPS. A sample ruleset used for this tutorial is below. However, for an enterprise grade enviroment, you should probably secure the source ip's to only known and trusted ips.


from IPython.display import HTML, display

 data = [["<b>Type</b>","<b>Protocol</b>","<b>Port Range</b>", "<b>Source</b>"],
         ["HTTPS","TCP",443,"0.0.0.0/0"],
         ["HTTP","TCP",80,"0.0.0.0/0"],
         ["SSH","TCP",22,"0.0.0.0/0"],
         ["Custom TCP Rule", "TCP", 8888, "0.0.0.0/0"]
         ]

 display(HTML(
    '<table><tr>{}</tr></table>'.format(
        '</tr><tr>'.join(
            '<td>{}</td>'.format('</td><td>'.join(str(_) for _ in row)) for row in data)
        )
 ))

Type 	Protocol 	Port Range 	Source
HTTPS 	TCP 	443 	0.0.0.0/0
HTTP 	TCP 	80 	0.0.0.0/0
SSH 	TCP 	22 	0.0.0.0/0
Custom TCP Rule 	TCP 	8888 	0.0.0.0/0

You can add these rules through the AWS console while setting up the instance as follows:


from IPython.display import Image
from IPython.core.display import HTML 
Image(url="https://s3-us-west-2.amazonaws.com/mxnetjupytersetup/SecurityGroup.jpg")


Optionally, you can create the security group using the AWS CLI beforehand and attach it to the instance. Keep in mind you need to update these commands to suit your own needs. In this tutorial we are using the latest Deep Learning AMI 2.0 (ami-dfb13ebf) in the us-west-2 region. The CLI commands for this are as follows:

```
#Setup the AWS CLI on your instance
aws configure
#Enter your access key and secret key 

# Create Security Group
aws ec2 create-security-group --group-name JupyterSG --description "JupyterSG"
#Add SG ruleset
aws ec2 authorize-security-group-ingress --group-name JupyterSG --protocol tcp --port 8888 --cidr 0.0.0.0/0
aws ec2 authorize-security-group-ingress --group-name JupyterSG --protocol tcp --port 22 --cidr 0.0.0.0/0
aws ec2 authorize-security-group-ingress --group-name JupyterSG --protocol tcp --port 443 --cidr 0.0.0.0/0
aws ec2 authorize-security-group-ingress --group-name JupyterSG --protocol tcp --port 80 --cidr 0.0.0.0/0

#Launch AWS instance
aws ec2 run-instances --image-id ami-dfb13ebf --count 1 --instance-type p2.xlarge --key-name <YOUR_KEY_NAME> --security-groups Jupyter
```


# Setup The Environment On Your New Instance

SSH into the instance using the instances DNS name

```
ssh -i <PATH_TO_PEM> ec2-user@ec2-xx-xxx-xx-xxx.eu-west-1.compute.amazonaws.com
```

Use the AMI's pre installed Anaconda3 environment managment tool to activate a default environment named "root" with the following command:
```

source src/anaconda3/bin/activate root
```

Verify that your environment is using the correct binaries with the following commands:
```

which python
~/src/anaconda3/bin/python
which  jupyter
~/src/anaconda3/bin/jupyter
```

If they are not pointing to the correct binaries change the PATH definition to the following:
```

PATH=$HOME/src/anaconda3/bin:$PATH:$HOME/.local/bin:$HOME/bin
export PATH
```

# Configure The Jupyter Server To Use HTTPS

Now that you have your AMI launched and your environment setup correctly we can begin to generate the configuration for Jupyter. In this tutorial, we will setup the configuration using HTTPS. If you want to use regular HTTP you can skip the following steps.
```

jupyter notebook --generate-config
key=$(python -c "from notebook.auth import passwd; print(passwd())")
```

Create a directory for your server certificates then use OpenSSL to self-sign a certificate
```

cd ~
mkdir certs
cd certs
certdir=$(pwd)
openssl req -x509 -nodes -days 365 -newkey rsa:1024 -keyout JupyterKey.key -out JupyterCert.pem
```
```
cd ~
sed -i "1 a\
c = get_config()\\
c.NotebookApp.certfile = u'$certdir/JupyterCert.pem'\\
c.NotebookApp.keyfile = u'$certdir/JupyterKey.key'\\
c.NotebookApp.ip = '*'\\
c.NotebookApp.open_browser = False\\
c.NotebookApp.password = u'$key'\\
c.NotebookApp.port = 8888" .jupyter/jupyter_notebook_config.py
```

# Running The Jupyter Server

Now that Jupyter is configured we can start the server. You can either start the server directly, or use session management to make sure that the server keeps running even after you log out of the instance. The steps for configuring the server to run with the session management tool screen are below:

```
screen -S jupyter
mkdir notebook
cd notebook
jupyter notebook
```

The above steps create a notebook directory and starts the Jupyter server with a default notebook.

You can now access your Jupyter server in your browser with the following URL: https://your-instances-public-ip:8888


from IPython.display import Image
from IPython.core.display import HTML 
Image(url="	https://s3-us-west-2.amazonaws.com/mxnetjupytersetup/JupyterNotebook.jpg")


Now that you have launched the AWS Deep Learning AMI, and configured Jupyter you can run the next tutorials for learning how to use Jupyter, and many other MXNet tutorials. Check out the MXNet GitHub Repo ( https://github.com/dmlc/mxnet-notebooks) for tutorials that make it easy to start using Jupyter for computer vision, natural language processing, and recommendation systems.
