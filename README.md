# Host-resume-on-AWS-S3
Host a resume as a static website on AWS S3 integrate with Route53, Certificate Manager and CloudFront
=============================================================================================================================================================================================================================

# Steps involved:
- Create a dedicated "S3 Bucket" and upload application files (Check the repo for resume template files- index.html, styles.css & script.js)
- Register a new Domain with "Route53".
- Create a Public TLS/SSL Certificate using AWS Certificate Manager.
- Create a CloudFront Distribution
=============================================================================================================================================================================================================================

# Let's deep dive into step by step
=> Create a dedicated S3 bucket:
Navigate to AWS Management Console and search for S3.
S3 -> Create Bucket -> General configuration ( Unique bucket name and select proper region)
- Object Ownership- enable ACLs disabled (recommended)
- Scrolling down, deselect the setting for Block all public access, then acknowledge the setting
 (NOTE: In most scenarios, this is not recommended, as you'll see from the warning you receive when you disable it. But because you're creating a public résumé that you DO want to be open to the world, then disabling 
  this is appropriate.)

- Use the defaults for the rest of the bucket settings and then click Create bucket.
- Now you have an empty bucket, but it's not quite ready for primetime in terms of website hosting. You'll need to make a couple more updates.

- Enable static website hosting
  For S3 to be able to serve your files up as a website, you'll need to enable that on the bucket.
  Click into the bucket you just created and go to the | Properties tab |
  >> Scroll all the way down to the bottom of the page, and in the Static website hosting section, click Edit.
  Edit the static website hosting setting, select 'Enable'. This will open additional options.
  >> For the Index document, enter index.html. This specifies the default home page for the site (your HTML code for your resume). Then click Save changes.
 
- At the top of the page, click the | Permissions tab | Scroll down to the Bucket policy section, and click Edit.
  > Add a bucket policy to allow the contents of the bucket to be publicly accessible.
  > Bucket policy JSON code:

```JSON
 {
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "PublicReadGetObject",
            "Effect": "Allow",
            "Principal": "*",
            "Action": [
                "s3:GetObject"
            ],
            "Resource": [
                "arn:aws:s3:::Bucket-Name/*"
            ]
        }
    ]
 }
```

# >> This policy says to "Allow" everyone (the Principal of "*") to take the action of "GetObject" (basically "read") on all files in your bucket ("Bucket-Name/*").
** IMPORTANT: Update "Bucket-Name" with the name of your bucket. Then click Save changes.

Now, you've got a bucket that's configured for static website hosting, and you've applied a policy that will let people access the site. 
Now it's time to add your code files (Provided in repo- modify your details accordingly).

>> On the top of the page, click the | Objects tab | Click the Upload button.
>> Drag and drop your four files to the browser.  This should include index.html, styles.css, script.js and headshot.jpg.
>> After the files have uploaded and all four display in the Files and folders section, click the Upload button.
>> Now it's time to test that your resume loads. To do this, you'll need to get the S3 bucket website endpoint.
>>>> Navigate to  the |Properties | tab of the bucket. -> Scroll all the way down to the bottom of the page, to the Static website hosting section. 
     Click on the Bucket website endpoint link (it will open in a new tab).

# >> If everything worked, you should see your resume as a website displayed in the browser.

=============================================================================================================================================================================================================================

# => Let's work on the domain name next, using Route 53, which is Amazon's domain name and DNS service.
 >>If you don't already have a domain name, then you can register one with AWS, using Route 53. 
 (If you already have a domain name with another provider, I'll give you some general guidance on using that further.)
>> Navigate to Route53, on the Route 53 Dashboard, simply enter the domain name you're interested in, then click Check.

>> If the domain name is available, you'll be able to Select it (and if it's not available, you'll see some alternate names).
   Selecting it will add it to your cart, take you through a checkout process, and then the charge will show up on your AWS bill.

>> Create a hosted zone and get your name servers Even if you aren't using a domain name from Route 53,
   you'll still need a hosted zone (again, this holds the records and rules that control how traffic is routed).

>> Click on Hosted zones on the left navigation, then click Create hosted zone.
>> Enter a unique domain name (example.com) and enable 'Public hosted zone' then click Create hosted zone.

>> Once your public hosted zone is created, you'll see four name servers listed. (** Make a note of your 4 name servers so you can enter them with the third-party provider.)
>> Navigate to the DNS settings for your current domain provider. Find your name server settings, and replace them with the name servers from Route 53.

>> Create an A Record with an Alias to Point to the S3 Website, now that you have a public hosted zone, you need to create a record that says how traffic should be routed when someone goes to your domain name.

>> Click into your hosted zone, and then click Create record.
NOTE: If you get the "wizard" view (Check top right corner of the page) click Switch to quick create.  (If you don't see this "tile" view, then you're already in quick create mode.)

# >> Fill out the details for the record.
Record name: Leave the subdomain blank, and just go with the root domain (like "amberaws.com").
Record type: A
Alias: Toggle this on. An alias lets you route to AWS resources like S3, CloudFront, Elastic Beanstalk and so on.

# >> Now fill in the details of where to route traffic. You can type into these dropdowns to filter the values.
- Alias to S3 website endpoint
- Your region 
- The final dropdown should automatically populate with your S3 website.
  NOTE: If nothing shows up here, it's likely because you didn't name your bucket the same as your domain name. You'll need to recreate the bucket with the exact name of your domain.

- For Routing policy, select Simple routing. For Evaluate target health, leave the default of Yes, then click Create records.
  (Choose routing policy, target health, and then create record.)

- It can take up to 60 seconds for your changes to take effect. You can view the status of propagation by clicking on the handy View status on top of the page.
  After the Status changes from PENDING to INSYNC, then you should be good to test out your changes. (Make sure the status says INSYNC before testing things out)

  
  # >>>>  type your domain name into a browser (like example.com), Route 53 should direct you to the S3 website, which means you should see your resume.

=============================================================================================================================================================================================================================

# => The Next step is to get a secure connection (HTTPS, with a TLS/SSL certificate) working so you can get rid of that "Not secure" message from your browser.

- Create a Public TLS/SSL Certificate using AWS Certificate Manager.
  If you need a refresher on certificates, these help ensure a secure connection between users and the server they're making a request to.

>> Navigate to Certificate Manager.
   (IMPORTANT!  For this section, you need to switch your region to us-east-1 (N. Virginia). If you create a certificate in another region,
    you won't be able to use it with CloudFront (where you'll eventually end up).

# >> From the Certificate Manager landing page, click Request a certificate. Select Request a public certificate and then click Next.

>> Enter your domain name (like "example.com"), leave the rest of the options as defaults, then click Request.
- The request was successful, but it will have a "pending validation" status until you validate DNS. Click View certificate.
  
>> Before a certificate can be issued, Amazon needs to confirm that you own this domain and that you're able to modify DNS settings (in Route 53).
   To start this process, click Create records in Route 53. There are various filters applied to this next screen, checking for validation status and
   whether your domain is found in Route 53. From here, you can click Create records, which will actually – wait for it – create a record in Route 53 for you.
   ( Create records, which will create a CNAME record in Route 53 )
>> If the record creation was successful, you should see a message as Successfully created a record in Route 53 to validate DNS.

> The record was created in Route 53. So navigate to Route 53, to your hosted zone you were working in earlier.
  >>>> You should see a new CNAME record that was created from Certificate Manager.
** Now you have a TLS/SSL certificate. 
> Your website files are currently hosted in S3, but unfortunately, you can't use a certificate on an S3 bucket.
=============================================================================================================================================================================================================================

# => A CloudFront distribution that points to the S3 bucket. And then the certificate is applied to the CloudFront distribution.

=> Create a CloudFront Distribution:
   Amazon's content delivery network, or CDN. It's used to get content to users faster by caching it at "edge locations" around the world. 
   This works great for things like videos and images, making them faster to load.  


=> Navigate to CloudFront -> On the CloudFront home page, click Create a CloudFront distribution.
>> The origin domain is where your website files live, which is in S3. If you type in S3 to filter, it should pull up your bucket.
>> You get a message about using the website endpoint rather than the bucket endpoint. Yes, that's what you want! Click Use website endpoint, and AWS will update the endpoint for you.
 (Use the website endpoint, not the bucket endpoint)

>> Scroll down to the Default cache behavior section, then under Viewer, select Redirect HTTP to HTTPS.
>> Scroll down to Web Application Firewall (WAF) and select Do not enable security protections.

# >> In the next section, Settings:
   For Alternate domain name (CNAME), enter your domain name (like "example.com"). 
   For Custom SSL certificate, select the certificate you set up earlier. 
   NOTE: if you set it up in a region other than us-east-1 (N. Virginia), it won't show up here. You'll need to recreate it in us-east-1.
   ( Enter an alternate domain name and the custom SSL certificate )

>> Scroll to the bottom of the page. For Default root object, enter index.html (your default home page) and then click Create distribution.

>> It will take several minutes for the CloudFront distribution to finish deploying (even if it says "Successfully created" at the top of the page).
   You'll know it's done when the Last modified value shows a date and time.

=> To test that everything is working with CloudFront and the TLS/SSL certificate, copy the Distribution domain name. Open a new tab in the browser and navigate to that address.
=============================================================================================================================================================================================================================

# > You should now see the all-important padlock icon in your browser, indicating that you're on a secure connection using the certificate set up through Certificate Manager.

>>>> Update Route 53 to Point to the CloudFront Distribution, at the moment, the A Record in Route 53 is pointing to the S3 bucket. 
     Instead, we want Route 53 to point to the CloudFront distribution, which then points to S3.
>> Navigate back to Route 53, to the hosted zone you've been working with. Select the A Record, then on the right of the screen, click Edit record.

>> Instead of routing traffic to S3, update the three dropdowns to point to your CloudFront distribution.
   - Alias to CloudFront distribution
   - US East (N. Virginia) (this option is selected for you and grayed out)
   - Choose your distribution (it should automatically populate in the third dropdown). Click Save.

===>> Now you should be able to navigate to your custom domain name and have it load your resume on a secure connection.
=============================================================================================================================================================================================================================

 #  IMPORTANT! Delete Your Resources  
** Make syre to set up an AWS Budget to be notified when charges reach a certain threshold.
=============================================================================================================================================================================================================================

# Delete the following:
- Disable and delete the CloudFront distribution
- Delete records from the Route 53 hosted zone
- Delete the hosted zone (optional)- You can also choose to delete your hosted zone in Route 53, but if you do, your domain might become unavailable on the internet.  
  If you plan to use your domain name at some point in the future, I'd recommend keeping the hosted zone.  Keeping the zone will cost you 50 cents per month.
  But if you'd like to go ahead with deletion, just select the hosted zone and click Delete.
- Confirm that you've completed the actions in this warning message, type "delete," and then click Delete.
  
# 
> Delete the certificate from Certificate Manager
> Empty the S3 bucket and then delete it. Before you can delete a bucket, you must first delete the files in it. AWS provides a handy link to do that. Click the link to empty bucket configuration.
> Click delete bucket configuration.

=============================================================================================================================================================================================================================



