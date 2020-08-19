APACHE-2.4 CIS
=========

Configure RHEL and Debian based Apache 2.4 servers to be [CIS] (https://www.cisecurity.org/cis-benchmarks/) compliant.

This role **will make changes to the system** that could break things. This is not an auditing tool but rather a remediation tool to be used after an audit has been conducted.

Based on [CIS Apache HTTP Server 2.4 Benchmark ](https://community.cisecurity.org/collab/public/index.php).


Requirements
------------

You should carefully read through the tasks to make sure these changes will not break your systems before running this playbook.
If you want to do a dry run without changing anything, set the below sections (apache_cis_section1-12) to false. 

Role Variables
--------------

There are many role variables defined in defaults/main.yml. This list shows the most important.

**apache_cis_section1**: CIS - Planning and Installation (Section 1) (Default: true)

**apache_cis_section2**: CIS - Minimize Apache Modules Mo (Section 2) (Default: true)

**apache_cis_section3**: CIS - Principles, Permissions, and Ownership (Section 3) (Default: true)

**apache_cis_section4**: CIS - Apache Access Control (Section 4) (Default: true)

**apache_cis_section5**: CIS - Minimize Features, Content and Options (Section 5) (Default: true)

**apache_cis_section6**: CIS - Operations - Logging, Monitoring and Maintenance (Section 6) (Default: true)  

**apache_cis_section7**: CIS - SSL/TLS Configuration (Section 7) (Default: true)

**apache_cis_section8**: CIS - Information Leakage (Section 8) (Default: true)

**apache_cis_section9**: CIS - Denial of Service Mitigations (Section 9) (Default: true)

**apache_cis_section10**: CIS - Request Limits (Section 10) (Default: true)

**apache_cis_section11**: CIS - Enable SELinux to Restrict Apache Processes (Section 11) (Default: true)

**apache_cis_section12**: CIS - Enable AppArmor to Restrict Apache Processes (Section 12) (Default: true)



##### Apache user and group declarations
apache_rhel_user is the user that the apache software will use for RHEL systems
apache_ubuntu_user is the user that the apache software will use for Ubuntu (Debian) systems
```
apache_rhel_user: apache
apache_ubuntu_user: apache
```
apache_rhel_group is the group the apache user will use for RHEL systems
apache_ubuntu_user is the group the apache user will use for Ubuntu (Debian) systems
```
apache_rhel_group: apache
apache_ubuntu_group: apache
```


##### Apache Principles, Permissions, and Ownership Settings
apache_cis_core_dump_location is the folder for core dumps
```
apache_cis_core_dump_location: /var/log/apache2
```

apache_cis_lockfile_location is the location to the lock file. This can not be the same location as as the DocumentRoot directory. Apache default is ServerRoot logs
The LockFile should be on a locally mounted driver rathare than an NFS mounted file system
apache_cis_lockfile_location = RHEL based
apache2_cis_lockfile_location = Debian based (Ubuntu)
```
apache_cis_lockfile_location: "{{ apache_cis_server_root_dir }}/logs"
apache2_cis_lockfile_location: "/var/lock/apache2"
```


##### Apache Minimize Features, Content and Options
This is the options setting for the web root directory vhost settings. Needs to be None or Multiviews to conform to CIS standards
```
apache_cis_webrootdir_options: None
```


##### Apache allowed file types
This is the list of allowed file types for the FilesMatch directive in httpd.conf/apache.conf
```
apache_cis_allowed_filetypes: "css|html?|js|pdf|txt|xml|xsl|gif|ico|jpe?g|png"
```


##### Apache top level server and IP/Port settings
The hostname of the top level server for RewriteCond %{HTTP_HOST} config section of httpd.conf/apache.conf
```
apache_cis_toplevel_svr: 'www\.example\.com'
```

This is the list of ip's and ports that apache will listen on. If multiples are in use a dash (-) list is used
```
apache_cis_listen_ip_port:
    - 10.0.2.15:80
```


##### Operations - Logging, Monitoring and Maintenance settings
all_mods is the level for everything but but core module. Value bust be notice or lower. The core_mod is the core mod setting and needs to be info or lower. 
`apache_cis_loglevel:
    all_mods: "notice"
    core_mod: "info"`

Path to the apache error logs
apache_cis_errorlog_path: "/var/log/apache2"
The facility setting for error logs. Any appropriate syslog facility can be used in place of local1 and will still conform to CIS standards
```
apache_cis_errorlog_facility: "local1"
```

apache_cis_log_format is the format that the log files will be created in. For compliance with the control
the following need to be present (order does not matter for the CIS control)
%h, %l, %u, %t, %r, %>s, %b, %{Referer}i, and %{User-agent}i
```
apache_cis_log_format: '"%h %l %u %t \"%r\" %>s %b \"%{Referer}i\" \"%{User-Agent}i\""'
```
apache_cis_custom_log is the path for the error log file
```
apache_cis_custom_log: "/var/log/apache2"
```

apache_cis_extra_packages are the extra packages that will need to be updated. Please make this in list format
example format apache_cis_extra_packages: "'apache2', 'telnet', 'openssl'"
```
apache_cis_extra_packages: "'apache2'"
```

To install/configure OWASP requires internet connections. If there is no internet available please set to false
```
apache_cis_owasp_automate: true
```

##### SSL/TLS Configuration settings
When apache_cis_custom_cert set to true the file in files/custom_cert will be copied to the /etc/ssl/certs folder
When apache_cis_custom_cert set to false the control will create a self signed certificate
```
apache_cis_custom_cert: false
```

The hostname used for certificate. It is important to remember that the browser will compare the host name in the URL to the common name in the 
certificate, so that it is important that all https: URL's match the correct host name.
Specifically, the host name www.example.com is not the same as example.com nor the
same as ssl.example.com.
```
apache_cis_hostname_cert: "example.com"
```

When using a cypher (aes128, aes256, etc) when generating an encrypted private key a passphrase is required
```
apache_cis_privatekey_passphrase: "letmein"
```

This will be the final location to your signed certificate
```
apache_cis_csr_folder: "/etc/ssl/private"
```

This is to add the hostname values to the openssl.cnf temp file.
It is recommented (not required) that the first alt name is the common name.
This is a list and must be in the format of DNS.X = <alternet host name>, where X is the next number sequentially
```
apache_cis_alt_names:
    - DNS:www.example.com
    - DNS:example.com
    - DNS:app.example.com
    - DNS:service.example.com
```

The settings below relate to req_distinguished_name section of the openssl.cnf file. The var with the value set relates to the setting it is named after.
```
apache_req_distinguished_name_settings:
    countryName_default: "GB"
    stateOrProvinceName_default: "Scotland"
    localityName_default: "Glasgow"
    organizationName_default: "Example Company Ltd"
    organizationalUnitName_default: "ICT"
    commonName_default: "www.example.com"
    email_address: "blah@mail.com"
```

apache_cis_tls_1_2_available will toggle TLS1.2 or TLSv1 set in ssl.conf. If TLS1.2 is available that is preferred but needs to be setup and TLSv1.0 and TLSv1.1 needs to removed/disabled
```
apache_cis_tls_1_2_available: true
```

apache_cis_sslciphersuite_settings are the settings for the SSLCipherSuite parameter in the ssl.conf configuration.
To conform to the CIS standard for 7.5 (weak ciphers disabled) these settings must have !NULL:!SSLv2:!RC4:!aNULL and it is not recommented to add !SSLv3. Example value: ALL:!EXP:!NULL:!LOW:!SSLv2:!RC4:!aNULL
to conform to the CIS standard for 7.8 (medium ciphers disables) these settings must have !3DES:!IDEA. Example value: ALL:!EXP:!NULL:!LOW:!SSLv2:!RC4:!aNULL:!3DES:!IDEA
```
apache_cis_sslciphersuite_settings: "ALL:!EXP:!NULL:!LOW:!SSLv2:!RC4:!aNULL:!3DES:!IDEA"
```

apache_cis_tls_redirect is the web address that will be used to redirect a tls website or similar
```
apache_cis_tls_redirect: "https://www.cisecurity.org/"
```


##### Information Leakage settings
apache_cis_servertokens needs to be set to either Prod or ProductOnly
```
apache_cis_servertokens: "Prod"
```


##### Denial of Service Mitigations settings
apache_cis_timeout is the apache server timeout, must be set to less than 10 seconds to conform to CIS standards
```
apache_cis_timeout: 10
```

apache_cis_maxkeepaliverequests is the max number of keep alive requests. Needs to be set to 100 or more to conform to CIS standards
```
apache_cis_maxkeepaliverequests: 100
```

apache_cis_keepalivetimeout is the keep alive timout value in seconds. Needs to be set to 15 or less to conform to CIS standards
```
apache_cis_keepalivetimeout: 15
```

apache_cis_reqread_timeout is the value or range of the request read timeout in seconds. The max length can not exceed 40 seconds to conform to CIS standards
```
apache_cis_reqread_timeout: 20-40
```

apache_cis_reqread_body is the value of the request read body timout in seconds. This needs to be set to 20 seconds or less to conform to CIS standards
```
apache_cis_reqread_body: 20
```


##### Request Limits settings
apache_cis_limitrequestline is the limit set to the request line. The value needs to be 512 or shorter to conform to CIS standards
```
apache_cis_limitrequestline: 512
```

apache_cis_limitrequestfields is the limit set to the number of fields. The value needs to be 100 or less to conform to CIS standards
```
apache_cis_limitrequestfields: 100
```

apache_cis_limitrequestfieldsize is the limit set for the size of the request headers. The value needs to be 1024 or less
```
apache_cis_limitrequestfieldsize: 1024
```

apache_cis_limitrequestbody is the limit set for the size of the request body. The value needs to be set to 102400 (100k) or less
```
apache_cis_limitrequestbody: 102400
```


##### Enable SELinux to Restrict Apache Processes settings
apache2_cis_selinux is if you are using AppArmor on Ubuntu instead of SELinux. AppArmor is installed by default with Ubuntu
AppArmor is not supported on RHEL based systems and this toggle will not work with the RHEL implimentation of the CIS role. 
```
apache2_cis_selinux: false
```

Dependencies
------------

Ansible > 2.6.5

Example Playbook
----------------

This sample playbook should be run in a folder that is above the main APACHE-2.4-CIS / APACHE-2.4-CIS-devel folder.

```
- name: Harden Server
  hosts: servers
  become: yes

  roles:
    - APACHE-2.4-CIS
```

Tags
----
Many tags are available for precise control of what is and is not changed.

Some examples of using tags:

```
    # Audit and patch the site
    ansible-playbook site.yml --tags="patch"
```
