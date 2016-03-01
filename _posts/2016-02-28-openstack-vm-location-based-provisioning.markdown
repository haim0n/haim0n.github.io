---
title: Openstack VM Location Based Provisioning
layout: post
categories: Openstack, Superfluidity, Nova, Activity Zones
---


# Real World Use Case - 5G Mobile App offloading
Taking advantage of location awareness, an ISP will offload many of the services that currently run on users' mobile devices to the cloud, providing extra security and reduced battery usage, while keeping a near local user experience.
Given the small footprint envisioned for the [`Superfluid`][superfluidity] Cloud guests, services can be deployed in a VM per user fashion, providing extreme personalization and unmatched security and privacy. The small footprint together with the mobility properties of the Superfluid Cloud, enable the processing units to follow the user around. Deployed in the edge, from the home gateway to the closest [`BTS`][bts-wiki], and therefore eliminating all sorts of overheads associated with centralized approaches, for a user experience that matches a local application.

# Mission Statement
Having the ability to migrate or instantiate a service, based on the end-user (application) physical location.


# Sample Setup
TBD.

# Nova Host Aggregates
A  [`host aggregate`][host-aggregates] is simply put, a group of hosts sharing some metadata. A host can be in more than one host aggregate. The concept of host aggregates is only exposed to cloud administrators.

A host aggregate may be exposed to users in the form of an `availability zone`. When you create a host aggregate, you have the option of providing an availability zone name. If specified, the host aggregate you have created is now available as an availability zone that can be requested upon provisioning.


Create a host aggregate that is exposed to users as an availability zone:
{% highlight bash %}
$ nova aggregate-create nyc-aggregate1 nyc:40.71:-74.00
+----+-----------------+-------------------+-------+----------+
| Id | Name            | Availability Zone | Hosts | Metadata |
+----+-----------------+-------------------+-------+----------+
| 1  | nyc-aggregate1  | nyc:40.71:-74.00  |       |          |
+----+-----------------+-------------------+-------+----------+
{% endhighlight %}

Add a host to the newly created nyc host aggregate:
{% highlight bash %}
$ nova aggregate-add-host 1 host1
Aggregate 2 has been successfully updated.
+----+-----------------+-------------------+-------------+----------------------------------------------+
| Id | Name            | Availability Zone | Hosts       | Metadata                                     |
+----+-----------------+-------------------+---------------+--------------------------------------------+
| 1  | nyc-aggregate   | nyc:40.71:-74.00  | [u'host1']  | {u'availability_zone': u'nyc:40.71:-74.00'}  |
+----+-----------------+-------------------+---------------+--------------------------------------------+
{% endhighlight %}

Now that the nyc availability zone has been defined and contains one host, we can create another zone for Austin, Tx:
{% highlight bash %}
$ nova aggregate-create txs-aggregate1 aus:30.26:-97.74
+----+-----------------+-------------------+-------+----------+
| Id | Name            | Availability Zone | Hosts | Metadata |
+----+-----------------+-------------------+-------+----------+
| 2  | txs-aggregate   | aus:30.26:-97.74  |       |          |
+----+-----------------+-------------------+-------+----------+
{% endhighlight %}

And assigning a host to it:
{% highlight bash %}
$ nova aggregate-add-host 2 host2
Aggregate 2 has been successfully updated.
+----+-----------------+-------------------+-------------+---------------------------------------------+
| Id | Name            | Availability Zone | Hosts       | Metadata                                    |
+----+-----------------+-------------------+---------------+-------------------------------------------+
| 1  | txs-aggregate   | aus:30.26:-97.74  | [u'host1']  | {u'availability_zone': u'aus:30.26:-97.74 '}|
+----+-----------------+-------------------+---------------+-------------------------------------------+
{% endhighlight %}

In order for the proper location decision to be made - we need an orchestration logic, which is out of current scope, but
we can come up with a simplified one for our purpuse.

Assuming there are only two data centeres, a simple table can be created:

| Geo Corrdinates | Availability Zone|
|-----------------|------------------|
| 40.71:-74.00    | nyc:40.71:-74.00 |
| 30.26:-97.74    | aus:30.26:-97.74 |



Our location decision function does the following: 

* gets the user's location
* travels the table and for each entry calculates the distance
* returns a list of placement candidates
* attempts to boot an instance with specified flavor

{% highlight bash %}
$ nova boot --flavor 84 --image 64d985ba-2cfa-434d-b789-06eac141c260 \
 --availability-zone nyc:40.71:-74.00 app1-instance
{% endhighlight %}






[superfluidity]: http://superfluidity.eu/about/specific-objectives/
[bts-wiki]: https://en.wikipedia.org/wiki/Base_transceiver_station
[host-aggregates]: http://docs.openstack.org/developer/nova/aggregates.html