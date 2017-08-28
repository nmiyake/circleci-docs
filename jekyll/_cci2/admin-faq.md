---
layout: classic-docs
title: Administration FAQ
category: [administration]
order: 3
description: "CircleCI Administration Frequently Asked Questions (FAQs)."
published: true
---

* TOC
{:toc}

#### Can I monitor available build containers?

Yes, basic reporting on the state of your build fleet is available in the CircleCI Admin application. Refer to the  [CloudWatch Monitoring]({{site.baseurl}}/enterprise/cloudwatch/) document for how to enable additional container monitoring for AWS.

#### How do I provision admin users?

The first user who logs in to the CircleCI application will automatically be designated an admin user. Options for designating additional admin users are found under the Users page in the Admin section at `https://[domain-to-your-installation]/admin/users`.

#### How can I retrieve the passphrase and private IP address if it is lost?

SSH into into the services box, and run the following:

```
$ # To get the passphrase
$ circleci get-secret-token
CIRCLE_SECRET_PASSPHRASE=xxxxxxxxxxxxxxxxxxxx
$
$ # To get private ip address
$ ifconfig eth0 | grep "inet addr"
          inet addr:10.0.0.235  Bcast:10.0.0.255  Mask:255.255.255.0
```

#### How can I change my passphrase?

1. Change your passphrase on the system console (services box port 8800) settings page.

2. Restart the application.

3. Update `CIRCLE_SECRET_PASSPHRASE` in the `init` script that you use to add builder boxes to your fleet.

New builder boxes joining the fleet will use the new passphrase. Existing builder boxes with the old passphrase will also continue functioning. But, it is best practice to restart these boxes as soon as you can to use the consistent passphrase across your fleet.


#### How can I gracefully shutdown a Builder instance?

1. Get an admin token from `/etc/circle-bot/.circle-bot-token` on the services box.

2. Run the following command, replacing *builder IP* with the IP address of the builder you want to shutdown and using the token from Step 1:

```
curl -k -X POST "https://<builder ip>/api/v1/admin/system/shutdown?circle-token=$TOKEN&unstoppable=true"
```

#### Is it possible to run iOS/macOS builds on CircleCI Enterprise?

We currently provide limited, early access for iOS builds on CircleCI that run on our own cloud of macOS machines. Contact <enterprise-support@circleci.com> to make a request for this fleet.

#### Why is Test GitHub Authentication failing?

This means that the GitHub Enterprise server is not returning the intermediate SSL certificates. Check your GitHub Enterprise instance with <https://www.ssllabs.com/ssltest/analyze.html> - it may report some missing intermediate certs. You can use tools like <https://whatsmychaincert.com/> to get the full certificate chain for your server.


#### Docker isn't working?

Please see [these instructions](https://circleci.com/docs/2.0/docker-builder-config/#sharing-the-docker-socket) to use Docker.

If you are using single-box mode, note that you'll have to have

`export CIRCLE_DOCKER_RUN_ARGUMENTS="-v /var/run/docker.sock:/var/run/docker.sock"`

in `/etc/circle-installation-customizations` for it to work. You also won't need to have docker specified in the `config.yml` as the container will have access to the docker socket in the background. That should get docker working in single-box mode.

#### Can I use a custom image in single box mode?

Yes, you can do this by adding the below export to a file called `/etc/circle-installation-customizations`:


```
export CIRCLE_CONTAINER_IMAGE_URI="docker://circleci/build-image:{{ site.data.trusty.versions-ubuntu-14_04-XXL.summary.build-image }}"
```

That will give you the same image that is currently used on circleci.com, but you can replace the URI image with anything that is currently pulled unto the machine or exists in dockerhub. This works with [custom images](https://github.com/circleci/image-builder) as well. 

#### Why isn't CircleCI using HTTPS?

While we create a self-signed cert when starting up, that certificate only applies to the management console and not the CircleCI product itself. If you want to use HTTPS, you'll have to give us certificates to use under the `Privacy` section of the settings in the management console.

#### Why doesn't terraform destroy every resource?

We set the services box to have termination protection in AWS. We also write to an s3 bucket. If you want terraform to destroy every resource, you'll have to either manually delete the instance, or turn off termination protection in the circleci.tf file. You'll also need to empty the s3 bucket that was created as part of the terraform install.'

#### How do I backup CircleCI?

As of 1.48.0 you can take snapshots right in the management console, and follow the steps below:

1: On the management console, https://services_ip:8800, create a snapshot

2: After it completes, backup the full snapshots directory `/var/lib/replicated/snapshots`, including the json file, to something safe.

3: Take down the services box and rebuild it, but do not run the setup wizard yet.

4: Copy the snapshots directory to the services box. I recommend `/home/ubuntu/ so to avoid permissions issues.

5: Run the startup. When it asks for the licence, instead click `restore from a snapshot`. 

6: Enter the path, for example, `/home/ubuntu/snapshots` and click browse.

7: Select the snapshot and choose restore.

8: Restore the full snapshot and that will return your licence, console passphrase and databases.

#### How do I upgrade builder instances?

The builder instances take their binaries directly from the services box when they initialize. Thus in order to update the builder boxes you'll need update the services box first, then create a new builder fleet (or roll the builder fleet using an autoscaling group). During this start-up phase the builders will take the new update from the services machines, and things will be up-to-date.


#### Do the builders store any state?

They can be torn down without worry as they don't persist any data. The builders will cache docker images if you've shared the docker socket.



#### How do I verify TLS settings are failing?

Make sure that your keys are in unencrypted PEM format, and that the certificate includes the entire chain of trust as follows:

```
-----BEGIN CERTIFICATE-----
your_domain_name.crt
-----END CERTIFICATE-----
-----BEGIN CERTIFICATE-----
intermediate 1
-----END CERTIFICATE-----
-----BEGIN CERTIFICATE-----
intermediate 2
-----END CERTIFICATE-----
...
```

#### How do I debug the Management Console (Replicated)?

If you're experiencing any issues with Replicated, here are a few ways to debug it.

- Check the current version of Replicated installed

First, make sure you have the CLI tool for Replicated installed:

```
replicated -version
```

- Restart Replicated and the CircleCI app

Try restarting Replicated services. You can do this by running the following commands on the service box:

```
sudo restart replicated-ui
sudo restart replicated
sudo restart replicated-agent
```

Then, go to your services box admin (i.e. https://YOUR-CCIE-INSTALL:8800) and try restarting with "Stop Now" and "Start Now".

- Trying to log into Replicated

Try to log in to Replicated. You can do this by running the following commands on the service box. You will only be asked to enter password, which is the same one used to unlock the admin (i.e.: https://YOUR-CCIE-INSTALL:8800).

```
replicated login
```

If you could login, then please run the following command too and give us the output.

```
sudo replicated apps
```

You are getting Error: `request returned Unauthorized for API route`.. error probably because you are not logged into Replicated, so please check if you are still getting the error after successful login.

- Check Replicated logs

You can find Replicated logs under `/var/log/replicated`.

- Check output of docker ps

Replicated starts many Docker containers to run CCIE, so it may be useful to check what containers are running.

You should see something similar to this output:

```
 sudo docker ps
CONTAINER ID        IMAGE                                                    COMMAND                  CREATED             STATUS              PORTS                                                              NAMES
03fb873adf26        <service-box-ip>:9874/circleci-frontend:0.1.149242-d650d3c   "/docker-entrypoint.s"   3 days ago          Up 3 days           0.0.0.0:80->80/tcp, 0.0.0.0:443->443/tcp, 0.0.0.0:4434->4434/tcp   e53e4f74259a6ec0a268d8c984ac6277
113b9ea03b46        <service-box-ip>:9874/circleci-slanger:0.4                   "/docker-entrypoint.s"   3 days ago          Up 3 days           0.0.0.0:4567->4567/tcp, 0.0.0.0:8081->8080/tcp                     d262cc492bd5d692d467f74d8cc39748
0a66adfbc2f0        <service-box-ip>:9874/postgres:9.4.6                         "/docker-entrypoint.s"   3 days ago          Up 3 days           0.0.0.0:5432->5432/tcp                                             423e0e6c4099fa99cd89c58a74355ffe
1c72cbef1090        <service-box-ip>:9874/circleci-exim:0.2                      "/docker-entrypoint.s"   3 days ago          Up 3 days           0.0.0.0:2525->25/tcp                                               94de52d61d464b7543f36817c627fe56
df944bb558ed        <service-box-ip>:9874/mongo:2.6.11                           "/entrypoint.sh mongo"   3 days ago          Up 3 days           0.0.0.0:27017->27017/tcp                                           04a57db9f97a250c99dfdbeec07c3715
66be98cd54fe        <service-box-ip>:9874/redis:2.8.23                           "/entrypoint.sh redis"   3 days ago          Up 3 days           0.0.0.0:6379->6379/tcp                                             e2ce5e702c4114648718d2d5840edc56
ac2faa662bbe        <service-box-ip>:9874/tutum-logrotate:latest                 "crond -f"               3 days ago          Up 3 days                                                                              34e4d4165947f14d185d225191ba4ce8
796013f64732        <service-box-ip>:9874/redis:2.8.23                           "/entrypoint.sh redis"   3 days ago          Up 3 days           0.0.0.0:32773->6379/tcp                                            dce3519e7aff9a365bd3b42ed3a6f77f
```

Providing us with the output of `sudo docker ps` in service box will be helpful in diagnosing the problem.