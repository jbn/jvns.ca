---
title: "How Kubernetes certificate authorities work"
juliasections: ['Kubernetes / containers']
date: 2017-08-05T09:11:50Z
url: /blog/2017/08/05/how-kubernetes-certificates-work/
categories: ["kubernetes"]
---

Today, let's talk about Kubernetes private/public keys & certificate authorities!

This blog post is about how to take your own requirements about how certificate
authorities + private keys should be organized and set up your Kubernetes
cluster the way you need to.

The various Kubernetes components have a TON of different places where
you can put in a certificate/certificate authority. When we were setting up a
cluster I felt like there were like 10 billion different command line arguments
for certificates and keys and certificate authorities and I didn't understand
how they all fit together.

There are not actually 10 billion command line arguments but there are quite a lot. For example! Let's just look at the command line arguments to the API server.

The API server has more than 16 different command line arguments to do with
certificates or keys (I actually deleted a bunch to cut it down to this
list).

```
--cert-dir string                           The directory where the TLS certs are located. If --tls-cert-file and --tls-private-key-file are provided, this flag will be ignored. (default "/var/run/kubernetes")
--client-ca-file string                     If set, any request presenting a client certificate signed by one of the authorities in the client-ca-file is authenticated with an identity corresponding to the CommonName of the client certificate.
--etcd-certfile string                      SSL certification file used to secure etcd communication.
--etcd-keyfile string                       SSL key file used to secure etcd communication.
--etcd-cafile string                        SSL Certificate Authority file used to secure etcd communication.
--kubelet-certificate-authority string      Path to a cert file for the certificate authority.
--kubelet-client-certificate string         Path to a client cert file for TLS.
--kubelet-client-key string                 Path to a client key file for TLS.
--proxy-client-cert-file string             Client certificate used to prove the identity of the aggregator or kube-apiserver when it must call out during a request. This includes proxying requests to a user api-server and calling out to webhook admission plugins. It is expected that this cert includes a signature from the CA in the --requestheader-client-ca-file flag. That CA is published in the 'extension-apiserver-authentication' configmap in the kube-system namespace. Components recieving calls from kube-aggregator should use that CA to perform their half of the mutual TLS verification.
--proxy-client-key-file string              Private key for the client certificate used to prove the identity of the aggregator or kube-apiserver when it must call out during a request. This includes proxying requests to a user api-server and calling out to webhook admission plugins.
--requestheader-allowed-names stringSlice   List of client certificate common names to allow to provide usernames in headers specified by --requestheader-username-headers. If empty, any client certificate validated by the authorities in --requestheader-client-ca-file is allowed.
--requestheader-client-ca-file string       Root certificate bundle to use to verify client certificates on incoming requests before trusting usernames in headers specified by --requestheader-username-headers
--service-account-key-file stringArray      File containing PEM-encoded x509 RSA or ECDSA private or public keys, used to verify ServiceAccount tokens. If unspecified, --tls-private-key-file is used. The specified file can contain multiple keys, and the flag can be specified multiple times with different files.
--ssh-keyfile string                        If non-empty, use secure SSH proxy to the nodes, using this user keyfile
--tls-ca-file string                        If set, this certificate authority will used for secure access from Admission Controllers. This must be a valid PEM-encoded CA bundle. Alternatively, the certificate authority can be appended to the certificate provided by --tls-cert-file.
--tls-cert-file string                      File containing the default x509 Certificate for HTTPS. (CA cert, if any, concatenated after server cert). If HTTPS serving is enabled, and --tls-cert-file and --tls-private-key-file are not provided, a self-signed certificate and key are generated for the public address and saved to /var/run/kubernetes.
--tls-private-key-file string               File containing the default x509 private key matching --tls-cert-file.
--tls-sni-cert-key namedCertKey             A pair of x509 certificate and private key file paths, optionally suffixed with a list of domain patterns which are fully qualified domain names, possibly with prefixed wildcard segments. If no domain patterns are provided, the names of the certificate are extracted. Non-wildcard matches trump over wildcard matches, explicit domain patterns trump over extracted names. For multiple key/certificate pairs, use the --tls-sni-cert-key multiple times. Examples: "example.crt,example.key" or "foo.crt,foo.key:*.foo.com,foo.com". (default [])
```

and here are the arguments for the controller manager:

```
--cluster-signing-cert-file string          Filename containing a PEM-encoded X509 CA certificate used to issue cluster-scoped certificates (default "/etc/kubernetes/ca/ca.pem")
--cluster-signing-key-file string           Filename containing a PEM-encoded RSA or ECDSA private key used to sign cluster-scoped certificates (default "/etc/kubernetes/ca/ca.key")
--root-ca-file string                       If set, this root certificate authority will be included in service account's token secret. This must be a valid PEM-encoded CA bundle.
--service-account-private-key-file string   Filename containing a PEM-encoded private RSA or ECDSA key used to sign service account tokens.
```

and for the kubelet:

```
--client-ca-file string                   If set, any request presenting a client certificate signed by one of the authorities in the client-ca-file is authenticated with an identity corresponding to the CommonName of the client certificate.
--tls-cert-file string                    File containing x509 Certificate used for serving HTTPS (with intermediate certs, if any, concatenated after server cert). If --tls-cert-file and --tls-private-key-file are not provided, a self-signed certificate and key are generated for the public address and saved to the directory passed to --cert-dir.
--tls-private-key-file string             File containing x509 private key matching --tls-cert-file.
```

In this post I'm going to assume you know basically how TLS certificates and
certificate authorities ("CAs") work. When setting this up, I knew how certs
work but I did not understand how all the different Kubernetes certificate
authorities fit together.

In this post I'll explain some of the main CAs you can set in Kubernetes and
how they fit together.

I'll also tell you about a few things I learned while setting all of this up:

* You can't use a CA to check the validity of the service account key. The service account key is a weird key which is handled differently from literally every other key.
* You can (and should!!) use an authenticating proxy if the way Kubernetes maps client certificates to usernames and groups doesn't work for you
* Setting up the API server to support too many different authentication methods ("more than 2") makes things confusing (though maybe this isn't too surprising :))

(as usual, let me know what mistakes you find in here, I think most of this is
right but it's a complicated topic!)

### PKI & Kubernetes

When I started reading about kubernetes I saw this term "PKI" a lot and I
wasn't sure what it meant.

If you have a Kubernetes cluster, you might have hundreds or thousands of
private & public keys (in client certificates, server certificates, anywhere!).
That is a lot of private keys!

If you just had thousands of unrelated independent keys that would be chaos.
Chaos is not great for security.  So instead the way you manage private/public
keys is you have certificate authorities ("CAs") issue certificates saying
"hey, this public key is OK, it's from me, you should trust it".

Your PKI ("public key infrastructure") is how you organize all of your keys --
which keys are signed by which certificate authorities. 

For example:

* You could have one CA per Kubernetes cluster, that's responsible for signing all the private keys in that cluster (this is the model Kubernetes docmentation usually assumes)
* You could have one global CA that's responsible for signing ALL your private keys
* You could have one CA you use for services that are externally visibile and another CA you use for internal-only services
* ... something else

I'm not a security expert and I'm **not gonna try to tell you how you should
manage the private keys + certificate authorities** in your infrastructure. But!
No matter what PKI model you want to use, I'm pretty sure you can make it work with Kubernetes.

This blog post is about how to take your own requirements about how certificate
authorities + private keys should be organized and set up your Kubernetes
cluster the way you need to.

### Does a Kubernetes cluster have to have a single root certificate authority? (no)

If you read a lot of guides to how to set up Kubernetes, you'll see a step like
"set up a certificate authority". Kelsey Hightower's great "kubernetes the hard way" doc has it as [Step 2](https://github.com/kelseyhightower/kubernetes-the-hard-way/blob/2983b28f13b294c6422a5600bb6f14142f5e7a26/docs/02-certificate-authority.md), and the [Trusting TLS in a cluster](https://kubernetes.io/docs/tasks/tls/managing-tls-in-a-cluster/) guide says:

> Every Kubernetes cluster has a cluster root Certificate Authority
> (CA). The CA is generally used by cluster components to validate the
> API server’s certificate, by the API server to validate kubelet client
> certificates, etc.


The way this basically works is:

1. Set up a certificate authority 
2. Use that certificate authority to generate a bunch of different certificates that you'll give to different parts of the Kubernetes infrastructure.

But what if you don't want to set up a new certificate authority for
each kubernetes cluster? We didn't want to do this for various reasons that I won't go into,
and at first I was worried that you **had** to set up a single root cluster CA.

It turns out that this sentence ("every Kubernetes cluster has a
[single] cluster root Certificate Authority") is not true -- you can
actually use certificates issued by several completely different CAs to
manage your Kubernetes cluster and it's fine.

To be clear -- I'm not saying you **shouldn't** have a single root certificate
for your Kubernetes cluster, I'm just saying you don't have to if for whatever
reason that way doesn't meet your requirements.

Let's break down some of these command line arguments about certificates and
how they relate to each other. Each of these sections is about one certificate
authority you can define. They're all independent -- none of them need to be the
same as any other one. (though in practice you will probably want to make some
of them the same, you don't want to be managing like 6 CAs probably).

### The API server's TLS certificate (and certificate authority)

```
 --tls-cert-file string             
    File containing the default x509 Certificate for HTTPS. (CA cert, if any,
    concatenated after server cert). If HTTPS serving is enabled, and
    --tls-cert-file and --tls-private-key-file are not provided, a self-signed
    certificate and key are generated for the public address and saved to
    /var/run/kubernetes.
 --tls-private-key-file string      
    File containing the default x509 private key matching --tls-cert-file.
```

You're probably using TLS to connect to your Kubernetes API server. These two
options (to the API server) let you pick what certificate the API server should use.

Once you set a TLS cert, you'll need to set up a kubeconfig file for the
components (like the kubelet and kubectl) that want to talk to the API server.

The kubeconfig file will look something like this:

```
current-context: my-context
apiVersion: v1
clusters:
- cluster:
    certificate-authority: /path/to/my/ca.crt # CERTIFICATE AUTHORITY THAT ISSUED YOUR TLS CERT
    server: https://horse.org:4443 # this name needs to be on the certificate in --tls-cert-file
  name: my-cluster
kind: Config
users:
- name: green-user
  user:
    client-certificate: path/to/my/client/cert # we'll get to this later
    client-key: path/to/my/client/key # we'll get to this later
```

One thing I found surprising about this is -- almost everything else in the
universe that uses TLS will look in /etc/ssl to find a list of certificate
authorities the computer trusts by default. But Kubernetes doesn't do that,
instead it mandates "no, you have to say exactly which CA issued the API
server's TLS cert".

You can pass this kubeconfig file into any kubernetes component with `--kubeconfig /path/to/kubeconfig.yaml`

So! We've met our first certificate authority: the CA that issues the API
server's TLS cert. This CA doesn't need to be the same as any of the other
certificate authorities we're going to discuss.

### The API server client certificate authority (+ certificates)

```
--client-ca-file string    
    If set, any request presenting a client certificate signed by one of the
    authorities in the client-ca-file is authenticated with an identity
    corresponding to the CommonName of the client certificate.
```

One way for Kubernetes components to authenticate to the API server is with
**client certificates**.

All of these client certs should be issued by the same CA (which, again,
doesn't need to be the same as the CA that issued the API server's server TLS
cert).

When using a kubeconfig file (like we talked about above), you set the client certificates in that file, like this:

```
kind: Config
users:
- name: green-user
  user:
    client-certificate: path/to/my/client/cert
    client-key: path/to/my/client/key
```

Kubernetes makes a lot of assumptions about how you've set up your client
certificates. (it sets the user to be the Common Name field and the group to be
the Organization field). If those assumptions don't match what you want,
the right thing to do is to **not use** client cert auth and instead use
an authenticating proxy.

### The request header certificate authority (or: using an authenticating proxy)

```
# API server arguments
--requestheader-allowed-names stringSlice                 
  	    List of client
        certificate common names to allow to provide usernames in headers specified by
        --requestheader-username-headers. If empty, any client certificate validated by
        the authorities in --requestheader-client-ca-file is allowed.
--requestheader-client-ca-file string                     
        Root certificate bundle to use to verify client certificates on incoming
        requests before trusting usernames in headers specified by
        --requestheader-username-headers
```

Another way to set up Kubernetes auth is with an **authenticating proxy**. If
you have a lot of opinions about what usernames and groups should be sent to the
API server, you can set up a proxy which passes usernames & groups to the API server in a HTTP header.

The docs basically explain how this works -- the proxy uses a client cert to
identify itself, and the `--requestheader-client-ca-file` tells the API server
which CA to use to verify that client cert.

I don't have too much to say about this except -- we learned pretty quickly
that having too many auth methods in your API server ("accept client
certificates OR an authenticating proxy OR a token OR...") makes things
confusing. It's probably better to pick a small number (like 1 or
2?) of authentication methods your API server supports because it makes it
easier to debug problems and understand your security model.

### serviceaccount private keys (which aren't signed by a certificate authority)

```
# API server argument
--service-account-key-file stringArray
    File containing PEM-encoded x509 RSA or ECDSA private or public keys, used to
    verify ServiceAccount tokens. If unspecified, --tls-private-key-file is used.
    The specified file can contain multiple keys, and the flag can be specified
    multiple times with different files.
# controller manager argument
--service-account-private-key-file string
    Filename containing a PEM-encoded private RSA or ECDSA key used to sign service
    account tokens.
```

The controller manager signs serviceaccount tokens with a private key. Unlike
every other private key that Kubernetes supports, the serviceaccount key
doesn't support "hey use a CA to check if this is the right key". This means
you have to give exactly the same private key file to every controller manager.

Anyway this key does not need to have a certificate and does not need to be
signed by any certificate authority. You can just generate a key with

```
openssl genrsa -out private.key 4096
```

and distribute it to every controller manager / API server.

Using `--tls-private-key-file` for this seems generally fine to me though, as
long as you give every API server the same TLS key (which I think you usually
would?). (I'm assuming here that you have a HA setup where you run more than
one API server and more than one controller manager)

If you give 2 different controller managers 2 different keys, they'll sign
serviceaccount tokens with different keys and you'll end up with invalid
serviceaccount tokens (see [this issue](https://github.com/kubernetes/kubernetes/issues/22351)). I think this isn't ideal (Kubernetes should probably
support these keys being issued from a CA like it does for ~every other private
key). From reading the source code I think the reason it's set up this way is
that [jwt-go](https://github.com/dgrijalva/jwt-go) doesn't support using a CA
to check signatures.

### kubelet certificate authorities

Let's talk about the kubelet! Here are the relevant command line arguments for the API server & kubelet: 

```
# API server arguments
--kubelet-certificate-authority string    Path to a cert file for the certificate authority.
--kubelet-client-certificate string       Path to a client cert file for TLS.
--kubelet-client-key string               Path to a client key file for TLS.
# kubelet arguments
--client-ca-file string                   If set, any request presenting a client certificate signed by one of the authorities in the client-ca-file is authenticated with an identity corresponding to the CommonName of the client certificate.
--tls-cert-file string                    File containing x509 Certificate used for serving HTTPS (with intermediate certs, if any, concatenated after server cert). If --tls-cert-file and --tls-private-key-file are not provided, a self-signed certificate and key are generated for the public address and saved to the directory passed to --cert-dir.
--tls-private-key-file string             File containing x509 private key matching --tls-cert-file.
```

It's useful to authenticate requests to the kubelet because the kubelet can
execute arbitrary code on your machines :) (in fact that's its job)

There are actually 2 CAs here. I won't go into too much detail because this is
the same as the setup for the APi server -- the kubelet has a TLS cert (that it
uses to serve TLS requests) and also supports client cert authentication.

You tell the API server what certificate authority to use to check the
kubelet's TLS cert, and what client certificate to use when talking to the
kubelet.

Again these 2 CAs could be totally different from each other.

### so many possible CAs

So far we have found 5 different certificate authorities you can specify as
part of setting up a Kubernetes cluster! They're all handled independently and
in theory they could all be totally different.

I didn't discuss every single CA setting that Kubernetes supports (there are
still more!) but hopefully this gives you some of the tools you need to read
the docs about the rest.

Again -- it almost certainly doesn't makes sense to make them **all** different
from each other but I think it's useful to understand how all this is set up if
you have your own requirements around how you want to handle your
certificate authorities for Kubernetes and don't want to do exactly what the
docs suggest.
