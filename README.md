Ansible Role: oracle-host-cron
==============================

Manage Oracle Database Server cron.

Copy scripts enlisted in `oracle_host_cron_templates_to_copy` list and schedule them as defined in `oracle_host_cron_config` variable.

Minutes are randomly assigned in CRON.

Sample CRON output after execution:

    #Ansible: backup_arch_emergency script to backup archivelogs when FRA utilisation threshold is exceeded
    */5 * * * * /home/oracle/scripts/backup_arch_emergency.ksh >/dev/null 2>&1
    #Ansible: rman_delete_expired script deletes records of the expired backups from the recovery catalog
    47 2 * * * /home/oracle/scripts/rman_delete_expired.ksh >/dev/null 2>&1
    #Ansible: rman_resync_catalog script to resync backups with the catalog database
    47 */6 * * * /home/oracle/scripts/rman_resync_catalog.ksh >/dev/null 2>&1


Supported OS:
-------------
* RedHat
* CentOS
* OracleLinux

Requirements
------------

This role uses `oracle` role.

Role Variables
--------------

Available variables are listed below, along with default values (see `defaults/main.yml`):

    # whether or not copy scripts enlisted in 'oracle_host_cron_templates_to_copy' to target hosts
    oracle_host_cron_copy_scripts: true
    
    # files to be copied to remote host
    oracle_host_cron_templates_to_copy:
      - file: backup_arch_emergency.ksh.j2
        dest: '{{ oracle_scripts_dir }}/backup_arch_emergency.ksh'
        owner: '{{ oracle_user }}'
        group: '{{ oracle_dba_group }}'
        mode: '0750'
        extra_vars:
          max_arch_usage: "{{ oracle_host_cron_backup_arch_emergency_max_arch_usage | default(70,true) }}"
          nsr_data_volume_pool: "{{ oracle_nsr_data_volume_pool | default('nsr_pool',true) }}"
      - file: rman_delete_expired.ksh.j2
        dest: '{{ oracle_scripts_dir }}/rman_delete_expired.ksh'
        owner: '{{ oracle_user }}'
        group: '{{ oracle_dba_group }}'
        mode: '0750'
        extra_vars:
          rmancat_connection_string: "{{ oracle_rmancat_connection_string | default('rmancat/secret@RMANCAT',true) }}"
      - file: rman_resync_catalog.ksh.j2
        dest: '{{ oracle_scripts_dir }}/rman_resync_catalog.ksh'
        owner: '{{ oracle_user }}'
        group: '{{ oracle_dba_group }}'
        mode: '0750'
        extra_vars:
          rmancat_connection_string: "{{ oracle_rmancat_connection_string | default('rmancat/secret@RMANCAT',true) }}"
    
    # cron config
    oracle_host_cron_config:
      - name: "backup_arch_emergency script to backup archivelogs when FRA utilisation threshold is exceeded"
        job: "{{ oracle_scripts_dir }}/backup_arch_emergency.ksh >/dev/null 2>&1"
        minute: "*/5"
        state: present
        disabled: no
      - name: "rman_delete_expired script deletes records of the expired backups from the recovery catalog"
        job: "{{ oracle_scripts_dir }}/rman_delete_expired.ksh >/dev/null 2>&1"
        hour: "{{ 11 |random }}"
        minute: "{{ 59 |random }}"
        state: present
        disabled: no
      - name: "rman_resync_catalog script to resync backups with the catalog database"
        job: "{{ oracle_scripts_dir }}/rman_resync_catalog.ksh >/dev/null 2>&1"
        hour: "*/6"
        minute: "{{ 59 |random }}"
        state: present
        disabled: no
    
    # whether or not manage cron jobs
    oracle_host_cron_manage_cron_jobs: true
    
    # whether or not disable cron jobs enlisted in 'oracle_host_cron_config' (for example: maintenance/patching)
    oracle_host_cron_disable_jobs: false


Dependencies
------------

This role uses `oracle` role.

Example Playbook
----------------

    - name: Configure Oracle database server cron
      hosts: ora-servers
      gather_facts: true
      become: true
      become_user: '{{ oracle_user }}'

      tasks:

      - import_role:
          name: oracle-host-cron
        vars:
          oracle_host_cron_copy_scripts: true
          oracle_host_cron_disable_jobs: false
          oracle_host_cron_manage_cron_jobs: true
        tags:
          - oracle_host_cron


Inside `vars/main.yml` or `group_vars/..` or `host_vars/..`:
    
    #----------------------------------
    # overrides role 'oracle' variables
    #----------------------------------
    
    # NetWorker backup data volume pool
    oracle_nsr_data_volume_pool: nsrpool
    
    # RMAN catalog TNS connection string
    oracle_rmancat_tnsstring: "RMANCAT = (DESCRIPTION=(ADDRESS=(PROTOCOL=TCP)(HOST=server1)(PORT=1521))(CONNECT_DATA=(SERVER=DEDICATED)(SERVICE_NAME=rmancatdb)))"
    
    # RMAN catalog connection string
    oracle_rmancat_connection_string: "rmancat/secret@RMANCAT"
    
    #--------------------------------------------
    # overrides role 'oracle-host-cron' variables
    #--------------------------------------------
    
    oracle_host_cron_backup_arch_emergency_max_arch_usage: 70
    
    
License
-------

GPLv3 - GNU General Public License v3.0

Author Information
------------------

This role was created in 2018 by [Krzysztof Lewandowski](mailto:Krzysztof.Lewandowski@fastmail.fm).

