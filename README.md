
## Lab: Explore a BigQuery Public Dataset
[source](https://googlecoursera.qwiklabs.com/focuses/11593489?parent=lti_session)

[img](img/recommendations.png)


### Task 1. Query a public dataset
Load the USA Names dataset

### Query the USA Names dataset

  SELECT name, gender, SUM(number) AS total
  FROM `bigquery-public-data.usa_names.usa_1910_2013`
  GROUP BY name, gender
  ORDER BY total DESC
  LIMIT 10
 
 
### Task 2. Create a custom table
Download the data to your local computer


### Task 3. Create a dataset
babynames


### Task 5. Query the table

SELECT
 name, count
FROM
 `babynames.names_2014`
WHERE
 gender = 'M'
ORDER BY count DESC LIMIT 5

***


## Lab: Recommend products using Cloud SQL and SparkML
[source](https://googlecoursera.qwiklabs.com/focuses/11593594?parent=lti_session)

### Task 1. Create a Cloud SQL instance
instancerentalsb.1

### Task 2. Create tables

CREATE DATABASE IF NOT EXISTS recommendation_spark;

USE recommendation_spark;

DROP TABLE IF EXISTS Recommendation;
DROP TABLE IF EXISTS Rating;
DROP TABLE IF EXISTS Accommodation;

CREATE TABLE IF NOT EXISTS Accommodation
(
  id varchar(255),
  title varchar(255),
  location varchar(255),
  price int,
  rooms int,
  rating float,
  type varchar(255),
  PRIMARY KEY (ID)
);

CREATE TABLE  IF NOT EXISTS Rating
(
  userId varchar(255),
  accoId varchar(255),
  rating int,
  PRIMARY KEY(accoId, userId),
  FOREIGN KEY (accoId)
    REFERENCES Accommodation(id)
);

CREATE TABLE  IF NOT EXISTS Recommendation
(
  userId varchar(255),
  accoId varchar(255),
  prediction float,
  PRIMARY KEY(userId, accoId),
  FOREIGN KEY (accoId)
    REFERENCES Accommodation(id)
);

SHOW DATABASES;


### Connect to this instance
Public IP address
34.67.191.189
Connection name
qwiklabs-gcp-00-0081b145c400:us-central1:rentals

Connect using Cloud Shell:
`gcloud sql connect rentals --user=root --quiet`


`gcloud auth login`
https://accounts.google.com/o/oauth2/auth?client_id=32555940559.apps.googleusercontent.com&redirect_uri=urn%3Aietf%3Awg%3Aoauth%3A2.0%3Aoob&scope=openid+https%3A%2F%2Fww
w.googleapis.com%2Fauth%2Fuserinfo.email+https%3A%2F%2Fwww.googleapis.com%2Fauth%2Fcloud-platform+https%3A%2F%2Fwww.googleapis.com%2Fauth%2Fappengine.admin+https%3A%2F%2Fwww
.googleapis.com%2Fauth%2Fcompute+https%3A%2F%2Fwww.googleapis.com%2Fauth%2Faccounts.reauth&code_challenge=6ubIHps1lfIY5pLVRNs0OrThhVOkjrTfGrrYZuAQ7Ew&code_challenge_method=S
256&access_type=offline&response_type=code&prompt=select_account

`gcloud sql connect rentals --user=root --quiet`
pwd: xxx

mysql> SHOW DATABASES;

mysql> USE recommendation_spark;

SHOW TABLES;

mysql> SELECT * FROM Accommodation;


## Task 3. Stage data in Cloud Storage
* Option 1: Use the command line
On another cloud shell tab:

echo "Creating bucket: gs://$DEVSHELL_PROJECT_ID"
gsutil mb gs://$DEVSHELL_PROJECT_ID

echo "Copying data to our storage from public dataset"
gsutil cp gs://cloud-training/bdml/v2.0/data/accommodation.csv gs://$DEVSHELL_PROJECT_ID
gsutil cp gs://cloud-training/bdml/v2.0/data/rating.csv gs://$DEVSHELL_PROJECT_ID

echo "Show the files in our bucket"
gsutil ls gs://$DEVSHELL_PROJECT_ID

echo "View some sample data"
gsutil cat gs://$DEVSHELL_PROJECT_ID/accommodation.csv


* (OR) Option 2: Use the Cloud Console UI
[accommodation.csv](https://storage.googleapis.com/cloud-training/bdml/v2.0/data/accommodation.csv)
[rating](https://storage.googleapis.com/cloud-training/bdml/v2.0/data/rating.csv)


## Task 4. Load data from Cloud Storage into Cloud SQL tables

qwiklabs-gcp-00-0081b145c400/accommodation.csv
with [Your-Bucket-Name]= qwiklabs-gcp-00-0081b145c400


## Task 5. Explore Cloud SQL data

USE recommendation_spark;

mysql> SELECT * FROM Rating
LIMIT 15;

mysql> SELECT COUNT(*) AS num_ratings
FROM Rating;


mysql> SELECT
    COUNT(userId) AS num_ratings,
    COUNT(DISTINCT userId) AS distinct_user_ratings,
    MIN(rating) AS worst_rating,
    MAX(rating) AS best_rating,
    AVG(rating) AS avg_rating
FROM Rating;

mysql> SELECT
    userId,
    COUNT(rating) AS num_ratings
FROM Rating
GROUP BY userId
ORDER BY num_ratings DESC;

mysql> exit


## Task 6. Launch Dataproc

region of your Cloud SQL instance: us-central1-b

Name: , Zone and Total worker nodes in your cluster.


echo "Authorizing Cloud Dataproc to connect with Cloud SQL"
CLUSTER=rentals
CLOUDSQL=rentals
ZONE=us-central1-a
NWORKERS=2

machines="$CLUSTER-m"
for w in `seq 0 $(($NWORKERS - 1))`; do
   machines="$machines $CLUSTER-w-$w"
done

echo "Machines to authorize: $machines in $ZONE ... finding their IP addresses"
ips=""
for machine in $machines; do
    IP_ADDRESS=$(gcloud compute instances describe $machine --zone=$ZONE --format='value(networkInterfaces.accessConfigs[].natIP)' | sed "s/\['//g" | sed "s/'\]//g" )/32
    echo "IP address of $machine is $IP_ADDRESS"
    if [ -z  $ips ]; then
       ips=$IP_ADDRESS
    else
       ips="$ips,$IP_ADDRESS"
    fi
done

echo "Authorizing [$ips] to access cloudsql=$CLOUDSQL"
gcloud sql instances patch $CLOUDSQL --authorized-networks $ips


## Task 7. Run the ML model

gsutil cp gs://cloud-training/bdml/v2.0/model/train_and_apply.py train_and_apply.py
cloudshell edit train_and_apply.py


# MAKE EDITS HERE
CLOUDSQL_INSTANCE_IP = '<paste-your-cloud-sql-ip-here>'   # <---- CHANGE (database server IP)
CLOUDSQL_DB_NAME = 'recommendation_spark' # <--- leave as-is
CLOUDSQL_USER = 'root'  # <--- leave as-is
CLOUDSQL_PWD  = '<type-your-cloud-sql-password-here>'  # <---- CHANGE


gsutil cp train_and_apply.py gs://$DEVSHELL_PROJECT_ID


## Task 8. Run your ML job on Dataproc

Task 9. Explore inserted rows with SQL

