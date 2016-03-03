# Datamechanics.io Uploader

This upload tool makes use of AWS S3 to host files for the Data Mechanics class. It relies on two libraries, [S3 CORS Uploader](https://github.com/frederickjansen/amazon-s3-cors) and [S3 Bucket Listing](https://github.com/Hariri-Institute-SAIL/s3-bucket-listing). The latter is a forked version to hide the root dir from the user.

### AWS Config

Some configuration steps are required to get this up and running on AWS S3. Start by creating a bucket that matches the name of the website you're using. Upload index.html to the root and create a data folder. Follow [this tutorial](https://docs.aws.amazon.com/AmazonS3/latest/dev/website-hosting-custom-domain-walkthrough.html) to set up static web hosting. Now add the following CORS Configuration:   

```
<?xml version="1.0" encoding="UTF-8"?>
<CORSConfiguration xmlns="http://s3.amazonaws.com/doc/2006-03-01/">
    <CORSRule>
        <AllowedOrigin>*</AllowedOrigin>
        <AllowedMethod>POST</AllowedMethod>
        <MaxAgeSeconds>3000</MaxAgeSeconds>
        <AllowedHeader>Authorization</AllowedHeader>
    </CORSRule>
    <CORSRule>
        <AllowedOrigin>*</AllowedOrigin>
        <AllowedMethod>GET</AllowedMethod>
        <AllowedHeader>*</AllowedHeader>
    </CORSRule>
</CORSConfiguration>
```

This allows indiscriminate GET requests, and authorized POSTs. For the Bucket Policy, add the following:

```
{
	"Version": "2012-10-17",
	"Statement": [
		{
			"Sid": "AllowPublicRead",
			"Effect": "Allow",
			"Principal": "*",
			"Action": "s3:GetObject",
			"Resource": "arn:aws:s3:::datamechanics.io/data/*"
		},
		{
			"Sid": "GetBucket",
			"Effect": "Allow",
			"Principal": "*",
			"Action": [
				"s3:ListBucket",
				"s3:GetBucketLocation"
			],
			"Resource": "arn:aws:s3:::datamechanics.io"
		},
		{
			"Sid": "AllowIndexRead",
			"Effect": "Allow",
			"Principal": "*",
			"Action": "s3:GetObject",
			"Resource": "arn:aws:s3:::datamechanics.io/index.html"
		}
	]
}
```

Now all users can read the index.html file, and anything inside of the data folder.
Next create an AWS user that will have limited read/write access to the bucket. Use the Access ID and Secret Key to generate the policies for the S3 CORS Uploader. Then add the following inline policy:  

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "s3:PutObject",
                "s3:PutObjectACL",
                "s3:GetObject"
            ],
            "Resource": [
                "arn:aws:s3:::datamechanics.io/data/*"
            ]
        }
    ]
}
```

This gives the user read and write access to just the data folder.

### DNS

If you want to use a custom domain name, follow the [same tutorial](https://docs.aws.amazon.com/AmazonS3/latest/dev/website-hosting-custom-domain-walkthrough.html) that was listed above.