nginx
=====

Ansible role which helps to install and configure Nginx web server.

The configuration of the role is done in such way that it should not be
necessary to change the role for any kind of configuration. All can be
done either by changing role parameters or by declaring completely new
configuration as a variable. That makes this role absolutely
universal. See the examples below for more details.

Please report any issues or send PR.


Usage
-----

```
# Example of basic usage of this role
- name: My play
  hosts: all
  roles:
    - nginx

# Example of how to create a new vhost
- name: My play
  hosts: all
  vars:
    # Override the default vhost
    nginx_vhost_config__default:
      default:
        - server:
          - listen 80
          - server_name example.com
          - root /srv/html
          - location /:
              - autoindex on
              - index index.html

    # Add another vhost
    nginx_vhost_config__custom:
      # Creates /etc/nginx/conf.d/test.conf
      test:
        - server:
          - listen 8080
          - server_name web1.example.com
          - "location /":
            - root html
            - index index.html index.htm
  roles:
    - nginx
```


Role variables
--------------

```
# Nginx package (you can specify exact version here)
nginx_pkg: nginx

# Nginx user and group
nginx_user: nginx
nginx_group: nginx

# Whether to install EPEL YUM repo
nginx_epel_install: "{{ yumrepo_epel_install | default(true) }}"

# EPEL YUM repo URL
nginx_epel_yumrepo_url: "{{ yumrepo_epel_url | default('https://dl.fedoraproject.org/pub/epel/$releasever/$basearch/') }}"

# EPEL YUM repo GPG key
nginx_epel_yumrepo_gpgkey: "{{ yumrepo_epel_gpgkey | default('https://dl.fedoraproject.org/pub/epel/RPM-GPG-KEY-EPEL-$releasever') }}"

# Additional EPEL YUM repo params
nginx_epel_yumrepo_params: "{{ yumrepo_epel_params | default({}) }}"

# Whether to install Nginx YUM repo
nginx_yumrepo_install: no

# Pick the right repo based on the distribution (centos|rhel)
nginx_yumrepo_url_distro: centos

# Nginx YUM repo URL
nginx_yumrepo_url: https://nginx.org/packages/{{ nginx_yumrepo_url_distro }}/$releasever/$basearch/

# Nginx YUM repo GPG key
nginx_yumrepo_gpgkey: https://nginx.org/packages/keys/nginx_signing.key

# Additional Nginx YUM repo params
nginx_yumrepo_params: {}

# GPG key for the APT repo
nginx_apt_repo_key: http://nginx.org/keys/nginx_signing.key

# APT repo distribution (ubuntu|debian)
nginx_apt_repo_string_distro: ubuntu

# APT repo string
nginx_apt_repo_string: deb http://nginx.org/packages/{{ nginx_apt_repo_string_distro }}/ {{ ansible_facts.distribution_release }} nginx

# Additional APT repo params
nginx_apt_repo_params: {}


# Path to the nginx.conf file
nginx_config_file: /etc/nginx/nginx.conf

# Values of the default core options
nginx_config_user: "{{ nginx_user }}"
nginx_config_worker_processes: 1
nginx_config_error_log: /var/log/nginx/error.log
nginx_config_pid: /var/run/nginx.pid

# Default core options
nginx_config__core__default:
  - user {{ nginx_config_user }}
  - worker_processes {{ nginx_config_worker_processes }}
  - error_log {{ nginx_config_error_log }}
  - pid {{ nginx_config_pid }}

# Custom core options
nginx_config__core__custom: []

# Core options
nginx_config__core: "{{
  nginx_config__core__default +
  nginx_config__core__custom
}}"

# Default values of events options
nginx_config_worker_connections: 1024

# Default events options
nginx_config__events__default:
  - worker_connections {{ nginx_config_worker_connections }}

# Custom events options
nginx_config__events__custom: []

# Events options
nginx_config__events:
  - events: "{{
      nginx_config__events__default +
      nginx_config__events__custom
    }}"

# Default values of http options
nginx_config_mime_include: /etc/nginx/mime.types
nginx_config_default_type: application/octet-stream
nginx_config_log_format: main '$remote_addr - $remote_user [$time_local] "$request" $status $body_bytes_sent "$http_referer" "$http_user_agent" "$http_x_forwarded_for"'
nginx_config_access_log: /var/log/nginx/access.log main
nginx_config_sendfile: 'on'
nginx_config_keepalive_timeout: 65
nginx_config_conf_include: /etc/nginx/conf.d/*.conf

# Default http options
nginx_config__http__default:
  - include {{ nginx_config_mime_include }}
  - default_type {{ nginx_config_default_type }}
  - log_format {{ nginx_config_log_format }}
  - access_log {{ nginx_config_access_log }}
  - sendfile {{ nginx_config_sendfile }}
  - keepalive_timeout {{ nginx_config_keepalive_timeout }}
  - include {{ nginx_config_conf_include }}

# Custom http options
nginx_config__http__custom: []

# Http options
nginx_config__http:
  - http: "{{
  nginx_config__http__default +
  nginx_config__http__custom
}}"

# Final Nginx config
nginx_config: "{{
  nginx_config__core +
  nginx_config__events +
  nginx_config__http
}}"


# Path to the fastcgi.conf file
nginx_fastcgi_config_file: /etc/nginx/fastcgi_params
nginx_fastcgi_config_file2: /etc/nginx/fastcgi.conf

# Values of FastCGI options
nginx_fastcgi_config_script_filename: $document_root$fastcgi_script_name
nginx_fastcgi_config_query_string: $query_string
nginx_fastcgi_config_request_method: $request_method
nginx_fastcgi_config_content_type: $content_type
nginx_fastcgi_config_content_length: $content_length
nginx_fastcgi_config_script_name: $fastcgi_script_name
nginx_fastcgi_config_request_uri: $request_uri
nginx_fastcgi_config_document_uri: $document_uri
nginx_fastcgi_config_document_root: $document_root
nginx_fastcgi_config_server_protocol: $server_protocol
nginx_fastcgi_config_gateway_interface: CGI/1.1
nginx_fastcgi_config_server_software: nginx/$nginx_version
nginx_fastcgi_config_remote_addr: $remote_addr
nginx_fastcgi_config_remote_port: $remote_port
nginx_fastcgi_config_server_addr: $server_addr
nginx_fastcgi_config_server_port: $server_port
nginx_fastcgi_config_server_name: $server_name
nginx_fastcgi_config_redirect_status: 200

# Provide the possibility to make the SCRIPT_FILENAME option optional for the pre-0.8.30 compatibility
nginx_fastcgi_config__default__script_name:
  - fastcgi_param  SCRIPT_FILENAME    {{ nginx_fastcgi_config_script_filename }}

# Provide the possibility to make the SERVER_NAME option optional for the use in PHP vhosts
nginx_fastcgi_config__default__server_name:
  - fastcgi_param  SERVER_NAME        {{ nginx_fastcgi_config_server_name }}

# Default FastCGI options
nginx_fastcgi_config__default:
  - fastcgi_param  QUERY_STRING       {{ nginx_fastcgi_config_query_string }}
  - fastcgi_param  REQUEST_METHOD     {{ nginx_fastcgi_config_request_method }}
  - fastcgi_param  CONTENT_TYPE       {{ nginx_fastcgi_config_content_type }}
  - fastcgi_param  CONTENT_LENGTH     {{ nginx_fastcgi_config_content_length }}
  - fastcgi_param  SCRIPT_NAME        {{ nginx_fastcgi_config_script_name }}
  - fastcgi_param  REQUEST_URI        {{ nginx_fastcgi_config_request_uri }}
  - fastcgi_param  DOCUMENT_URI       {{ nginx_fastcgi_config_document_uri }}
  - fastcgi_param  DOCUMENT_ROOT      {{ nginx_fastcgi_config_document_root }}
  - fastcgi_param  SERVER_PROTOCOL    {{ nginx_fastcgi_config_server_protocol }}
  - fastcgi_param  GATEWAY_INTERFACE  {{ nginx_fastcgi_config_gateway_interface }}
  - fastcgi_param  SERVER_SOFTWARE    {{ nginx_fastcgi_config_server_software }}
  - fastcgi_param  REMOTE_ADDR        {{ nginx_fastcgi_config_remote_addr }}
  - fastcgi_param  REMOTE_PORT        {{ nginx_fastcgi_config_remote_port }}
  - fastcgi_param  SERVER_ADDR        {{ nginx_fastcgi_config_server_addr }}
  - fastcgi_param  SERVER_PORT        {{ nginx_fastcgi_config_server_port }}
  - fastcgi_param  REDIRECT_STATUS    {{ nginx_fastcgi_config_redirect_status }}

# Custom FastCGI options
nginx_fastcgi_config__custom: []

# Final FastCGI options
nginx_fastcgi_config: "{{
  nginx_fastcgi_config__default__script_name +
  nginx_fastcgi_config__default +
  nginx_fastcgi_config__default__server_name +
  nginx_fastcgi_config__custom
}}"


# Path to the conf.d directory
nginx_vhost_path: /etc/nginx/conf.d

# Values of the default core options
nginx_vhost_config__default_listen: 80 default_server
nginx_vhost_config__default_server_name: _
nginx_vhost_config__default_include: /etc/nginx/default.d/*.conf

# Default core options
nginx_vhost_config__default__core:
  - listen {{ nginx_vhost_config__default_listen }}
  - server_name {{ nginx_vhost_config__default_server_name }}
  - include {{ nginx_vhost_config__default_include }}

# Values of the default root location options
nginx_vhost_config__default__location_root_root: /usr/share/nginx/html
nginx_vhost_config__default__location_root_index: index.html index.htm

# Default root location options
nginx_vhost_config__default__location_root__default:
  - root {{ nginx_vhost_config__default__location_root_root }}
  - index {{ nginx_vhost_config__default__location_root_index }}

# Custom root location options
nginx_vhost_config__default__location_root__custom: []

# Final root location options
nginx_vhost_config__default__location_root:
  - "location /": "{{
      nginx_vhost_config__default__location_root__default +
      nginx_vhost_config__default__location_root__custom
    }}"

# Values of the default 400 error options
nginx_vhost_config__default__location_400_root: /usr/share/nginx/html
nginx_vhost_config__default__location_400_code: 404
nginx_vhost_config__default__location_400_page: /400.html

# Default 400 error options
nginx_vhost_config__default__location_400__default:
  - root {{ nginx_vhost_config__default__location_400_root }}

# Custom 400 error options
nginx_vhost_config__default__location_400__custom: []

# Final 400 error location section
nginx_vhost_config__default__location_400:
  - error_page {{ nginx_vhost_config__default__location_400_code }} {{ nginx_vhost_config__default__location_400_page }}
  # Ansible doesn't replace variables in dict keys :o(
  - "location = /400.html": "{{
      nginx_vhost_config__default__location_400__default +
      nginx_vhost_config__default__location_400__custom
    }}"

# Values of the default 50x error options
nginx_vhost_config__default__location_500_root: /usr/share/nginx/html
nginx_vhost_config__default__location_500_code: 500 502 503 504
nginx_vhost_config__default__location_500_page: /50x.html

# Default 50x error options
nginx_vhost_config__default__location_500__default:
  - root {{ nginx_vhost_config__default__location_500_root }}

# Custom 50x error options
nginx_vhost_config__default__location_500__custom: []

# Final 50x error location section
nginx_vhost_config__default__location_500:
  - error_page {{ nginx_vhost_config__default__location_500_code }} {{ nginx_vhost_config__default__location_500_page }}
  # Ansible doesn't replace variables in dict keys :o(
  - "location = /50x.html": "{{
      nginx_vhost_config__default__location_500__default +
      nginx_vhost_config__default__location_500__custom
    }}"

# Custom options of the default vhost
nginx_vhost_config__default__custom: []

# Default list of vhosts
nginx_vhost_config__default:
  # The "default" here is actually the filename of the "default.conf" file
  default:
    - server: "{{
        nginx_vhost_config__default__core +
        nginx_vhost_config__default__location_root +
        nginx_vhost_config__default__location_400 +
        nginx_vhost_config__default__location_500 +
        nginx_vhost_config__default__custom
      }}"

# Custom vhosts
nginx_vhost_config__custom: {}

# Final list of vhost files and their configurations
nginx_vhost_config: "{{
  nginx_vhost_config__default | combine(
  nginx_vhost_config__custom) }}"
```


Dependencies
------------

- [`config_encoder_filters`](https://github.com/jtyr/ansible-config_encoder_filters)


License
-------

MIT


Author
------

Jiri Tyr
