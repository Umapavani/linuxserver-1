# Udacity-Linux-Server-Configuration
### By Vanitha Vuppalapati

### About
  This is the Udacity project 6 about the Configuring the Linux the server.

### Server Details
Server IP Address 35.154.196.1

port : 2200

Hosted site Url [http://35.154.196.1.xip.io/](http://35.154.196.1.xip.io/)

### Configuring Linux ServerLogin to the grader

# Login to the grader
First open puttygen and then load the lightsail1.pem to genarate private key

It is saved as privatekey1.ppk

Now open putty give the static ip:35.154.196.1 and port:2200

Now go to ssh and in ssh goto to auth load the privatekey1

login as : ubuntu

su - grader

password for grader:vanitha23

# Making .ssh folder
mkdir .ssh

creates .ssh directory

touch .ssh/authorized_keys

create authorized_keys in .ssh folder

nano .ssh/authorized_keys

In git bash ssh-keygen

.ssh file will be created with id_rsa files

now copy the contents of id_rsa.pub file (microsoft office file) and paste it in the grader.

#### Updating all packages
```
sudo apt-get update
sudo apt-get upgrade
```

#### Creating grader User:
  ```
  sudo adduser grader
  ```
  This will add new user
  Password for this user : vanitha23
  ```
  sudo nano /etc/sudoers
  ```
  Below the Root user append the following line
  ```
  grader  ALL=(ALL:ALL) ALL
  ```
  This will grant sudo permission to grader
  #### Creating a ssh key pair for grader
   On your local machine in terminal/command prompt
   ```
   ssh-keygen
   ```
   This will generate public and private ssh keys which is saved to .ssh folder
   
   Then in your virtual machine change to newly created user
   ```
   su - grader
   ```
   Create a new directory .ssh and new file authorized_keys in that directory
   ```
   mkdir .ssh
   sudo nano .ssh/authorized_keys
   ```
   Copy the public key with .pub extension to authorized_keys and save the file
   ```
   chmod 700 .ssh
   chmod 644 .ssh/authorized_keys
   ```
   - 700 will give read write and execute permission to user.
   - 644 prevent other user from writting in to file.
   Then restart ssh server
   ```
   service ssh restart
   ```
   
   Now from your log in to grader with private key generated 
   ```
   ssh -i .ssh/id_rsa grader@35.154.196.1
   ```
  #### Changing the ssh port to 2200
   ```
   sudo nano /etc/ssh/sshd_config
   ```
   Change port 22 to port 2200
    
   Restart the ssh server
   
   ```
   service ssh restart
   ```
   
   >Note: Before Logging using ssh add custom TCP port 2200 under lightsaail firewall in networking tab in lightsail instance console  
   
   Now Login using command like this
   ```
   ssh -i .ssh/id_rsa -p 2200 grader@35.154.196.1
   ```
   
  #### Disabling ssh login as root
  `sudo nano /etc/ssh/sshd_config`
  
  make change `PermitRootLogin no`
  
  #### Configurating  Ufw firewall

  Goto amazon aws lightsail account 

  Goto networking then change firewall as follows
  ```
  Custom        UDP        123
  
  Custom        TCP        2200
  
  Configurating Ufw firewall
    sudo ufw allow 2200/tcp
    sudo ufw allow 80/tcp
    sudo ufw allow 123/udp
    sudo ufw enable
This will allow all required ports and enables the ufw

After that

   sudo ufw status
It will display all allowed ports
  ```
  
  #### Installing Apache2 
  In terminal 
  
  ```sudo apt-get install apache2```
  
  Now mod_wsgi
  
  ```sudo apt-get install python-setuptools libapache2-mod-wsgi```
  
  Enable mod_wsgi
  
  ```sudo a2enmod wsgi ```
  ##### Setting up your flask application to work with apache2
   Creating a flask app
   
   In /var/www directory create a new folder
   `sudo mkdir FlaskApp`
   
   Install git 
   
   `sudo apt-get install git`
   
   move to the FlaskApp `cd FlaskApp`
   
   In that direcory clone your github repository
   
   `sudo git clone 'https://github.com/vuppalapativanitha/myclothingcatalog.git'`
   
   Renaming the repository to FlaskApp
   
   Then rename main.py file to `__init__.py`
   
   Make Following changes in __init__.py
   ```
   PROJECT_ROOT = os.path.realpath(os.path.dirname(__file__))
   json_url = os.path.join(PROJECT_ROOT, 'client_secrets.json')
   CLIENT_ID = json.load(open(json_url))['web']['client_id']
   ```
   Use json_url instead client_secrets.json in script
   
   
  ##### Install and configuring postgresql for project
   Install Postgres `sudo apt-get install postgresql`
   
   login to postgres `sudo su - postgres`
   
   postgres shell `psql`
   
   create user `CREATE USER catalog WITH PASSWORD 'password';`
   
   permit user to createdb `ALTER USER catalog CREATEDB;`
   
   Create a db name  catalog with user catalog `CREATE DATABASE catalog WITH OWNER catalog;`
   
   connect to db `\c catalog`
   
   revoke all permission to public `REVOKE ALL ON SCHEMA public FROM public;`
   
   Give schema permission to user catalog `GRANT ALL ON SCHEMA public TO catalog;`
   
   exit from db and postgres `\q and exit`
   
   Change the database connection in both database_setup.py and `__init__.py` as `engine =       create_engine('postgresql://catalog:password@localhost/catalog')`
   
   Now we are ready with our applicatiom
  #### Configure and Enable a New Virtual Host
   `sudo nano /etc/apache2/sites-available/FlaskApp.conf`
   
   In this add the following code
   ```
   <VirtualHost *:80>
		ServerName 35.154.196.1
		ServerAdmin admin@mywebsite.com
		WSGIScriptAlias / /var/www/FlaskApp/flaskapp.wsgi
		<Directory /var/www/FlaskApp/FlaskApp/>
			Order allow,deny
			Allow from all
		</Directory>
		Alias /static /var/www/FlaskApp/FlaskApp/static
		<Directory /var/www/FlaskApp/FlaskApp/static/>
			Order allow,deny
			Allow from all
		</Directory>
		ErrorLog ${APACHE_LOG_DIR}/error.log
		LogLevel warn
		CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
   ```
   Enable the virtual host 
   `sudo a2ensite FlaskApp`
   
   Disabling the default apache2 page
   `sudo a2dissite 000-default.conf`
   
  #### Create the .wsgi File
    ```
    cd /var/www/FlaskApp
    sudo nano flaskapp.wsgi 
    ```
   Add the following code
   
   ```
   #!/usr/bin/python
    import sys
    import logging
    logging.basicConfig(stream=sys.stderr)
    sys.path.insert(0,"/var/www/FlaskApp/")

    from FlaskApp import app as application
    application.secret_key = 'Add your secret key'
   ```
   save and exit
   
   Deploying flask app with apache2 is referred from [Digital ocean]
   (https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps)
   
   #### Installing require modules
   You can either install all modules on your machine or create a virtual environment for the project and install the modules
   ` pip install flask sqlalchemy psycopg2 requests oauth2client`
   
   #### Setting up Google Oauth2
   
   - Go to [Google Cloud Plateform](https://console.cloud.google.com/).
  
   - Click `APIs & services` on left menu.
   
   - Click `Credentials`.
   
   - Create an OAuth Client ID (under the Credentials tab), and add http://35.154.196.1.xip.io 
     as authorized JavaScript origins.
   
   - Add the below three links as authorized javascript origins
     
     1. http://35.154.196.1.xip.io/

     2. http://35.154.196.1.xip.io/gconnect

     3. http://35.154.196.1.xip.io/callback 

   - Download the corresponding JSON file, open it et copy the contents.

   - Open `/var/www/FlaskApp/FlaskApp/client_secret.json` and paste the previous contents into the this file.

   - Replace the client ID in app.js file in the project directory.
   
   #### Final Step
   
   Restart apache2 server
  
   `sudo service apache2 restart`

 #   Test1.pem file
 
 -----BEGIN RSA PRIVATE KEY-----
MIIEowIBAAKCAQEA3EC8k8rBr9PTcAPEbAos59FIR4DMPQZs6Kau9+kKi3+kBX4K
ni7RA9XHDcVajyPPxlblW11cY5gxoNiauBhRfg55dW49T+iqRAFMLPygB6rKEDsu
duntKZjEuzSSGHK6irYiNtSk6fCEnuCu7k84yfISPDes13zPvqByUMhp8LZDfd8o
LBl4YTnaOVkSFotvDoAFvT+kWatns3K9Hwf5PeF65ieCP2+Ku9l/99ONaW1vFTU4
rjVfpJ451T4ICIsPNduOyn5Y7zmObT4hIvHg3IK5+NqvclsUPlRiCfkIMUHVms/n
OgviXszMsd3/bnvzHR2/KkmPIhpi/X16R0E4rQIDAQABAoIBAHsRJQ2DjmP7fToq
sLcZnGvPcY6adgRnMbVxZXSaX00A/hofijlGuX1mFvon2uj+Ppt5dGBvsy7nHFve
i9zvoaFI3y7xcQrUW0byXqkIYzbFhHA7UUQag2zpsotT3YoRmQ/mowl2Gcupm8DW
lIgN9hJI002YYlhpof7G0fk3cpKPiugkV5vGZfjz4Tb/f7nzSbev40SRmslHkwIW
UDnklyuHp4vCphDxwncyJ3xbioHZA1EDVO24B0CG6nTizLmSnZLA630zWbHCAkR9
Ru56MxPnX1ROTt5XyLUtm/4ZLbB6lw1P4lJJa8hgT1QnUTDpxvk6Ey2oBe9pRiqy
NeaYa2ECgYEA/2KZY+vD6HBAeXUZCbyJt7HnwvbvyRgFhJZXN440kF3jMyT+/6+T
A61JjiJntvCNv6XFGQfUIK57wH23bRjbA5uEZCxX0O4MekBvuQNSa4vAjdHxTcby
Rc5kxUiAYwaYHeQjmumHCJtJMClorGsB6noPIh/6ad2I00Yu5rP+/QUCgYEA3Mh8
BlevDfwh9YNSEjLZvNJfkmemIsDUvsU1Zl7wo2GLAbd7QW43Jdd2bhm3GIiUJbw4
uJqGCg6xSpTDrGr9q9wfGQOKbpsam9DrxchMbi1RhPWw8x/zQs0YNt2GSvyBzYaf
/yXQNIOcDcEoMrwmERbDY3GuqkmrpbzTvfgtXYkCgYAv21E1OUQP9aEPYZMckPkZ
tDvi/BU5EMhP7UBQx9QvzXg66E7kqQkaoklrWiUnUfKuHClQJHhq22eTTbumtQat
qWHox6p5G3K5IgQNnoK+ZoThzpqyYXqa/C9EDO8KH3039L17VRGZ2kefv9K+pJrK
Tq8xTN7HId13AereDpLU5QKBgQCm6YKqINwdDIJ34+HGFF98WucZ/fYhy/qKhvkJ
/bibLAE1OQubucFDgJLuRc6gY5DsvlF7bobrT5RFOBZ+YRyKMw3nkT+0wtno9pdo
nTb7DJPWmxA9negAlqE5yVvfkOppAOAwutue9+iglWjYglmdDcKFicpsvulfkVPb
CKbzUQKBgCTpX2yuKuBRalW0QbR1KE5bsaiOlcx8YLVvC5l7BEQfwVE4serKeTfj
Wr7mhMTojjFkekejYChDVX32WFE8Gv4MvF0HPhxOE+N+PFFhMMgv0eTEeVFJLHax
IZyVyHnpkeIWCGGQbFvZlrk/j2hAf/+XKMfFNzEfWkLc33GxAy4o
-----END RSA PRIVATE KEY-----

# Privatekey1.ppk file

PuTTY-User-Key-File-2: ssh-rsa
Encryption: none
Comment: imported-openssh-key
Public-Lines: 6
AAAAB3NzaC1yc2EAAAADAQABAAABAQDcQLyTysGv09NwA8RsCizn0UhHgMw9Bmzo
pq736QqLf6QFfgqeLtED1ccNxVqPI8/GVuVbXVxjmDGg2Jq4GFF+Dnl1bj1P6KpE
AUws/KAHqsoQOy526e0pmMS7NJIYcrqKtiI21KTp8ISe4K7uTzjJ8hI8N6zXfM++
oHJQyGnwtkN93ygsGXhhOdo5WRIWi28OgAW9P6RZq2ezcr0fB/k94XrmJ4I/b4q7
2X/3041pbW8VNTiuNV+knjnVPggIiw81247KfljvOY5tPiEi8eDcgrn42q9yWxQ+
VGIJ+QgxQdWaz+c6C+JezMyx3f9ue/MdHb8qSY8iGmL9fXpHQTit
Private-Lines: 14
AAABAHsRJQ2DjmP7fToqsLcZnGvPcY6adgRnMbVxZXSaX00A/hofijlGuX1mFvon
2uj+Ppt5dGBvsy7nHFvei9zvoaFI3y7xcQrUW0byXqkIYzbFhHA7UUQag2zpsotT
3YoRmQ/mowl2Gcupm8DWlIgN9hJI002YYlhpof7G0fk3cpKPiugkV5vGZfjz4Tb/
f7nzSbev40SRmslHkwIWUDnklyuHp4vCphDxwncyJ3xbioHZA1EDVO24B0CG6nTi
zLmSnZLA630zWbHCAkR9Ru56MxPnX1ROTt5XyLUtm/4ZLbB6lw1P4lJJa8hgT1Qn
UTDpxvk6Ey2oBe9pRiqyNeaYa2EAAACBAP9imWPrw+hwQHl1GQm8ibex58L278kY
BYSWVzeONJBd4zMk/v+vkwOtSY4iZ7bwjb+lxRkH1CCue8B9t20Y2wObhGQsV9Du
DHpAb7kDUmuLwI3R8U3G8kXOZMVIgGMGmB3kI5rphwibSTApaKxrAep6DyIf+mnd
iNNGLuaz/v0FAAAAgQDcyHwGV68N/CH1g1ISMtm80l+SZ6YiwNS+xTVmXvCjYYsB
t3tBbjcl13ZuGbcYiJQlvDi4moYKDrFKlMOsav2r3B8ZA4pumxqb0OvFyExuLVGE
9bDzH/NCzRg23YZK/IHNhp//JdA0g5wNwSgyvCYRFsNjca6qSaulvNO9+C1diQAA
AIAk6V9srirgUWpVtEG0dShOW7GojpXMfGC1bwuZewREH8FROLHqynk341q+5oTE
6I4xZHpHo2AoQ1V99lhRPBr+DLxdBz4cThPjfjxRYTDIL9HkxHlRSSx2sSGclch5
6ZHiFghhkGxb2Za5P49oQH//lyjHxTcxH1pC3N9xsQMuKA==
Private-MAC: c7f2777eb3264c34e7bce23394e041e898d64d9b

# id_rsa file

-----BEGIN RSA PRIVATE KEY-----
MIIEowIBAAKCAQEA3PzpTCS06rLjxRI40Njr7y32ixlPX3QneEl7FSXaw2Yr1Nta
JbAc+AmhhXmKh6VHuZol4rTtftbnF3F8yremYaWqR7vzUxf0oUanH1MNrSR/pWLD
60Pkt/jF2tN+w1wXNbnTxoj3xY7anJgiT/0g6lgG6NuF21Mz9PvU4F5pw8AWpvS8
3+Tzl6bjpY3qeTI51DWG0/CmjOX7QudsOpqCzjfP/D4Y56krGC1ixv1gtkqs/2KS
L9i/7ghX/+pLZXu9sw1qibPPsiuE+PalW8Js64ZG3mc7Ng9ni8mOiWKEcn/l5yR0
oqK7ptp2PqZuFl/jYhh4v1cqy88Qwmbjbcr3BwIDAQABAoIBACDEx/tofgNHX4r7
dr1RTTr8P9DEggaPfMLTcpLiOBw9bEZ1+FoaUVFebDsUmLwggBA/kVqapZTnXQEW
7QBKVzuniyZz8lLh/H5lsaZtdFu2S89EY/Tg7mtxUjVuox9o6nAnDAYmjUcYNcZ+
sKfXyye0weGJm8G4Br5PEXPrzBcggDACshdcmfXkwI+ffiFY3VzxN2DgCZfjaZBm
sTuPu3gJ7eXE95eVbiLpwNqeFsEiJnyjX6HRiUVWCt17i7KjWkHNqEwrwVEgvjHI
zU76lHgyBtyS/Dy9phr8FEudnWpGCDKRu6KXoN6JXBQJSkOyRa+Z9wiyOEWYWQt+
O1VpA4ECgYEA8L8b+12hvZXvGEcFu6FqFI5EAqFI+tzWiwUEIYtOEd+E+jCcaoFn
+r1QecVCi144Ltvhh1cveCVOx3FsbgSZN5VbS3uBATGATz6ktL6PBeWaYEd/GqDi
xzcTp08TP1hGFyl6n5cxgbesmjEMOZX9wXbe7JMkae9Zi6mHAIaXWOECgYEA6v1R
uDwRH+sY1F5fW6gFt220tWdopWuK18Yt2gk1LfmRVCTORSvnk+wD2eBNXnPrTfaR
GrncrEkV3WKKem4HU+BEN1kdvu/jBZyhuuFhCvXx043ZTRwACChloQ7zn02pYjFB
+7UcGa+v0bCg8OE0oNeVC8KVQAymcrbK8sVeROcCgYBwaTveliy0gnLeyiLiJo+K
w5b8B2U1RbKjvRbdttcgP1cvH02Z6YyspoMSKMpWmwruzlqzQEF4/yqWs95mTJ1i
N8omJ6fn774ywlRT1PqhTUFVHW06+M6LKKtzntek50nq/MI2DHngUOw2HxrPNLsE
/8U9f8Mr98e/D8xqsW2v4QKBgF9CN4lu2CZPQG5+nzthnoegMlxDQjmkodEcpmO3
zdYIUHCCxxdlV+gwCdOdyN9cMGwXYvUpmRpCOlnXY3mD9vZ6eEzTlGpdhnM07p76
VEOENfbjjs5iZmToM2KZ1AqlCeAjRbNK1MxY2vYvGt7q/FGjcukkhSpEtojULutB
NRjLAoGBAJPuePnFtZkbEniY3EFsfhVvUYmPuexlGCkRkEoDUCCqIBUGRLcKDGRV
S0AV4MwphgXSxjLtBxOVfGz9rrfBoDlin8O1BflK+K2Rwr9oGp7/4fbZ5TXVwwt8
7MfjfykoulCp9U49PYF/NvAHFP+fRCz9hTPVMMHANB6jJwSw7CV7
-----END RSA PRIVATE KEY-----

# id_rsa.pub file

ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDc/OlMJLTqsuPFEjjQ2OvvLfaLGU9fdCd4SXsVJdrDZivU21olsBz4CaGFeYqHpUe5miXitO1+1ucXcXzKt6ZhpapHu/NTF/ShRqcfUw2tJH+lYsPrQ+S3+MXa037DXBc1udPGiPfFjtqcmCJP/SDqWAbo24XbUzP0+9TgXmnDwBam9Lzf5POXpuOljep5MjnUNYbT8KaM5ftC52w6moLON8/8PhjnqSsYLWLG/WC2Sqz/YpIv2L/uCFf/6ktle72zDWqJs8+yK4T49qVbwmzrhkbeZzs2D2eLyY6JYoRyf+XnJHSiorum2nY+pm4WX+NiGHi/VyrLzxDCZuNtyvcH Acer@DESKTOP-E3L7FP3

## References

   1. https://github.com/rrjoson/udacity-linux-server-configuration

   2. https://github.com/boisalai/udacity-linux-server-configuration

   3. https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps


