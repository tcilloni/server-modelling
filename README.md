# Probabilistic model checking of failures for specific servers architecture

## Contributors
Thomas Cilloni', t.cilioni17@student.xjtlu.edu.cn  
Gurugautham Anandakumar', g.anandakumar17@student.xjtlu.edu.cn  
Jean-Christophe Taillandier', j.taillandier17@student.xjtlu.edu.cn  
William Henarto Tengku'

_'Department of Computer Science and Software Engineering, Xi’an Jiatong Liverpool University, Suzhou, Jiangsu, P.R. China_

> _Abstract - A server system is a process often forgot today, although it is still a center piece to any type of network. For this reason, we use PRISM probabilistic model checker based on Markov chains in order to determine the failure rate of different server architectures, ultimately looking for an optimized architecture._

## Introduction
We decided to model a server system consisting of three types of nodes. Very few studies analyzing such thing exists as it would require an enormous amount of resources, and with fast changing technologies, the time requirements would make such study irrelevant by the time it comes out. Nonetheless we here analyze a very broad study simulating the impact of adding a rebooting mechanism to individual servers in a specific server architecture. The architecture is built as follows: a load balancer (lb) receives a request and transmits it to one of two main servers (ms1 & ms2). The main server that receives the request then passes it on to the servers that will execute the request. As long as the selected main server functions properly, it will pass down the request to all three of its associated server, which will, in turn, process the request. Note that throughout the experiment, we specified that the minimum number of running servers has to be 1 and the minimum number of Main servers has to be 1, in order for the system to function.
![server architecture](/res/architecture.jpg)

## Initial Model
Using PRISM probabilistic model checking, we initially
created a very simple model, where each node in the system
had an associated rate of failure, and where we could run
experiments with different conditions regarding the minimum
number of servers needed for the system to keep functioning.
From there we created three modules;

#### First module: Load Balancer
The load balancer has two states, up or down, which
we initially set as up. The system first checks the probability
that the load balancer fails (which will result in a system shut
down), and then selects which main server it will send the
request to. We understand this process depends on many
factors in real life, notably their level of activity but we had
the load balancer determine where to send the request based
on two factors only. First it will check whether one of the
main servers is already down, in which case he will send it to
the other server. A case where both main servers are down
will result in a failure of the system. In the case that both main
servers are up, it will randomly choose a server, with
probabilities of 50-50 for each.

#### Second Module: Main server
Both main servers have the same two possible states
as the load balancer; up or down. This module calculates the
probability of failure of the chosen Main server from previous
module. Failure of a main server at this point would result in
the whole system shutting down.

#### Third module: Servers
This module deals with the servers that handle the request
from their main servers. Recall that each main server has three
servers at their disposition. We assume all three servers are
functioning at the arrival of the request. Then we check for
each server with the probability of them going down.
Additionally, we need to make sure that the number of servers
up at the end is greater than what was specified at the
launching of the experiment (2).
![v1](/res/v1.jpg)

This graph models the failure rate, over T time (30 days) of the system, because of each type of node. For example, the green line indicates the probability after T days that the system fails because of a server failure as described in the labels section of our model.

As we see within a month (31 days), this type of architecture, and especially given the high failure rate of the servers, the system becomes unusable within 20 days. It is therefore important to make change to our architecture. For the next experiment we created a third ‘transient’ state for all nodes, and a reboot mechanism in case of failure. These numbers seem quite poor when compared with data from statistical study website Statista (1) giving a worldwide failure rate of about 5% over the first year of a server.

## Revised Model
For every module of our modules in case of failure, the node would now fall into a ‘transient’ state, which would trigger the rebooting sequence. Rebooting sequence is based of a probability of successful reboot, which we set at 90% and a maximum amount of reboot tries. The maximum amount of reboot tries can be changed when launching a new experiment and is different for Servers, Master servers and Load balancer. In our experiment we set maximum reboot at 1 for load balancer, 2 for main servers and 3 for servers, all the while keeping the minimum number of servers and main servers up and running at 1. Due to a lack of time and processing power, we were not able to test the model over 30 days as we did for our initial, simpler model. The complexity of our revised model allowed us to only test over 7 days, which still required a lot of computing power, and nonetheless allows us to see trends and infer some conclusion regarding our change in architecture.
![v2](/res/v2.jpg)

The difference with the previous graph is significant even considering this graph models only 7 days, the probability of failure at every stage of the process is a fraction of what it was before, while the failure rate remain the same as before. These failure rate also fit in a category labelled ‘very low’ by Yavari et. al.(2) who analyzed server failure rates in Service Oriented architecture. Compared to Statista’s data however, the server seems to have quite a high failure rate, even if it were to flatten over time, it is already well above world average.

In fact, both the Load Balancer, and the Main Server failure is barely above 0. The load balancer in particular, with only one chance at rebooting, seem to be extremely reliable, keeping the same failure rate as before (1/(60*60*24*365) ). Similar behaviour for the Main Server, who was modeled with two chances of reboot.

As for the Servers, it may seem worrying that the curve looks like an exponential function. However, it also did in the previous graph over the first 7 days, which is why we are not worried that have we had been able to keep the experiment running for a few more days, we would have surely seen a flattening of the Server curve on a probability far lower that of the previous graph. Without such assumption, we can still clearly see that the failure rate associated to any given point at T time, is considerably lower than in the previous architecture.

## Conclusion
In conclusion we saw the huge difference that a third state and rebooting capacity has on a server architecture, even with such a small size and even though the failure rates remained constant. We could infer that for a real size server farm, such impact would be exponentially more significant. If time were to permit, our next step would be to test with a re routing capacity in case of failure. That is, if the servers, once they received the request are in a failure state, the parent node (here Main server) would be able to catch this failure, and then reroute the request through the other main server. The same could be modeled with Main Server and Load Balancer.

According to a research done by Microsoft, hardware failure in consumer PC is making it exponentially more likely for the system to crash again (3), which is also not taken into account in this analysis. It would be interesting to, when accounting for failure of individual component, increase the future probability of failure of this very component.

### References
[1] Statista, “Frequency of server failure based on the age of the server (per year)”, February 2014 online: https://www.statista.com/statistics/430769/annual-failure-rates-of-servers/  
[2] A. Yavari, M. Musavi, H. Momeni and M. Hamzenhia, “Measuring the failure Rate in Service Oriented Architechture Using Fuzzy Logic,” Journal of Mathematics and Computer Science, vol. 7, pp. 160-170, July 2013.  
[3] E, Nightingale, J Douceur, and V. Orgovan, “Cycles, cells and Platters: An Empirical Analysis of Hardware Failures on a Million Consumer PC”, Proceedings of EuroSys 2011, April 1st 2011
