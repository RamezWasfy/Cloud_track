# Description

- Deployments provide powerful automation for application management in Kubernetes. One of the things deployments can do is to allow you to perform rolling updates. Rolling updates enable you to update your containers to new versions gradually, without incurring downtime. In this lab, you will have a chance to learn, hands-on, how to perform a rolling update, as well as a rollback to previous version.

# Additional Information and Resources

- Your company's developers have just finished developing a new version of their candy-themed mobile game. They are ready to update the backend services that are running in your Kubernetes cluster. There is a deployment in the cluster managing the replicas for this application. The deployment is called candy-deployment. You have been asked to update the image for the container named candy-ws in this deployment template to a new version, linuxacademycontent/candy-service:3.

- After you have updated the image using a rolling update, check on the status of the update to make sure it is working. If it is not working, perform a rollback to the previous state.

# Solution 

- Perform a rolling update of the container version

- Update the deployment to the new version like so:

`kubectl set image deployment/candy-deployment candy-ws=linuxacademycontent/candy-service:3 --record`

- Check the progress of the rolling update:

`kubectl rollout status deployment/candy-deployment`

- If the update is not finished after a few minutes, something may be wrong with the update.

- Roll back to the previous working state

- In this scenario, the rolling update should fail. This is because the specified image, linuxacademycontent/candy-service:3, does not exist. Roll back to the previous version to get things working again.

- Get a list of previous revisions.

`kubectl rollout history deployment/candy-deployment`

- Undo the last revision.

`kubectl rollout undo deployment/candy-deployment`

- Check the status of the rollout.

`kubectl rollout status deployment/candy-deployment`

- This command should complete soon, saying the rollout was successful.