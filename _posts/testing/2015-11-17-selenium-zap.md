---
layout: post
title: " Up and Running with Selenium Web Driver, OWASP ZAP and Jenkins"
modified:
categories: testing
comments: true
excerpt: How easy it is to combine web driver with OWASP ZAP.
tags: [selenium, ruby, owasp zap]
image:
  feature:
date: 2015-11-17
---

Here's how easy it is to use web driver and OWASP ZAP to scan your site for vunelerabilities. The goal is to get a report from ZAP and run tests in your continuos integration pipeline. This is a basic job configuration, advanced ZAP attacks and reporting can be added in the future.

### Steps

First thing you will need to do is get OWASP ZAP extracted into a directory on the build machine. You can skip this step if zap is already on the server.

{% highlight bash %}
wget -q https://github.com/zaproxy/zaproxy/releases/download/2.4.2/ZAP_2.4.2_Linux.tar.gz
tar xvf ZAP_2.4.2_Linux.tar.gz
cd ZAP_2.4.2
{% endhighlight %}

Next we will start the ZAP Proxy which will give us access its API. All you need is at least Java 7 to run ZAP.

`nohup ./zap.sh -daemon -port 8787 -config api.disablekey=true &`

Note:

* I start ZAP on port `8787` but you can choose any port that you prefer
* I'm using `nohup` and `&` in the command so it will run as a background process that we shut down later
* We disable the api key here `api.disablekey=true` to keep this job configuration simple, this was turned off by default in ZAP before but turning it on causes some issues for some other additional ZAP tools like client-side libraries. Zap is only running on your machines localhost but you can re-enable this for additional security in the future if your ZAP client-side libraries don't have issues. 
* ZAP will start in a passive mode where it just scans requests for vunrelbilities but won't actively start attacking your site, those attacks need to be turned on via the ZAP REST api calls you can make later. 

More can be read about the API here: [API Docs](https://github.com/zaproxy/zaproxy/wiki/ApiDetails)
and [Command Line Docs](https://github.com/zaproxy/zap-core-help/wiki/HelpCmdline)

You will need a build step to run some selenium tests and use a profile to proxy your browser automation traffic into zap. This is well documented for all the major web driver libraries here: 
[Firefox](http://www.seleniumhq.org/docs/04_webdriver_advanced.jsp#firefox)
and
[Other Browsers](http://www.seleniumhq.org/docs/04_webdriver_advanced.jsp#using-a-proxy)

For Ruby and Firefox it was as easy as the following:

{% highlight ruby %}
PROXY = 'localhost:8087'

profile = Selenium::WebDriver::Firefox::Profile.new
profile.proxy = Selenium::WebDriver::Proxy.new(
  :http     => PROXY,
  :ftp      => PROXY,
  :ssl      => PROXY
)

driver = Selenium::WebDriver.for :firefox, :profile => profile
{% endhighlight %}

After your web driver tests have successfully run we can now get a HTML report of anything ZAP has found during its test run via a simple `curl` command.

`curl http://localhost:8787/OTHER/core/other/htmlreport/>>$WORKSPACE/zap_report.html`

Note:

* I'm using `localhost:8787` because I started ZAP on port `8787` so you can substitute it with whatever port you decided to use.
* This will save a file named `zap_report.html` to your current jenkins workspace via the `$WORKSPACE` jenkins environment variable. You can download or view the `zap_report.html` file right in jenkins, just click on the Workspace tab to find your report:

![Workspace](https://i.gyazo.com/e4cfbfe6138d8791987bf6ea78c1aaec.png)

Now we can shut down zap via the api.

`curl http://localhost:8787/json/core/action/shutdown/`
	
Thats it! ZAP is now shutdown.

### Putting it all together
*A basic job configuration in jenkins with the configuration from above.*

![jenkins](https://i.imgur.com/o8epqxN.png)

That's all you need to get up and running! Be careful with how you use OWASP ZAP since its a powerful security tool and always get permission before you start enabling any advanced attacks in the ZAP API.

*More Information*

[OWASP ZAP Project Home](https://www.owasp.org/index.php/OWASP_Zed_Attack_Proxy_Project)

[OWASP ZAP Language API's(python, ruby, java etc..)](https://github.com/zaproxy/zaproxy/wiki/ApiDetails)
