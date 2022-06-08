caktus.k8s-hosting-services
===========================

|pre-commit|

.. |pre-commit| image:: https://github.com/caktus/ansible-role-k8s-hosting-services/actions/workflows/test.yml/badge.svg
   :target: https://github.com/caktus/ansible-role-k8s-hosting-services/actions/workflows/test.yml

An Ansible role to allow Caktus Kubernetes projects to take advantage of common
Caktus-provided services. These include:

* backing up the Postgres database to the backup AWS S3 bucket.
* backing up the environment S3 bucket to the backup AWS S3 bucket.
* More coming soon! (Papertrail, NewRelic infra, etc)

License
-------

This Ansible role is released under the BSD License.  See the `LICENSE
<https://github.com/caktus/ansible-role-aws-web-stacks/blob/master/LICENSE>`_
file for more details.

Development sponsored by `Caktus Consulting Group, LLC
<http://www.caktusgroup.com/services>`_.


How do I add this role to my project
------------------------------------

1. Add to your ``requirements.yaml``:

   .. code-block:: yaml

      ---
      - src: https://github.com/caktus/ansible-role-k8s-hosting-services
        name: caktus.k8s-hosting-services
        version: v0.5.0

   If you're using `invoke-kubesae`, run: `inv deploy.install` to install the
   Ansible requirements.

#. Create a ``deploy/deploy-hosting-services.yaml`` file which you'll run once to set up
   the backup process (and then infrequently whenever you'd like to make changes to it).
   Put this inside of it:

   .. code-block:: yaml

      - name: caktus hosting services
        hosts: production
        vars:
          ansible_connection: local
          ansible_python_interpreter: "{{ ansible_playbook_python }}"
        gather_facts: false
        roles:
          - role: caktus.k8s-hosting-services  # backups only
        # Install configured monitoring tools (optional):
        tasks:
           - import_role:
               name: caktus.k8s-hosting-services
               tasks_from: monitoring


#. Ensure the ``production`` host is in your ``deploy/inventory`` file:

   .. code-block:: ini

      ...
      [k8s]
      staging
      production

#. Method 1: Client AWS account and S3 bucket (preferred)

   Clients typically prefer backups to be stored within their own AWS account.
   This doesn't necessarily need to be the same account as the production K8s
   cluster, but for simplicity it is usually set up this way.

   #. Create new S3 bucket to store backups, e.g. `project-production-backups`
   #. Create new `caktus-backup` IAM user in client AWS account and create a new
      access key. Save the `AWS_SECRET_ACCESS_KEY` for use below.
   #. Attach a restricted IAM policy (remember to fill in the real bucket name)
      to the user:

   .. code-block:: json

      {
         "Version": "2012-10-17",
         "Statement": [
            {
                  "Sid": "ListObjectsInBucket",
                  "Effect": "Allow",
                  "Action": [
                     "s3:ListBucket"
                  ],
                  "Resource": [
                     "arn:aws:s3:::project-production-backups"
                  ]
            },
            {
                  "Sid": "ObjectActions",
                  "Effect": "Allow",
                  "Action": "s3:PutObject",
                  "Resource": [
                     "arn:aws:s3:::project-production-backups/*"
                  ]
            }
         ]
      }

#. Method 2: Caktus Hosting Services AWS Account and `caktus-hosting-services` S3 bucket

   The `AWS_SECRET_ACCESS_KEY` is in the LastPass shared entry: "Caktus Website
   Hosting Services k8s secrets".

#. You'll need to encrypt at least 2 variables for this role to work:

   A. Your production `DATABASE_URL`.
   #. The `AWS_SECRET_ACCESS_KEY` for the backup IAM user.

   The first should be obtained from your project's configuration and
   `AWS_SECRET_ACCESS_KEY` is found above.

   Use `ansible-vault` to encrypt them:

   .. code-block:: sh

      ansible-vault encrypt_string SECRET-VALUE-HERE

#. Once you have those encrypted values, add the following to
   ``deploy/group_vars/production.yaml``:

   .. code-block:: yaml

      k8s_hosting_services_project_name: "your-project-name"
      k8s_hosting_services_healthcheck_url: "<... project healthcheck url ...>"
      k8s_hosting_services_database_url: "<... secret from ansible-vault output ...>"
      k8s_hosting_services_backup_base_bucket: "<... project backup S3 bucket ...>" OR "caktus-hosting-services"
      k8s_hosting_services_aws_access_key: "<... project iam user access key ...>"
      k8s_hosting_services_aws_secret_access_key: "<... secret from ansible-vault output ...>"

   * You should generate a new healtcheck url at healtchecks.io and provide that
     to the developer for their `k8s_hosting_services_healthcheck_url` variable.
   * ``k8s_hosting_services_project_name`` will be the directory in S3 under which these
     backups will be stored.

#. By default, this role will run backups on a daily, weekly, monthly and yearly
   schedule. If you don't need all of those, or if you need a custom schedule, then
   override ``k8s_hosting_services_cron_schedules``. Ideally, you should do this before
   your run this role for the first time, because this role does not delete existing
   schedules. If you do need to delete a schedule, you can do it manually using k8s
   commands. Here is an example of customizing your schedule to remove daily backups
   and add a rule to backup every 2 hours:

   .. code-block:: yaml

      k8s_hosting_services_cron_schedules:
        - label: weekly
          schedule: "@weekly"
        - label: monthly
          schedule: "@monthly"
        - label: yearly
          schedule: "@yearly"
        - label: every2hours
          schedule: "0 */2 * * *"

#. Review ``defaults/main.yml`` in this repo to see other variables that you can override.

#. See the next section for the commands to deploy this role to your cluster.


How do I deploy this role to my cluster
---------------------------------------

Once you have configured the role as described above (or any time you need to make a
change to the configuration), you can deploy this to your kubernetes cluster.

* Using invoke-kubesae:

  .. code-block:: sh

     inv deploy.playbook -n deploy-hosting-services.yaml

* Without invoke-kubesae:

  .. code-block:: sh

     cd deploy/
     ansible-playbook deploy-hosting-services.yaml -vv


Run a database backup job manually
---------------------------------------

By default, the shortest backup `cronjob` interval is **daily**, which means
that the first `job` won't run for roughly 24 hours after the initial setup.
This makes it difficult to test if backups are configured correctly.

You can manually invoke a `job` like so:

.. code-block:: shell

   kubectl create job --from=cronjob/backup-job-daily test-job-0001 -n hosting-services


Papertrail
---------------------------------------

Add the following for each cluster to monitor:

   .. code-block:: yaml

      k8s_papertrail_logspout_destination: syslog+tls://YYYYY.papertrailapp.com:NNNNN
      k8s_papertrail_logspout_syslog_hostname: "{{ k8s_cluster_name }}"


New Relic Infrastructure
---------------------------------------

New Relic's [Helm Charts](https://github.com/newrelic/helm-charts/) are used to
install New Relic Infrastructure monitoring.

Add the following for each cluster to monitor:

   .. code-block:: yaml

      # https://github.com/newrelic/helm-charts/releases
      k8s_newrelic_chart_version: "2.22.3"
      k8s_newrelic_license_key: !vault...


Maintainer information
----------------------

If you are working on the role itself (rather than just using the role), make sure to
set up a Python 3 virtualenv and then set up pre-commit:

.. code-block:: sh

   pip install -Ur requirements.txt
   pre-commit install  # <- only needs to be done once

The pre-commit tasks will run on each commit locally, and will run in Github Actions for
each pull request.

.. |master-status| image::
    https://github.com/caktus/ansible-role-k8s-hosting-services/workflows/test/badge.svg?branch=master
    :alt: Build Status
    :target: https://github.com/caktus/ansible-role-k8s-hosting-services/actions?query=branch%3Amaster
