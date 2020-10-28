---
layout: post
title: Consolidate Terraform services state file on AWS S3
excerpt_separator:  <!--more-->
tags:
  - terraform
  - s3
# comments: true
---

I have been working on reorgnizing Terraform service repo a little bit, i.e.
combine some terraform resources into a bigger one as they are needed to
bootstrap all other terraform services on our OpenStack Cloud. Thus I need to
figure out how to move the status of these resources managed by terraform to
the new one.

<!--more-->
After googling around on the Internet I found some related but not perfect
solutions, and after poking around I finally ended up with following:

1. Create a new one which contains all the resources declarations

2. Import all resources state to the new service folder

      ```bash
      terraform import openstack_compute_keypair_v2.<kp> <kp_name>
      terraform import openstack_networking_secgroup_rule_v2.<sgr> <sgr_uuid>
      terraform import openstack_networking_secgroup_v2.<sg> <sg_uuid>
      ```

3. Remove resources state in original folder

      ```bash
      terraform state rm openstack_compute_keypair_v2.<kp>
      terraform state rm openstack_networking_secgroup_rule_v2.<sgr>
      terraform state rm openstack_networking_secgroup_v2.<sg>
      ```

4. Refresh the terraform state in new folder with `terraform refresh` and run
`terraform plan` to verify there is nothing need to be done.

5. Delete the remote backend state file and key in DynamoDB of old resources.
