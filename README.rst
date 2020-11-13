caktus.k8s-hosting-services
===========================

An Ansible role to allow Caktus Kubernetes projects to easily back up their Postgres
database to the proper Caktus Hosting Services AWS S3 bucket.

License
~~~~~~~~~~~~~~~~~~~~~~

This Ansible role is released under the BSD License.  See the `LICENSE
<https://github.com/caktus/ansible-role-aws-web-stacks/blob/master/LICENSE>`_
file for more details.

Development sponsored by `Caktus Consulting Group, LLC
<http://www.caktusgroup.com/services>`_.


Requirements
~~~~~~~~~~~~~~~~~~~~~~

* ``pip install openshift kubernetes-validate``


Installation and Configuration
------------------------------

1. Add to your ``requirements.yaml``:

   .. code-block:: yaml

      ---
      - src: https://github.com/caktus/ansible-role-k8s-hosting-services
        name: caktus.k8s-hosting-services
        version: v0.0.1

#. Create a ``deploy/deploy-hosting-services.yaml`` file which you'll run once to set up
   the backup process (and then infrequently whenever you'd like to make changes to it).
   Put this inside of it:

   .. code-block:: yaml

      - name: caktus hosting services
        hosts: cluster
        vars:
          ansible_connection: local
          ansible_python_interpreter: "{{ ansible_playbook_python }}"
        gather_facts: false
        roles:
          - role: caktus.k8s-hosting-services

#. Add a ``cluster`` group to your ``deploy/inventory`` file:

   .. code-block:: ini

      ...
      [cluster]
      aws.amazon.com

#. You'll need to obtain and encrypt at least 3 variables for this role to work:

   A. Your production DATABASE_URL.
   #. The hosting services AWS_SECRET_ACCESS_KEY.
   #. The hosting services BACKUP_HEALTHCHECK_URL.

   The first should be obtained from your project's configuration and the other two
   items will be given to you by the Caktus TS team.

   Use ansible-vault to encrypt them:

   .. code-block:: sh

      ansible-vault encrypt_string SECRET-VALUE-HERE

#. Once you have those encrypted values, add the following to
   ``deploy/group_vars/cluster.yaml``:

   .. code-block:: yaml

      k8s_hosting_services_project_name: "your-project-name"
      k8s_hosting_services_aws_secret_access_key: "<... secret from ansible-vault output ...>"
      k8s_hosting_services_database_url: "<... secret from ansible-vault output ...>"
      k8s_hosting_services_healthcheck_url: "<... secret from ansible-vault output ...>"

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
          schedule: ""* */2 * * *"

#. Review ``defaults/main.yml`` in this repo to see other variables that you can override.


Usage
-----

Once you have configured the role as described above, you can deploy this to your
kubernetes cluster.

* Using invoke-kubesae:

  .. code-block:: sh

     inv playbook deploy-hosting-services.yaml

* Without invoke-kubesae:

  .. code-block:: sh

     cd deploy/
     ansible-playbook deploy-hosting-services.yaml -vv
