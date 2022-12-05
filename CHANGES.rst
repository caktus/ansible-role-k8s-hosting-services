caktus.k8s-hosting-services Releases
====================================


v0.8.0 on 2022-12-02
~~~~~~~~~~~~~~~~~~~~
* Add ``deploy.db-restore`` task to manage database backup restore process in Ansible (#19)


v0.7.0 on 2022-04-06
~~~~~~~~~~~~~~~~~~~~
* Add optional support for `SOURCE_AWS_CA_BUNDLE` and `SOURCE_AWS_NO_VERIFY_SSL` environment variables
* Default to `0.4.0-postgres12` Docker image tag


v0.6.1 on 2022-03-01
~~~~~~~~~~~~~~~~~~~~
* Set default tag to 0.3.0-postgres12


v0.6.0 on 2022-02-24
~~~~~~~~~~~~~~~~~~~~
* Add S3 bucket backup support


v0.5.0 on 2022-02-17
~~~~~~~~~~~~~~~~~~~~
* Add variables to support enabling encryption
* Default to PostgreSQL 12 client version


v0.4.0 on 2021-09-29
~~~~~~~~~~~~~~~~~~~~
* Fix docs (#15)


v0.3.0 on 2021-09-15
~~~~~~~~~~~~~~~~~~~~
* Add detailed documentation for new setups using non-Caktus S3 buckets
* Add ``k8s_hosting_services_backup_base_bucket``, ``k8s_hosting_services_aws_region``, and ``k8s_hosting_services_aws_access_key`` to allow further role customization
* Document New Relic Infrastructure


v0.2.0 on 2021-05-25
~~~~~~~~~~~~~~~~~~~~
* Add New Relic Helm chart


v0.1.0 on 2021-05-24
~~~~~~~~~~~~~~~~~~~~
* Add Papertrail support


v0.0.1 on 2020-11-13
~~~~~~~~~~~~~~~~~~~~
* Initial release
