---
type: post
title: "Octopress+Heroku move your assets to Amazon S3"
date: 2013-11-22 07:53:21 -0400
published: true
share: true
comments: true
tags: [Web, Blogging, Octopress, Heroku, Amazon, S3]
image:
  feature: /images/abstract-4.jpg
---

On this blog post I walk you through how to lightweight you web [Heroku] instance by moving all your assets (CSS, Javascipt and images) to a [CDN] (Content Delivery Network). We will be using [Amazon S3] web services for the storage and [Amazon CLoudFront] for the CDN.

<!--more-->

## Octopress

{% comment %}
### What is Octopress
{% endcomment %}

[Octopress] is a framework designed by [Brandon Mathis] for [Jekyll], the blog aware static site generator powering [Github Pages]. To start blogging with [Jekyll], you have to write your own HTML templates, CSS, Javascripts and set up your configuration. But with Octopress All of that is already taken care of. Simply clone or fork Octopress, install dependencies and the theme, and you're set.

[Jekyll] is a simple, blog-aware, static site generator. It takes a template directory containing raw text files in various formats, runs it through a converter (like [Markdown]) and our [Liquid] renderer, and spits out a complete, ready-to-publish static website suitable for serving with your favorite web server. Jekyll also happens to be the engine behind [GitHub Pages], which means you can use Jekyll to host your project’s page, blog, or website from GitHub’s servers for free.

### Why use Octopress ?

[Octopress] make your blogging experience easier (with some limit), Octopress provide all the organization layer like themes, layouts, generators and plugins and taking advantage of what Jekyll provide. But Octopress/Jekyll is not for everyone unless your are the kind of person that like to get your hand dirty.

>Octopress: a blogging framework for hackers  
>-- By [Octopress]

If you don't like command line amd coding stuff: «go your way» otherwise keep reading.

## Heroku

{% comment %}
### What is Heroku
{% endcomment %}

[Heroku] is a [PaaS] (Platform as a Service) provider, It let you focus on your app and take care of the deployment and configuration stuff for you. There are a lot to talk about [Heroku] and I don't like reinvent the wheel, so go ahead to their [site] for more information

{% comment %}
### Why use Heroku?

Well, good question, its a personal choice I am Developer/IT guy and I love control. [Heroku] provide me all the tools I need to do that.
{% endcomment %}

## Amazon AWS

{% comment %}
### What is Amazon AWS?
{% endcomment %}

Amozon is an [IaaS] (Infrastructure as a Service) provider, Amazon provide much more than Heroku, as Heroku is mostly an application oriented platform, Amazon is a broad and deep core infrastructure services (Compute, Storage & Content Delivery, Database, Network...).

{% comment %}
### Why use Amazon AWS?

Well its the first that come to my mind and the free plan offer more than you need for a blog.
{% endcomment %}

{{< alert class="alert-info" >}}
**Update:** Since I wrote this post, I found [CloudFlare](https://www.cloudflare.com/). CloudFlare is a website optimizer. Very easy to get up running.
{{< /alert >}}

## Step 1: Amazon AWS

Follow this [guide]. The documentation is pretty well done, you will go through it wihout any problem and if you are stuck at some place feel free to contact via the comment section.

## Step 2: Get API keys

Login to your Amazon AWS account and go to thw shell Management:

{% link_img center /images/aws-image-01.png %}

Go to **Identity & Access Management shell**:

{% link_img center /images/aws-image-02.png %}

This screen show up

{% link_img center /images/aws-image-03.png %}

1. On the left side bar select **Users**
2. Click **Create New Users**

{% link_img center /images/aws-image-04.png %}

1. Enter a new username in the field whatever you what (until it make sense)
2. Click **Create**

Then on the next page

{% link_img center /images/aws-image-05.png %}

1. Click **Show user security credentials** and copy them in a secure place
2. Click **Download credentials** (Save it in a secure place)

{{< alert class="alert-warning" >}}
**This is a one time download after that you will not be able to re-download it.**
{{< /alert >}}

Click on the user that you just created to edit it

{% link_img center /images/aws-image-06.png %}

scroll to the bottom of the page until the **Permissions** section

{% link_img center /images/aws-image-07.png %}

Click **Attach Policy**

{% link_img center /images/aws-image-08.png %}

1. Search for **s3**
2. Click the checkbox next to **AmazonS3FullAccess**
3. Click **Attach Policy**


## Step 3: Add some gem dependencies

```ruby GemFile
gem 'aws_sdk', '2'
gem 'ruby-progressbar'
gem 'mimetype-fu'
```

then run:

```shell
$ bundle install
```

## Step 4: Set up the SDK

Create a credentials config file:

* ~/.aws/credentials (Linux/Mac)
* %USERPROFILE%\.aws\credentials  (Windows)

```ini ~/.aws/credentials
[default]
aws_access_key_id = ACCESS_KEY
aws_secret_access_key = SECRET_KEY
```

## Step 5: Create some Rake tasks  

Add these line after the existing require statement in your RakeFile

```ruby Rakefile
require "aws-sdk"
require "ruby-progressbar"
require "mimetype_fu"
```

Declare this global variable after ``server_port`` in your Rakefile

```ruby Rakefile
@aws_bucket = "<the bucket name created when following the guide in Step 1>"
```

Add these task at the end of your RakeFile

```ruby RakeFile
######################
# Working with Amazon #
#######################

desc "Upload images assets to Amazon S3"
task :upload_assets => [:upload_javascript, :upload_stylesheet, :upload_images] do
  puts "All Assets uploaded!"
end

desc "Upload javascript assets to Amazon S3"
task :upload_javascript do
  raise "public directory doesn't exist" unless File.directory?(public_dir)
  Dir.chdir(public_dir)

  if File.directory?("javascripts") then
    assets = Dir.glob("javascripts/**/*.*")
    @progress = ProgressBar.create()
    @progress.total = assets.count
    @progress.format = "[Javascripts] Processing: %c from %C | %t"

    for asset in assets
      upload(asset)
      @progress.increment
    end
  end

  Dir.chdir(File.dirname(__FILE__))
end

desc "Upload stylesheet assets to Amazon S3"
task :upload_stylesheet => [:generate] do
  raise "public directory doesn't exist" unless File.directory?(public_dir)
  Dir.chdir(public_dir)

  if File.directory?("stylesheets") then
    assets = Dir.glob("stylesheets/**/*.*")

    @progress = ProgressBar.create()
    @progress.total = assets.count
    @progress.format = "[Stylesheets] Processing: %c from %C | %t"

    for asset in assets
      upload(asset)
      @progress.increment
    end
  end

  Dir.chdir(File.dirname(__FILE__))
end

desc "Upload images assets to Amazon S3"
task :upload_images do
  raise "public directory doesn't exist" unless File.directory?(public_dir)
  Dir.chdir(public_dir)

  if File.directory?("images") then
    assets = Dir.glob("images/**/*.*")

    @progress = ProgressBar.create()
    @progress.total = assets.count
    @progress.format = "[Images] Processing: %c from %C | %t"

    for asset in assets
      upload(asset)
      @progress.increment
    end
  end

  Dir.chdir(File.dirname(__FILE__))
end


def upload(asset)
  begin
    options = {
      acl: "public-read",
      cache_control: "public; max-age=86400",
      content_type: File.mime_type?(asset)
    }

    s3 = Aws::S3::Resource.new
    bucket = s3.bucket(@aws_bucket)

    if !bucket.exists? then
      abort("rake aborted!") if ask("#The bucket {@aws_bucket} doesn't exists. Do you want to create it?", ['y', 'n']) == 'n'
      bucket = s3.create_bucket(bucket: @aws_bucket)
    end

    force_update = !!ENV["FORCE_UPDATE"] || false

    force = "[OVERWRITTING]" if force_update
    object = bucket.object(asset)

    object_exist = object.exists?
    asset_changed = File.size(asset) != object.size if object_exist

    if !object_exist || force_update || asset_changed then
      @progress.title = "Uploading #{File.basename(asset)} #{force}"
      bucket.object(asset).upload_file(asset, options)
    else
      @progress.title = "Skipping #{File.basename(asset)} [UP TO DATE]"
    end
  rescue Aws::S3::MultipartUploadError => e
    puts e.errors
  end
end
```

## Step 6: Use your CDN

Now your are all setup you need to modify some layout file:

* source/_includes/head.html
* source/_includes/scripts.html

replace

```plain
{{ "{{" }} {{ "root_url" }} {{ }} }}
```

by

```plain
{{ "{{" }} {{ "site.cdn_url || root_url" }} {{ }} }}
```

If you use the ``img`` liquid tag plugin to reference images in your post, instead of modify each post you can modify the plugin to auto detect the url to use.

```ruby plugins/img_tag.rb
def render(context)
  config = context.registers[:site].config

  @img['src'] = if config['cdn_url']
    URI::HTTP.build({
      :host => URI.parse(config['cdn_url']).host,
      :path => @img['src']
    })
  else
    File.join("/", @img["src"].split("/"))
  end

  # ...code omitted...
end
```

then add this to your ``_config.yml``

```yaml
cdn_url: <YOUR_CDN_URL>
```
<hr/>

{{< alert class="alert-info" >}}
_**YOUR\_CDN\_URL:**_

* Can be the Amazon S3 url: http://``bucket``.s3.amazonaws.com or http://``bucket``.s3-``aws-region``.amazonaws.com
* Can be the Amazon CloudFront url: http://``generated_domain_name``.cloudfront.net
* Can be a custom domain name url: http://cdn.yourblogdomain.tld (need some DNS and CloudFront settings)  

{{< /alert >}}

## Step 7: Test the new URL  

Open your browser and the dev tools, hit your blog and you should see this for example for the ``screen.css``

{% link_img center /images/aws-image-09.png %}

That's it ! Now you have a supercharge blog and a lightweight Heroku web instance.

# Resources

- [Amazon S3]
- [Amazon CloudFront]
- [Jekyll]
- [Octopress]

[Amazon S3]: http://aws.amazon.com/s3/
[CDN]: http://en.wikipedia.org/wiki/Content_delivery_network
[Amazon CloudFront]: http://aws.amazon.com/cloudfront/
[guide]: http://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/GettingStarted.html
[CloudFlare]: https://www.cloudflare.com/
[Markdown]: http://daringfireball.net/projects/markdown/
[Liquid]: https://github.com/Shopify/liquid/wiki
[Github Pages]: http://pages.github.com/
[Jekyll]: http://github.com/mojombo/jekyll
[Octopress]: http://octopress.org/
[Heroku]: https://www.heroku.com
[IaaS]: http://en.wikipedia.org/wiki/Cloud_computing#Infrastructure_as_a_service_.28IaaS.29
[PaaS]: http://en.wikipedia.org/wiki/Platform_as_a_service
[site]: https://www.heroku.com/features
