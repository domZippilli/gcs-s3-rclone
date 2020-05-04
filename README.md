# Using rclone to synchronize AWS S3 to GCP GCS

[`rclone`](https://www.rclone.org) is a great open source tool for synchronizing data across various storage platforms. Of the many it offers, AWS S3 and GCP's GCS are just two.

This repo will show some hints and tips to get you started for using rclone in GCP with GCS. For the purposes of this repo, we'll use a GCE VM, but other runtimes such as GKE can do this as well, with minimal modification.

## Create a service account that can write to GCS

Your service account can use the Object Creator role, or simply Storage Admin. You can optionally set IAM conditions where the resource must be a particular bucket.

## Get a VM

You'll need a VM to perform the copy. It's a good idea to put this VM geographically close to your AWS bucket. Here's an example:

``` shell
gcloud beta compute \
--project=YOUR_PROJECT \
instances create object-sync-worker \
--zone=us-east1-b \
--machine-type=n1-standard-8 \
--subnet=default \
--network-tier=PREMIUM \
--maintenance-policy=MIGRATE \
--service-account=object-sync@YOUR_PROJECT.iam.gserviceaccount.com \
--scopes=https://www.googleapis.com/auth/cloud-platform \
--image=debian-10-buster-v20200413 \
--image-project=debian-cloud \
--boot-disk-size=10GB \
--boot-disk-type=pd-standard \
--boot-disk-device-name=object-sync-worker \
--no-shielded-secure-boot \
--shielded-vtpm \
--shielded-integrity-monitoring \
--reservation-affinity=any
```

## SSH to the VM

Something like this should do it:

`gcloud beta compute ssh \
--zone "us-east4-c" "object-sync-vm" \
--project "YOUR_PROJECT"`

You might need to open a firewall rule for this to work. Alternatively, you can connect via the GCP UI's **SSH** button in the VM listing.

## Install rclone

To ensure you get the latest version, download the `*-amd64.deb` for the latest [here](https://downloads.rclone.org/).

For this README, we will use version `1.51.0`, downloaded with: `curl -O https://downloads.rclone.org/v1.51.0/rclone-v1.51.0-linux-amd64.deb`

Install it with `sudo dpkg -i ./rclone-v1.51.0-linux-amd64.deb`.

## Configure rclone

### Clone this repository (optional)

Install git: `sudo apt install -y git`.

Clone this repository to the VM with `git clone https://github.com/domZippilli/gcs-s3-rclone.git`.

Change PWD to the repo root with `cd gcs-s3-rclone`.

### Create the config file

Find out the rclone config file path by running `rclone config file`.

``` text
$ rclone config file
Configuration file doesn't exist, but rclone will use this path:
/home/YOUR_USERNAME/.config/rclone/rclone.conf
```

Create this file by copying the rclone.conf in this directory over it (unless you already have one, in which case you should concatenate it).

`cp rclone.conf /home/$USER/.config/rclone/rclone.conf`

Alternatively, you can create a symlink to the `rclone.conf` in this repo:

`ln -s $PWD/rclone.conf /home/$USER/.config/rclone/rclone.conf`

(Just be sure not to commit and push any secrets!!!)

### Edit the config file

The `[gcs]` remote for rclone will use the environment metadata from your VM to configure itself, so all you need to add to the configuration file is information to authenticate with AWS.

To do so, put your Access Key and Secret Access Key in the rclone.conf file with an editor as appropriate:

``` text
access_key_id = AK_ID
secret_access_key = SK_ID
```

If you'd like, there are other options like environment variables you can use. See more in the [rclone docs](https://rclone.org/s3/#authentication).

## Run a sync

Now you're all set. At the simplest level, a [sync](https://rclone.org/commands/rclone_sync/) between two buckets should look like this:

`rclone sync -P s3:AWS_BUCKET gcs:GCS_BUCKET`

Other rclone commands, such as `lsd` and `ls`, should work with both providers as well. For a full list of commands, see [here](https://rclone.org/commands/).

## Important Disclaimer

Work in this repo is written by a Googler, but this project is not supported by Google in any way. As the `LICENSE` file says, this work is offered to you on an "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied, including, without limitation, any warranties or conditions of TITLE, NON-INFRINGEMENT, MERCHANTABILITY, or FITNESS FOR A PARTICULAR PURPOSE. You are solely responsible for determining the appropriateness of using or redistributing the Work and assume any risks associated with Your exercise of permissions under this License.

## License

Apache 2.0 - See [the LICENSE](/LICENSE) for more information.

## Copyright

Copyright 2020 Google, Inc.
