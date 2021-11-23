# SimplyHost
## Simplify Hosting of Websites and Webapplications, without vendor lockin

The problem: Hosting a website or webapplication in 2021 is not easy. 

For most people it is impossible to host something on a computer in their home. Either because they don't have a 24/7 uptime computer, because their internet provider doesn't provide a stable IP or their connection is behind one or more layers of firewall and NAT.

Setting up and maintaining your own (virtual) server is technically challenging and costly.

So you have to rent space on a server from a provider. Each provider has its own unique setup. Usually you only have access through a web portal and (S)FTP. You cannot access the webserver configuration files directly and even if you could, those files are absurdly arcane.

## A real-life example

With SimplyEdit we've created a purely browser-based CMS system. But eventually you will want to save your changes to a webserver. Webservers have the tools to force users to login, using basic authentication. They also have tools to allow you to write files to a server, using webdav for example. However we found no way to create a configuration file that you could just add to your hosting account that would set this up correctly.

In the case of Apache, you can add a .htaccess and .htpasswd files and configure some stuff there. However you run into problems immediately. To configure the .htpasswd file location, you need to specify a full absolute path, you cannot use a path relative to the .htaccess file itself. Worse, there is no way in a .htaccess to tell Apache to allow webdav access to a specific part of your website.

So we made a PHP backend instead. It was designed to be as simple and generic as possible, use basic authentication and a .htpasswd file. And still there are problems. Each hosting company uses a different setup. Sometimes the webserver runs as the user 'www-data' (or another specific web account). In this case you need to grant write access to this user for your writable directories.

Then even if you get your PHP running, the code needs to parse the request headers to handle authentication. And depending on the configuration those authentication headers will differ. We ended up checking for anything up to 'redirect_redirect_redirect_authentication'.

Then to top it off, some hosting providers didn't even send the correct method in the request headers. Everything would turn into a GET request, only by looking for 'redirect_method', or 'redirect_redirect_method' would you get the actual method sent from a browser.

And this is just for Apache, that at least can support a .htaccess file. In the case of nginx you need to alter the main configuration file. So anything not exposed in your providers web portal of choice is not accessible at all.

## A general solution

What would a solution to this problem look like? From an end-user perspective the solution would allow them to have full control to the data (and code) for their website or webapplication. 

All the information needed to configure hosting for their site or app should be in one (or more) configuration files, under their control. There should be one or more applications to help write those configuration files. 

Then when they want to move their site or app to a new hosting provider, all that they need to do is to upload that configuration, or point to a url containing the configuration. Then they need to alter the DNS information to point to the new hosting provider.

Thats it. 

The configuration file should contain all the information needed to setup the server. It should contain all the technical requirements, so the hosting provider can check that everything needed is available. They may even suggest a switch to a different account type (more/less complete, more/less expensive) based on those requirements. Or if some requirement cannot be met, abort and tell the user that their site or app cannot be hosted by this provider.

The configuration file format should be an open standard, but extendable. The data needed for the site or app, the html pages, css, files, code, etc. should not need to move from their current location. That location could be a dropbox folder or a Solid Datapod. The hosting provider should just request access to the data location. Then the webserver can fetch the files and cache them if needed. Caching instructions can be added to the configuration file.

## Enter Solid

[Solid](https://solidproject.org/) is a suitable candidate to build most elements of a SimplyHost solution. It is heavily dependant on Linked Data, and linked data is an obvious choice for the configuration file format.

Solid divides solutions into three parts: an identity, a datapod and an application. Each of these can be hosted seperately. So we can re-use solid identities and solid datapods, as is. All we need is to build a solid application that can serve websites or webapplications based on the contents of a datapod. And it can grant/deny access to parts of the site or app based on your solid identity.

We can probably generalize this a bit, so SimplyHost could accept any OpenID Connect identity, not just Solid. And it could serve any content readonly as long as there is a URL to read it from.

But to write changes back, there must be a well defined protocol. Solid provides this in an open standard and in a decentralized way.

## A roadmap

### Version 0: upload files

Create a SimplyHost MVP that allows you to enter a domain name and upload static html files from your computer. Either through multiple file upload or through the browser Filesystem API.

### Version 1: Static hosting

Each route has a target location which contains the data for that route. Each folder of the target is mapped to a folder in the route.

Each route can also specify the name of an index file. The default is index.html.

When you add a new hostname to the configuration, the hosting provider must check that you have control over its DNS configuration. The hosting provider must provide a unique and random control sequence. The user must then add that sequence to a text record in the DNS configuration. Only if this check is passed should the hosting provider setup or change the configuration for this hostname.

### Version 2: Static hosting with write-back

You can add different methods (PUT/DELETE) to routes. If these are set, the webserver will passthrough these methods to the corresponding target urls. If the target url is part of a solid datapod, it will also handle/passthrough a login request.

### Version 3: Static site generation

You can configure a static site generation pipeline. E.g. the datapod contains markdown files. You can configure a static site generator like Jekyll and the theme to use. The hosting provider watches the source files and when it detects a change, it will re-generate the static site from the changes.

Solid Applications can receive notifications through the [Linked Data Notifications protocol](https://www.w3.org/TR/ldn/).

### Later: Other data stores

Add support for data stores other than Solid Data PODs. Potential candidates:

- Hyper (decentralized P2P)
- Nextcloud
- Dropbox
- Google Drive
- Microsoft One Drive
- ...

Add support for headless CMS systems like Contentful, headless e-commerce systems, etc.

