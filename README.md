# Simple Web Application

This is a simple web application using [Python Flask](http://flask.pocoo.org/) and [MySQL](https://www.mysql.com/) database. 
This is used in the demonstration of development of Ansible Playbooks.
  
  Below are the steps required to get this working on a base linux system.
  
  - Install all required dependencies
  - Install and Configure Database
  - Start Database Service
  - Install and Configure Web Server
  - Start Web Server
   
## 1. Install all required dependencies
  
  Python and its dependencies

    apt-get install -y python python-setuptools python-dev build-essential python-pip python-mysqldb

   
## 2. Install and Configure Database
    
 Install MySQL database
    
    apt-get install -y mysql-server mysql-client

## 3. Start Database Service
  - Start the database service
    
        service mysql start

  - Create database and database users
        
        # mysql -u <username> -p
        
        mysql> CREATE DATABASE employee_db;
        mysql> GRANT ALL ON *.* to db_user@'%' IDENTIFIED BY 'Passw0rd';
        mysql> USE employee_db;
        mysql> CREATE TABLE employees (name VARCHAR(20));
        
  - Insert some test data
        
        mysql> INSERT INTO employees VALUES ('JOHN');
    
## 4. Install and Configure Web Server

Install Python Flask dependency

    pip install flask
    pip install flask-mysql

- Copy app.py or download it from source repository
- Configure database credentials and parameters 

## 5. Start Web Server

Start web server

    FLASK_APP=app.py flask run --host=0.0.0.0
    
## 6. Test

Open a browser and go to URL

    http://<IP>:5000                            => Welcome
    http://<IP>:5000/how%20are%20you            => I am good, how about you?
    http://<IP>:5000/read%20from%20database     => JOHN
    
    

# playbook.yaml
```
- name: Deploy a web application
  hosts: db_and_web_server1,db_and_web_server2
  tasks:
    - name: Install all required dependencies
      apt: name={ { item } } state=installed                # So the first dependency is python.(apt)module
      with_items:
        - python
        - python-setuptools
        - python-dev
        - build-essential
        - python-pip                                        # I will use {{ item }} so that it replaces loop.

    - name: Install MYSQL database
      apt: name={{ item }} state=installed
      with_items:
        - mysql-server
        - mysql-client
     
    - name: Start  MYSQL Services
      services:
        name: mysql
        state: started
        enabled: yes

    - name: Create Database user
      mysql_user:
        name: db_user
        password: Passw0rd
        priv: '*.*:ALL'
        state: present
        host: '%'

    - name: Install Python Flask dependency
      pip:
         name: {{ item }}
         state: present
      withe_items:
         - flask
         - flask-mysql

     - name: Copy source code
       copy: src=app.py dest=/opt/app.py

     - name: Start Web Server
       shell: FLASK_APP=/opt/app.py nohup flask run --host=0.0.0.0

```


    
