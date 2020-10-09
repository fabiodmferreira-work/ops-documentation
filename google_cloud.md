Google Cloud Integration
========================

Ops can integrate with your existing Google Cloud Platform (GCP) account. You can use Ops CLI to create and upload an image in GCP account.
Once, you have uploaded image, you can also create an instance with a particular image using CLI.

## Pre-requisites

1. Create a Service Account (SA) in your GCP account and download the Service Account key json file.
2. Please make sure your Service Account has access to the Google Compute Engine and Google Storage.
3. Get the name of your Google Cloud account project where you would be creating images and instances.
4. Create a bucket in Google Cloud storage for image artifacts storage.
5. Please make sure you export `GOOGLE_APPLICATION_CREDENTIALS` with the Service Account key json file path, before invoking below commands.
```sh
$ export GOOGLE_APPLICATION_CREDENTIALS=~/service-key.json
```

## Image Operations
### Create Image

If you have already created an Ops project, you can update your existing `config.json`.
You need to add [CloudConfig](configuration.md#cloudconfig) which mentions specific details like `ProjectID`, `BucketName`.

```json
{
    "CloudConfig" :{
        "ProjectID" :"prod-1000",
        "Zone": "us-west1-b",
        "BucketName":"my-deploy"
    },
    "RunConfig" : {
        "Memory": "2G"
    }
}
```

Once, you have updated `config.json` you can create an image in Google Cloud with the following command.

```sh
$ ops image create -c config.json <image_name> -t gcp
```

For creating an image using a particular package, you need to provide the package name to `ops image create` command with `-p` option.

```sh
$ ops image create -c config.json -p node_v14.2.0 -a ex.js -i <image_name> -t gcp
```

### List Images

You can list existing images on Google cloud with `ops image list`.

```sh
$ ops image list
+--------------------+--------+-------------------------------+
|        NAME        | STATUS |            CREATED            |
+--------------------+--------+-------------------------------+
| nanos-main-image   | READY  | 2019-03-21T15:06:17.567-07:00 |
+--------------------+--------+-------------------------------+
| nanos-node-image   | READY  | 2019-04-16T23:16:03.145-07:00 |
+--------------------+--------+-------------------------------+
| nanos-server-image | READY  | 2019-03-21T15:50:04.456-07:00 |
+--------------------+--------+-------------------------------+
```

### Delete Image

`ops image delete <imagename>` can be used to delete an image from Google Cloud.

```
$ ops delete image nanos-main-image
```

## Instance Operations
### Create Instance

After the successful creation of an image in Google Cloud, we can create an instance from an existing image.

You need to export `GOOGLE_APPLICATION_CREDENTIALS` and pass project-id and zone with cli options.

```sh
$ export GOOGLE_APPLICATION_CREDENTIALS=<credentials_file_path>
$ ops instance create -g prod-1000 -z us-west1-b -i <image_name> -t gcp
```

Alternatively, you can pass config, if you have mentioned project-id and zone in project's config.json.
```
$ ops instance create -c config.json -i <image_name>
```

### List Instances

You can list instance on Google Cloud using `ops instance list` command.

You need to export `GOOGLE_APPLICATION_CREDENTIALS`, `GOOGLE_CLOUD_PROJECT` and `GOOGLE_CLOUD_ZONE` before firing command.

```sh
$ export GOOGLE_CLOUD_PROJECT=prod-1000
$ export GOOGLE_CLOUD_ZONE=us-west1-b
$ ops instance list
+-----------------------------+---------+-------------------------------+-------------+--------------+
|            NAME             | STATUS  |            CREATED            | PRIVATE IPS |  PUBLIC IPS  |
+-----------------------------+---------+-------------------------------+-------------+--------------+
| nanos-main-image-1556601450 | RUNNING | 2019-04-29T22:17:34.609-07:00 | 10.240.0.40 | 34.83.204.40 |
+-----------------------------+---------+-------------------------------+-------------+--------------+
```

Alternatively you can pass project-id and zone with cli options.
```sh
$ ops instance list -g prod-1000 -z us-west1-b
```

## Get Logs for Instance

You can get logs from serial console of a particular instance using `ops instance logs` command.

You need to export `GOOGLE_APPLICATION_CREDENTIALS`, `GOOGLE_CLOUD_PROJECT` and `GOOGLE_CLOUD_ZONE` before firing command.

```sh
$ export GOOGLE_CLOUD_PROJECT=prod-1000
$ export GOOGLE_CLOUD_ZONE=us-west1-b
$ ops instance logs <instance_name> -t gcp
```

Alternatively you can pass project-id and zone with cli options.
```sh
$ ops instance logs -g prod-1000 -z us-west1-b
```

### Delete Instance

`ops instance delete` command can be used to delete instance on Google Cloud.

You need to export `GOOGLE_APPLICATION_CREDENTIALS`, `GOOGLE_CLOUD_PROJECT` and `GOOGLE_CLOUD_ZONE` before firing command.

```sh
$ export GOOGLE_CLOUD_PROJECT=prod-1000
$ export GOOGLE_CLOUD_ZONE=us-west1-b
$ ops instance delete my-instance-running
```

Alternatively you can pass project-id and zone with cli options.
```sh
$ ops instance delete -g prod-1000 -z us-west1-b my-instance-running
```

## Volume Operations
### Create Volume

You need to set the `BucketName`, `ProjectID` and `Zone` in the `CloudConfig` section of your configuration file and export `GOOGLE_APPLICATION_CREDENTIALS` before firing the command.

```json
{
    "CloudConfig" :{
        "ProjectID" :"prod-1000",
        "Zone": "us-west1-b",
        "BucketName":"my-deploy"
    }
}
```

```sh
$ export GOOGLE_APPLICATION_CREDENTIALS=<credentials_file_path>
$ ops volume create <volume_name> -t gcp -c <configuration_file_path>
```

For create a volume with existing files you can add the `-d` flag and the directory path.
```sh
$ export GOOGLE_APPLICATION_CREDENTIALS=<credentials_file_path>
$ ops volume create <volume_name> -t gcp -c <configuration_file_path> -d <directory_path>
```

### List Volumes

You can list volumes on Google Cloud using `ops volume list -t gcp -c <configuration_file_path>` command.

You need to set the `ProjectID` and `Zone` in the `CloudConfig` section of your configuration file and export `GOOGLE_APPLICATION_CREDENTIALS` before firing the command.

```sh
$ ops instance list -t gcp -c <configuration_file_path>
+-----------------------------+---------+-------------------------------+-------------+--------------+
|            NAME             | STATUS  |            CREATED            | PRIVATE IPS |  PUBLIC IPS  |
+-----------------------------+---------+-------------------------------+-------------+--------------+
| nanos-main-image-1556601450 | RUNNING | 2019-04-29T22:17:34.609-07:00 | 10.240.0.40 | 34.83.204.40 |
+-----------------------------+---------+-------------------------------+-------------+--------------+
```


### Delete Volume

`ops volume delete` command can be used to delete an instance on Google Cloud.

You need to set the `ProjectID` and `Zone` in the `CloudConfig` section of your configuration file and export `GOOGLE_APPLICATION_CREDENTIALS` before firing the command.

```sh
$ export GOOGLE_APPLICATION_CREDENTIALS=<credentials_file_path>
$ ops volume delete <volume_name> -t gcp -c <configuration_file_path>
```

### Attach Volume

For attaching a volume you need a running instance using a image configured with a mount point. This means you have to create a volume before running the instance. After the volume created you have to specify the volume label with the same name of the volume created. You can create the image running the next command.
```sh
$ ops image create <image_name> -c config.json  --mounts <volume_label>:<mount_path>
```

After having the instance running you can attach a volume using `ops volume attach <instance_name> <volume_name> <volume_name> -t gcp -c <configuration_file_path>`.

**Note:** You need to stop and start the instance to see the changes applied.

### Detach Volume

You can detach a volume from a running instance using `ops volume detach <instance_name> <volume_name> -t gcp -c <configuration_file_path>`.
