# How to Setup Superset with enterprise level architecture using Superset, Celery, Redis and Separate Postgres as Metadata database

### Environment 
1.  Python and Python-venv (Version 3.10)
2.  Postgres (Metadata db)
3.  Redis

Everything I am installing on single vm as creating multiple will cost a lot but you should be able to understand.


Process is going to be same as normal installation we saw with few changes. Let's start.

```
sudo apt update -y & sudo apt upgrade -y
```


### Installing Redis

```
sudo apt install redis-server
```


### Installing Apache Superset
* Install dependencies

    ```
    sudo apt-get install build-essential libssl-dev libffi-dev python3.10-dev python3.10-pip libsasl2-dev libldap2-dev default-libmysqlclient-dev python3.10-venv libpq-dev 
    ```

    If 3.10 gives error then you can add repo using 

    ```
    sudo add-apt-repository ppa:savoury1/python
    sudo apt update
    ```


* Create app directory for superset and dependencies

    ```
    sudo mkdir /app
    sudo chown user /app
    cd /app
    ```

* Create python environment


    ```
    mkdir superset
    cd superset
    python3 -m venv superset_env
    . superset_env/bin/activate
    pip install --upgrade setuptools pip
    ```

* Install python dependencies  
    ```
    pip install pillow
    pip install apache-superset
    pip install psycopg2
    pip install gunicorn
    pip install celery
    pip install gevent
    ```
* Create superset config file and set environment variable
    ```
    touch superset_config.py
    export SUPERSET_CONFIG_PATH=/app/superset/superset_config.py
    ```

* Edit `superset_config.py` using `nano superset_config.py` and put following code in it

    ```
    # Superset specific config
    ROW_LIMIT = 5000

    # Flask App Builder configuration
    # Your App secret key will be used for securely signing the session cookie
    # and encrypting sensitive information on the database
    # Make sure you are changing this key for your deployment with a strong key.
    # Alternatively you can set it with `SUPERSET_SECRET_KEY` environment variable.
    # You MUST set this for production environments or the server will not refuse
    # to start and you will see an error in the logs accordingly.
    SECRET_KEY = 'YOUR_OWN_RANDOM_GENERATED_SECRET_KEY'

    # The SQLAlchemy connection string to your database backend
    # This connection defines the path to the database that stores your
    # superset metadata (slices, connections, tables, dashboards, ...).
    # Note that the connection information to connect to the datasources
    # you want to explore are managed directly in the web UI
    # The check_same_thread=false property ensures the sqlite client does not attempt
    # to enforce single-threaded access, which may be problematic in some edge cases
    SQLALCHEMY_DATABASE_URI = 'sqlite:////app/superset/superset.db?check_same_thread=false'

    TALISMAN_ENABLED = False
    WTF_CSRF_ENABLED = False

    # Set this API key to enable Mapbox visualizations
    MAPBOX_API_KEY = ''


    # Celery Redis configuration for async query execution
    class CeleryConfig(object):
    broker_url = "redis://localhost:6379/0"
    imports = (
        "superset.sql_lab",
        "superset.tasks.scheduler",
    )
    result_backend = "redis://localhost:6379/0"
    worker_prefetch_multiplier = 10
    task_acks_late = True
    task_annotations = {
        "sql_lab.get_sql_results": {
            "rate_limit": "100/s",
        },
    }

    CELERY_CONFIG = CeleryConfig

    # On Redis
    from flask_caching.backends.rediscache import RedisCache
    RESULTS_BACKEND = RedisCache(
        host='localhost', port=6379, key_prefix='superset_results')

    ```


* Please replace YOUR_OWN_RANDOM_GENERATED_SECRET_KEY in above file with the code returned by following command
    ```
    openssl rand -base64 42
    ```



* Once Done let us inititlize database with following commands 

    ```
    # Create an admin user in your metadata database (use `admin` as username to be able to load the examples)
    export FLASK_APP=superset

    superset db upgrade

    superset fab create-admin

    # As this is going to be production I have commented load example part but if you need you can run this
    # superset load_examples

    # Create default roles and permissions
    superset init

    ```

* Now Our environment is ready lets try running it..
To run superset I have created a sh script that you can run in order to run the server. To create create script using following command.

    ```
    nano run_superset.sh
    ```

    and paste following code in it.

    ```
    #!/bin/bash
    export SUPERSET_CONFIG_PATH=/app/superset/superset_config.py
    . /app/superset/superset_env/bin/activate
    gunicorn \
        -w 10 \
        -k gevent \
        --timeout 120 \
        -b  0.0.0.0:8088 \
        --limit-request-line 0 \
        --limit-request-field_size 0 \
        --statsd-host localhost:8125 \
        "superset.app:create_app()"
    ```


* In order to run it we need to grant it run permission. To do that lets run following command.
    ```
    chmod +x run_superset.sh
    ```

 * Lets run and test if it works?

    ```
    sh run_superset.sh
    ```

* check if you are able to login using admin creds on server-ip-address:8088. If everything is working fine then we can go ahead and create service that will start automatically as soon as server starts or in case it reboots.

> [!NOTE]  
> Now if you try to connect with database and async query is enabled it will just run indefinitely as celery worker is not running. We will create celery once superset service is created


Lets create service called superset using following command

    sudo nano /etc/systemd/system/superset.service
    
paste following code in it 

    [Unit]
    Description = Apache Superset Webserver Daemon
    After = network.target

    [Service]
    PIDFile = /app/superset/superset-webserver.PIDFile
    Environment=SUPERSET_HOME=/app/superset
    Environment=PYTHONPATH=/app/superset
    WorkingDirectory = /app/superset
    limit-re>
    ExecStart = /app/superset/run_superset.sh
    ExecStop = /bin/kill -s TERM $MAINPID


    [Install]
    WantedBy=multi-user.target

    

    once copied run following command to enable and start service

    systemctl daemon-reload
    sudo systemctl enable superset.service
    sudo systemctl start superset.service
    


#### Run and Test Celery 
* To run celery I have created a sh script that you can run in order to run the server. To create create script using following command.

    ```
    nano run_celery.sh
    ```

    and paste following code in it.

    #!/bin/bash
    export SUPERSET_CONFIG_PATH=/app/superset/superset_config.py
    . /app/superset/superset_env/bin/activate

    celery --app=superset.tasks.celery_app:app worker --pool=prefork -O fair -c 4 &
    celery --app=superset.tasks.celery_app:app beat

    

    In above script `-c 4` represents how many worker processes should run in a worker.

* In order to run it we need to grant it run permission. To do that lets run following command.
    ```
    chmod +x run_celery.sh
    ```

 * Lets run and test if it works?

    ```
    sh run_celery.sh
    ```

* Create Celery service edit/create file `sudo nano /etc/systemd/system/celery.service` and paste following code in it

    ```
    [Unit]
    Description = Apache Celery worker Daemon
    After = network.target

    [Service]
    PIDFile = /app/superset/celery.PIDFile
    Environment=SUPERSET_HOME=/app/superset
    Environment=PYTHONPATH=/app/superset
    WorkingDirectory = /app/superset
    ExecStart = /app/superset/run_celery.sh
    ExecStop = /bin/kill -s TERM $MAINPID


    [Install]
    WantedBy=multi-user.target

    ```


    once copied run following command to enable and start service

    
    ```
    systemctl daemon-reload
    sudo systemctl enable celery.service
    sudo systemctl start celery.service
    ```

### YEY! Your Enterprise Server is Up and running you can test it by restarting the server...
If you have any issues you can contact me on [contact@shantanukhond.me](mailto://contact@shantanukhond.me) . 
