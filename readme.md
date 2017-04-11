Repository Init Content
=======================

# Employee Task Aproval Project

## Initial Repo Creation steps.
* Add groups and users

```
cd /Users/anuragsaran/Documents/MW/summit2017/employee-task-approval-process/employee-task-approval
mkdir configuration
touch configuration/application-users.properties
touch configuration/application-roles.properties
echo "kaviitha=d0f9e8fd2ffa6504b7b96db34a139262" > configuration/application-users.properties
echo "satish=d0f9e8fd2ffa6504b7b96db34a139262" >> configuration/application-users.properties
echo "josh=d0f9e8fd2ffa6504b7b96db34a139262" >> configuration/application-users.properties
echo "justin=d0f9e8fd2ffa6504b7b96db34a139262" >> configuration/application-users.properties
echo "anurag=d0f9e8fd2ffa6504b7b96db34a139262" >> configuration/application-users.properties
echo "joe=d0f9e8fd2ffa6504b7b96db34a139262" >> configuration/application-users.properties
# Password = pass@123 
echo "kaviitha=kie-server,agent" > configuration/application-roles.properties
echo "satish=kie-server,agent" >> configuration/application-roles.properties
echo "josh=kie-server,agent" >> configuration/application-roles.properties
echo "justin=kie-server,reviewer" >> configuration/application-roles.properties
echo "anurag=kie-server,reviewer,admin" >> configuration/application-roles.properties
echo "joe=kie-server,reviewer" >> configuration/application-roles.properties

```

## Deployment Steps On Openshift
* ArtifactID=GroupID:ArtifactID:Version
* employee-task-approval=com.acme.empexpenses:employee-task-approval:2.0.5
* Always check : 1.0.5
```
cd /Users/anuragsaran/Documents/MW/summit2017/employee-task-approval-process/employee-task-approval/configuration
oc project employee-task-approval-process
oc create -f processserver-mysql-persistent-s2i.yaml
oc create -f processserver-63-is.yaml
export application_name=taskapproval
export source_repo=https://github.com/anurag-saran/employee-task-approval-process.git
export context_dir=employee-task-approval
export nexus_url=http://nexus-nexus.apps.anuragsdemo.com/
export kieserver_password=kieserver1!
export is_namespace=employee-task-approval-process
export kie_container_deployment="employee-task-approval=com.acme.empexpenses:employee-task-approval:1.0.5"
oc new-app --template=processserver63-mysql-persistent-s2i -p APPLICATION_NAME=$application_name,SOURCE_REPOSITORY_URL=$source_repo,CONTEXT_DIR=$context_dir,KIE_SERVER_PASSWORD=$kieserver_password,IMAGE_STREAM_NAMESPACE=$is_namespace,KIE_CONTAINER_DEPLOYMENT=$kie_container_deployment,KIE_CONTAINER_REDIRECT_ENABLED=false,MAVEN_MIRROR_URL=$nexus_url/content/groups/public/
```



## Steps to make rest calls
* export policyquote_app=<URL of the policyquote app route>
* cd /Users/anuragsaran/Documents/MW/summit2017/employee-task-approval-process/employee-task-approval/configuration

```
export policyquote_app=taskapproval-dev-env.apps.anuragsdemo.com
export kieserver_password=kieserver1!
```

Check the health of the Process Server:
```
curl -X GET -H "Accept: application/json" --user kieserver:$kieserver_password "$emprequest_app/kie-server/services/rest/server"
```

Check which containers are deployed on the server:
```
curl -X GET -H "Accept: application/json" --user kieserver:$kieserver_password "$emprequest_app/kie-server/services/rest/server/containers"
```

Start a process
```
curl -X POST -H "Accept: application/json" -H "Content-Type: application/json" --user kieserver:$kieserver_password -d @empreq-start-process-payload.json "$emprequest_app/kie-server/services/rest/server/containers/employee-task-approval/processes/employee-task-approval.employeerequest-process/instances"
```

Check that the process instance is running:
```
curl -X GET -H "Accept: application/json" --user kieserver:$kieserver_password "$emprequest_app/kie-server/services/rest/server/queries/containers/employee-task-approval/process/instances"
```

Query for the tasks that have anurag as potential owner:
```
curl -X GET -H "Accept: application/json" --user anurag:pass@123 "$emprequest_app/kie-server/services/rest/server/queries/tasks/instances/pot-owners"
```

As anurag, start the task:
```
curl -X PUT -H "Accept: application/json" --user anurag:pass@123 "$emprequest_app/kie-server/services/rest/server/containers/employee-task-approval/tasks/1/states/started"
```

Again as anurag, complete the task specifying the policy price as payload of this call, using the task_price task output variable:
```
curl -X PUT -H "Accept: application/json" --user anurag:pass@123 -d '{ "task_cost" : 600 }' "$emprequest_app/kie-server/services/rest/server/containers/employee-task-approval/tasks/1/states/completed"
```


