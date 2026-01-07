---
title: 'Creating a dotnet development certificate'
date: '15:29 08-01-2023'
taxonomy:
    category:
        - docs
---

https://stackoverflow.com/a/50788371

https://wiki.archlinux.org/title/Transport_Layer_Security

----


I was looking for sources of how to trust a certificate for local development.

I found this https://andrewlock.net/creating-and-trusting-a-self-signed-certificate-on-linux-for-use-in-kestrel-and-asp-net-core/

I have copied the original article with the intent of going over the content to learn some more especially for chromium based browsers as Blazor WASM is hard to debug without a trusted certificate.


> ## Using Open SSL to create a self-signed certificate
> 
> On Windows, creating a self-signed development certificate for development is often not necessary - Visual Studio automatically creates a development certificate for use with IIS Express, so if you run your apps this way, then you shouldn't have to deal with certificates directly.
> 
> On the other hand, if you want to host Kestrel directly over HTTPS, then you'll need to work with certificates directly one way or another. On Linux, you'll either need to create a cert for Kestrel to use, or for a reverse-proxy like Nginx or HAProxy. After much googling, I took the approach described in this post.
> Creating a basic certificate using openssl
> 
> Creating a self-signed cert with the openssl library on Linux is theoretically pretty simple. My first attempt was to use a script something like the following:
> ``` 
> openssl req -new -x509 -newkey rsa:2048 -keyout localhost.key -out localhost.cer -days 365 -subj /CN=localhost
> openssl pkcs12 -export -out localhost.pfx -inkey localhost.key -in localhost.cer
> ```
> This creates 3 files:
> ```
> localhost.cer - The public key for the SSL certificate
> localhost.key - The private key for the SSL certificate
> localhost.pfx - An X509 certificate containing both the public and private key. This is the file that will be used by our ASP.NET Core app to serve over HTTPS.
> ```
> 
> The script creates a certificate with a "Common Name" for the localhost domain (the -subj /CN=localhost part of the script). That means we can use it to secure connections to the localhost domain when developing locally.
> 
> The problem with this certificate is that it only includes a common name so the latest Chrome versions will not trust it. Instead, we need to create a certificate with a Subject Alternative Name (SAN) for the DNS record (i.e. localhost).
> 
> The easiest way I found to do this was to use a .conf file containing all our settings, and to pass it to openssl.
> ## Creating a certificate with DNS SAN
> 
> The following file shows the .conf config file that specifies the particulars of the certificate that we're going to create. I've included all of the details that you must specify when creating a certificate, such as the company, email address, location etc.
> 
> If you're creating your own self signed certificate, be sure to change these details, and to add any extra DNS records you need.
> 
> ```
> [ req ]
> prompt              = no
> default_bits        = 2048
> default_keyfile     = localhost.pem
> distinguished_name  = subject
> req_extensions      = req_ext
> x509_extensions     = x509_ext
> string_mask         = utf8only
> 
> # The Subject DN can be formed using X501 or RFC 4514 (see RFC 4519 for a description).
> #   Its sort of a mashup. For example, RFC 4514 does not provide emailAddress.
> [ subject ]
> countryName     = GB
> stateOrProvinceName = London
> localityName            = London
> organizationName         = .NET Escapades
> 
> 
> # Use a friendly name here because its presented to the user. The server's DNS
> #   names are placed in Subject Alternate Names. Plus, DNS names here is deprecated
> #   by both IETF and CA/Browser Forums. If you place a DNS name here, then you 
> #   must include the DNS name in the SAN too (otherwise, Chrome and others that
> #   strictly follow the CA/Browser Baseline Requirements will fail).
> commonName          = Localhost dev cert
> emailAddress            = test@test.com
> 
> # Section x509_ext is used when generating a self-signed certificate. I.e., openssl req -x509 ...
> [ x509_ext ]
> 
> subjectKeyIdentifier        = hash
> authorityKeyIdentifier  = keyid,issuer
> 
> # You only need digitalSignature below. *If* you don't allow
> #   RSA Key transport (i.e., you use ephemeral cipher suites), then
> #   omit keyEncipherment because that's key transport.
> basicConstraints        = CA:FALSE
> keyUsage            = digitalSignature, keyEncipherment
> subjectAltName          = @alternate_names
> nsComment           = "OpenSSL Generated Certificate"
> 
> # RFC 5280, Section 4.2.1.12 makes EKU optional
> #   CA/Browser Baseline Requirements, Appendix (B)(3)(G) makes me confused
> #   In either case, you probably only need serverAuth.
> # extendedKeyUsage  = serverAuth, clientAuth
> 
> # Section req_ext is used when generating a certificate signing request. I.e., openssl req ...
> [ req_ext ]
> 
> subjectKeyIdentifier        = hash
> 
> basicConstraints        = CA:FALSE
> keyUsage            = digitalSignature, keyEncipherment
> subjectAltName          = @alternate_names
> nsComment           = "OpenSSL Generated Certificate"
> 
> # RFC 5280, Section 4.2.1.12 makes EKU optional
> #   CA/Browser Baseline Requirements, Appendix (B)(3)(G) makes me confused
> #   In either case, you probably only need serverAuth.
> # extendedKeyUsage  = serverAuth, clientAuth
> 
> [ alternate_names ]
> 
> DNS.1       = localhost
> 
> # Add these if you need them. But usually you don't want them or
> #   need them in production. You may need them for development.
> # DNS.5       = localhost
> # DNS.6       = localhost.localdomain
> # DNS.7       = 127.0.0.1
> 
> # IPv6 localhost
> # DNS.8     = ::1
> ```
> 
> We save this config to a file called localhost.conf, and use it to create the certificate using a similar script as before. Just run this script in the same folder as the localhost.conf file.
> 
> ```
> openssl req -config localhost.conf -new -x509 -sha256 -newkey rsa:2048 -nodes \
>     -keyout localhost.key -days 3650 -out localhost.crt
> openssl pkcs12 -export -out localhost.pfx -inkey localhost.key -in localhost.crt
> ```
> 
> This will ask you for an export password for your pfx file. Be sure that you provide a password and keep it safe - ASP.NET Core requires that you don't leave the password blank. You should now have an X509 certificate called localhost.pfx that you can use to add HTTPS to your app.
> ## Trusting the certificate
> 
> Before we use the certificate in our apps, we need to trust it on our local machine. Exactly how you go about this varies depending on which flavour of Linux you're using. On top of that, some apps seem to use their own certificate stores, so trusting the cert globally won't necessarily mean it's trusted in all of your apps.
> 
> The following example worked for me on Ubuntu 16.04, and kept Chrome happy, but I had to explicitly add an exception to Firefox when I first used the cert.
> 
> ## Install the cert utils
> ```
> sudo apt install libnss3-tools
> ```
> ## Trust the certificate for SSL 
> ```
> pk12util -d sql:$HOME/.pki/nssdb -i localhost.pfx
> ```
> ## Trust a self-signed server certificate
> ```
> certutil -d sql:$HOME/.pki/nssdb -A -t "P,," -n 'dev cert' -i localhost.crt
> ```
> 
> > As I said before, I'm not a Linux guy, so I'm not entirely sure if you need to run both of the trust commands, but I did just in case! If anyone knows a better approach I'm all ears :)
> 
> We've now created a self-signed certificate with a DNS SAN name for localhost, and we trust it on the development machine. The last thing remaining is to use it in our app.
> Configuring Kestrel to use your self-signed certificate
> 
> For simplicity, I'm just going to show how to load the localhost.pfx certificate in your app from the .pfx file, and how configure Kestrel to use it to serve requests over HTTPS. I've hard-coded the .pfx password in this example for simplicity, but you should load it from configuration instead.
> 
>  > Warning You should never include the password directly like this in a production app.
> 
> The following example is for ASP.NET Core 2.0 - Shawn Wildermuth has an example of how to add SSL in ASP.NET Core 1.X (as well as how to create a self-signed cert on Windows).
> 
> ```
> public class Program
> {
>     public static void Main(string[] args)
>     {
>         BuildWebHost(args).Run();
>     }
> 
>     public static IWebHost BuildWebHost(string[] args) =>
>         return WebHost.CreateDefaultBuilder()
>             .UseKestrel(options =>
>             {
>                 // Configure the Url and ports to bind to
>                 // This overrides calls to UseUrls and the ASPNETCORE_URLS environment variable, but will be 
>                 // overridden if you call UseIisIntegration() and host behind IIS/IIS Express
>                 options.Listen(IPAddress.Loopback, 5001);
>                 options.Listen(IPAddress.Loopback, 5002, listenOptions =>
>                 {
>                     listenOptions.UseHttps("localhost.pfx", "testpassword");
>                 });
>             })
>             .UseStartup<Startup>()
>             .Build();
> }
> ```
> Although CreateDefaultBuilder() adds Kestrel to the app anyway, you can call UseKestrel() again and specify additional options. Here we are defining two URLs and ports to listen on (The IPAddress.Loopback address corresponds to localhost or 127.0.0.1):
> 
>     http://localhost:5001 - An unsecured end point
>     https://localhost:5002 - Secured using our SSL cert
> 
> We add HTTPS to the second Listen() call with the UseHttps() extension method. There are several overloads of the method, which allow you to provide an X509Certificate2 object directly, or as in this case, a filename and password to a certificate.
> 
> If everything is configured correctly, you should be able to view the app in Chrome, and see a nice, green, Secure padlock: