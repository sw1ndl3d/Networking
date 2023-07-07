# IPTABLES juicy goodness

---

##### Beginnings of IPTABLES

To change the default value of any of the default policies:

```sudo iptables -P [INPUT|FORWARD|OUTPUT] [ACCEPT|DROP|REJECT?]```

The following command will accept all connections inbound from a given IP or CIDR to any destination.

```sudo iptables -s 192.168.x.x/24 -p all -A INPUT```

To list the changes we just made.

```sudo iptables -L```

For the sake of learning, we can add some more arbitrary rules to the INPUT chain.

Lets add the localhost as the source (-s) and destination (-d)

```sudo iptables -s 127.0.0.1 -d 127.0.0.1 -A INPUT```

Lets add another <IP> as the source (-s) with the TCP protocol (-p tcp)

```sudo iptables -s <IP> -p tcp -A INPUT```

Verify our changes took effect and are accurate once again.

We will no append another firewall rule for the name ocalhost and observe what happens. We will add it as the source address:

```sudo iptales -s localhost -A INPUT```

We can see that there are two listings now with the name localhost. This is due to localhost being resolved to the IPv4 and IPv6 addresses. These resolve back to localhost in the output.
(kinda doesnt make sense as of now 04/17/2023)

Now lets run through and delete the duplicate lines:

```sudo iptables -s localhoust -D INPUT```

The (-D) option deletes rules. It did not delete the rule we have set with a different destination set. We only wanted to delete one rule of the two we just deleted. Lets add them again

```sudo iptables -s localhost -A INPUT```

```sudo iptables -L --line-numbers```

Now we can delete specific rules:

```sudo ip tables -D INPUT 5```

Its important to note that firewalls work from the top of the list and go through each line until a match is found. This being said, if there are conflicts between rules, the first line will be the one used in the firewall configuration.

If we run into issues, here is how we can rearrange our IPTABLE structure:

```sudo iptables -s <IP> -I INPUT 1```

```sudo iptables -L --line-numbers```

```sudo iptables -D INPUT 4``` : remove the duplicate after ensuring our new one is at the top of the list.

If we wanted to view some of the traffic going across the rules we just made:

```sudo iptables -nvL```

`-v` : allows us to see traffic in terms of packets and bytes

`-n` : will display the IP addresses in stead of the canonical names
 
##### IN ORDER TO SAVE IPTABLE RULES

```sudo iptables-save```

##### Extended Rules and Default Policies

Going back on what was done, if we dont specify with (-p) the policy we want to use, the default is automatically inherited. For an example, lets replace one of the rules we previously created:

```sudo iptables -R INPUT 2 -s 192.168.x.x/24 -j DROP```

`-R` : Replace INPUT|OUTPUT|FORWARD

`-j` : Secifies we want DROP|ACCEPT|REJECT

We have now blocked all traffic on the /24 CIDR for that ip range. However, because the rule that comes first is 192.168.1.37, the traffic for that is still allowed through.

Now lets change the access of this 192.168.1.31 host:

```sudo iptables -R INPUT 1 -s 192.168.1.31 -d 127.0.0.1 -p tcp --dport 8080```

We specify we wanna (-R) replace INPUT 1 (-s) source (-d) destination (-p) protocol (--dport) destination port

IF we know what the source port is we will be working with, we can even specifiy the:
```--sport``` : source port

There are some more options that can be used to create rules based on the type of conncetion made. This option is part of the conntrack odule, which used the (-m conntrack) option. The subest of the module is (--ctstate) which stems from the 'iptables-extensions' package
(older linux systems use --state instead of --ctstate)


- INVALID: The packet is associated with no known connection.

- NEW: The packet has started a new connection or otherwise associated with a connection that has not seen packets in both directions.

- ESTABLISHED: The packet is associated with a connection that has seen packets in both directions.

- RELATED: The packet is starting a new connection, but is associated with an existing connection, such as an FTP data transfer or an ICMP error.

- UNTRACKED: The packet is not tracked at all, which happens if you explicitly un-track it by using -j CT --notrack in the raw table.

Using conntrack will make the iptables a stateful firewall.

Examples:

```sudo iptables -I INPUT 1 -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT```

This rule will keep track of any existing connections.

It would be good practie to add a DROP action for any packet that has an INVALID conectio state. LEts start by inserting this rule at the beginning of the INPUT chain.

```sudo iptables -I INPUT 2 -m conntrack --ctstate INVALID -j DROP```

With this configuration, any traffic that originates from our host will be accepted back into our host. The INVALID conections will be dropped immediately.