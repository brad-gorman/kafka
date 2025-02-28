Docker Images
=============

This directory contains scripts to build, test, push and promote docker image for kafka.

Repository Setup
----------------
Make sure the `DOCKERHUB_USER` and `DOCKERHUB_TOKEN` secrets are added and made available to Github Actions in Github Repository settings. This is required for pushing the docker image.

Local Setup
-----------
Make sure you have python (>= 3.7.x) and java (>= 17) (java needed only for running tests) installed before running the tests and scripts.

Run `pip install -r requirements.txt` to get all the requirements for running the scripts.

Make sure you have docker installed with support for buildx enabled. (For pushing multi-architecture image to docker registry)

Bulding image and running tests locally
---------------------------------------
- `docker_build_test.py` script builds and tests the docker image.
- kafka binary tarball url along with image name, tag and type is needed to build the image. For detailed usage description check `python docker_build_test.py --help`.
- Sanity tests for the docker image are present in test/docker_sanity_test.py.
- By default image will be built and tested, but if you only want to build the image, pass `--build` (or `-b`) flag and if you only want to test the given image pass `--test` (or `-t`) flag.
- An html test report will be generated after the tests are executed containing the results.

Example command:-
To build and test an image named test under kafka namespace with 3.6.0 tag and jvm image type ensuring kafka to be containerised should be https://downloads.apache.org/kafka/3.6.0/kafka_2.13-3.6.0.tgz (it is recommended to use scala 2.13 binary tarball), following command can be used
```
python docker_build_test.py kafka/test --image-tag=3.6.0 --image-type=jvm --kafka-url=https://archive.apache.org/dist/kafka/3.6.0/kafka_2.13-3.6.0.tgz
```

Bulding image and running tests using github actions
----------------------------------------------------
This is the recommended way to build, test and get a CVE report for the docker image.
Just choose the image type and provide kafka url to `Docker Build Test` workflow. It will generate a test report and CVE report that can be shared with the community.

kafka-url - This is the url to download kafka tarball from. For example kafka tarball url from (https://archive.apache.org/dist/kafka). For building RC image this will be an RC tarball url.

image-type - This is the type of image that we intend to build. This will be dropdown menu type selection in the workflow. `jvm` image type is for official docker image (to be hosted on apache/kafka) as described in [KIP-975](https://cwiki.apache.org/confluence/display/KAFKA/KIP-975%3A+Docker+Image+for+Apache+Kafka)

Example command:-
To build and test a jvm image type ensuring kafka to be containerised should be https://archive.apache.org/dist/kafka/3.6.0/kafka_2.13-3.6.0.tgz (it is recommended to use scala 2.13 binary tarball), following inputs in github actions workflow are recommended.
```
image_type: jvm
kafka_url: https://archive.apache.org/dist/kafka/3.6.0/kafka_2.13-3.6.0.tgz
```

Creating a Release Candidate
----------------------------
- `docker_release.py` script builds a multi-architecture image and pushes it to provided docker registry.
- Ensure you are logged in to the docker registry before triggering the script.
- kafka binary tarball url along with image name (in the format `<registry>/<namespace>/<image_name>:<image_tag>`) and type is needed to build the image. For detailed usage description check `python docker_release.py --help`.

Example command:-
To push an image named test under kafka dockerhub namespace with 3.6.0 tag and jvm image type ensuring kafka to be containerised should be https://archive.apache.org/dist/kafka/3.6.0/kafka_2.13-3.6.0.tgz (it is recommended to use scala 2.13 binary tarball), following command can be used. (Make sure you have push access to the docker repo)
```
# kafka/test is an example repo. Please replace with the docker hub repo you have push access to.

python docker_release.py kafka/test:3.6.0 --kafka-url https://archive.apache.org/dist/kafka/3.6.0/kafka_2.13-3.6.0.tgz
```

Please note that we use docker buildx for preparing the multi-architecture image and pushing it to docker registry. It's possible to encounter build failures because of buildx. Please retry the command in case some buildx related error occurs.

Creating a Release Candidate using github actions
-------------------------------------------------
This is the recommended way to push an RC docker image.
Go to `Build and Push Release Candidate Docker Image` Github Actions Workflow.
Choose the `image_type` and and provide `kafka_url` that needs to be containerised in the `rc_docker_image` that will be pushed to github.

Example:-
If you want to push a jvm image which contains kafka from https://archive.apache.org/dist/kafka/3.6.0/kafka_2.13-3.6.0.tgz to dockerhub under the namespace apache, repo name as kafka and image tag as 3.6.0-rc1 then following values need to be added in Github Actions Workflow:-
```
image_type: jvm
kafka_url: https://archive.apache.org/dist/kafka/3.6.0/kafka_2.13-3.6.0.tgz
rc_docker_image: apache/kafka:3.6.0-rc0
```

Promoting a Release Candidate
-----------------------------
`docker_promote.py` provides an interactive way to pull an RC Docker image and promote it to required dockerhub repo.

Promoting a Release Candidate using github actions
--------------------------------------------------
This is the recommended way to promote an RC docker image.
Go to `Promote Release Candidate Docker Image` Github Actions Workflow.
Choose the RC docker image (`rc_docker_image`) that you want to promote and where it needs to be pushed to (`promoted_docker_image`), i.e. the final docker image release. 

Example:-
If you want to promote apache/kafka:3.6.0-rc0 RC docker image to apache/kafka:3.6.0 then following parameters can be provided to the workflow.
```
rc_docker_image: apache/kafka:3.6.0-rc0
promoted_docker_image: apache/kafka:3.6.0
```

Using the image in a docker container
-------------------------------------
Please check [this](./examples/README.md) for usage guide of the docker image.

Steps to release docker image
-----------------------------
- Make sure you have executed `release.py` script to prepare RC tarball in apache sftp server.
- Use the RC tarball url (make sure you choose scala 2.13 version) as input kafka url to build docker image and run sanity tests.
- Trigger github actions workflow using the RC branch, provide RC tarball url as kafka url.
- This will generate test report and CVE report for docker images.
- If the reports look fine, RC docker image can be built and published.
- Execute `docker_release.py` script to build and publish RC docker image in your dockerhub account.
- Share the RC docker image, test report and CVE report with the community in RC vote email.
- Once approved and ready, take help from someone in PMC to trigger `docker_promote.py` script and promote the RC docker image to apache/kafka dockerhub repo
