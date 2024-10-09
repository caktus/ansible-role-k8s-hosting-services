caktus.k8s-hosting-services Releases
====================================

v0.13.0 on 2024-10-09
~~~~~~~~~~~~~~~~~~~~~
* Add ability to backup Redis databases with ``k8s_hosting_services_backup_redis_url``


v0.12.0 on 2024-02-08
~~~~~~~~~~~~~~~~~~~~~
* Add ability to run ``pg_restore`` with alteranate flags during ``deploy.db-restore`` task by adding default value ``k8s_restore_pg_restore_command``


v0.11.0 on 2023-04-25
~~~~~~~~~~~~~~~~~~~~~
* Support Kubernetes v1.25 by using the ``batch/v1`` API version for ``CronJob``, available since v1.21.


v0.10.0 on 2023-04-25
~~~~~~~~~~~~~~~~~~~~~
* Switch ``community.kubernetes.helm``, which breaks on newer Ansible versions, to ``kubernetes.core.k8s``
* Use full ``kubernetes.core.k8s`` path elsewhere


v0.9.0 on 2022-12-09
~~~~~~~~~~~~~~~~~~~~
* Update New Relic configuration keys to match latest Helm chart keys (#20)
* Enable New Relic ``lowDataMode`` support by default (#20)
* Add support to run custom SQL commands with ``k8s_restore_sql_commands`` after a database restore (#21)

v0.8.0 on 2022-12-05
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
