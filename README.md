satellite_maintenance
=========

This repository is a collection of Ansible playbooks that can be used from Ansible Tower to do things like the following:

* Publish new versions of Content Views and Composite Content Views called `cv_publish.yml`
* Remove old Content Views and Composite Content Views call `cv_cleanup.yml`

Requirements
------------

---

This playbook or playbooks require the `theforeman.foreman` ansible collection. So add the following to the `requirements.yml` under the collections directory:

```
collections:
  - name: theforeman.foreman
```

See this website for documentation:

[foreman-ansible-modules](https://github.com/theforeman/foreman-ansible-modules)

satellite_maintenance Variables
--------------

---

`cv_cleanup.yml` requires two variable to be passed on the command-line or an User Survey in Ansible Tower in a template:

* `survey_content_view_version_cleanup_keep` -- is the number of versions of Content Views or Composite Content Views to keep.

* `survey_content_view_version_cleanup_search` -- is a search critria that can be used in the following manner:

  `name ~ some_view`



Dependencies
------------
---

See above in the Requirements sections.

satellite Project Playbook
----------------

Here is the `cv_cleanup.yml` playbook:

```
---
# cv_cleanup.yml for satellite

- name: Clean Up Satellite Content Views
  hosts: localhost
  gather_facts: false
  vars_files:
    - vars/server.yml

  roles:
    - name: remove all unused content views
      role: theforeman.foreman.content_view_version_cleanup
      vars:
        content_view_version_cleanup_keep: '{{ survey_content_view_version_cleanup_keep }}' 
        content_view_version_cleanup_search: '{{ survey_content_view_version_cleanup_search }}'

```

Here is the `cv_publish.yml` playbook:

```
---
# cv_publish.yml for satellite

- name: Clean Up Satellite Content Views
  hosts: localhost
  gather_facts: false
  vars_files:
    - vars/server.yml
  environment: "{{ foreman_env }}"

  tasks:
  - name: Find All CVs
    theforeman.foreman.resource_info:
      resource: content_views
    register: content_views_list
  
  - name: Print Out Content View
    debug:
      msg: '{{ item.name }}'
    with_items:
      - "{{ content_views_list.resources }}"
    when: 
      - item.name != 'Default Organization View'
      - item.composite == false

  - name: Publish New Versions of the Content Views
    theforeman.foreman.content_view_version:
      organization: '{{ organization }}'
      content_view: '{{ item.name }}'
    with_items:
      - "{{ content_views_list.resources }}" 
    when: 
      - item.name != 'Default Organization View'
      - item.composite == false

```

License
-------
---

GPL

Author Information
------------------
---

Scott Parker - Red Hat North American Consulting

Credits
-------------------
---
Credit goes to the authors for the [foreman-ansible-modules](https://github.com/theforeman/foreman-ansible-modules) 