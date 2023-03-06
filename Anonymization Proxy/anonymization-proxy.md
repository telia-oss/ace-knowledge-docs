## Overview

Anonymization Proxy detects and anonymizes Personally Identifiable Information (PII) that originates from the use of ACE Knowledge Widgets (e.g., performing a search, chatting with an ACE Knowledge Chat Bot, etc.).

## Features

* Anonymization of PII such as personal names, social security numbers, telephone numbers.
* Client IP masking - IP address of the user that works with ACE Knowledge Widgets is automatically masked as it is only visible to Anonymization Proxy application. Anonymization Proxy does not store this information and does not forward it to ACE Knowledge backend.

## How can I use Anonymization Proxy?

In order to be able to use Anonymization Proxy, please consider the following:
  * Anonymization Proxy currently only works with public Widgets
  * Since anonymization of PII is done outside of ACE Knowledge backend, it means that it is no longer possible to search on PII in Widgets when using Anonymization Proxy
  * It is not possible to use Anonymization Proxy for v1-v3 Widget implementations
  * Widgets distributed through Web provisions cannot be configured to use Anonymization Proxy

### Enabling Anonymization Proxy

Please contact ACE Knowledge support team if you wish to start using Anonymization Proxy. Team will enable the feature for your specified tenant(s).

#### Portals

Portal type of Widgets hosted on ACE Knowledge do not require additional configuration as long as they are **publicly accessible**. Please make sure accces mode is set to `Public access` in Widget's Security Settings:

![](images/security-settings.png)

Then, you can simply open Portal Widget by clicking `Preview` on the Widget and choosing `Yes` to `Preview using Anonymization Proxy` option as per below image:

![](images/preview.png)

### Widgets embedded on your website

In order to embed Widget on your web site and configure it use Anonymization Proxy, please follow the general Widget installation instructions in `Embed on your website` section of Widget Editor. However, one small modification is needed for installation script.

1. Click on `Show installation code` as per below image:

![](images/embed.png)

2. A dialog showing the installation code similar to below will pop up:

![](images/dialog.png)

3. Copy the installation code and replace `humany.net` domain in the link of the last line of the script tag to `anon-proxy.ace.teliacompany.net`.

For example, if the link is `//tenant.humany.net/v5/embed.js`, after the change it should become `//tenant.anon-proxy.ace.teliacompany.net/v5/embed.js`.
  
4. Follow the rest of the steps as they are defined in `Embed on your website` section.