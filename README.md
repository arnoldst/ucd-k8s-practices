# ucd-k8s-practices

# Installation on Openshift
If you're installing UrbanCode Deploy (UCD) onto IBMs Redhat Openshift Kuberbetes Service(ROKS), then you can use the automated install scripts from this git repository to create a single ucd server using mysql, all running inside Openshift.
https://github.com/arnoldst/ucd-install-on-roks

Other topologies are available, in particular using the DB2 Cloud Service (https://cloud.ibm.com/catalog/services/db2) to provide a managed database solution, whilst hosting the UCD application inside Openshift.  


# Scaling the UCD server - horizontally and vertically
Discussion on scaling the ucd server stateful set to get more instances. (does this actually work - how does the server know ?)  
How do you configure memory size in the tomcat server - are there environment properties/ config maps that you need to do ?

# Setting up regular agents to connect to ucd server
How to do this ?

# Scaling the UCD agents
Discussion on scaling the ucd agent stateful set to get more instances. 
Scaling up and down ???

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


