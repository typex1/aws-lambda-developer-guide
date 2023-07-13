CodeDeploy:
------------------
EC2/on-prem
  Need CD agent installed
  Traffic direction: in-place or blue/green
Lambda
  Apps that consist of an updated version of a L. function
  Traffic: canary, linear, all-at-once between two versions
ECS
  Deploy containerized app as a task set
  Blue/green = install updated version as a new replacement task set
