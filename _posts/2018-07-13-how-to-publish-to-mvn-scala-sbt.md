---
layout: post
title: Guide to publish Scala artifact using sbt
---

> Disclaimer: this guide is actual for MacOS and GnuPG 2.1 in July 2018. In the future, something may get wrong.

### Create an account on Sonatype

Follow this [guide](http://central.sonatype.org/pages/ossrh-guide.html) and create a JIRA account and a ticket for claiming your groupId. You should own domain for grouipId or use something like io.github.yourname. Check this [guide](https://central.sonatype.org/pages/choosing-your-coordinates.html) for more information.

### Configure sbt

Add plugins to project/plugins.sbt
{% highlight java %}
addSbtPlugin("org.xerial.sbt" % "sbt-sonatype" % "2.3")

addSbtPlugin("com.jsuereth" % "sbt-pgp" % "1.1.1")
{% endhighlight %}

Add to build.sbt and write down actual information

{% highlight java %}
name := "artifactName"
organization := "io.organization"
version := "1.0"

homepage := Some(url("https://github.com/awesome/project"))
scmInfo := Some(ScmInfo(url("https://github.com/awesome/project"),
                            "git@github.com:awesome/project.git"))
developers := List(Developer("name",
                             "Firstname Secondname",
                             "meemailru",
                             url("https://github.com/username")))
licenses += ("Apache-2.0", url("http://www.apache.org/licenses/LICENSE-2.0"))
publishMavenStyle := true

// Add sonatype repository settings
publishTo := Some(
  if (isSnapshot.value)
    Opts.resolver.sonatypeSnapshots
  else
    Opts.resolver.sonatypeStaging
)
{% endhighlight %}

Add to ~/.sbt/1.0/sonatype.sbt

{% highlight java %}
credentials += Credentials("Sonatype Nexus Repository Manager",
        "oss.sonatype.org",
        "username",
        "password")
{% endhighlight %}

### Configure gpg

You should sign an artifact with a private key. In the guide, I use GnuPG on MacOS. First of all, create public/private key pair (if you don’t have the one).

{% highlight bash %}
$ gpg --gen-key
{% endhighlight %}

list your keys

{% highlight bash %}
$ gpg --list-keys
/Users/khamutov/.gnupg/pubring.kbx
----------------------------------
pub   rsa2048 2018-03-05 [SC] [expires: 2020-03-04]
      16F64A138832C316EF8B2CD84399A163BBF065D1
uid           [ultimate] khamutov <some@email.com>
sub   rsa2048 2018-03-05 [E] [expires: 2020-03-04]
{% endhighlight %}

Long hexadecimal value is your keyid. Send key to keyservers. It can take up to 24 hours to sync the key across all keyservers.

{% highlight bash %}
$ gpg --keyserver hkps://hkps.pool.sks-keyservers.net --send-key 16F64A138832C316EF8B2CD84399A163BBF065D1
{% endhighlight %}

Configuration sbt for working with gpg is a bit tricky. Add following to ~/.sbt/1.0/sonatype.sbt 

{% highlight java %}
// use external gpg instead BouncyCastle
useGpg := true

// GnuPG 2.1 has no secring file, but setting pubring.kbx as secring works.
pgpSecretRing := file("~/.gnupg/pubring.kbx")

// optional passphrase for key if you don't want to typing it each time. Should be array of chars.
pgpPassphrase := Some(Array('p','a','s','s','w','o','r','d'))
{% endhighlight %}

### Publish to sonatype staging repository

Run 
{% highlight bash %}
sbt publishSigned
{% endhighlight %}

if you see error 
**gpg: signing failed: Inappropriate ioctl for device**

fix it with
{% highlight bash %}
$ GPG_TTY=$(tty) 
$ export GPG_TTY
{% endhighlight %}

### Promote to central

Search your artifact on [https://oss.sonatype.org/](https://oss.sonatype.org/) and follow guide for releasing [https://central.sonatype.org/pages/releasing-the-deployment.html](https://central.sonatype.org/pages/releasing-the-deployment.html)

Congratulations, you are just released your first artifact.🎉
