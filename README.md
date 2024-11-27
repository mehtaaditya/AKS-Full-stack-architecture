#This is not a step-by-step deployment. Project is still improving and lot of new features such as agro rollout, more testing piplelines will be integrated soon and some features will be removed. If the site is down that means that i might be working on it. You are welcome to checkout the github repo.

LINK TO SITE: [(http://devops.adityacloud.online/)](http://devops.adityacloud.online/)
This site leverages all important Devops tools needed to accomplish a full stack deployment from coding to containerization, testing, deployment and monitoring. The project follows the principle of CI/CD, continuous testing, continuous monitoring and Finops principle

Services extensively used are Azure Kubernetes cluster, Azure service bus, Docker Desktop, Helm, Github Actions, Selenuim, Azure load test, Azure key vault , Azure Devops , Azure VM, Prometheus, Grafana and more.


How the complete workflow works: -

1. As soon as new commit is created in the GitHub repo by the developer, the Docker images are composed on the GitHub runner and pushed to ACR(Deployed Beforehand) in the cloud using GitHub actions workflow files.

2. And then the second GitHub Actions runs, and deployment takes place in the AKS cluster which is previously deployed

3. Also, the deployment depends on the which file the commit is done to. If the commit is done to the Docker file or the development code folder, the whole workflow including 1 and 2 takes place.

4. If the commit is only done to AKS files(manifest) which include CPU usage, images used, credential used etc, then only the 2nd workflow takes place and not triggered on the main branch commit.

5. A service bus which I already deployed for storing the orders from the customers is also used in the manifest files (GithubAction 2) using the key vault secret for service bus password in the AKS yaml mainfest. This way passwords are never stored in plain text. Passwords are not even stored in GitHub secrets variable(other than az login credentials). Azure key vault provides way better security and accountability. AKS secrets feature alone could also be used here.Also GPG key for github used in the commits makes sure that code commits are valid and verified.

6. A service principal is used to login to azure and only the appropriate permissions are added to the SP for the AKS deployment using Kubernetes RBAC.

7. The 2nd GitHub action has a trigger for Azure Devops pipeline in the end which triggers it after deploying to AKS and consequently the testing phase of the deployment begins.

8. The AKS cluster follows the principle of Blue-Green Deployments, so the AKS cluster will be using two namespace named green and blue. The live version is initially blue and after the deployment to AKS various testing happens in Azure devops pipeline.

9. App is firstly tested using selenium for UI functionality. (Pipeline 1)

10. The second azure Devops pipeline consist of OWASP zap security testing to find out the vulnerabilities. (Pipeline 2)

11. The third test is done using Azure Load test pipeline which gets the metrics from the test and store the results in the developer specified file. (Pipeline 3)

12. After the testing phase all traffic from blue is shifted to green namespace and the first namespace can be deleted manually or in the pipeline.

13. Of course, at each stage a developer need to take manual look at what might be wrong if any stage comes up with any issue either in testing or deployment

14. IAC terraform file are later extracted using aztfexport tool and the configuration files are stored in HCP cloud for redeployment in case of DR.

15. For the purpose of cost saving and efficiency, I made an executable file from poweshell using PS2EXE which runs using task scheduler in windows at morning 9:00 and 22:00 to start and stop the AKS cluster

16. I also did the same in Linux VM for redundancy if I case my personal laptop is offline. I used cronjobs using scripts for this. The same VM is used as the agent for the deployment in Azure pipelines. The VM is using the is using autoshut down feature which unfortunately is not present in AKS at the moment.

17. The AKS cluster is using autoscale feature which can use upto five nodes and can be configured to use as low as 0 nodes when the load is low.

18. The observability and monitoring of the project is done using Prometheus and Grafana which are deployed using helm chart deployment feature. The alerts, health and dashboards of AKS are configured for continuous visibility.

This only shows the structure that needs to be followed but not a step by step detailed tutorial.

1. The code used in this deployment is leveraged by Microsoft code samples which can be viewed here.

2. Website used VUE as JavaScript framework and used MongoDB and RabbitMQ as the backend storage. Service bus is also additionally added. The code samples have lot of other features which are not deployed in the deployment.

3. The first step will be to clone the deployment in your own repository and working from there.

4. Firstly, we will be manually deploying the resources and then if everything works correctly, we will be using automation. Resources I will be deploying using azure cli are AKS cluster, service bus and ACR. Full deployment can be viewed here with steps. After following the step-by-step tutorial for MS learn AKS app, you can come back here for next steps.

5. Next step is to docker compose using the file named docker-compose-quickstart.yml.

6. After checking locally that the app works as intended, we are going to push the images to the Azure container registry.

7. For the step above we used the command docker tag and then docker push citing the name of the container registry. Like this

docker tag rabbitmq:3.13.2-management-alpine acr777aditya.azurecr.io/rabbitmq:3.13.2-management-alpine

docker push acr777aditya.azurecr.io/rabbitmq:3.13.2-management-alpine

8. After the images show up in the ACR, we will be using the manifest file to deploy the pods using kubectl command. Kubectl can be installed like this. I used the file aks-store-quickstart.yml for this deployment.

9. The site should be working fine. Check if the pods are running fine( kubectl get pods) and check the ip address of the front store pod to access it(kubectl get service store-front –watch)

10. Also for the testing purpose use the service bus password in the yaml manifest to first test if the order can be seen after the purchasing the service bus queue.

11. The environment variable for the az login needs to stores as a secret in github secrets section for the actions to work

12. Now the automation can be started using GitHub actions and the first work flow to build the images and push them to azure acr can be done(please check the GitHub workflow folder for yml).

13. Then GitHub action for deployment to the AKS cluster needs to be done and at last step the trigger for Azure pipeline needs to be added. The workflow files will mostly be using the az cli commands.

14. Azure Devops organization accounts need to be created if not already present. The azure pipelines self-hosted agent is configured for the windows using the step mentioned here.

15. First pipeline is triggered from the completion of the 2nd GitHub action and the selenium UI test is added in the pipeline

16. Second pipeline is triggered by the completion the first pipeline and it does the OWASP zap security plugins in Azure Devops

17. Third pipeline triggered by the second use azure load test and the testing phase can be concluded here.

18. To now use Prometheus and Grafana for monitoring they first needs to be deployed in an environment, and for this we will deploy them in a different namespace in the same cluster. Using helm, we deployed it like this.

19. The alerting is configured for the app store namespace and can alert when load limit(set to 80%) is surpassed in the logs. Autoscaling is also configured in the AKS cluster with the minimum of 1 and 4 nodes respectively.

20. After all the infrastructure us configured I used aztfexport utility to get the terraform files for the deployment. I can use these to quickly redeploy the infrastructure in case to a disaster.

21. 1. Couldn’t find anything good which is efficient as well as secure to automate the start and stopping of an AKS cluster so I made a simple script where I logged in using az login and used az aks stop command to stop the nodes. I then used PS2EXE to change into executable. Task secduler(win. Feature) is used to automate this on schedule.
