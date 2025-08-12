---
layout: post
title: "The Power of AWS IAM Policies"
date: 2024-01-27
tags:
  - Jekyll
  - Markdown
author: Mers Wafula
category: AWS
---
As an aspiring AWS enthusiast on the journey toward mastering cloud security, I recently stumbled upon an enlightening AWS re:Invent talk by the captivating speaker, Bridgit. The session was a treasure trove of knowledge, particularly focusing on the intricate world of AWS Identity and Access Management (IAM) policies. Inspired by Bridgit’s insights, this article embarks on a journey to unravel the essence of IAM policies and their pivotal role in securing AWS environments.

What is an IAM Policy?

An IAM policy is like a set of rules for controlling who can do what in a computer system. Think of it as a way to manage permissions for different users or programs. It helps ensure that the right people have the right access to files, databases, or other resources, while keeping things secure. In simple terms, it’s like setting up boundaries and rules for who gets to do what in a digital space.

For example, an IAM policy for an AWS S3 bucket might grant a user permission to read objects from that bucket but deny the ability to delete them. IAM policies are a critical component of securing and managing access to resources in cloud environments, ensuring that users have the appropriate level of access while maintaining security and compliance.

Structure of an IAM Policy:

An example of an IAM policy.: AmazonS3ReadOnlyAccess


An example of an IAM policy.: AmazonS3ReadOnlyAccess
The structure of an IAM policy:

Effect: maybe Allow or Deny

Principal: The entity (user, group, or role) to which the policy is applied.

Example: A specific user, a group of users, or a role assumed by an application.

Action: The specific operation or task that is allowed or denied by the policy.

Example: s3:GetObject for reading an object from an S3 bucket, or ec2:StartInstances for starting EC2 instances.

Resource: The AWS resource to which the policy is applied, specifying what the action can be performed on.

Example: An S3 bucket ARN (Amazon Resource Name) or an EC2 instance ID.

Condition: Additional criteria or constraints under which the policy is enforced. Conditions can include factors such as Multifactor authentication, Region, time of day, IP address, or other contextual information.

Example: Allowing access only if the request is made from a specific IP address or during a certain time range.

{

“Version”: “2012–10–17”,

“Statement”: [

{

“Effect”: “Deny”,

“Action”: “s3:*”,

“Resource”: [

“arn:aws:s3:::mersssbucket/*”

],

“Condition”: {

“Bool”: {

“aws:MultiFactorAuthPresent”: false

}

}

}

]

}

an example of IAM policy with MFA condition.

Stay tuned as I continue to unravel the layers of AWS security :)
