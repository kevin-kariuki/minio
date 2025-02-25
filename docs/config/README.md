# MinIO Server Config Guide [![Slack](https://slack.min.io/slack?type=svg)](https://slack.min.io) [![Docker Pulls](https://img.shields.io/docker/pulls/minio/minio.svg?maxAge=604800)](https://hub.docker.com/r/minio/minio/)

## Configuration Directory

Till MinIO release `RELEASE.2018-08-02T23-11-36Z`, MinIO server configuration file (`config.json`) was stored in the configuration directory specified by `--config-dir` or defaulted to `${HOME}/.minio`. However from releases after `RELEASE.2018-08-18T03-49-57Z`, the configuration file (only), has been migrated to the storage backend (storage backend is the directory passed to MinIO server while starting the server).

You can specify the location of your existing config using `--config-dir`, MinIO will migrate the `config.json` to your backend storage. Your current `config.json` will be renamed upon successful migration as `config.json.deprecated` in your current `--config-dir`. All your existing configurations are honored after this migration.

Additionally `--config-dir` is now a legacy option which will is scheduled for removal in future, so please update your local startup, ansible scripts accordingly.

```sh
minio server /data
```

### Certificate Directory

TLS certificates by default are stored under ``${HOME}/.minio/certs`` directory. You need to place certificates here to enable `HTTPS` based access. Read more about [How to secure access to MinIO server with TLS](https://docs.min.io/docs/how-to-secure-access-to-minio-server-with-tls).

Following is the directory structure for MinIO server with TLS certificates.

```sh
$ mc tree --files ~/.minio
/home/user1/.minio
└─ certs
   ├─ CAs
   ├─ private.key
   └─ public.crt
```

You can provide a custom certs directory using `--certs-dir` command line option.

#### Region
| Field                     | Type     | Description                                                                                                                                                                             |
|:--------------------------|:---------|:----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| ``region name=my_region`` | _string_ | `region` describes the physical location of the server. By default it is blank. You may override this field with `MINIO_REGION_NAME` environment variable. If you are unsure leave it unset. |

Example:

```sh
export MINIO_REGION_NAME="my_region"
minio server /data
```

#### Worm

| Field                  | Type     | Description                                                                                                                                       |
|:-----------------------|:---------|:--------------------------------------------------------------------------------------------------------------------------------------------------|
| ``worm state=on`` | _string_ | Enable this to turn on Write-Once-Read-Many. By default it is set to `off`. You may override this field with ``MINIO_WORM`` environment variable. |

Example:

```sh
export MINIO_WORM_STATE=on
minio server /data
```

### Storage Class

| Field                          | Type     | Description                                                                                                                                                                                  |
|:-------------------------------|:---------|:---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| ``storageclass``               |          | Set storage class for configurable data and parity, as per object basis.                                                                                                                     |
| ``storageclass standard=EC:4`` | _string_ | Value for standard storage class. It should be in the format `EC:Parity`, for example to set 4 disk parity for standard storage class objects, set this field to `EC:4`.                     |
| ``storageclass rrs=EC:2``      | _string_ | Value for reduced redundancy storage class. It should be in the format `EC:Parity`, for example to set 3 disk parity for reduced redundancy storage class objects, set this field to `EC:3`. |

By default, parity for objects with standard storage class is set to `N/2`, and parity for objects with reduced redundancy storage class objects is set to `2`. Read more about storage class support in MinIO server [here](https://github.com/minio/minio/blob/master/docs/erasure/storage-class/README.md).

### Cache
| Field                                                        | Type       | Description                                                                                                 |
|:-------------------------------------------------------------|:-----------|:------------------------------------------------------------------------------------------------------------|
| ``cache drives="/mnt/drive1;/mnt/drive2;/mnt/cache{1...3}"`` | _[]string_ | List of mounted file system drives with [`atime`](http://kerolasa.github.io/filetimes.html) support enabled |
| ``cache exclude="*.pdf;mybucket/*"``                         | _[]string_ | List of wildcard patterns for prefixes to exclude from cache                                                |
| ``cache expiry=90``                                          | _int_      | Days to cache expiry                                                                                        |
| ``cache quota=70``                                           | _int_      | Percentage of disk available to cache                                                                       |
|                                                              |            |                                                                                                             |

#### Notify

| Field                    | Type | Description                                                                                                                           |
|:-------------------------|:-----|:--------------------------------------------------------------------------------------------------------------------------------------|
| ``notify_amqp``          |      | [Configure to publish MinIO events via AMQP target.](https://docs.min.io/docs/minio-bucket-notification-guide#AMQP)                   |
| ``notify_nats``          |      | [Configure to publish MinIO events via NATS target.](https://docs.min.io/docs/minio-bucket-notification-guide#NATS)                   |
| ``notify_elasticsearch`` |      | [Configure to publish MinIO events via Elasticsearch target.](https://docs.min.io/docs/minio-bucket-notification-guide#Elasticsearch) |
| ``notify_redis``         |      | [Configure to publish MinIO events via Redis target.](https://docs.min.io/docs/minio-bucket-notification-guide#Redis)                 |
| ``notify_postgresql``    |      | [Configure to publish MinIO events via PostgreSQL target.](https://docs.min.io/docs/minio-bucket-notification-guide#PostgreSQL)       |
| ``notify_kafka``         |      | [Configure to publish MinIO events via Apache Kafka target.](https://docs.min.io/docs/minio-bucket-notification-guide#apache-kafka)   |
| ``notify_webhook``       |      | [Configure to publish MinIO events via Webhooks target.](https://docs.min.io/docs/minio-bucket-notification-guide#webhooks)           |
| ``notify_mysql``         |      | [Configure to publish MinIO events via MySql target.](https://docs.min.io/docs/minio-bucket-notification-guide#MySQL)                 |
| ``notify_mqtt``          |      | [Configure to publish MinIO events via MQTT target.](https://docs.min.io/docs/minio-bucket-notification-guide#MQTT)                   |

### Accessing configuration file
All configuration changes can be made using [`mc admin config` get/set commands](https://github.com/minio/mc/blob/master/docs/minio-admin-complete-guide.md). Following sections provide brief explanation of fields and how to customize them. A complete example of `config.json` is available [here](https://raw.githubusercontent.com/minio/minio/master/docs/config/config.sample.json)

## Environment only settings

### Browser

Enable or disable access to web UI. By default it is set to `on`. You may override this field with `MINIO_BROWSER` environment variable.

Example:

```sh
export MINIO_BROWSER=off
minio server /data
```

### Domain

By default, MinIO supports path-style requests that are of the format http://mydomain.com/bucket/object. `MINIO_DOMAIN` environment variable is used to enable virtual-host-style requests. If the request `Host` header matches with `(.+).mydomain.com` then the matched pattern `$1` is used as bucket and the path is used as object. More information on path-style and virtual-host-style [here](http://docs.aws.amazon.com/AmazonS3/latest/dev/RESTAPI.html)
Example:

```sh
export MINIO_DOMAIN=mydomain.com
minio server /data
```

For advanced use cases `MINIO_DOMAIN` environment variable supports multiple-domains with comma separated values.
```sh
export MINIO_DOMAIN=sub1.mydomain.com,sub2.mydomain.com
minio server /data
```

## Explore Further
* [MinIO Quickstart Guide](https://docs.min.io/docs/minio-quickstart-guide)
* [Configure MinIO Server with TLS](https://docs.min.io/docs/how-to-secure-access-to-minio-server-with-tls)
