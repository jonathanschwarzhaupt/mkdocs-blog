---
date:
  created: 2024-11-27
tags:
  - dltHub
  - Python Tooling
categories:
  - Python
---

# Impressions from dltHub's product launch event in Berlin

The dlt team has been on a global roadshow for the last few weeks, making the stop in their home-city of Berlin last Tuesday.

<!-- more -->

The evening was packed with presentations, guest speakers, and product demos. And even though one speaker fell ill, it went well over the planned schedule. If it was up to me, it could have continued for a good while longer - I was really fascinated by the insights shared from the members of the community. It was really cool to put a face to the names you typically see in Slack or in articles.

## dlt+
![Introduction of dlt+](../images/dlt+.jpeg){ align=left width=400 }

dltHub's journey started as a open source python library for easily extracting and loading data. For 2025 they focus on building-out and establishing their commercial product, dlt+.

With dlt+, dltHub is moving into the data platform space. The goal is to give users instant access to datasets, enable local exploration and processing, and allow the user to share back their results. dltHub coins it the "[Portable data lake](https://dlthub.com/blog/portable-data-lake)".

I find the reasons why someone would use dlt+ really compelling. That is, only if it solves a key problem you are trying to solve. Luckily, dlt+ potentially covers many use cases:
- Collaboration in multidisciplinary teams
- Quicker access to data -> faster, more accurate insights
- Quicker onboarding of users -> Save engineering time
- Reduce cloud costs through local exploration, development, and testing
- Semantic contracts maintain data quality and compliance
- Leverages open standards (Delta, Iceberg) thus breaking vendor lock-in (but isn't dlt+ a vendor lock-in? 🧐)

## Deeper look
The rough workflow in dlt+ would look a little something like this:
1. Data Engineer creates a dlt package* (python, SQL, yaml) and pushes their changes to GitHub. The Data Engineers uses locally:
    - Python, SQL, yaml
    - Delta Lake
    - DuckDB, Arrow, Parquet
    - dbt
2. The Infrastrucuter Engineer adds security (profiles, audit) and builds a private PyPi package. They use tools like Docker and Terraform/ Pulumi
3. The Data Scientist uses and shares the data. Within their local notebook environment, they can use the dlt package as a regular python package, analyze and wrangle data, and write-back the data - all with embedded security and schema contracts.

*A dlt package contains data sources, dlt pipeline(s) and (dbt) transformations

The definition of the dlt project, `dlt_project.yml`, will already look familiar to many:
![dlt project definition](../images/dlt_project1.jpeg){ align=left width=800 }
![dlt project definition with profiles](../images/dlt_project2.jpeg){ align=right width=800 }

After building of the pipeline, initial transformations, and deployment, the Data Scientist can then get to work on the data without needing to be connected to expensive cloud resources or needing to deal much with secrets. They would interact with the portable data lake something like this:
```python
# Import package from private PyPi repository
import dlt_company_package as dlt_cp

# Inspect available datasets
catalogue = dlt_cp.catalogue()

# This shows the datasets available for the profile interacting with the catalogue
print(catalogue)

# Shows the table of the dataset, assuming the user has access via their profile
print(catalogue.my_dataset)
```

Then, after performing some data science work, they can write back their results. Here, dlt enforces the data contracts specified in the dlt_profile. It could for example allow the insertion of records, but disallow the change of columns.
```python
# Write into data lake, if sufficient permissions
print(catalogue.my_dataset.save(df, table_name="my_table"))
```
If a table contract is violated, dlt raises an error, which would look something like this:
```
DataValidationError: In schema: out source Table: my_table Column: id. Contract on columns with freeze mode is violated. Trying to add column id to table my_table but columns are frozen
```

## Thoughts
Personally, I am really excited about the possibilities of dlt+! I have experienced first-hand the creeping cloud costs of easy-to-setup and use cloud-native analytics platforms (looking at you Azure Synapse) and wished for ways I could simply leverage my own laptop for e.g. quick exploration, or the creation of an ad-hoc analysis.

And if you really think about it, [most companies (certainly all I encounter in my work) really do not need distributed computing](https://motherduck.com/blog/redshift-files-hunt-for-big-data/) - which is what all modern, cloud-native datawarehouses are built on top of. So why are we still recommending *everyone* to move to one of the big DWH - regardless of data volume?

The above is a scenario more and more data practicioners and decision makers face. OSS software like dlt and DuckDB are massively in gaining popularity, because they effectively address these very real problems.

With dlt's focus on open standards, performant and quick data ingestion, and now enterprise features such as data contracts, collaboration, and security all while leveraging local compute, I think dlt+ is positioned very attractively in a promising space!
