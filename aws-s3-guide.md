# AWS S3 BUCKET COMPREHENSIVE GUIDE PREPARED BY mathemartins
  COPIED FROM JOINCFE.COM(http://www.codingforentrepreneurs.com)

# 1. [!CREATE AWS ACCOUNT HERE](http://aws.amazon.com/)

# 2. Create AWS User Credentials

    A. [Navigate to IAM Users](https://console.aws.amazon.com/iam/home?#users)
    B. Click on users
    C. Select Create New Users
    D. Enter awsbean as a user name (or whatever you prefer)
    E. Ensure Programmatic Access is selected, hit Next.
    F. Select Download credentials and keep them safe.
    G. Open the credentials.csv file that was just downloaded/created

    ** Note the Access Key Id and Secret Access Key as they are needed for a future step. **
    ** # These will be referred to as ```<your_access_key_id>``` and ```<your_secret_access_key>``` **

# 3. CREATE NEW S3 BUCKET

    A. Navigate to S3 through the [Console](https://console.aws.amazon.com/) or this [Link](https://console.aws.amazon.com/s3)
    B. Click Create Bucket
    C. Create a unique Bucket Name such as your-project-bucket or any other name you choose.
    D. Select Region relative to your primary users' location.
    E. Click Create
    
   ** Make note of the Bucket Name you created here. It will replace ```<your_bucket_name>``` below. **

# 4. Add Default Access Policies to your IAM User:

    A. Navigate to the user's account such as: https://console.aws.amazon.com/iam/home?#/users
    B. Select user.
    C. Click on Permissions tab.
    D. Click on Attach Existing Policies Directly and add any policies you'd like to give this user access to or add a custom policy.

# 5. Add Custom Policy / Permissions to your IAM User

    A. Navigate IAM Home: https://console.aws.amazon.com/iam/home?#/users
    B. Select user and click on click on Permissions tab.
    C. Click tab for Add Permissions
    D. Click on Attach Existing Policies Directly and add any policies you'd like to give this user access to or add a custom policy.
    E. Click Create Policy then select Create Your Own Policy.
    F. Set Policy Name to S3Django (or any name you decide)
    G. Set Policy Document to below. Change all ```<your_bucket_name>``` to the name of your bucket in S3 (set above). Do not change version date.

    ## Code To Paste In There
    ```javascript
    {
        "Version": "2012-10-17",
        "Statement": [
            {
                "Effect": "Allow",
                "Action": [
                    "s3:ListAllMyBuckets"
                ],
                "Resource": "arn:aws:s3:::*"
            },
            {
                "Effect": "Allow",
                "Action": [
                    "s3:ListBucket",
                    "s3:GetBucketLocation",
                    "s3:ListBucketMultipartUploads",
                    "s3:ListBucketVersions"
                ],
                "Resource": "arn:aws:s3:::<your_bucket_name>"
            },
            {
                "Effect": "Allow",
                "Action": [
                      "s3:*Object*",
                    "s3:ListMultipartUploadParts",
                    "s3:AbortMultipartUpload"
                ],
                "Resource": "arn:aws:s3:::<your_bucket_name>/*"
            }
        ]
    }
    ```
    ## 
    A. The Actions that we choose to set are based on what we want this user to be able to do. The line "s3:*Object*", will handle a lot of our permissions for handling objects for the specified bucket within the Recourse Value.

    B. Add CORS policy to bucket:

      ** note => just go back to where you created your bucket, you will see it there **
      i.   Click on bucket
      ii.  Permissions > Cors Configuration
      iii. Paste:
       ```html
        <CORSConfiguration>
          <CORSRule>
            <AllowedOrigin>*</AllowedOrigin>
            <AllowedMethod>GET</AllowedMethod>
            <MaxAgeSeconds>3000</MaxAgeSeconds>
            <AllowedHeader>Authorization</AllowedHeader>
          </CORSRule>
        </CORSConfiguration>
       ```
  
# 6. Django Setup

  1. Install boto, boto3 and Django Storages:
    ```pip install boto boto3 django-storages```

  2. Update INSTALLED_APPS in settings.py:

    ```
      INSTALLED_APPS = [
        ...
        'storages',
        ...
      ]
    ```

  3. Run migrate:

    ```$ python manage.py migrate```

  4. Create aws module in same directory as settings.py:

    ```bash
      $ pwd
      /path/to/<your-virtualenv>/src/<your-project>/
      $ ls
      __init__.py settings.py wsgi.py urls.py
      $ mkdir aws && cd aws
      $ touch __init__.py
      $ touch utils.py
      $ touch conf.py
      5. In utils.py add the following:
    
        from storages.backends.s3boto3 import S3Boto3Storage

        StaticRootS3BotoStorage = lambda: S3Boto3Storage(location='static')
        MediaRootS3BotoStorage  = lambda: S3Boto3Storage(location='media')
    ```

  6. In your conf.py add the following:

  ```python
    import datetime
    AWS_ACCESS_KEY_ID = "<your_access_key_id>"
    AWS_SECRET_ACCESS_KEY = "<your_secret_access_key>"
    AWS_FILE_EXPIRE = 200
    AWS_PRELOAD_METADATA = True
    AWS_QUERYSTRING_AUTH = True

    DEFAULT_FILE_STORAGE = '<your-project>.aws.utils.MediaRootS3BotoStorage'
    STATICFILES_STORAGE = '<your-project>.aws.utils.StaticRootS3BotoStorage'
    AWS_STORAGE_BUCKET_NAME = '<your_bucket_name>'
    S3DIRECT_REGION = 'us-west-2'
    S3_URL = '//%s.s3.amazonaws.com/' % AWS_STORAGE_BUCKET_NAME
    MEDIA_URL = '//%s.s3.amazonaws.com/media/' % AWS_STORAGE_BUCKET_NAME
    MEDIA_ROOT = MEDIA_URL
    STATIC_URL = S3_URL + 'static/'
    ADMIN_MEDIA_PREFIX = STATIC_URL + 'admin/'

    two_months = datetime.timedelta(days=61)
    date_two_months_later = datetime.date.today() + two_months
    expires = date_two_months_later.strftime("%A, %d %B %Y 20:00:00 GMT")

    AWS_HEADERS = { 
        'Expires': expires,
        'Cache-Control': 'max-age=%d' % (int(two_months.total_seconds()), ),
    }
  ```

  7. In Django settings.py:

    ```from <your-project>.aws.conf import *```

  8. Run python manage.py collectstatic and you should be all setup.

  ## thank you -- copied from justin mitchel coding for entrepreneurs + few modificcations of my own
