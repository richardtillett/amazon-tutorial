# Using AWS S3 storage, EC2 compute, and IAM identity management

June 29, 2021

## Purpose and scope

We wish to use cloud services for various tasks, including computation, raw data storage, and result storage. In this guide, we'll describe how to

- Upload folders of data to S3 buckets via the web
- Create an EC2 compute instance
- Create IAM credentials needed to talk to S3 from EC2
- Sync an S3 bucket to a folder on EC2
- Sync changes made on EC2 back to our S3 bucket

## Upload folders of data to S3 bucket via the web

Here we'll upload some folder or folders of data to S3 manually, using our web browser. There are command line ways of doing this too, and we'll get a look at those later on.

### Grabbing test files from Github

If you want to use Richard's cat-centric test file set for bucket creation, perform the following steps.

1. Go to <https://github.com/richardtillett/amazon-tutorial>
2. Click on for_aws_steps.zip
3. Click the Download button
4. Find the zip file on your local computer, move it if you like, and unzip it.
5. Look inside and see that there are 2 folders inside, one with a picture and one with text. Check out the cat picture, but avoid reading the text file for a fun surprise later on.

### Making a bucket on the web

See the [S3 getting started guide](https://docs.aws.amazon.com/AmazonS3/latest/userguide/creating-bucket.html)

1. Sign in to the AWS S3 console <https://console.aws.amazon.com/s3/>
2. Choose **Create bucket**
3. Enter a **bucket name** with just lowercase and numbers (dashes are allowed)
4. Choose region (probably whatever region we decide to stick with)
5. Leave **block public access** blocked.
6. Click **Create bucket**

_By leaving public access blocked, we are being very secure, but also making things a little harder on ourselves. We cannot use `wget` to get objects, for example._

### Filling bucket with stuff

_Section copied nearly verbatim from [S3 getting started guide. Step 2](https://docs.aws.amazon.com/AmazonS3/latest/userguide/uploading-an-object-bucket.html)_ The change made is that we will be uploading a folder with stuff in it instead of a single file. Note: I don't think empty folders work, so make sure there is at least one file in the folder.

1. Open the Amazon S3 console at <https://console.aws.amazon.com/s3/>.
2. In the Buckets list, choose the name of the bucket that you want to upload your object to.
3. On the Objects tab for your bucket, choose Upload.
4. Under Files and folders, choose Add **folder**.
5. Choose the **folders** to upload, upload both the **media** and **text** folders from the test set, and then choose Open.
6. Choose Upload.

## Create and log in to an EC2 compute instance

_Skip to the logging in subsection if using an EC2 that already exists, is awake, and for which you have the correct **pem file**_

### Create an EC2 compute instance

Taken from the official [EC2 tutorial](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EC2_GetStarted.html).

1. Open the Amazon EC2 console at <https://console.aws.amazon.com/ec2/>.
2. From the console dashboard, choose Launch Instance.
3. The Choose an Amazon Machine Image (AMI) page displays a list of basic configurations, called Amazon Machine Images (AMIs), that serve as templates for your instance. Select an HVM version of Amazon Linux 2\. Notice that these AMIs are marked "Free tier eligible."
4. On the Choose an Instance Type page, you can select the hardware configuration of your instance. Select the t2.micro instance type, which is selected by default. The t2.micro instance type is eligible for the free tier. In Regions where t2.micro is unavailable, you can use a t3.micro instance under the free tier. For more information, see AWS Free Tier.
5. On the Choose an Instance Type page, choose Review and Launch to let the wizard complete the other configuration settings for you.
6. On the Review Instance Launch page, under Security Groups, you'll see that the wizard created and selected a security group for you. You can use this security group, but see the tutorial above if you need to make a new one.
7. On the Review Instance Launch page, choose Launch.
8. When prompted for a key pair, select Choose an existing key pair, or create one now. Make note of the name of this key pair and the name of the `pem file`

  **Warning:** Don't select Proceed without a key pair. If you launch your instance without a key pair, then you can't connect to it. When you are ready, select the acknowledgement check box, and then choose Launch Instances.

9. A confirmation page lets you know that your instance is launching. Choose View Instances to close the confirmation page and return to the console.

10. On the Instances screen, you can view the status of the launch. It takes a short time for an instance to launch. When you launch an instance, its initial state is pending. After the instance starts, its state changes to running and it receives a public DNS name. (If the Public IPv4 DNS column is hidden, choose the settings icon ( ) in the top-right corner, toggle on Public IPv4 DNS, and choose Confirm.

11. It can take a few minutes for the instance to be ready so that you can connect to it. Check that your instance has passed its status checks; you can view this information in the Status check column.

12. While you're here, you can name your instance by clicking in the blank Name field and typing.

### Logging in to your EC2 instance

This subsection is written for Linux or Mac OS users with native terminal clients and ssh support. It should generally work just fine for Windows users, if they're using PuTTY, WSL, or "Git for Windows", but some early steps will slightly differ and program names will differ.

1. Launch your Linux or MacOS Terminal application.
2. Locate the `pem file` from Step 8 of the previous subsection. If it isn't already in your home directory, move it to your home. If the pem file `importantcats.pem` is in your ~/Downloads folder, the move command will be

  ```
  mv ~/Downloads/importantcats.pem ~/
  ```

3. Change the "mode" of `importantcats.pem` to mode 400 with `chmod`

  ```
  chmod 400 importantcats.pem
  ```

4. If you didn't leave the web browser open after Step 12 of the previous section, get to your list of running EC2 instances like this.

  - Sign in to <https://aws.amazon.com>
  - Look for the black search bar up top and search for **EC2**
  - Click on the service just named **EC2** to load the EC2 console
  - In the Resources section of the page, click on **Instances (running)**

5. Find the instance you created previously, and click it's blank check-box on the left.

6. Click the previously greyed out **Connect** button

7. If using the "new experience," (new is default) click the **SSH client** tab for a usable example `ssh` command for your specific instance. It will look like

  ```
  ssh -i "importantcats.pem" ec2-user@ec2-18-223-107-249.us-east-2.compute.amazonaws.com
  ```

8. Paste that example command in your Terminal to connect to your instance.

  _Hacker voice: "I'm in."_

## Create IAM credentials needed to talk to S3 from EC2

Steps taken directly from [Amazon CLI user guide, configure section](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-quickstart.html#cli-configure-quickstart-config). This is an important step to do securely, so I have simplified not a single word.

**To create access keys for an IAM user**

1. Sign in to the AWS Management Console and open the IAM console at <https://console.aws.amazon.com/iam/>.
2. In the navigation pane, choose Users.
3. Choose the name of the user whose access keys you want to create, and then choose the Security credentials tab.
4. In the Access keys section, choose Create access key.
5. To view the new access key pair, choose Show. You will not have access to the secret access key again after this dialog box closes. Your credentials will look something like this:

  ```
  Access key ID: AKIAIOSFODNN7EXAMPLE
  Secret access key: wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
  ```

6. To download the key pair, choose Download .csv file. Store the keys in a secure location. You will not have access to the secret access key again after this dialog box closes.

  Keep the keys confidential in order to protect your AWS account and never email them. Do not share them outside your organization, even if an inquiry appears to come from AWS or Amazon.com. No one who legitimately represents Amazon will ever ask you for your secret key.

7. After you download the .csv file, choose Close. When you create an access key, the key pair is active by default, and you can use the pair right away.

## Sync an S3 bucket to a folder on EC2

See also [Amazon EC2 guide, storage section](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AmazonS3.html) for more variations on this theme.

1. `ssh` in to your running EC2 instance
2. Configure the aws CLI using `aws configure`. When prompted, enter (paste) in your

  ```
  Access key ID:
  Secret access key:
  ```

  You can just hit Return/Enter to skip the last two prompts that follow the Secret key entry.

3. Make a folder in your EC2 instance that will hold the bucket and contents. For, example, `mkdir ~/cats` if you want your bucket to live in `~/cats`

4. Sync the bucket to the EC2's folder using `aws s3 sync`. This 3-word command needs the s3 URI for your bucket, and this is `s3://my-bucket-name-exactly`, and it also needs the name of the folder you made in Step 3\. The sync command should look something like this.

  ```
  aws s3 sync s3://my-bucket-name-exactly ~/cats
  ```

5. Prove to yourself that you have the contents of the bucket by **finally** reading the README.txt in the text folder.

  ```
  cat ~/cats/text/README.txt
  ```

Now that whole bucket has been copied onto the EC2 instance, and is ready to be worked on, analyzed, changed, etc. Make some changes (or perform some work) and move to the final section.

## Sync changes made on EC2 back to our S3 bucket

If you haven't made any changes yet, maybe just make a file named `foo` somewhere in `~/cats` so we can prove to ourselves that the changes synced. `touch ~/cats/text/foo` will make a blank file, and will work for such a demo.

There's just two steps left here, not counting the one above. Repeat the sync step from the previous section, but reverse the order of the folder name and bucket URI. Then we will check it back on aws S3

1. Sync back with folder name and URI positions swapped in the command.

  ```
  aws s3 sync ~/cats sync s3://my-bucket-name-exactly
  ```

2. Prove to yourself that the changes (foo) made it back to the S3 bucket at <https://console.aws.amazon.com/s3/>.
