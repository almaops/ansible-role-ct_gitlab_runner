freehck.gitlab_runner
=========

This role deploys gitlab-runner in docker container, and register it in gitlab.

Role Variables
--------------
`gitlab_runner_name`: the name that will be seen on runners page, default is `"{{ inventory_hostname }}"`

`gitlab_runner_ct_name`: container name, default is `"gitlab-runner"`, change it if you're going to run multiple runners on the same host

`gitlab_runner_registration_token`: this token is needed to register, can be acquired from project for specific runners, and from admin panel for shared runners; mandatory parameter, no default

`gitlab_runner_coordinator_url`: address of your gitlab, by default is configured to `"https://gitlab.com/ci"`; change it if you have an on-premise gitlab installation

`gitlab_runner_tags`: subj; default is `[]`

`gitlab_runner_executor`: subj; default is `"docker"` as it's the most convenient way; no other executors are tested

`gitlab_runner_parallel_builds_number`: the default is `"1"`

`gitlab_runner_docker_image`: the default is `"alpine"`

`gitlab_runner_srv_dir`: base dir to determine where to store service configs, default is `"/srv"`

`gitlab_runner_cfg_dir`: final dir, where service configs are stored, default is `"{{ gitlab_runner_srv_dir }}/{{ gitlab_runner_ct_name }}/config"`

`gitlab_runner_cfg_file`: runner config file, default is `"{{ gitlab_runner_cfg_dir }}/config.toml"`; do not change it, it's really `config.toml`, I swear

`gitlab_runner_ct_image`: image of gitlab-runner container, default is `"gitlab/gitlab-runner:latest"`; change only if you want to have some specific runner version

`gitlab_runner_ct_restart_policy`: container restart policy, default is `"always"`

`gitlab_runner_ct_state`: container state after deploy, default is `"started"`

`gitlab_runner_ct_restart`: if we need to restart container after deploy, default is `false`

`gitlab_runner_ct_pull`: if we need to try to pull the image while deploy, default is `true`

`gitlab_runner_cache_s3_enabled`: if we're going to use S3/Minio for caches, default is `false`

`gitlab_runner_cache_s3_server_address`: subj, mandatory if s3 enabled, no default

`gitlab_runner_cache_s3_access_key`: subj, mandatory if s3 enabled, no default

`gitlab_runner_cache_s3_secret_key`: subj, mandatory if s3 enabled, no default

`gitlab_runner_force_reg`: if we want to re-register this runner; default is `false`

`gitlab_runner_docker_volumes`: additional mounts for containers, default is `[]`; useful if you want to pass `docker.sock` or to mount `/cache` volume

`gitlab_runner_dind_enabled`: set to `true` if you need your docker containers to run in priveleged mode; default is `false`

Example Playbook
----------------

    - hosts:
        - gitlab_runners
      become: true
      vars:
        minio_bind_ip: "{{ backnet_ip }}"
        minio_access_key: "<...>"
        minio_secret_key: "<...>"
        gitlab_runner_registration_token: "<...>"
        gitlab_runner_cache_s3_enabled: true
        gitlab_runner_cache_s3_server_address: "{{ backnet_ip }}:9000"
        gitlab_runner_cache_s3_access_key: "{{ minio_access_key }}"
        gitlab_runner_cache_s3_secret_key: "{{ minio_secret_key }}"
        gitlab_runner_docker_volumes:
            - "/srv/{{ gitlab_runner_name }}/cache:/cache"
            - "/var/run/docker.sock:/var/run/docker.sock"
        gitlab_runner_names:
          - "gitlab-runner"
      pre_tasks:
        - name: create cache dirs
          file:
            path: "/srv/{{ gitlab_runner_name }}/cache"
            state: directory
            mode: 0755
            recurse: true
          loop: "{{ gitlab_runner_names }}"
          loop_control:
            loop_var: gitlab_runner_name
      roles:
        - role: freehck.gitlab_runner
          loop: "{{ gitlab_runner_names }}"
          loop_control:
            loop_var: gitlab_runner_name
          tags:
            - runner


License
-------
MIT

Author Information
------------------
Dmitrii Kashin, <freehck@freehck.ru>
