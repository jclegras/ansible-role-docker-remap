Role Name
=========

This Ansible role allow people to enable the docker remapping feature: https://integratedcode.us/2015/10/13/user-namespaces-have-arrived-in-docker/

Requirement
------------

**Docker version 1.10** min installed on your system with an OS with user namespace enabled (3.10 Linux kernel)

Role Variables
--------------

    # Basic remapping options
    dockremap_docker_options_userns_remap: dockremap
    dockremap_docker_options_directory: /etc/docker
    
    # Subordinates files configuration
    dockremap_subordinate_file_length: 65536
    dockremap_subordinate_file_reference_user: "{{ dockremap_docker_options_userns_remap }}"
    dockremap_subordinate_file_numerical_subordinate_id: 500000
    dockremap_subordinate_file_group: /etc/subgid
    dockremap_subordinate_file_user: /etc/subuid
    
    # Groups and users for mapping
    dockremap_groups:
      reference_user:
        name: "{{ dockremap_docker_options_userns_remap }}"
        gid: 0
    
    dockremap_users:
      reference_group:
        name: "{{ dockremap_docker_options_userns_remap }}"
        group: "{{ dockremap_docker_options_userns_remap }}"
        uid: 0

Dependencies
------------

None

Example Playbook
----------------
       
1) Change the reference user (default dockremap)

    ---
    hosts: all
    become: true
    vars:
      dockremap_docker_options_userns_remap: myreference
    roles:
      - jclegras.docker-remap         
      
2) Change the numerical subordinate ID (default 500000)

    ---
    hosts: all
    become: true
    vars:
      dockremap_subordinate_file_numerical_subordinate_id: 100000
    roles:
      - jclegras.docker-remap   
      
3) Add new users and groups for mapping

    ---
    - hosts: all
      become: true
      vars:
        dockremap_groups:
          dockremap_factory:
            name: dockremap-jenkins
            gid: 1000
            
        dockremap_users:
          dockremap_factory:
            name: dockremap-jenkins
            group: dockremap-jenkins
            uid: 1000
      roles:
        - jclegras.docker-remap
         
Note
----

*Let dockremap_subordinate_file_numerical_subordinate_id = 100000 and*
*let dockremap_subordinate_file_length = 65536*

Your mapping will be:

      | User Host ID   |  User Container ID|
      |----------------|:------------------|
      | 100000         | 0                 |
      | ...            | ...               |
      | 101000         | 1000              |
      | ...            | ...               |
      | 165536         | 65536             |

  
**Rule:**

*offset in [0..dockremap_subordinate_file_length]*
      
    User Host ID = User Host ID + offset
    User Container ID = offset
    
Caveat
------

For Red Hat, you have to enable the user namespace first.
http://goyalankit.com/blog/2016/06/25/user-namespace-in-red-hat-enterprise-linux-7-dot-2/

License
-------

BSD

Author Information
------------------

Jean-Charles Legras
