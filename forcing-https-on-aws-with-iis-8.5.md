## Introduction

Previously we talked about setting up a [free letsencrypt HTTPS certificate](https://richardhildebrand.wordpress.com/2017/02/04/using-letsencrypt-with-aws-and-iis) on an AWS EC2 server provided by elastic beanstalk. Now we would like to take full advantage of this certificate by forcing all traffic to HTTPS.


## Opening the URL Rewrite module

Open IIS manager. Select your server, and then under the IIS section, open the Url Rewrite module.


## Configuring the URL Rewrite module

Click add a blank inbound rule. Name the rule `force https`.

Set the match url paramaters as follows:

	Requested URL: Matches the Pattern
    Using: Regular Expressions
    Pattern: (.*)
    Ignore Case: checked


Set the condition parameters as follows:

	Logical Groupings: Match All
    Input: {HTTPS}
    Type: Matches the Pattern
    Pattern: ^OFF$
	Track capture groups across conditions: unchecked


Ignore Server Variables

Set action parameters as follows:

	Action type: Redirect
    Redirect URL: https://{HTTP_HOST}/{R:1}
    Append query string: checked
    Redirect type: See Other (303)


Apply the changes and test your site with HTTP and HTTPS!