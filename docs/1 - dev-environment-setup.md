# Development environment setup

You will need to install a few prerequisites in order to take part in this tutorial. Please follow the instructions below:

## Prerequisites

- Docker and docker-compose: These are used for running postgres databases
- Python 3.10: Unfortunately airflow doesn't officially support 3.11 yet

## Overall structure

If you take a look at the src directory, you'll see a few things:

```
airflow_home/
database/
django_project_1/
django_project_2/
requirements.txt
```

- Airflow has a concept called `airflow_home`. THis is where your airflow configuration and dags can be found. 
- database: Our django applications will be storing their data in Postgres databases. These are defined through docker-compose
- django_project_*: We will be showing how airflow can play nice with 2 separate django projects. This is useful because many organisations are likely to run multiple projects
- requirements.txt: The usual

## Setup

### 1. Set up your virtual environment

In a real-world project it's likely that Airflow and the Django projects would be developed in different places. Perhaps they would have different repos, and they would definitely have different virtual-environments. We're just using one virtual environment for all of the things so we can keep things simple.

Note: Airflow does not yet support any package managers beyond pip. Eg `poetry` is not officially supported. So we are doing things in a bit of an old-school way:

```
cd src 
python3.10 -m venv venv
source venv/bin/activate 

pip install -r requirements.txt 
```

If you get an error that includes the following:

```
  ./psycopg/psycopg.h:36:10: fatal error: libpq-fe.h: No such file or directory
     36 | #include <libpq-fe.h>
```

then you'll need to install some postgres development headers and try again. On Ubuntu you would do this:

```
sudo apt install libpq-dev  
```

Once you have installed the prerequisites then try `pip install -r requirements.txt` again.

### 2. Install Airflow


Airflow installation is a little bit unusual. To keep things explicit we'll do this in a separate step:


```
AIRFLOW_VERSION=2.7.0
PYTHON_VERSION="$(python --version | cut -d " " -f 2 | cut -d "." -f 1-2)"
CONSTRAINT_URL="https://raw.githubusercontent.com/apache/airflow/constraints-${AIRFLOW_VERSION}/constraints-${PYTHON_VERSION}.txt"
pip install "apache-airflow==${AIRFLOW_VERSION}" --constraint "${CONSTRAINT_URL}"
```

Learn more here: https://airflow.apache.org/docs/apache-airflow/stable/installation/installing-from-pypi.html 


### 3. Set AIRFLOW_HOME 

Airflow needs to know where your airflow_home directory is. It has a default location that doesn't make sense for a development environment. We will use an environmental variable to tell Airflow what's up:

1. get the absolute path to your airflow_home directory

If you are using bash (or similar) you could do this to print out the absolute path.

```
cd src/airflow_home
pwd
```

Mine looks like this:

```
/home/sheena/workspace/airflow-tutorial/src/airflow_home
```

2. Figure out where your activate script is

We want to make sure that there is an environmental variable that is set every time we activate our virtual environment. This is just a nice trick to save us from unnecessary suffering as we continue our work.

To do this we need to figure out where our virtual environment's activate script is.


If you followed the instructions exactly then there should be a venv directory inside the src directory. Your `activate` script will then be at `src/venv/bin/activate`.  You can move onto step 3.

If you used a different tool to create your virtual environment then your path might be different. To find out where your activate script is do the following:

```
# 1. activate your venv then

# 2. Find the path to the Python executable
which python 
```

The Python executable will be inside a directory named `bin`. Your activate script will be there too.

3. Edit your `activate` script

Now open the activate script with any editor you want. Eg VSCode.

Put the following at the top of your activate script:

```
export AIRFLOW_HOME=/path/to/your/airflow-tutorial/src/airflow_home
```

NOTE: Replace the path above with the actual path to your airflow home.


4. Check that it worked

Open up a new shell and activate your virtual environment.

Check that the environmental variable is as it should be.

```[bash]
echo $AIRFLOW_HOME
```

This should print out the value you put in your activate script.


### 4. Make sure your database runs

Our Django projects are going to be plugging into some Postgres databases. We'll use Docker and docker-compose to make life easy.

```
cd database
docker-compose up
```
