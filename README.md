# ucd-k8s-practices

# Installation on Openshift
If you're installing UrbanCode Deploy (UCD) onto IBMs Redhat Openshift Kuberbetes Service(ROKS), then you can use the automated install scripts from this git repository to create a single ucd server using mysql, all running inside Openshift.
https://github.com/arnoldst/ucd-install-on-roks

Other topologies are available, in particular using the DB2 Cloud Service (https://cloud.ibm.com/catalog/services/db2) to provide a managed database solution, whilst hosting the UCD application inside Openshift.  


# Scaling the UCD server - horizontally and vertically
Discussion on scaling the ucd server stateful set to get more instances. (does this actually work - how does the server know ?)  
How do you configure memory size in the tomcat server - are there environment properties/ config maps that you need to do ?

## Challenges for scaling
If we start looking at scaling an individual server instance that basically means CPU and memory.  At the k*s level that's fairly straight forward but would require a restart of the server which would loose you all the current executing processes and login sessions.  To actually make a change to the JVM heap memory you need to edit the set_env script in the server bin directory to change the value and then again, start the server - if that value in set_env was possible to take from an environment variable then it would be easy to inject that into a restart via a config from k*s.  Scaling down the memory would be the same in reverse.  However given that a restart would be necessary, it would not be seemless.  Also at higher heap sizes there are more complex heap considerations with an IBM JVM which allow more fine grained control over how the heap is used and this is kind of important to avoid 'pauses' when you get a mega garbage collection.

For scaling out more server instances that is probably relatively straight forward but there are some things that would need to happen
  * The first instance would always have to be configured as an HA system
  * The second and subsequent systems would need to be married into the cluser by adding the new servers into the cluster otherwise they won't talk to each other or share anything
  * There is a loadbalancer to consider to make sure that user sessions always go to the same instance otherwise the user will loose their session - or at least it gets 'lost' and possibly they might get it back again if there are again swicthed.  So sticky sessions for users is a must.
  * Scaling down again would be a whole lot harder because you would have to somehow get the server you wanted to loose out of the cluster.  I don't think you can just remove its connection from the cluster otherwise the agent and user sessions to that instance would become orphaned - or maybe the server wouldn't take any notice of that if they cache server connections.  So a restart of the whole HA might be necessary.
  * I've not played with front end servers yet but maybe that is where you would look at scaling rather than the back end.  The front end servers do a lot of the donkey work for user sessions.  So I guess if you stopped making new connections to a front-end server you could offload over time the user sessions.  How you tell when a front end server has no user sessions anymore i don't know.

So scaling up possily isn't too much of a stretch but I guess scaling down again is going to be a much harder problem.

# Setting up regular agents to connect to ucd server
How to do this ?

# Scaling the UCD agents
Discussion on scaling the ucd agent stateful set to get more instances. 
Scaling up and down ???

## Some thoughts on how we might do this
It might be possible to 'pre-calculate' a bunch of agent_ids and have a known list.  So we have a map of agent-name to agent-id and when you want a new agent you take the next name in the list and set it up in its installed.propreties with the corresponding id.  This might work if the server takes first contact with a new id as a new agent and not an imposter - depends on who actually generates the ID and the protocol exchange for new agents.  Might be worth a go.  So then if we scale back, the agent just goes offline but when we scale up if we can get the same ID, it will come back online.  This also assumes that there is nothing clever in an agent id apart from it being a random string.  Then the agents would just go offline and come online.
One possibility might be to scale up to the maximum agents to get them online and harvest their ID's and then scale back to the number you wanted.
The only reasonable use of such agents would i think be as agentpools.  Can't see how else they would work.  One of the downsides of agentpools is that they deal with a large number of potential components and hence with working directory can get very large.  I guess if the working directory is in the container then they get automatically cleaned on an agent restart.
The only other challenge on scaling back I can see is only doing it when an agent isn't busy - i guess the server could deal with that but there are dependencies on steps with the content of the working directory so it basically kills the component process(s) completely but leaves the environment in a relatively unkown state.  Guess the server would have to restart the process - never tried this.  With a normal agent, you bring it back and it carries on, but with a container agent the workspace context is lost unless they have persistent volumes ?  Agent pools - i don't know what happens, maybe ucd will wait for the assigned agent to cme back so that it can carry on.


# Agent configuration
Setting up a ucd service account for use by the agents (long lived log in)
Configuring the service account so its got access to the right namespaces, with the right permssions. Follow the best practice of the minimum amount of permissions to get the job done.  So whilst giving admin rights is easy - probably better to restrict the scope.
Multiple service accounts ?  So different teams can have different access ?
Do you need to allocate agents to teams ?
Still need to have controlled environments (namespaces) for which you need special access / roles

# User integration with k8s 
Is this even possible ?  Or do you just integrate with the same ldap ?

# Monitoring
Can just use prometheus / grafana to get a view of the load of the ucd server.  Can we build out an example dashboard for this ?

# Logging
Should be able to use the EFK stack to get at agent and server logs.

# Integration with tekton
Tekton handover to ucd, (maybe velocity?)

AFAIK there is no Tekton plugin available for ucd nor velocity. possibly in development -> will ask.


