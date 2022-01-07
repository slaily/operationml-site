# How to upload train, test, validation datasets with DVC into AWS S3 Bucket

Looking to store Datasets into Remote Storage and track transformations on them? Datasets in practice are often split into `train`, `test`, and `validation`. Store them into an [AWS S3 Bucket](https://aws.amazon.com/s3/). When a change like dropping a column on the Dataset occurs because the data presented is not meaningful, a tool that helps track such kinds of transformations is [DVC](https://dvc.org/).

This is the approach that helped the project to have data consistency, reproducibility, and traceability. Developers independently experiment building different Machine Learning and Deep Learning Models based on the same data.

## Step 1: Setup DVC

Assuming DVC is already [installed](https://dvc.org/doc/install), let's initialize it by running inside the [Git](https://git-scm.com/) project:
```console
dvc init
```

## Step 2: Track a directory - DVC

The `train`, `test` and `validation` datasets are under directory named `datasets`:
```console
datasets
├── test.csv
├── train.csv
└── validation.csv
```
To start tracking the directory with the `.csv` files inside:
```console
dvc add datasets/
```
New file will be created `datasets.dvc`, it stores metadata information about the actual data. Push the file to the [Git](https://git-scm.com/) project, it'll be our main point of reference when we do operations with the data like `download`.
```console
git add datasets.dvc .gitignore
git commit -m "Add train, test and validation datasets"
```

## Step 3: Store the datasets into a S3 Bucket

Assume having an [AWS Account](https://aws.amazon.com/premiumsupport/knowledge-center/create-and-activate-aws-account/), inside there created a [Bucket](https://docs.aws.amazon.com/AmazonS3/latest/userguide/creating-bucket.html) with name `ml-classifier` for an example. 

Install the DVC package support for Amazon S3 storage:
```console
pip install "dvc[s3]"
# If using Poetry
poetry add dvc --extras "s3"
# If using Pipenv
pipenv install "dvc[s3]"
```
Configure DVC to upload `train`, `test`, and `validation` into the S3 Bucket:
```console
 dvc remote add -d aws-s3-storage s3://ml-classifier/datasets/
```
Set the AWS Credentials that DVC uses to authenticate in order to `upload` and `download` the datasets.

Create a file `credentials` inside the project root directory with the following structure:
```console
[default] 
aws_access_key_id=AKIAIOSFODNN7EXAMPLE 
aws_secret_access_key= wJalrXUtnFEMIK7MDENGbGmLyxfIstl
```
Just replace values of `aws_access_key_id` and `aws_secret_access_key` with your AWS Account [Access and Secret Keys](https://docs.aws.amazon.com/powershell/latest/userguide/pstools-appendix-sign-up.html).

Configure DVC to get the AWS Credentials from the `credentials` file:
```console
dvc remote modify aws-s3-storage credentialpath credentials
```
Open `.dvc/config` file and it should look like:
```console
[core]
    remote = aws-s3-storage
['remote "aws-s3-storage"']
    url = s3://ml-classifier/datasets/
    credentialpath = ../credentials
```
Ready to go! Let's upload the datasets via DVC into the S3 Bucket:
```console
dvc push
```
Now our `train`, `test` and `validation` datasets appear into `ml-classifier` S3 Bucket.

## Final thoughts

The benefit of this solution is to store the data in a single place with a structural policy for access and actions on it, along with a history of the data transformations.