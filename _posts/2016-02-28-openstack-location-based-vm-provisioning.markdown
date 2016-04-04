---
title: Openstack Location Based VM Provisioning
layout: post
categories: Openstack, Superfluidity, Nova, 5G, Location Based Services, MEC, RAN
comments: true
tags:
    -OpenStack
    -Mobile Edge Compute
---


# Real World Use Case - Emergency Breaklight In Autonomous Cars 
Normally, even an advanced driver assistance systems can only react to a direct surroundings of the vehicle.
With connected driving via `5G` mobile network, the cars will be able to communicate nearly in real time over a large distance
and provide driving information beyond the driver's line of sight. This will allow the car to slowdown in advance, in 
the case of an upcoming road hazard. 

A more advanced use case can be considered as well: two cars coordinating a lane change and negotiating the priority and risk of the 
situation, while taking into an account such parameters as speed, location, and 
weather conditions.

# The Challenge
Communication between cars via a `central cloud` has an end to end latency up to seconds, and this is untolerable for the
car to car communication use case.
![autonomous cars](/assets/openstack-location-based-vm/central-cloud-car-2-car.png)
*Car to car communication via central cloud introduces high latency*

# The Straightforward Solution 
The "simple" solution would be bringing the cloud infrastructure closer to the end users (cars) in our case. 
One of the possible solutions is being developed by [ETSI](http://www.etsi.org/technologies-clusters/technologies/mobile-edge-computing) 
and described in following section.
   
# Mobile Edge Computing To The Rescue
Mobile-edge Computing `(MEC)` enables *cloud* computing capabilities and an IT service environment within the Radio Access Network `(RAN)` in 
close proximity to mobile subscribers. Simply put, MEC architecture provides small data centers `(cloudlets)` in close proximity to 
mobile devices, sensors, and end users (cars in our case). These data centers are aimed to be located just one wireless 
hop away from the user at the radio access network RAN facilities which is reffered to as `network edges` i.e.; the nearest `base station`.
Compute and storage resources being right next to the user enables applications that require low latency responses.

#### MEC components
The architecture consists of several parts:
 
* A distributed wireless network comprised of a remote radio Unit `(RRU)` and antennas.
 
* A high-bandwidth and low-latency optical transport network.

* A centralized baseband unit `(BBU)` pool consisting of general purpose processors with Network Function Virtualization `(NFV)` support.

* MEC application server or simply `(MEC server)`, which is integrated at the RAN element. This server provides computing
resources, storage capacity, connectivity and access to RAN information. It supports a multitenancy run-time and hosting
environment for applications and services.

![MEC arch](/assets/openstack-location-based-vm/mec-arch.jpg)
*MEC components*


# Connecting Mobile Edge Computing And The Use Case
Adding mobile edge computing and geo service applications to the base stations makes communication much faster, so that
base stations with MEC cloudlets will show an end to end latency of a few **miliseconds**. 
![autonomous cars](/assets/openstack-location-based-vm/autonomous-cars-mec.gif)
*Car to car communication via MEC cloudlet, reduces the latency by several magnitude orders*

# OpenStack And MEC Cloudlets Placement
OpenStack is being the de facto in virtualization platform, and in our case it is the heart of IT infrastructure, provisioning
the MEC resources.

The requirement for the infrastructure is to being able of dynamically adapting to the application needs and scaling 
according to resource requirements.

In the case of the autonomous cars, that would mean, that in the case of multiple traffic the platform would be required
to power up  more cpu resources at some specific cloudlet. 
 
 


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
* attempts to boot an instance with specified `app1-instance` flavor

{% highlight bash %}
$ nova boot --flavor 84 --image 64d985ba-2cfa-434d-b789-06eac141c260 \
 --availability-zone nyc:40.71:-74.00 app1-instance
{% endhighlight %}






[superfluidity]: http://superfluidity.eu/about/specific-objectives/
[bts-wiki]: https://en.wikipedia.org/wiki/Base_transceiver_station
[host-aggregates]: http://docs.openstack.org/developer/nova/aggregates.html