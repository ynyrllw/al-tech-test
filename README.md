#AL Tech Test Ynyr Submission

This is the technical test for job application at al.

The objective was for 2 web servers with a load balanacer.

This should work with both windows and linux operating systems

Time spent: well over 24h

#Software dependencies
See below for software versions used
* Vagrant
* Virtual Box

#Instructions 

git clone 

cd al-tech-test

vagrant up

#Thoughts
I made this far more challenging for myself than I needed to and I'm not happy with myself about it. Although it was an interesting challenge then just asking someone what dev ops means and got to try some new things.

I think I definitely tried to rush which ended up taking more time in de-bugging in the end, especially with all the different tools in the mix it was bound to not work to begin with. Usually I would have iteratively added a new tool on an exiisting stack and I'd have it in a known state.

#Tweaks
From reading the brief it seemed slightly out of date, no containerisation as an example. Main change is using a control machine VM for ansible and then containers instead of a web server or load balancer VM.

In hindsight this would have been finished far sooner if I hadn't made this decision.

I initially spent a long while trying to decide where docker would fit. Do I use docker as the provider within vagrant and then ansible as a container? I wanted to try and be as vagrant as possible to keep honest to the brief but quickly decided if I did this there wouldn't be any room for ansible as you could use docker compose for everything. I didn't really count docker compose as a configuration management tool so felt it didn't tick the box.

There are alot of resources arguing both ways that you should use ansible to build images and containers, It feels a bit strange to get set up but Ansible is definitely more powerful. In this use case if the brief didn't ask specifically for a CM tool I would have used compose. Usually you would use ansible to set up the host but as vagrant does that already and we dont have any corporate requirement it feels a bit moot so needed to use ansible for building the containers.



#Improvements
* Add tests in its own yml file.
* much more testing - test suite
* stop using docker logs for the testing, syslog or journald
* tagging for ansible tasks
* Ch
* Improve Scalibility - It's somewhat easy to scale currently (add a new command and edit nginx conf) but it would be great to be able to have this a bit more slick to scale easier. (then look at k8s)
* Use something like consul
* Improve Monitoring & logging
* Add a DB and pull data (currently static hello world)
* Use more ansible galaxy roles
* Security - Plug in AD, file permissions, https, close ports (currently the containers are exposed to the host too)
* Built docker images need version
* polish the ansible playbook
* extra VM for docker, and one for control machine
* stick to version in requirements.txt for the web server (currently pulling latest)

#Issues
Adding docker always adds an extra layer of complexity, but the main trouble I had was connecting the web servers with nginx which was in another container, one issue I didn't find out for awhile was the container was publishing to the localhost of the container itself, so nginx couldnt't access it.

Not exactly an issue but deciding where ansible fit, if I either used it inside a container or used it to create them.

I think by attempting to rush and spending time on each tool/part I put them all together then tried to get it working and with a complex and unique solution it was harder to spot what was exactly going wrong.

Something that unfortunately I sunk a bit of time the partition on my fedora OS was pretty low where I couldn't build a vagrant box half way through when trying to install docker on the VM. I'll need to spend a weekend re-formatting and increasing the primary partition. Tried to take some off the windows partition but needed to either boot to a live disk or re-install the OS.

Decided it wasn't worth the hassle and moved to windows and use ansible_local with a control machine. Plus for this now people can run this on both windows and linux.


#Software design choices
##Web Server
###Python

Decided early on to use Python as it's what I've used and quite comfortable with. Now I would probably chosen something much simpler like node js. Had experience with tomcat but felt it was too heavy for what we were doing. 

* Django
Options from what I saw was between Django or flask, went for Django as its the most popular.
* UWSGI
UWSGI is the standard usually to use for python web servers
* Docker


##Configuration Management
2 configuration management tools I was between was puppet and ansible. The master slave architecture really would  not have fitted with what I was aiming for, so ansible was naturally the fit as its a serverless architecture which doesn't require much dependencies except python(which I've already stuck to) and openssh + it's what I've been using the most recently.

Usually puppet is more powerful but for what I wanted the CM to do which was prep the host and deploy the containers, there wa senough support in ansible.

Didn't consider others such as Chef as I though best stick with what I know but if it was a simpler solution would have been good to try and see if it fit the use case.

#Loadbalancer
Didn't think this too much. Nginx is basically a standard that most people use, I've had experience with it before and it has strong compatbility with UWSGI.

#Software Versions
Vagrant 2.2.7
Vrtual box 6.1.4

#Long Form
Warning: this was my thought process while working so it's quite long and convoluted but wrote it as I went. I've tidied it slightly and made it more comprehensible but tried to keep it mostly intact. Treat it as a work log or a comment on a JIRA ticket.


2 options for automated testing

first is check its up: relatively easily make sure the output is = to "hello world"

second automated testing: making sure the load balancing in nginx is working correctly. We aren't outputting anything special to deffrentiate between the 2 containers (say hello world A on the first and hello world B on the other). So the only way to differentiate is using nginx.

Nginx defaults to round robin. Easiest test would be to constantly ping the server then check the nginx access logs to show the container switching from one to the other. Some logic to scan for both container names in the log

second option would be a bit heavier and more prone to issues. Nginx has a default max_fail of 1 and fail_timeout is 10 seconds. We could increase the timeout of one server to only re-try after say 1 minute, and check the server if you can still get hello world within that 1 minute(this isn't testing any specific load balancing strategy but ensuring that it is flipping to another container and you can get output) Round robin wouldn't be the best option to choose here.



can also add slow_start which stops all the connections hammering a server when its getting back up.

i'd prefer the first option, will just have to pull the logs out of the container to the host first before reading it with ansible.

Couldn't find anything in logging module. But in the upstream module which is what you use for load balancing there is: upstream_addr "keeps the IP address and port, or the path to the UNIX-domain socket of the upstream server." which looks ideal to what we want


We now need to update the log format so we can get this info
log_format upstreamlog '[$time_local] $remote_addr - $remote_user - $server_name $host to: $upstream_addr: $request $status upstream_response_time $upstream_response_time msec $msec request_time $request_time';

so added that to the nginx conf file and added set it to send /var/log/nginx/access.log as the nginx image automatically is sym lynced to /dev/stdout so we can easily use docker logs from the vagrant host.

so ran wget a couple of times then checked docker logs and we can see it moving between "
172.17.0.2:8000 - - [05/Apr/2020:21:39:14 +0000] "GET / HTTP/1.1" 200 21 "-" "Wget/1.19.4 (linux-gnu)"rt=0.000 uct="0.000" uht="0.000" urt="0.000"
172.17.0.3:8000 - - [05/Apr/2020:21:39:14 +0000] "GET / HTTP/1.1" 200 21 "-" "Wget/1.19.4 (linux-gnu)"rt=0.000 uct="0.000" uht="0.000" urt="0.000"

Only concern now is if this ip address will be the same each time

Loadbalance option. For now i've conciously chosen to stay with round robin, you can also add weights to which container you want most connections on. Least time looks the best which conciously chooses the container with the least latency and active connection but is nginx plus only. Least connections would be the smartest option i think otherwise, ip hash if you don't mind or expect to get swamped.

tried using $upstream_http_server or $upstream_http_name but zilch. will stick with upstream_addr for now


so can use lineinfile to check files, although as we're using docker we can check the logs with "docker logs" can use shell but its not the most ansible way to use.

Another option is we can set up a volume so the logs are always printed on the host

https://docs.docker.com/config/containers/logging/configure/


need to configure the log driver with log opts then set in the container creation of the ansible task

so only viable available log driver really is journald as docker ce is limited and still want to be able to use docker logs.
 Now to brush up on how to filterering with it or a way to do it with ansible ideally

https://docs.ansible.com/ansible/latest/modules/docker_container_module.html
https://docs.docker.com/config/containers/logging/configure/


ended up deciding to use docker logs directly for now.

Only issue with this test is it needs to be a fresh installation to check or it will give a false positive

also for test i added to check the wget for the Hello World output
