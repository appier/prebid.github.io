---
layout: page_v2
title: Get Started with Prebid Server
description: Get Started with Prebid Server
pid: 27
top_nav_section: dev_docs
nav_section: prebid-server
sidebarType: 5

---

<div class="bs-docs-section" markdown="1">

# Get Started with Prebid Server
{:.no_toc}

Prebid Server improves your page's performance by running the header bidding auction on a server.
This will improve your page's load time, which should improve your users' experience.

Prebid Server supports different adapters from Prebid.js. For info about Prebid Server adapters, see
the [Prebid Server Bidders]({{site.baseurl}}/dev-docs/prebid-server-bidders.html) page.

{: .alert.alert-success :}
**Prebid Server is open source!**
Prebid Server is an open source project.  [The source code is hosted under the Prebid organization on GitHub](https://github.com/prebid/prebid-server).

* TOC
{:toc}

## Step 1. Choose Your Server Host

- If you plan to host Prebid Server on your own server, please skip to **Step 2** below. If you do *not* have your own server, please choose from one of the Prebid.org members below to get started.

- **AppNexus**
  - Go to the [Prebid Server sign-up page](https://prebid.adnxs.com) and click the button to sign up.
  - Fill out the form details, including your email address.
  - When approved, you will receive an email with your assigned `accountId`. You will need this for configuring Prebid.js to use Prebid Server.

- **Rubicon Project**
  - [Learn more](https://rubiconproject.com/demand-manager-hosted-prebid-server/) about Rubicon Project's Hosted Prebid Server offering.


## Step 2. Download Prebid.js with Prebid Server enabled

- Go to [the Prebid.org download page]({{site.baseurl}}/download.html), select all the demand adapters you want to work with, and include "Prebid Server".
  - Some Prebid Server demand adapters may not have a corresponding client-side adapter that's present on the download page.  In this case, just ensure to select "Prebid Server" as part of your build and you will be able to interact with these Prebid Server only bidders.
  - We still strongly recommend to select any listed demand adapters in your build.  These additional selections will allow any client-side features (such as userSyncs) to function as well as allow you to easily use any of the adapter's currently registered aliases.

- For example, if you want to use AppNexus, Index Exchange, and Rubicon with Prebid Server, select:
  - *AppNexus*
  - *Index Exchange*
  - *Rubicon*
  - *Prebid Server*

- Then, click **Get Custom Prebid.js** and follow the instructions.

## Step 3. Update your site with the new build of Prebid.js

Update your site's hosted copy of Prebid.js to use the new build you just generated.  (For example, you may host your copy on your site's servers or using a [CDN](https://en.wikipedia.org/wiki/Content_delivery_network) such as Fastly or Akamai.)

## Step 4. Configure S2S bidder adapters

The Prebid Server settings (defined by the [`pbjs.setConfig`]({{site.baseurl}}/dev-docs/publisher-api-reference.html#module_pbjs.setConfig) method) go in the same anonymous function where you define your ad units.  This method must be called before `pbjs.requestBids`.

The code in your Prebid configuration block should look something like the following (unless you want to show video ads, in which case see [Using Prebid Server to show video ads](#prebid-server-video-openrtb) below).

See [The `s2sConfig` object]({{site.baseurl}}/dev-docs/publisher-api-reference.html#setConfig-Server-to-Server) for definitions of the keys in the `s2sConfig` object.

{% highlight js %}
var pbjs = pbjs || {};

pbjs.que = pbjs.que || [];

pbjs.que.push(function() {

    pbjs.setConfig({
        s2sConfig: {
            accountId: '1',
            enabled: true,
            bidders: ['appnexus', 'pubmatic'],
            timeout: 1000,
            adapter: 'prebidServer',
            endpoint: 'https://prebid.adnxs.com/pbs/v1/openrtb2/auction',
            syncEndpoint: 'https://prebid.adnxs.com/pbs/v1/cookie_sync'
        }
    });

    var adUnits = [{
        code: '/19968336/header-bid-tag-1',
        sizes: sizes,
        bids: [{
            /* Etc. */
        }]
    }];
});
{% endhighlight %}

Optionally, if you chose to use one of the existing Prebid.org members as your server host, you can also use the defaultVendor property.  This property represents the vendor's default settings for the s2sConfig.  When used, these settings will be automatically populated to the s2sConfig, saving the need to individually list out all data points. Currently 'appnexus' and 'rubicon' are supported values.

These defaults can still be overridden by simply including the property with the value you want in the config.  The following example represents the bare minimum configuration required when using the defaultVendor option:

{% highlight js %}
var pbjs = pbjs || {};

pbjs.que.push(function() {

    pbjs.setConfig({
        s2sConfig: {
            accountId: '1',
            bidders: ['appnexus', 'pubmatic'],
            defaultVendor: 'appnexus'
        }
    });

    var adUnits = [{
        code: '/19968336/header-bid-tag-1',
        sizes: sizes,
        bids: [{
            /* Etc. */
        }]
    }];
});
{% endhighlight %}

{: .alert.alert-info :}
**OpenRTB Endpoint**
If your `s2sConfig.endpoint` points to a url containing the path `/openrtb2/`, such as the AppNexus-hosted endpoint https://prebid.adnxs.com/pbs/v1/openrtb2/auction', Prebid will communicate with that endpoint using the OpenRTB protocol.

{: .alert.alert-info :}
**Aliasing Prebid Server only bidders**
If you wish to set/use an alias for a Prebid Server only bidder, simply list the alias in your `s2sConfig.bidders` field and call the [`pbjs.aliasBidder` method]({{site.baseurl}}/dev-docs/publisher-api-reference.html#module_pbjs.aliasBidder) in your prebid code (prior to the `pbjs.requestBids`) to register the alias.

<a name="prebid-server-video-openrtb" />

### Using Prebid Server to show video ads

If you are using Prebid Server and you want to show video ads, you must use [OpenRTB video parameters](https://www.iab.com/guidelines/real-time-bidding-rtb-project/) in your Prebid ad unit as shown below.

{: .alert.alert-warning :}
The `mimes` parameter is required by OpenRTB.  For all other parameters, check with your server-side header bidding partner.

```javascript
var adUnit1 = {
    code: 'videoAdUnit',
    mediaTypes: {
        video: {
            context: 'instream',
            mimes: ['video/mp4'],
            playerSize: [400, 600],
            minduration: 1,
            maxduration: 2,
            protocols: [1, 2],
            w: 1,
            h: 2,
            startdelay: 1,
            placement: 1,
            playbackmethod: [2]
            // other OpenRTB video params
        }
    },
    bids: [
        // ...
    ]
}
```

## Related Topics

+ [Prebid.js Developer Docs]({{site.baseurl}}/dev-docs/getting-started.html)
+ [Add a Bidder Adapter to Prebid Server]({{site.baseurl}}/dev-docs/add-a-prebid-server-adapter.html)

</div>
