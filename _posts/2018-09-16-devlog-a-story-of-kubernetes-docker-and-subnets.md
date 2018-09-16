---
layout: post
title: 'Devlog: A story of kubernetes, docker and subnets'
tags:
  - devlog
  - networks
date: 2018-09-16 18:11 +0530
---

This post is around *why knowledge on subnetting* needed and one real world example which I have faced while working on project. Before going into details let me give some conext.

We have a nodeJS app which interacts with kubernetes through API.
This nodeJS app responsible for configuring and deploying kafka consumers in kubernetes. We use minikube and docker for local dev setup. Our pods communicate with kafka.


We had a bug where we can only see it in production. We wanted to reproduce that bug locally with the similar load.
We decided to use a production kafka mirror. We can only access kafka mirror through VPN. Let's say mirror kafka address is `172.17.1.6`. From now on I refer to mirror kafka with `172.17.1.6`.

We have configured our nodeJS app to include this ip (`172.17.1.6`) for deploying new consumers in kubernetes cluster. Basically it's a environment variable in a kube deployment file.

When we deployed our new pods which points to `172.17.1.6` in local kubernetes they were not able to connect to `172.17.1.6`. Initially we thought our pods were not able to connect it because it's served under VPN (*Dumb me.... Didn't know that minikube shares host network and host network anyways connected with VPN. Ideally those pods should be able to connect to `172.17.1.6`*). So we started searching for things like *'how to share VPN connection with minikube'*. But didn't get any useful results. We tried different options for more than a hour. Still no luck. We checked with other people in team. One team mate mentioned that it might be because of *subnet collison*.

Our containers running in two networks deep (Minikube VM and containers running in VM) which we have to look into. Quick `ifconfig` in minikube gave us network IPs and their subnet masks. Our `172.17.1.6` didn't fall under that network range.

We looked at containers running in minikube it has a subnet of `172.17.0.0/16`. Address range for this subnet is `172.17.0.0 - 172.17.255.255`. `172.17.1.6` comes under this address range. So containers tries to find the host with ip `172.17.1.6` inside the docker network itself, that means request doesn't even reach the host machine. We need to tell minikube that start docker with different network range. Fortunately minikube allows you to pass different config options for docker. We restarted the minikube with this command
```
minikube start --docker-opt=bip=172.17.0.0/28
```
The address range for this would be `172.17.0.0-172.17.0.15`. This way we reduced the network range and now our containers able to connect to `172.17.1.6`. Please note that with `172.17.0.0/28`, we can only spawn 14 containers since this address range allows 14 ips. You can start docker with different network ip range alltogether if you want. 