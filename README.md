# ASLens

I noticed a few things had changed on the original ASLens repository and wanted to get this working.

## Before Deployment

- Add your device to the DeepLens console.
- Ensure MXNet is updated on the device.
- Install any required modules (see below).
- Copy contents of letters directory to `/opt/awscam/artifacts/` directory.

### Pip Modules

```
root@AWSDeepLens:~# pip list
adium-theme-ubuntu (0.3.4)
awscli (1.18.19)
boto3 (1.9.137)
botocore (1.15.19)
certifi (2019.3.9)
chardet (3.0.4)
colorama (0.4.3)
ConfigArgParse (0.14.0)
decorator (4.4.0)
dlr (1.0)
docutils (0.15.2)
enum (0.4.7)
futures (3.3.0)
graphviz (0.8.4)
gyp (0.1)
idna (2.6)
jmespath (0.9.5)
mxnet (1.3.0)
mxnet-mkl (0.12.0)
numpy (1.11.0)
pexpect (4.8.0)
pip (8.1.1)
playsound (1.2.2)
ptyprocess (0.6.0)
pyasn1 (0.4.8)
pygobject (3.20.0)
python-dateutil (2.8.1)
PyYAML (5.3)
requests (2.18.4)
rsa (3.4.2)
s3transfer (0.3.3)
setuptools (20.7.0)
six (1.14.0)
unity-lens-photos (1.0)
urllib3 (1.25.8)
wheel (0.29.0)
```

### Sound Modules

#### Credit

- Thanks to these folks for great instructions, but providing the steps below as well to remove dependencies.
- `https://forums.aws.amazon.com/thread.jspa?messageID=826041&#826041`
- `https://github.com/matthew1000/dee`

#### Steps to get Playsound working On DeepLens

- On the DeepLens device, open `/opt/awscam/greengrass/ggc/deployment/group/group.json`.
- Find the section containing `/dev/dri/renderD128`.
- Copy it twice, and change the SourcePath in one to `/dev/snd/controlC0` and the other to `/dev/snd/pcmC0D0p`. Also change the `Id` field to `audio1` and `audio2`.
- Within the first ‘ResourceAccessPolicies’ array further down, add two more entries for `audio1` and `audio2`.
- Ensure you have a speaker attached to your DeepLens.
- Add audio devices to Greengrass group (ensure that group is set to `audio`). Add resources to Lambda function. Deploy the group.
- `sudo systemctl restart greengrassd.service --no-block`

## Upload Lambda

- There used to be a greengrass hello world Lambda example which was removed.
- You need to author a new lambda.
- Copy all contents of the Lambda folder into files such as `greengrassHelloWorld.py`, and the various other folders and files.
- The lambda needs a service role.
- The lambda needs to be published.
- The lambda must start with `deeplens`.

## Upload DeepLens Model

- Navigate to the S3 console, create a private S3 bucket, in this example I will refer to it as `my-bucket-name`.
- Upload two json files and params files from this repo's model directory.
- Go to the DeepLens console, navigate to `Models` (left nav pane), `Import Model` button.
- Select `Externally Trained Model`.
- For `Model Artifact Path`, use `s3://my-bucket-name/`.
- For `Model Framework`, select `MXNet`.
- Give it a name and description.
- Select the `Import Model` button.

## Create DeepLens Project

- Go to the DeepLens console, navigate to `Projects` (left nav pane), `Create New Project` button.
- `Create a New Blank Project` button, `Next`.
- Give it a name and a description.
- Under `Project Content`, select the model we just imported, and add the Lambda function we published.

## Deploy DeepLens Project to Device

- Go to the DeepLens Projects dashboard. 
- Select your Project and click the `Deploy to Device` button.
- Select the correct device, select `Review`, then select `Deploy`.

## Monitor Changes via DeepLens Linux Console

- You can tail the following logs. The first is for monitoring the downloading of the model. The other is for monitoring the output of the Lambda function, which is also available in Greengrass and CloudWatch Logs.
- `/opt/awscam/greengrass/ggc/var/log/user/{region}/{aws-account-id}/deeplens_admin_model_downloader.log`
- `/opt/awscam/greengrass/ggc/var/log/user/{region}/{aws-account-id}/{project-name}.log`
