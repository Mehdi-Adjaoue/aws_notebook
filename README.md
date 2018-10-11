## Supervised Learning Setup

### Prerequisites

*Structure (optional, but HIGHLY recommended)*

* Create a folder, name it `aws_utilities` for example, and use it as your workspace.

*AWS Access*

* You have an account with access to AWS console.
* You have enough privileges to access EC2 services.
* (if not already done) Register your accound on [AWS Educate](https://www.awseducate.com/registration).

*For windows users*

* On your favorite browser, open [Putty Download Page](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html).
* Download ssh client : [putty.exe (the SSH and Telnet client)](https://the.earth.li/~sgtatham/putty/latest/w64/putty.exe).
* Download RSA key formatter : [puttygen.exe (a RSA and DSA key generation utility)](https://the.earth.li/~sgtatham/putty/latest/w64/puttygen.exe).
 
 
### 1. AWS EC2 Service


First, launch a distant computation instance (EC2) :


1. Connect to AWS console.
1. Set the region to UE (Paris) : top-right corner of your AWS console homepage.
1. Select [EC2 Service](https://eu-west-3.console.aws.amazon.com/ec2/), and click on **launch instance**.
1. Select `Ubuntu Server 18.04 LTS`, then `t2.micro` and click `Next: Configure Instance Details`.
1. In step **6. Configure security group** : click `Add Rule`, and add the following rule (be sure to **add** a rule, do not overwrite existing rules) :

    ```
    <type==Custom TCP Rule ; Port range == 8888 ; Source==Anywhere>
    ```
    
    
1. Click **launch**.
1. **/!\ Very Important /!\\** create a key pair, name it `SL_key_pair` for example, and download it to your workspace.
1. You should be able to click **Launch instances** by now.


### 2. SSH To Your instance

First, you need to get some information:

1. Go back to your EC2 instances, select the one you've created and click on **Connect**. A modal window will be shown.
1. The link given as an example should resemble the following:

    ```
    ssh -i "SL_key_pair.pem" ubuntu@ec2-35-180-118-68.eu-west-3.compute.amazonaws.com
    ```
    
    
1. You want to retrieve the `username`, the `DNS address`, and the `key_pair`. For this example:

    ```
    username==ubuntu ; DNS address==ec2-35-180-118-68.eu-west-3.compute.amazonaws.com ; key_pair==SL_key_pair.pem
    ```
    
**Your OS is Windows**

Windows does not include a built-in ssh client. You'll use Putty :
1. Open PuttyGen, and load your <key_pair.pem>. If you used the suggested name, you would load `SL_key_pair.pem`. Then save the private key.
1. Open Putty.exe
1. Put your DNS address in the "Host Name (or IP address)" field.
1. Click Connection-> Data, and put your username in the "Auto-login username" field.
1. Click Connection-> SSH -> Auth -> Browse, and select the private key you saved in step 1.
1. Click Open.


**Your OS is Unix based (Mac/Linux)**
1. On your terminal, change directory to your workspace :

    ```
    cd /path/to/your/workspace
    sudo chmod 400 SL_key_pair.pem
    ```
    
    
1. Then ssh using the the ssh link provided by AWS :

    ```
    ssh -i "SL_key_pair.pem" ubuntu@ec2-35-180-118-68.eu-west-3.compute.amazonaws.com
    ```


### Installation and Configuration

By now you should be connected to your distant instance. Next : install python prerequisites.

1. install pip :

    ```
    sudo apt-get update
    sudo apt-get install -y python3-pip
    ```
    
    
1. Using a virtual environment is a best practice. It will be used for this tutorial:

    ```
    sudo apt-get install virtualenv
    ```
    
    
1. And then create a virtual environment named **venv**:

    ```
    virtualenv -p $(which python3) venv
    source venv/bin/activate
    ```
    
    
1. The username will be preceded by `(venv)` on your terminal, this is normal.
1. Install jupyter notebook :

    ```
    pip3 install jupyter
    ```


Now you need to configure jupyter in order to access it publicly.

1. First, setup a password :
    ```
    ipython
    # >> inside ipython :
    from IPython.lib import passwd
    passwd()
    # >> type your password
    # Output >> something like 'sha1:c94aa6ffc3df:851b65eeea53725bfbce6a97bf3c5e7179a54307' 
    ```
    
    
1. Store this password somewhere, you will need it later. 
1. Now close ipython using `exit()` command. 
1. Then, generate a configuration:

    ```
    jupyter notebook --generate-config
    ```
    
1. You will need SSL certificats to be compatible with new policies of browsers. To generate them :

    ```
    mkdir certificats
    cd certificats
    sudo openssl req -x509 -nodes -days 365 -newkey rsa:1024 -keyout certificat.key -out certificat.pem
    sudo chmod 666 certificat.key certificat.pem
    ```
    
1. Complete the configuration of the notebook server :

    ```
    cd ~/.jupyter
    nano jupyter_notebook_config.py
    ```
    
1. On the top of the file, paste the following code (remember to put the password you generated (u'sha1:...') in the correct place) :

    ```
    c = get_config()
    
    # Kernel config
    c.IPKernelApp.pylab = 'inline'
    
    # Notebook config
    c.NotebookApp.certfile = u'/home/ubuntu/certificats/certificat.pem' 
    c.NotebookApp.keyfile = u'/home/ubuntu/certificats/certificat.key'
    c.NotebookApp.allow_remote_access = True
    c.NotebookApp.open_browser = False
    c.NotebookApp.password = <YOUR PASSWORD GOES HERE> # example u'sha1:c94aa6ffc3df:851b65eeea53725bfbce6a97bf3c5e7179a54307'
    c.NotebookApp.port =  8888
    c.NotebookApp.allow_origin = '*'  # allow all origins
    c.NotebookApp.ip = '0.0.0.0'  # listen on all IPs
    ```
    
1. Most of your problems will come from the previous step. Remember, the file you are writing is a `.py`, so pay attention to indentation. You can save : `Ctrl+O`, then quite : `Ctrl+X`.
1. Finally you can launch your notebook:

    ```
    jupyter notebook
    ```
    
1. On your machine, open your browser on the url : `https://<your-dns-address>:8888`. For this example it would be :

    ```
    https://ec2-35-180-118-68.eu-west-3.compute.amazonaws.com:8888/
    ```

### Few Tips:
1. Each time you ssh to the instance, don't forget to activate your virtualenv :

    ```source venv/bin/activate```
    
1. Each time you disconnect from the ssh session, the server will be closed. If you want to keep it running :
    
    ```
    nohup jupyter notebook &
    (press Enter)
    ```
    
1. To install a package, use pip (pip will default to pip3 once you activate your virtualenv):
    
    ```
    pip install <package-name>
    ```
    
If you want to install multiple packages, you can put them in a requirements file and install them using pip:
1. create a requirements file: 

    ```
    touch requirements.txt
    nano requirements.txt
    ```
    
1. Fill in all the dependencies you want to install. `Ctrl+O` to save, `Ctrl+X` to quit :

    ```
    numpy
    scipy
    matplotlib
    pandas
    scikit-learn==0.17.1
    ```

1. And run the following command:

    `pip install -r requirements.txt`
   
### Questions ?
Contact Mehdi Adjaoue
