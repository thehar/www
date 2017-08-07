+++
categories = ["tools"]
date = "2016-04-15"
tags = ["aws", "s3", "hugo", "terraform","travis-ci", "namecheap"]
title = "Setup for Changin Times"

+++

# Step 0) Get a domain name from NameCheap.

[NameCheap](https://www.namecheap.com) This is a pretty great registrar and they don't support LGBT hating causes like GoDaddy.

# Step 1) Setup a Github Account and Organization.

It's pretty simple - you [signup](https://github.com/join),
[create an organization](https://help.github.com/articles/creating-a-new-organization-from-scratch/),
and then [create a new git repository](https://help.github.com/articles/creating-a-new-repository/)
under that github organization.

If this is truly your first time ever using github you're going to have to
[create an ssh key-pair](https://help.github.com/articles/generating-ssh-keys/),
which is used to authenticate you so you can push up your code.

# Step 2) Start blogging locally with Hugo.

[Hugo](https://github.com/spf13/hugo) is a more modern, much faster, version of
[Jekyll](https://github.com/jekyll/jekyll).

The main advantage I see for using Hugo over Jekyll is that Hugo makes it much
easier to build static websites that are not blogs, and you don't need ruby to
build website - you simply need a static Go binary.

Follow [this introduction](http://gohugo.io/overview/introduction/) and within
two minutes you should be well on your way to setting up a blog.

```
Side funny note: Don't use Wordpress, ever. Unless you want to pay to host it
with wordpress.com.  It's a pile of crap and you have a much larger attack vector
to deal with when you host a WP site yourself. (Feel free to email and ask about this)
```

# Step 3) Create an AWS Account.

AWS is a great way to serve up your website for pennies in the first year.  You get
a free tier.

But for now simply create a new account, and figure out away to get yourself
some admin keys (AWS_ACCESS_KEY and AWS_SECRET_KEY) because you will need these
to setup your AWS account to host this blog.

You will need to install `aws-cli` in order to run `aws configure` and maintain
your keys locally correctly.  Please see the [Changin Times] entry from 04-15-2017.

# Step 4) Use a Terraform script to setup your AWS account for the blog.

I built you out a simple [Terraform](https://terraform.io/) script which you
can use to create all the required things to host a website in S3.

I personally use `us-west-2`, Oregon's AWS region, because it is the cheapest and
you get all the shiny new products.

Here is the [script](https://github.com/thehar/www/blob/master/tf/setup.tf):
```terraform
provider "aws" {
  access_key = "AWS_ACCESS_KEY"
  secret_key = "AWS_SECRET_KEY"
  region = "us-west-2"
}

variable "domain_name" {
  default = "domain.com"
}

resource "aws_s3_bucket" "blog" {
  bucket = "${var.domain_name}"
  region = "us-west-2"
  acl = "public-read"
  policy = <<EOF
{
  "Version": "2012-10-17",
  "Statement": [{
    "Sid": "AddPerm",
    "Effect": "Allow",
    "Principal": "*",
    "Action": "s3:GetObject",
    "Resource": "arn:aws:s3:::${var.domain_name}/*"
  }]
}
EOF

  website {
    index_document = "index.html"
    error_document = "error.html"
  }
}

resource "aws_s3_bucket" "wwwblog" {
  bucket = "www.${var.domain_name}"
  region = "us-west-2"
  acl = "public-read"
  website {
    redirect_all_requests_to = "${var.domain_name}"
  }
}


resource "aws_route53_record" "blog" {
  zone_id = "${aws_route53_zone.primary.zone_id}"
  name = "${var.domain_name}"
  type = "A"
  alias {
    name = "${aws_s3_bucket.blog.website_domain}"
    zone_id = "${aws_s3_bucket.blog.hosted_zone_id}"
    evaluate_target_health = true
  }
}

resource "aws_iam_user" "blog_deploy" {
  name = "${var.domain_name}_blog_deploy"
  path = "/s3/"
}

resource "aws_iam_access_key" "blog_deploy" {
  user = "${aws_iam_user.blog_deploy.name}"
}

resource "aws_iam_user_policy" "blog_deploy_rw" {
  name = "${var.domain_name}_rw"
  user = "${aws_iam_user.blog_deploy.name}"
  policy = <<EOF
{
  "Statement": [{
    "Effect": "Allow",
    "Action": [
      "s3:ListBucket",
      "s3:GetBucketLocation",
      "s3:ListBucketMultipartUploads"
    ],
    "Resource": "arn:aws:s3:::${var.domain_name}",
    "Condition": {}
  }, {
    "Effect": "Allow",
    "Action": [
      "s3:AbortMultipartUpload",
      "s3:DeleteObject",
      "s3:DeleteObjectVersion",
      "s3:GetObject",
      "s3:GetObjectAcl",
      "s3:GetObjectVersion",
      "s3:GetObjectVersionAcl",
      "s3:PutObject",
      "s3:PutObjectAcl",
      "s3:PutObjectAclVersion"
    ],
    "Resource": "arn:aws:s3:::${var.domain_name}/*",
    "Condition": {}
  }, {
    "Effect": "Allow",
    "Action": "s3:ListAllMyBuckets",
    "Resource": "*",
    "Condition": {}
  }]
}
EOF
}
```

Before you can use this you'll need to substitute out `AWS_ACCESS_KEY`,
`AWS_SECRET_KEY`, and `domain.com` with the appropriate values.

If you really are against using Terraform to setup this configuration you can
of course always use [the guide](http://docs.aws.amazon.com/AmazonS3/latest/dev/website-hosting-custom-domain-walkthrough.html)
from Amazon directly.

Don't worry, we'll have more blogs about Terraform and other AWS + CM tools.

# Step 5)

# Step 6) Get a Travis CI account.

You might want to follow [this guide](https://docs.travis-ci.com/user/for-beginners)
to get your feet wet with Travis CI.

# Step 7) Add a .travis.yml to your blog repository.

Here is a basic `.travis.yml` you'll want to use which will publish your blog
upon every push to your repository.

~~~yaml
language: go
install: go get -v github.com/spf13/hugo
script:
  - hugo
  - python --version
  - sudo pip install s3cmd
  - s3cmd sync --delete-removed -P -M -r public/ s3://thehar.com/
notifications:
    email:
        on_failure: always
~~~

Then, [as described here](https://docs.travis-ci.com/user/environment-variables/#Defining-Variables-in-Repository-Settings),
you will want to add the `AWS_ACCESS_KEY` and `AWS_SECRET_KEY` keys of the S3 user,
which was created via the Terraform script, as build environment variables in the
Travis CI project.

The keys can be found in the `*.tfstate` file created when the Terraform script
was applied.

Note, we chose to use s3cmd because the provided Travis CI S3 deploy step
doesn't allow you to actually 'sync'. Meaning if files are removed from your
blog, their deploy step doesn't remove them, and all files are always uploaded
to the bucket, even if they haven't changed, slowing down the deploy, and costing you money.

### Some credit belongs to continuousfailure.com for this post
