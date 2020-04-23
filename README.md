
# Configure letsencrypt certificates via dehydrated

- configured in three steps
- the client sofware includes the role with the right parms, creating all
  expected cert files with snake-oil contents
- the client software configures a webserver with the right
  pass through for .well-known URLs and loading the right cert filenames
- the presence of a "dehydrated" key in host_vars allows the
  main dehydrated module to run, which creates the real certs and schedules
  weekly checks for updates

Note: for this process to work, the dehydrated role must come after any roles
that install and configure the webserver serving the acme-challenges.

Warning: due to the fail-safe nature of the snake-oil used to make sure
the client software always sees valid certificates, the first time a new
domain is added, a manual dehydrated run is needed:
    sudo dehydrated -c --force $domain

(Yes, this is something that should be fixed)

## Example webserver config

nginx:

```
location /.well-known/acme-challenge {
    alias /var/lib/dehydrated/acme-challenges;
    try_files $uri =404;
}
```

## Example include from client

```
- name: Register domain with dehydrated
  include_role:
    name: dehydrated
    tasks_from: client
  vars:
    param:
        hostname: site.example.com
```

## Example enable dehydrated

host vars
```
dehydrated:
```
