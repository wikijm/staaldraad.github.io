---
layout: post
title:  "Abusing File Converters"
date:   2015-08-22 10:23:00
categories: fileformats security 
---
Every now and then you run into a new file format and you find that you may not have a tool to parse that file. Or you are looking for an easy to use solution for you mom to access the photo's you sent her in a .tar archive. This is where file conversion services come in, a quick Google for "online file converter" will yield multiple results. One thing to keep in mind when converting files, is that different file formats may support different features. 

An example of this would be the .zip and .rar formats' support for symlinks. Symlinks can be thought of as the *nix world's shortcut files, although more powerful in a lot of ways. If we wanted to create a symlink to a file, say the /etc/passwd file, we could simply use the `ln` command.

{% highlight bash %}
$ ln -s /etc/passwd linktopasswd
$ ls -l linktopasswd
lrwxrwxrwx. 1 user user 11 Aug 22 14:49 linktopasswd -> /etc/passwd
{% endhighlight %}

This simply creates a softlink in our current folder and any command run against this link will actually be executed against the original file. When you archive a folder containing symlinks, most file formats will follow the symlink and include the original file in the archive. We can however choose to preserve symlinks under certain file-formats, such as .zip and .tar, so that a copy of the symlink is included in the archive, but not the actual linked file. Other file formats, particularly those that originate in the Windows world, such as .rar, don't allow symlinks and will always include the linked file in the archive. 

At this point it should become rather obvious that this difference in symlink support could possibly be abused. If we create a .zip file, with a symlink to /etc/passwd and that file is converted to .rar, what would happen to the symlink? Would the symlink be preserved, or would the linked file be included in the new archive? 
To test this, we create a .zip with symlinks to common Linux files. Symlinks can be preserved by including the --symlinks option when creating the .zip archive.

{% highlight bash %}
$ mkdir /tmp/links
$ ln -s /etc/passwd /tmp/links/etcpasswd
$ ln -s /proc/self/environ /tmp/links/environ
$ ln -s /proc/mounts /tmp/links/mounts
$ zip --symlinks -r /tmp/archive.zip /tmp/links
{% endhighlight %}

Now it's simply a matter of uploading our .zip file and having it converted to a format that doesn't contain symlink support. 

![Use online converter](assets/conv_zip_to_rar.png)

The converted file can then be downloaded, un-archived and the contents viewed. The conversion process resulted in the server-side files being included in the newly created archive. Giving us arbitrary file read on the server. This would be limited by file-permissions and an attacker having to know the full path to files on the server.

![Unrar and view contents](assets/viewlinks.png)

This is a pretty simple attack and it's always worth-while checking whether services that allow .zip (and similar formats) to be uploaded, actually extract the file contents on the server-side and use these in later actions. If files are extracted and echoed back in anyway we have arbitrary file read, if the files are actually altered, it may result in a denial of service attack, if critical files are overwritten. 

*Disclaimer* The above conversion provider was contacted about this behaviour and they acknowledged the risk and accepted it as each conversion was being done in a newly spawned docker container. 
