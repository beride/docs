---
layout: docs
title: On-Premise Installation
category: registry
forkurl: https://github.com/coreos/docs/blob/master/enterprise-registry/initial-setup/index.md
weight: 5
---

# On-Premise Installation

CoreOS Enterprise Registry requires five components to operate successfully:

- A supported database (MySQL, Postgres)
- A Redis instance (for real-time events)
- A config.yaml file
- An SMTP server
- The Enterprise Registry image


## Preparing the Database

A MySQL RDBMS or Postgres installation with an empty database is required, and a login with full access to said database. The schema will be created the first time the registry image is run. The database install can either be pre-existing or run on CoreOS via a [Docker container]({{site.url}}/docs/enterprise-registry/mysql-container).

Please have the url for the login and database available in the SQLAlchemy format:

### For MySQL:

```
mysql+pymysql://<username>:<url escaped password>@<hostname>/<database_name>
```

### For Postgres:

```
postgresql://<username>:<url escaped password>@<hostname>/<database_name>
```

## Setting up Redis

Redis stores data which must be accessed quickly but doesn’t necessarily require durability guarantees. If you have an existing Redis instance, make sure to accept incoming connections on port 6379 (or change the port in the config.yaml) and then feel free to skip this step.

To run redis, simply pull and run the Quay.io Redis image:

```
sudo docker pull quay.io/quay/redis
sudo docker run -d -p 6379:6379 quay.io/quay/redis
```

**NOTE**: This host will have to accept incoming connections on port 6379 from the hosts on which the registry will run.


## Enterprise Registry Config File

CoreOS Enterprise Registry requires a `config.yaml` file, that stores database connection information, the storage location of your containers and other important settings.

Sample configuration can be found below. Any fields marked as `(FILL IN HERE)` are required to be edited.

```yaml
# A unique secret key. This should be a UUID or some other secret
# string.
SECRET_KEY: '(FILL IN HERE: secret key)'

# Should be 'https' if SSL is used and 'http' otherwise.
PREFERRED_URL_SCHEME: '(FILL IN HERE: "https" or "http")'

# The HTTP host (and optionally the port number) of the location
# where the registry will be accessible on the network.
SERVER_HOSTNAME: '(FILL IN HERE: registry.mycorp.com)'

# A logo to use for your enterprise
ENTERPRISE_LOGO_URL: '(FILL IN HERE: http://someurl/...)'

# Contact information for the contact page.
# Supported URL forms:
#   email: mailto:some@email.com
#   IRC: irc://server:port/channel
#   telephone: tel:number
#   twitter: https://twitter.com/twitterhandle
#   generic URL: http(s)://*
CONTACT_INFO:
  - 'mailto:(FILL IN HERE: support@email.com)'
  - 'tel:(FILL IN HERE: +1-555-555-5555)'

# Settings for SMTP and mailing. This is *required*. Note: "localhost" will not work since the
# registry is run inside a container. Delete or use null values for username and password if
# you are running an auth-less mailserver.
MAIL_PORT: 587
MAIL_PASSWORD: '(FILL IN HERE: password)'
MAIL_SERVER: '(FILL IN HERE: hostname)'
MAIL_USERNAME: '(FILL IN HERE: username)'
MAIL_USE_TLS: true

# To change the mail sender:
# MAIL_DEFAULT_SENDER: '(EMAIL ADDRESS: validated email address)'

# The database URI for your MySQL or Postgres DB.
DB_URI: '(FILL IN HERE: database uri)'

# REDIS connection information. The 'host' entry in the dictionary is
# *required* and should refer to the REDIS host setup above. Note that the
# host should *not* include the port.
#
# Additional options:
#   port: The port to use when connecting to REDIS.
#   password: The password to use when connecting to REDIS.
#
BUILDLOGS_REDIS: {'host': '(FILL IN HERE: redis host)'}
USER_EVENTS_REDIS: {'host': '(FILL IN HERE: redis host)'}

# The usernames of your super-users, if any. Super users will
# have the ability to view and delete other users.
FEATURE_SUPER_USERS: false
SUPER_USERS: []

# Either 'Database' or 'LDAP'.
# If LDAP, additional configuration is required below.
AUTHENTICATION_TYPE: 'Database'

# Should always be 'local'.
DISTRIBUTED_STORAGE_PREFERENCE: ['local']

# Defines the kind of storage used by the registry:
#  LocalStorage: Registry data is stored on a local mounted volume
#
#     Required fields:
#       storage_path: The path under the mounted volume
#
#  S3Storage: Registry data is stored in Amazon S3
#
#     Required fields:
#       storage_path: The path under the S3 bucket
#       s3_access_key: The S3 access key
#       s3_secret_key: The S3 secret key
#       s3_bucket: The S3 bucket
#
#  GoogleCloudStorage: Registry data is stored in GCS
#
#     Required fields:
#       storage_path: The path under the GCS bucket
#       access_key: The GCS access key
#       secret_key: The GCS secret key
#       bucket_name: The GCS bucket
#
#  RadosGWStorage: Registry data is stored in Ceph Object Gateway (RADOS)
#                  See: http://ceph.com/docs/master/radosgw/admin/
#
#     Required fields:
#       hostname: The hostname at which RADOS is running
#       is_secure: Whether to use a secure connection
#       storage_path: The path under RADOS
#       access_key: An object gateway user access key
#       secret_key: An object gateway user secret key
#       bucket_name: The bucket under RADOS
DISTRIBUTED_STORAGE_CONFIG:
 local:
    # The name of the storage provider
    - LocalStorage

    # Fields, in dictionary form
    - {'storage_path': '/datastorage/registry'}

# LDAP information (only needed if `LDAP` is chosen above).
# LDAP_URI: 'ldap://localhost'
# LDAP_ADMIN_DN: 'cn=admin,dc=devtable,dc=com'
# LDAP_ADMIN_PASSWD: 'secret'
# LDAP_BASE_DN: ['dc=devtable', 'dc=com']
# LDAP_EMAIL_ATTR: 'mail'
# LDAP_UID_ATTR: 'uid'
# LDAP_USER_RDN: ['ou=People']

# If Active Directory is used for LDAP services:
# LDAP_URI: 'ldap://globalcatalog.dev.company.com:3268'
# LDAP_ADMIN_DN: 'cn=service_user,ou=serviceaccounts,dc=dev,dc=company,dc=com'
# LDAP_ADMIN_PASSWD: 'secret'
# LDAP_BASE_DN: ['dc=dev','dc=company','dc=com']
# LDAP_EMAIL_ATTR: 'mail'
# LDAP_UID_ATTR: 'sAMAccountName'
# To search for users across all OUs and CNs in AD simply leave the following line commented out:
# LDAP_USER_RDN: ['cn=Users']

# Where user files (uploaded build packs, other binary data)
# are stored. Must match a key under DISTRIBUTED_STORAGE_CONFIG.
USERFILES_LOCATION: 'local'

# The path under the storage where user files are stored. If the storage has a storage_path,
# this will be a sub-directory under that path.
USERFILES_PATH: 'userfiles/'

# Required constants.
TESTING: false
USE_CDN: false
FEATURE_USER_LOG_ACCESS: true
FEATURE_BUILD_SUPPORT: false
```

## Setting up the Directories

CoreOS Enterprise registry requires a storage directory and a configuration directory containing the `config.yaml`, and, if SSL is used, two files named `ssl.cert` and `ssl.key`:

	mkdir storage
	mkdir config
	mv config.yaml config/config.yaml
	cp my-ssl-cert config/ssl.cert
	cp my-ssl-key config/ssl.key


## Accessing the Enterprise Registry Container

After signing up you will receive a `.dockercfg` file containing your credentials to the `quay.io/coreos/registry` repository.
Save this file to your CoreOS machine in `/home/core/.dockercfg` and `/root/.dockercfg`.
You should now be able to execute `docker pull quay.io/coreos/registry` to download the container.


## Running the Registry

The CoreOS Enterprise Registry is run via a `docker run` call, with the `config` and `storage` being the directories created above.

	docker run -p 443:443 -p 80:80 --privileged=true -v /local/path/to/config:/conf/stack -v /local/path/to/storage:/datastorage -d quay.io/coreos/registry


## Verifying the Registry status

Visit the `/status` endpoint on the registry hostname and verify. The expected values are:

Variable Name     | Expected Value | Error Description
----------------- | -------------- | ------------------------------------------------------------------
db_healthy        | true           | If `false`, then the SQL connection failed.
buildlogs_healthy | true           | If `false`, then the Redis connection failed.
is_testing        | false          | If `true`, then the configuration volume was not mounted properly.

Note: `is_testing` may be missing from older installations.

## Logging in

### If using database authentication:

Once the Enterprise Registry is running, new users can be created by clicking the `Sign Up` button. The sign up process will require an e-mail confirmation step, after which repositories, organizations and teams can be setup by the user.


### If using LDAP authentication:

Users should be able to login to the Enterprise Registry directly with their LDAP username and password.

To aid in LDAP debugging you can [tail the logs of the Enterprise Registry container]({{site.url}}/docs/enterprise-registry/log-debugging) from the Docker host.

## Ensuring storage is set up correctly

### If using local storage

There's nothing left to do! Enjoy a brand new Enterprise Registry!

### If using a storage service

When using a storage service, the service must have its CORS (Cross-Origin Resource Sharing) configuration altered. This is usually done in the service's web console while configuring the settings for a given bucket. Enterprise Registry requires that both GET and PUT methods be accessible from any origin because userfiles (Dockerfiles and archives with Dockerfiles in them) are uploaded directly from the browser to storage. If this is not properly set up, uploading a Dockerfile/archive will yield an Internal Server Error (500) page.

An example Amazon S3 configuration looks like:

```
<?xml version="1.0" encoding="UTF-8"?>
<CORSConfiguration xmlns="http://s3.amazonaws.com/doc/2006-03-01/">
    <CORSRule>
        <AllowedOrigin>*</AllowedOrigin>
        <AllowedMethod>GET</AllowedMethod>
        <MaxAgeSeconds>3000</MaxAgeSeconds>
        <AllowedHeader>Authorization</AllowedHeader>
    </CORSRule>
    <CORSRule>
        <AllowedOrigin>*</AllowedOrigin>
        <AllowedMethod>PUT</AllowedMethod>
        <MaxAgeSeconds>3000</MaxAgeSeconds>
        <AllowedHeader>Content-Type</AllowedHeader>
        <AllowedHeader>x-amz-acl</AllowedHeader>
        <AllowedHeader>origin</AllowedHeader>
    </CORSRule>
</CORSConfiguration>
```

For more information see [Amazon's CORS documentation](http://docs.aws.amazon.com/AmazonS3/latest/dev/cors.html).

An example Google Cloud Storage configuration looks like:

```
<?xml version="1.0" encoding="UTF-8"?>
<CorsConfig>
  <Cors>
    <Origins>
      <Origin>*</Origin>
    </Origins>
    <Methods>
      <Method>GET</Method>
      <Method>PUT</Method>
    </Methods>
    <ResponseHeaders>
      <ResponseHeader>Content-Type</ResponseHeader>
    </ResponseHeaders>
    <MaxAgeSec>3000</MaxAgeSec>
  </Cors>
</CorsConfig>
```

For more information see [Google's CORS documentation](https://cloud.google.com/storage/docs/cross-origin).
