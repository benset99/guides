# Part of the Learning Azure series
### My attempt at using azure to improve my cloud skills

## Creating a mariaDB database in Azure 

1. Create a MariaDB database 

`az mariadb server create -g learn-azure -n dben-84 --admin-user benset99 --admin-password TestTest1234 --sku-name MO_Gen5_2 --location eastus` 

2. Create a firewall rule so that your machine can connect to the remote database

`az mariadb server firewall-rule create -g learn-azure --server dben-84 --name allowIP --start-ip-address [Your IP address] --end-ip-address [Your IP address]`

3. Connect to the database using mySQL

`mysql -h dben-84.mariadb.database.azure.com -u benset99@dben-84 -p`

4. Once you logged in, create a database and add entries to it

```
CREATE DATABASE market;
USE market;

CREATE TABLE fruit (
id serial PRIMARY KEY,
name VARCHAR(50),
quantity INTEGER
);

INSERT INTO fruit (id, name, quantity) VALUES (1, 'bananas', 150);
INSERT INTO fruit (id, name, quantity) VALUES (2, 'apples', 132);
INSERT INTO fruit (id, name, quantity) VALUES (3, 'grapes', 24);
```

5. Check to see if your table has been setup correctly

`SELECT * FROM fruit;`

## Let's make a python script that reads the database remotely

1. Install python3

`sudo apt install python3 python3-venv python3-pip`

2. Create a virtual environment

```
mkdir ~/db_remote_read
cd ~/db_remote_read

python3 -m venv ben

source ~/db_remote_read/ben/bin/activate
```

3. Install Gunicorn and Flask to host our web app

`pip install gunicorn flask`

4. Create the Flask app 

`vim app.py`

```
from flask import Flask
app = Flask(__name__)
@app.route('/')
def hello_world():
    return "Hello World!"
if __name__ == '__main__':
    app.run(debug=True,host='0.0.0.0')
```

5. Test if our Flask app works

```
python app.py
curl http://locatlhost:5000
```

Output:

`Hello World!`

6. Create a WSGI entry point

`vim wsgi.py`

```
from app import app

if __name__ == "__main__":
    app.run()
```

`gunicorn --bind 0.0.0.0:5000 wsgi:app`

7. Replace the app.py with code that will read our database remotely

```
from flask import Flask
from flask import jsonify
import login
import mysql.connector
from mysql.connector import errorcode

#Obtain connection string information from the portal
app = Flask(__name__)

@app.route('/')
def db_connect():
	try:
		conn = login.connect()
		print("Connection established.")
	except mysql.connector.Error as err:
		if err.errno == errorcode.ER_ACCESS_DENIED_ERROR:
			print("Something wrong with username or password")
		elif err.errno == errorcode.ER_BAD_DB_ERROR:
			print("Database does not exist")
		else:
			print(err)
	else:
		cursor = conn.cursor()
		
		# Read Data
		cursor.execute("SELECT * FROM fruit;")
		rows = cursor.fetchall()
		print("Read", cursor.rowcount, "row(s) of data.")

		# Print all rows
		my_list = []
		for row in rows:
			my_list.append ("Data row = (%s, %s, %s)" %(str(row[0]),
				str(row[1]), str(row[2])))

		# Cleanup
		conn.commit()
		cursor.close()
		conn.close()
		return jsonify(my_list)
		
if __name__ == '__main__':
	app.run(debug=True,host='0.0.0.0')
```

Note: Make sure you include the file `config.ini`:
```
[mysqlDB]
host = 
user =
password = 
database =
```

8. Edit NGINX config 

```
sudo usermod -a -G ben nginx

chmod 710 ~/db_remote_read
```

`sudo vim /etc/nginx/proxy_params`

```
proxy_set_header Host $http_host;
proxy_set_header X-Real-IP $remote_addr;
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
proxy_set_header X-Forwarded-Proto $scheme;
```

`sudo vim /etc/nginx/sites-available/ben.conf`

```
server {
    listen 80;
    server_name;

location / {
  include proxy_params;
  proxy_pass http://unix:/home/ben/src/app.sock;
    }
}
```

9. Check if NGINX config works and restart

```
sudo nginx -t 
sudo systemctl restart nginx
```

10. That's it! You should have a working MariaDB database and a python web app that will read it remotely and display it. 
