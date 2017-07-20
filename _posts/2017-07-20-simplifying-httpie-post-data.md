---
layout: post
title: Simplifying HTTPie post data
excerpt: "Use Vipe to simplify passing post data to HTTPie"
categories: ["Web Development"]
tags: ["CLI","HTTPie", "Vipe", "HTTP POST"]
comments: true
image:
  feature: https://i.imgur.com/HOxC5NG.png
---

[HTTPie](https://github.com/jakubroztocil/httpie) is a wonderful little tool to work with REST API's in the terminal. You can very easily make requests with header and data. A simple yet powerful example could be.

```
http PUT example.org X-API-Token:123 name=John
```

* Do HTTP POST request
* To example.org
* Set `X-API-Token` header to 123
* Pass `{"name":"John"}` as post data

Another wonderful feature is the ability for HTTPie to accept data from stdin. This enables us to create a premade file that we can pass to HTTPie, this file can for example contain xml or BASE64 encoded data.

This is cool, but we do need to edit this file every time. What if we could get an editor prompt before sending the request, a bit like how you can set your editor of choise as the "prompt" from writing git commit messages.

Good news! The moreutils package of tools contains the `vipe` command.
On macOS X this can be installed with homebrew

```
brew install moreutils
```
This enables us to insert a text editor into a pipe. Now we can create a template file with out data, pipe it to vipe which will open your editor of choice and pipe the edited text to HTTPie.

Once we are happy with out template (In my case an xml file for adding documents to Solr) we can pull the trigger on our HTTPie command

```
cat doctemplate.txt | vipe | http -v POST http://localhost:32770/solr/mytestcore/update "Content-Type: text/xml"
```